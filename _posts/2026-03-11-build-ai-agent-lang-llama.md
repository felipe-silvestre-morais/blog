---
title: "Building Your First AI Agent with LangChain and LlamaIndex"
date: 2026-03-11 15:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [langchain, llamaindex, openai, agents, python, tutorial]
---

I remember the first time I really understood what an AI Agent is. I was reading about ReAct — that famous "Reasoning and Acting" paper — and I thought: "okay so the model just decides when to call tools? That's it?" And then I actually built one and realized: yes, that's kind of it, but also it's really powerful once you see it running.

In this post I want to show you how to build a simple but real AI Agent using two popular frameworks: **LangChain** and **LlamaIndex**. Both use OpenAI as the underlying model. We will build the same agent with both frameworks so you can compare the approaches side by side.

Let's go.

---

## What Even Is an Agent?

Before touching any code, let me make sure we are aligned on the terminology.

A regular LLM call is a one-shot thing. You send a prompt, you get a response, done. An **agent** is different. It runs in a loop:

1. The model receives a question
2. It decides if it needs to call a tool (or not)
3. If yes, it calls the tool, gets the result, and loops back to step 2
4. When it has enough information, it produces the final answer

The agent decides by itself whether to call tools, which tool to call, and what arguments to pass. The human is mostly out of the loop during execution. That's the key insight — it's autonomous reasoning, not just autocomplete.

The tools can be anything: web search, a database query, a calculator, a file reader, an API call. The agent decides when and how to use them.

---

## The Example We Are Building

We are going to build a **Smart Assistant Agent** that has three tools:

- **Calculator** — evaluates a math expression and returns the result
- **Get Current Date** — returns today's date (useful for time-sensitive questions)  
- **Lookup Knowledge Base** — searches a small in-memory dictionary of facts

This is intentionally simple. But it's real — the agent will actually reason about which tool to call, and you will see it working. Once you understand the pattern, you can plug in any tools you want: web search, databases, your own APIs.

---

## Setup

First, let's install the dependencies. I'm assuming you have Python 3.11+ and a virtual environment ready.

```bash
pip install langchain langchain-openai llama-index llama-index-llms-openai openai python-dotenv
```

Create a `.env` file in your project root with your OpenAI key:

```env
OPENAI_API_KEY=sk-your-key-here
```

Project structure for reference:

```
my-agent/
├── .env
├── agent_langchain.py
└── agent_llamaindex.py
```

---

## Part 1: Building the Agent with LangChain

LangChain is probably the most widely used framework in this space. It gives you abstractions for chains, agents, memory, tools, and a lot more. Let me walk through the code step by step.

### Step 1 — Define the Tools

Tools are the core of any agent. In LangChain, you define a tool using the `@tool` decorator. The docstring is important — the model reads it to decide when to use each tool.

```python
# agent_langchain.py

import os
import math
from datetime import date
from dotenv import load_dotenv
from langchain_core.tools import tool

load_dotenv()

# ---- Define the tools ----

@tool
def calculator(expression: str) -> str:
    """
    Evaluates a mathematical expression and returns the result.
    Use this tool when the user asks for calculations, arithmetic,
    percentages, or any kind of math.
    Examples: '2 + 2', '15% of 200', 'sqrt(144)', '100 / 4'
    """
    try:
        # We use eval with math module available, but limit the scope
        # for safety. In production, use a proper math parser.
        result = eval(expression, {"__builtins__": {}}, vars(math))
        return f"Result: {result}"
    except Exception as e:
        return f"Error evaluating expression: {e}"


@tool
def get_current_date(dummy: str = "") -> str:
    """
    Returns today's date.
    Use this tool when the user asks about the current date,
    what day it is, or how many days until/since a specific date.
    """
    today = date.today()
    return f"Today is {today.strftime('%B %d, %Y')} ({today.isoformat()})"


@tool
def lookup_knowledge_base(query: str) -> str:
    """
    Searches a knowledge base for information about specific topics.
    Use this tool when the user asks about programming languages,
    AI frameworks, or technology topics.
    Returns a short description if the topic is found.
    """
    knowledge_base = {
        "python": "Python is a high-level, interpreted programming language created by Guido van Rossum in 1991. Known for simplicity and readability.",
        "langchain": "LangChain is an open-source framework for building applications powered by large language models. It provides abstractions for chains, agents, memory, and tools.",
        "llamaindex": "LlamaIndex (formerly GPT Index) is a data framework for LLM applications. It excels at connecting LLMs with external data sources through indexing and retrieval.",
        "openai": "OpenAI is an AI research company known for GPT-4, GPT-4o, and the o-series reasoning models. They also created the widely-used ChatGPT product.",
        "react": "ReAct (Reasoning and Acting) is a prompting strategy for LLM agents where the model alternates between reasoning steps and tool calls to solve complex tasks.",
        "langraph": "LangGraph is a library built on top of LangChain for building stateful, multi-actor applications with LLMs, using a graph-based state machine approach.",
    }

    query_lower = query.lower().strip()

    # Try exact match first, then substring match
    if query_lower in knowledge_base:
        return knowledge_base[query_lower]

    for key, value in knowledge_base.items():
        if key in query_lower or query_lower in key:
            return value

    return f"No information found for '{query}' in the knowledge base."
```

