---
title: "I Built a Multi-Agent System Without a Framework. Here's What I Learned."
date: 2026-05-13 10:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [multi-agent, python, openai, anthropic, llm, agents, architecture, redis, rag]
---

A few weeks ago I decided to do something a bit uncomfortable: build a multi-agent system from scratch, with no LangChain, no LlamaIndex, no CrewAI, no framework at all. Just Python, the raw LLM SDKs, and whatever I needed to wire it together.

The project is an e-commerce customer service assistant — the kind of thing you see on every shop's website. You ask about your order, you ask to cancel something, you ask about a product. Simple domain, but a perfect sandbox for exploring multi-agent patterns because it naturally decomposes into distinct responsibilities.

The repo is here: [ecommerce-multi-agent-assistan](https://github.com/felipe-silvestre-morais/ecommerce-multi-agent-assistan)

Here's what I built, why I made the choices I made, and what surprised me along the way.

---

## Why No Framework?

This is the obvious question. LangChain exists. LlamaIndex exists. They are good. I have written about both.

My reason was simple: I wanted to understand what the framework is actually doing for me before I let it do it. Every time I've used a framework, there's this layer of magic that I trust without fully understanding. That works fine until something breaks in a weird way, and then I'm debugging three levels of abstraction.

Building without a framework forces every design decision into the open. How does routing work? I have to write it. How do tools get dispatched? I have to write it. How do agents share context? I have to figure it out.

The trade-off is obvious — more code, more things to maintain. But the understanding you gain is worth it at least once.

---

## The Architecture

The system has three moving parts:

**An orchestrator** that reads the user's message and classifies intent. It doesn't do any business logic — it just decides who should handle this.

**Three specialist agents**, each focused on exactly one thing:
- `OrderTrackingAgent` — finds out where your order is
- `OrderCancellationAgent` — evaluates if you can cancel and executes it
- `ProductInfoAgent` — answers questions about products using a RAG pipeline

**A session layer** backed by Redis that keeps conversation history and shared context across turns.

The flow looks like this:

```
User message
     │
     ▼
Orchestrator (intent classification)
     │
     ├──► OrderTrackingAgent
     ├──► OrderCancellationAgent
     └──► ProductInfoAgent
              │
              ▼
         Response + context update
```

The orchestrator classifies the intent into one of four buckets: `order_cancellation`, `order_tracking`, `product_info`, or `unclear`. That's it. It never touches an order or queries a product database. That separation is deliberate — I wanted the routing logic to stay completely independent from the business logic.

---

## Two LLM Calls Per Request

Here's the core pattern that makes this work: every user message triggers two LLM calls.

**Call 1** goes to the orchestrator. It reads the conversation history and the new message, and outputs a single intent classification.

**Call 2** goes to the relevant specialist agent. That agent has its own system prompt, its own tools, and its own logic for how to respond.

The specialist agent responds in a strict JSON schema:

```json
{
  "response": "Your order 1234 is on the way and should arrive by Friday.",
  "tool_calls": [
    {
      "tool": "get_order_status",
      "arguments": { "order_id": "1234" }
    }
  ],
  "context_updates": {
    "last_order_id": "1234"
  }
}
```

Enforcing a JSON response schema turned out to be one of the most important decisions in the whole project. When the LLM always returns structured data, parsing and tool dispatch become trivially simple. No regex. No string manipulation. Just `json.loads()` and you're done.

---

## No Framework Means Writing Your Own Tool Dispatch

In LangChain, you put `@tool` on a function and the framework handles the rest. Here, I had to write that myself.

It's not complicated, but it's instructive:

```python
TOOL_REGISTRY = {
    "get_order_status": get_order_status,
    "cancel_order": cancel_order,
    "check_cancellation_eligibility": check_cancellation_eligibility,
}

def dispatch_tools(tool_calls: list[dict]) -> list[dict]:
    results = []
    for call in tool_calls:
        tool_name = call["tool"]
        args = call.get("arguments", {})
        
        if tool_name not in TOOL_REGISTRY:
            results.append({"tool": tool_name, "error": "Unknown tool"})
            continue
            
        result = TOOL_REGISTRY[tool_name](**args)
        results.append({"tool": tool_name, "result": result})
    
    return results
```

The agent's JSON response says which tools to call and with what arguments. This function looks up the tool, runs it, and collects the results. Then those results go back into the next LLM call as context.

You can see how a framework would wrap this into something more elegant. But there's nothing magic about it — it's just a dict lookup and a function call.

---

## Supporting Multiple LLM Providers Without Framework Lock-in

One thing I wanted from the start: I didn't want to be locked to OpenAI. So I built a thin provider abstraction.

```python
class LLMProvider(ABC):
    @abstractmethod
    def complete(self, messages: list[dict], **kwargs) -> str:
        pass

class OpenAIProvider(LLMProvider):
    def complete(self, messages, **kwargs):
        response = self.client.chat.completions.create(
            model=self.model,
            messages=messages,
            **kwargs
        )
        return response.choices[0].message.content

class AnthropicProvider(LLMProvider):
    def complete(self, messages, **kwargs):
        response = self.client.messages.create(
            model=self.model,
            messages=messages,
            **kwargs
        )
        return response.content[0].text
```

Every agent gets injected with a provider at startup. Switching from GPT-4o to Claude is just a config change — the agents don't know or care which model they're talking to.

This kind of abstraction is something frameworks often give you for free. Worth knowing how thin it actually is when you write it yourself.

---

## Session Management with Redis

Multi-turn conversations need shared memory. I used Redis for this — each session gets a key with a TTL of one hour, and the value is a JSON blob containing:

- The last 10 messages in the conversation
- Explicit context signals: `last_order_id`, `last_product_mentioned`

The explicit context signals are the interesting part. When a user says "can I cancel it?" — without saying *what* — the agent needs to know what "it" refers to. Instead of relying on the LLM to figure that out from conversation history every time, I have the agents explicitly update named context fields in their JSON response. The next turn reads those fields and can inject them directly into the prompt.

This is a small thing but it makes conversations feel a lot more natural. And it's much cheaper than feeding the entire conversation history into every prompt.

---

## The RAG Service for Product Questions

The `ProductInfoAgent` doesn't just query an API — it does semantic search against a vector database.

I spun this up as a separate microservice (called it `rag_service`) using:
- `sentence-transformers` with the `all-MiniLM-L6-v2` model for embeddings
- Qdrant as the vector store

The agent calls the RAG service as a tool, gets back the most relevant product chunks, and uses those as context to answer the user's question. Clean separation — the agent doesn't know anything about embeddings or vector search, it just gets results back from a tool.

---

## What Actually Surprised Me

A few things I didn't expect:

**JSON schema enforcement works really well.** I was worried about the LLM returning malformed JSON or drifting from the schema. With a clear system prompt and a well-specified schema, it's reliable. The occasional failure is easy to catch and handle.

**The orchestrator is the easiest part.** I expected routing to be tricky, but intent classification with 4 categories is a task modern LLMs are very good at. The hardest cases are ambiguous messages that could reasonably be two things — "where is my order and can I still cancel it?" — and I handle those by defaulting to the more common intent and surfacing clarifying questions.

**Session management needs more thought than you expect.** Keeping the last 10 turns sounds simple. But what happens when a user switches topics mid-conversation? Stale `last_order_id` values cause the agent to act on the wrong order. I added explicit context invalidation logic — some topics clear certain context fields — but this is the part of the system that feels most fragile.

**Logging matters a lot.** Without a framework to instrument everything for you, you have to be intentional about observability. I ended up logging every LLM call with: intent classified, agent invoked, tools called, token counts, duration, and the raw response. This was annoying to add but invaluable when debugging.

---

## Would I Do It Again?

For a production system handling real users? Probably not — the frameworks exist for good reasons, and the maintenance overhead of owning all this plumbing adds up.

But for understanding? Absolutely yes. After building this, I have a much cleaner mental model of what LangChain's `AgentExecutor` is actually doing, what CrewAI's orchestration pattern looks like under the hood, and where the actual complexity in multi-agent systems lives.

The complexity isn't in the LLM calls. It's in state management, tool dispatch reliability, and making conversations feel coherent across turns. Frameworks help with all of these, but they don't make the underlying problems disappear.

If you're thinking about building something with agents and you haven't gone framework-free at least once — I'd recommend it. It's a bit of work, but the clarity you get on the other side is worth it.

---

*The full project is on [GitHub](https://github.com/felipe-silvestre-morais/ecommerce-multi-agent-assistan). If you have questions or want to talk multi-agent architecture, find me there.*
