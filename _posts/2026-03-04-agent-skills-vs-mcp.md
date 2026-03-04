---
title: "Agent Skills vs MCP: What's the Difference?"
date: 2026-03-04 12:00:00 -0300
categories: [AI, Agents]
tags: [mcp, ai-agents, skills, llm, anthropic]
author: Felipe Silvestre Morais
---

Okay, minha gente — let's talk about something that I genuinely got giddy about this week. If you're building AI agents (or thinking about it), you've probably already heard of **MCP**. But have you heard of **Agent Skills**? Because if you haven't, buckle up. This is one of those "aha!" moments that makes you want to call your developer friends at midnight.

## First: Why Do AI Models Need Tools at All?

AI models are getting **insanely good**. There's a benchmark called GPQA — hard questions about biology, chemistry, physics — and modern models are hitting near **100%** on it. There's also a real-world software engineering benchmark (actual GitHub issues!), and models went from 20% to **80% accuracy**.

But here's the thing: a smart model with no tools is like a genius locked in a room with no internet. Not very useful.

Imagine you ask an AI: *"Find bugs in my codebase."* Without tools, it says: *"I don't have access to your codebase."* Cool, thanks. Very helpful.

But if you give it access to GitHub → it reads the code and finds obvious bugs.  
Add a code interpreter → now it runs tests and finds *silent* bugs.  
Add documentation → now it follows *your team's coding standards*.

Same model. Completely different results. **The model is only as good as the tools and context you give it.**

## Enter MCP: The USB-C Port of AI

This is the problem that **Model Context Protocol (MCP)** was built to solve. Released by Anthropic in **November 2024**, MCP is an open standard — a universal way to give LLMs the right tools and context.

The analogy: MCP is the **USB-C port of AI applications**. Just like your laptop can connect to any device via USB-C, MCP lets any AI app (Claude, ChatGPT, Cursor…) connect to any tool (Gmail, Slack, Notion, web search…).

### How MCP Works — The Coffee Shop Analogy

MCP follows a **client-server architecture**:

- **You** = the MCP client (making requests)
- **The barista** = the MCP server (responding)

You ask: *"Can I get a pumpkin spice latte?"*  
The barista responds: *"You got it."* ☕

In MCP terms: the client asks the server to list available tools. The server responds with all tools and their schemas. Simple, elegant, and widely adopted.

## The Problem with MCP: Context Rot 🧟

MCP is great — but it has a dirty little secret: **it's token-heavy.**

When the MCP client starts up, it asks the server to list all available tools. The server responds with a verbose description of every tool and schema. Most of those tools are irrelevant to your current task, but they're eating up precious context window space.

This is called **context rot** — irrelevant information degrading the model's performance. More tokens = more money 💸, and it can actively make the AI *worse*.

## Enter Agent Skills: Context on Demand 🎯

**Agent Skills** are a folder with instructions that an agent loads *only when needed*. The structure:

```
my-skill/
  skill.md         ← main instructions file
  references/      ← optional documentation
  assets/          ← templates, resources
  scripts/         ← executable Python/JS code
```

The magic is the **Progressive Disclosure** model — three levels:

| Level | What's Loaded | Token Cost |
|-------|--------------|-----------|
| 1 | Skill metadata (front matter) | ~100 tokens |
| 2 | Full skill body | Up to 5,000 tokens |
| 3 | Extra files, scripts, references | Practically unlimited |

The agent only goes deeper if it needs to. It's like handing someone a 200-page manual vs. showing them the table of contents and letting them flip to the right chapter.

## MCP vs Skills: The Breakdown

| | MCP | Agent Skills |
|---|---|---|
| **Purpose** | Gives agents **tools** | Gives agents **instructions** |
| **Context loading** | All schemas at startup | Loaded progressively, on demand |
| **Setup** | Requires code | Plain English |

The key mental model:

> **MCP = tool access. Skills = task instructions.**

Example: connect Claude to Notion using the **Notion MCP server** (tool access), then use a **Skill** to tell the agent *how* to create a specific type of Notion page (task instructions). One gives you the hammer. The other tells you how to build the house.

## Final Thoughts

MCP took 3–6 months to become widely adopted after its release. Skills are following a similar path and already picking up steam. A year from now, I'd bet they'll be just as ubiquitous.

We're no longer just building features — **we're designing how agents think and act**. That's a different kind of engineering, and honestly? It's the most exciting thing I've worked with in years.

Now if you'll excuse me, I need to go rewrite all my agents with Skills. 🚀