A few things to notice here:

- The `@tool` decorator wraps a regular Python function and makes it a LangChain tool
- The docstring is literally what the model reads to decide when to use the tool — write it like documentation for the model, not for humans
- The function signature types (`str`) are used to generate the tool schema automatically
- Error handling inside tools is important: if the tool crashes, the agent should receive an error message, not an exception that kills the whole program

### Step 2 — Build the Agent

Now we put it all together. We create the LLM, bind the tools to it, and then wire up the agent loop using `create_react_agent`.

```python
# (continuing agent_langchain.py)

from langchain_openai import ChatOpenAI
from langchain.agents import create_react_agent, AgentExecutor
from langchain import hub

# ---- Build the LLM ----

llm = ChatOpenAI(
    model="gpt-4o",          # or "gpt-4o-mini" for lower cost
    temperature=0,            # 0 = more deterministic, better for agents
    api_key=os.environ["OPENAI_API_KEY"],
)

# ---- Register the tools ----

tools = [calculator, get_current_date, lookup_knowledge_base]

# ---- Load a ReAct prompt template ----
# LangChain hub has a battle-tested ReAct prompt ready to use.
# You can view it at: https://smith.langchain.com/hub/hwchase17/react

prompt = hub.pull("hwchase17/react")

# ---- Create the agent and executor ----

agent = create_react_agent(
    llm=llm,
    tools=tools,
    prompt=prompt,
)

agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,        # prints the agent's reasoning steps — keep this True while learning
    max_iterations=10,   # prevents infinite loops
    handle_parsing_errors=True,
)
```

What's happening here:

- `ChatOpenAI` is the LangChain wrapper for OpenAI models. Temperature 0 makes the model more predictable, which is what you want for agents — creativity is great for writing, not so great for deciding which tool to call.
- `hub.pull("hwchase17/react")` downloads the standard ReAct prompt from LangChain Hub. This prompt teaches the model to output its thoughts in a structured format: `Thought → Action → Action Input → Observation → Thought → ...`. You can inspect and customize this prompt later, but starting from the battle-tested version is smart.
- `AgentExecutor` is the runtime loop. It takes the agent (which is just the LLM + prompt + tools), handles the tool calls, feeds results back into the context, and runs until the agent produces a final answer or hits `max_iterations`.

### Step 3 — Run It

```python
# (continuing agent_langchain.py)

def run(question: str):
    print(f"\n{'='*60}")
    print(f"Question: {question}")
    print('='*60)
    result = agent_executor.invoke({"input": question})
    print(f"\nFinal Answer: {result['output']}")
    return result["output"]


if __name__ == "__main__":
    run("What is 15% of 847?")
    run("What is today's date?")
    run("Tell me about LangChain, and also what is 2 to the power of 10?")
```

### Step 4 — See It In Action

Run it:

```bash
python agent_langchain.py
```

With `verbose=True`, you will see the agent's reasoning printed to the terminal. It looks something like this for the first question:

```
Thought: I need to calculate 15% of 847. I should use the calculator tool.
Action: calculator
Action Input: 0.15 * 847
Observation: Result: 127.05000000000001
Thought: I now have the result.
Final Answer: 15% of 847 is approximately 127.05.
```

That output is not me writing it — that's the model's actual thought process. This is the ReAct pattern in action. It thinks, it acts, it observes the result, it thinks again.

For the third question (asking about LangChain *and* a calculation), you will see the agent call two tools in sequence: first `lookup_knowledge_base`, then `calculator`. The model understands it needs both, calls them one at a time, and combines the results into a single coherent answer.

---

## Part 2: Building the Same Agent with LlamaIndex

LlamaIndex takes a different approach. It was originally focused on RAG (Retrieval-Augmented Generation), but it has grown into a full agent framework. The abstractions are a bit different from LangChain — tools are called `FunctionTool`, and the agent is created with `FunctionCallingAgentWorker` or `ReActAgent`.

Let me build the same agent.

### Step 1 — Define the Tools

In LlamaIndex, you wrap your Python functions with `FunctionTool.from_defaults`. Same idea as LangChain's `@tool`, slightly different syntax.

```python
# agent_llamaindex.py

import os
import math
from datetime import date
from dotenv import load_dotenv
from llama_index.core.tools import FunctionTool

load_dotenv()

# ---- Define the raw functions ----

def calculator(expression: str) -> str:
    """
    Evaluates a mathematical expression and returns the result.
    Use for any arithmetic, percentage, or math calculation.
    Examples of valid expressions: '2 + 2', '0.15 * 847', 'math.sqrt(144)'
    """
    try:
        result = eval(expression, {"__builtins__": {}}, vars(math))
        return f"Result: {result}"
    except Exception as e:
        return f"Error: {e}"


def get_current_date(dummy: str = "") -> str:
    """
    Returns today's date in a human-readable format.
    Use when the user asks what day or date it is.
    """
    today = date.today()
    return f"Today is {today.strftime('%B %d, %Y')} ({today.isoformat()})"


def lookup_knowledge_base(query: str) -> str:
    """
    Looks up information about a programming or AI topic in the knowledge base.
    Topics available: python, langchain, llamaindex, openai, react, langraph.
    """
    knowledge_base = {
        "python": "Python is a high-level, interpreted programming language created by Guido van Rossum in 1991. Known for simplicity and readability.",
        "langchain": "LangChain is an open-source framework for building applications powered by large language models. It provides abstractions for chains, agents, memory, and tools.",
        "llamaindex": "LlamaIndex (formerly GPT Index) is a data framework for LLM applications. It excels at connecting LLMs with external data sources through indexing and retrieval.",
        "openai": "OpenAI is an AI research company known for GPT-4, GPT-4o, and the o-series reasoning models. They also created the widely-used ChatGPT product.",
        "react": "ReAct (Reasoning and Acting) is a prompting strategy for LLM agents where the model alternates between reasoning steps and tool calls to solve complex tasks.",
        "langraph": "LangGraph is a library built on top of LangChain for building stateful, multi-actor applications with LLMs, using a graph-based state machine approach.",
    }

    query_lower = query.lower().strip()

    if query_lower in knowledge_base:
        return knowledge_base[query_lower]

    for key, value in knowledge_base.items():
        if key in query_lower or query_lower in key:
            return value

    return f"No information found for '{query}'."


# ---- Wrap functions as LlamaIndex tools ----

calculator_tool = FunctionTool.from_defaults(
    fn=calculator,
    name="calculator",
    description="Evaluates a mathematical expression. Input should be a valid Python math expression string.",
)

date_tool = FunctionTool.from_defaults(
    fn=get_current_date,
    name="get_current_date",
    description="Returns today's date. Call with an empty string if no input is needed.",
)

knowledge_tool = FunctionTool.from_defaults(
    fn=lookup_knowledge_base,
    name="lookup_knowledge_base",
    description="Searches a knowledge base for info about tech topics like python, langchain, llamaindex, openai.",
)
```

### Step 2 — Build the Agent

LlamaIndex has a few agent implementations. I'll use `ReActAgent` here because it's the most transparent — you can see the model's reasoning steps clearly, which is great for learning.

```python
# (continuing agent_llamaindex.py)

from llama_index.llms.openai import OpenAI
from llama_index.core.agent import ReActAgent

# ---- Configure the LLM ----

llm = OpenAI(
    model="gpt-4o",
    temperature=0,
    api_key=os.environ["OPENAI_API_KEY"],
)

# ---- Create the agent ----

tools = [calculator_tool, date_tool, knowledge_tool]

agent = ReActAgent.from_tools(
    tools=tools,
    llm=llm,
    verbose=True,       # prints reasoning steps
    max_iterations=10,
)
```

One thing I like about LlamaIndex's `ReActAgent` is that it also uses the ReAct pattern under the hood, so the mental model is identical to LangChain. The differences are mostly in how you wire things together and the extra features each framework offers.

### Step 3 — Run It

```python
# (continuing agent_llamaindex.py)

def run(question: str):
    print(f"\n{'='*60}")
    print(f"Question: {question}")
    print('='*60)
    response = agent.chat(question)
    print(f"\nFinal Answer: {response}")
    return str(response)


if __name__ == "__main__":
    run("What is 15% of 847?")
    run("What is today's date?")
    run("Tell me about LlamaIndex, and also what is 2 to the power of 10?")
```

Run it:

```bash
python agent_llamaindex.py
```

You will see similar reasoning traces to the LangChain version. The output format is slightly different but the behavior — Thought → Action → Observation loop — is the same.

---

## LangChain vs LlamaIndex: What I Actually Think

Now that we built the same agent twice, let me share my honest opinion.

**LangChain** has a bigger community, more integrations, and more examples online. For production agents, LangGraph (the graph-based extension) is really powerful for complex multi-step workflows with state management. If you are building something serious and need lots of community resources, LangChain is the safer bet.

**LlamaIndex** shines when your agent needs to work with documents, databases, or external knowledge — not just tools. It has first-class support for vector stores, retrieval pipelines, and knowledge graphs built right into the framework. If your agent needs to read 500 PDFs and answer questions about them while also calling APIs, LlamaIndex's data layer is ahead.

For simple tool-calling agents? They are honestly very similar. Pick the one whose API feels more natural to you.

---

## A Few Things That Tripped Me Up

Let me save you some debugging time:

**Docstrings really matter.** If the agent is calling the wrong tool or not calling any tool at all, check the docstrings first. The model uses them to route decisions. Vague docstrings = confused agent.

**Temperature should be 0 for agents.** Higher temperature makes the model creative and unpredictable. For agents that need to follow a structured output format (Thought/Action/Input), predictability is what you want.

**`verbose=True` is your best friend while learning.** Without it, you will see the final answer but not understand why the agent did what it did. Always start with verbose on and only turn it off in production.

**`max_iterations` prevents infinite loops.** It can happen — the agent gets stuck in a loop calling the same tool repeatedly. Set a reasonable limit (10-15) and handle the `max_iterations` case gracefully.

**Eval in the calculator is a security risk in production.** Using `eval()` is fine for learning and local experiments, but never expose this to user input in a real application. Use a proper math parsing library like `numexpr` or `sympy` instead.

---

## What to Build Next

Now that you have the pattern, the fun part starts. Here are some ideas for extending this:

**Add a web search tool** using the Tavily API or DuckDuckGo. One import and a few lines and your agent can look up real information from the internet.

**Add memory** so the agent remembers previous messages in a conversation. LangChain has `ConversationBufferMemory` for this. LlamaIndex's `agent.chat()` already maintains conversation history automatically between calls in the same session.

**Connect to a real database** using a SQL tool. There are built-in integrations in both frameworks that let the agent write and run SQL queries against a database.

**Build a RAG + agent hybrid** where the knowledge base tool actually queries a vector database with thousands of documents. That's where LlamaIndex especially shines.

---

## Final Thoughts

Building AI agents is one of those things that sounds more complicated than it is, until you build one and realize: okay, it's just a loop. The LLM reasons, calls a tool, gets a result, reasons again.

The hard parts come later — handling failures gracefully, managing costs when the agent calls lots of tools, testing agent behavior reliably, keeping context from ballooning too large. But those are problems for a more advanced post.

For now, you have a working agent with both frameworks. That's the foundation for everything else.

If you build something cool with this, let me know. I'm always curious what people are actually building with these tools.

---

*Full code for both examples is available in this post. Dependencies: `langchain`, `langchain-openai`, `llama-index`, `llama-index-llms-openai`, `openai`, `python-dotenv`.*