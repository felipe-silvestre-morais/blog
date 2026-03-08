---
title: "Claude Skills: The Feature That Changed How I Use AI in My Daily Work"
date: 2026-03-05
categories: [ai]
tags: [skill, agent, llm]
---

If you use Claude regularly for engineering work, you probably already know that the quality of your output depends heavily on how good your instructions are. You give it a vague prompt, you get a vague result. You give it a detailed and specific prompt, suddenly things start to get impressive. That's just how LLMs work.

But here's the problem I kept running into: writing those detailed instructions every single time is annoying. And copy-pasting them from some Notion page every time I want Claude to do something specific is not exactly the "AI-powered future" I signed up for.

That's why I got really excited when I learned about **Claude Skills**. Let me explain what they are and why I think this changes things.

---

## What Are Claude Skills?

At its core, a skill is just a set of reusable instructions that Claude can automatically pick up whenever they are relevant to what you are doing. You write the instructions once, save them as a skill, and then Claude knows when to use them — you don't need to paste anything or even ask for it explicitly.

Think about how many times you've done something like this: you open Claude, you start explaining the context, you write your specific requirements... and then you realize you literally did the exact same setup last week for a similar task. That repetition? Skills eliminate it.

The examples that clicked for me as a developer:

- A **code review skill** that tells Claude exactly how you like your PRs reviewed — what kind of feedback format you prefer, whether you care more about performance or readability, if you follow specific naming conventions in your codebase.
- A **SQL query optimizer skill** with knowledge of your particular database setup and which indexes you have, so Claude doesn't suggest something generic that doesn't fit your schema.
- A **PR description generator** that knows your team's template, the JIRA format you use, and the level of detail your tech leads expect.

You create these skills once. Claude uses them automatically when it recognizes the context. That's the idea.

---

## How Skills Actually Work (The Technical Part)

Okay so this is where it gets interesting from an engineering perspective.

A skill is literally just a folder with a file inside called `skill.md`. That's it. The file has two parts:

**Metadata** — a name (max 64 characters) and a short description (max ~1,024 characters). This is what Claude reads upfront to understand what the skill is about.

**Body** — the actual instructions. This is your full prompt, written in Markdown. It can go up to around 5,000 tokens.

Now here's the clever bit. Claude doesn't load your entire skill into its context window from the start. It only loads the metadata — that lightweight name + description. Only when Claude decides the skill is relevant does it actually read the full body. This is called **progressive disclosure**, and it's a really smart design decision.

Why does this matter? Because context windows are precious. Imagine you have a bunch of different skills — a code review skill, a SQL optimizer, a documentation generator, a debugging assistant. If Claude loaded all of them every single conversation, you'd be burning thousands of tokens on instructions that have nothing to do with what you're currently doing. With progressive disclosure, only the metadata gets loaded up front (roughly 100 tokens per skill), and the full content comes in only when needed.

---

## Skills Can Go Deeper Than One File

One thing that surprised me: a skill is not just limited to a single `skill.md` file. Your skill folder can contain additional Markdown files and even subfolders.

Let's say you have a debugging skill. The main `skill.md` has the core instructions. But you also have a separate file called `common-errors.md` with a list of the most frequent issues in your specific stack. The main skill can reference that file, and Claude will load it only if it feels relevant — for example, if it runs into an error it's seen before.

You can also add **scripts**. Claude Code runs in an environment with Python and Node.js available, which means you can include actual executable tools as part of your skill. For example: a skill for API documentation could include a Python script that automatically fetches your OpenAPI spec and parses it. Claude would then run that script as part of doing its job, instead of you manually feeding it the spec every time.

Three levels of context, loaded progressively:

1. **Metadata** — always loaded, very small (~100 tokens)
2. **Skill body** — loaded when the skill is triggered (up to ~5,000 tokens)
3. **Extra files and scripts** — loaded on demand, practically no size limit

This is really good context management. You get the power of detailed, specific instructions without paying the token cost upfront.

---

## Okay, But How Do I Actually Set It Up?

This is the part I wish someone had shown me earlier. It's simpler than it sounds.

Skills live in a specific folder on your machine: `~/.claude/skills/`. That's it. Every subfolder inside `skills/` is treated as one skill. So your structure would look something like this:

```
~/.claude/
└── skills/
    ├── code-reviewer/
    │   └── skill.md
    ├── sql-optimizer/
    │   ├── skill.md
    │   ├── schema-notes.md
    │   └── scripts/
    │       └── fetch_indexes.py
    └── pr-description/
        └── skill.md
```

Each skill folder needs at least one file: `skill.md`. That file must start with a metadata block at the top, followed by the actual instructions. Here's what a minimal `skill.md` looks like for the code reviewer example:

```markdown
---
name: Code Reviewer
description: >
  Use when the user asks for a code review, PR feedback, or wants
  to improve code quality. Follows team conventions and focuses on
  readability and maintainability over premature optimization.
---

## How to Review Code

When reviewing code, always start by understanding the intent before
pointing out issues. Structure your feedback in three sections:

1. **What looks good** — acknowledge what is working well
2. **Suggestions** — changes that would improve the code, with reasoning
3. **Nitpicks** — minor style things that are optional to fix

Avoid rewriting entire functions unless asked. Prefer explaining
*why* something should change, not just *what* to change.

...and so on
```

The metadata block (between the `---` lines) is what Claude reads upfront — the name and description. The rest is the body, loaded only when the skill is actually needed.

If your skill needs extra context files, you just add them to the same folder and reference them in the body:

```markdown
If the user mentions a specific table or query performance issue,
load `schema-notes.md` for context about our database structure.
```

And if you want to include a script, you add it to a `scripts/` subfolder and tell Claude how to run it:

```markdown
To fetch the current index list from the database, run:
python scripts/fetch_indexes.py
```

That's the whole setup. No config files, no CLI commands, no registration step. You drop the folder in the right place and Claude picks it up automatically next time you start a session.

---

## Skills vs MCP: What's the Difference?

If you've been following the AI ecosystem, you've probably heard about MCP (Model Context Protocol). Skills and MCP can feel similar because they both give Claude additional tools and instructions. But they serve different purposes.

The biggest difference: **MCP is an open standard that works with any LLM**, while Skills are Claude-only. If you need a clean integration with an external service — say, connecting Claude to your GitHub, your database, or your ticketing system — MCP is the right tool. There are already tons of MCP servers built for popular tools, and using them saves you from having to build the integrations yourself.

Skills are better when you want to **teach Claude how to use tools it already has**. You're not adding new capabilities — you're adding specialized knowledge and workflows on top of what Claude can already do.

There's also a practical token difference. A full MCP server for something like Notion dumps all its tool metadata into the context window at startup — we're talking ~20,000 tokens right away. A Claude skill for the same use case? About 100 tokens at startup, growing only when needed. That's two orders of magnitude less noise in your context.

In practice, the best setup uses both. MCP servers for complex tool integrations, and Skills to give Claude the domain knowledge and workflow instructions to use those tools well.

---

## Where Sub-Agents Fit In

If you use Claude Code, there's another piece to this puzzle: sub-agents. These are specialized agents that run with their own separate context window, which means they don't pollute your main coding session with tokens from side tasks.

The pattern I find most useful: while working on a feature, instead of asking Claude Code to stop what it's doing and go research some library for me, I can spin up a sub-agent with a specific MCP server (like one that fetches updated documentation) to go figure that out. It does its research, returns a summary, and my main session stays clean.

The great thing is that sub-agents inherit all the skills you've already set up. So your carefully crafted instructions are available everywhere, not just in the main session.

---

## My Take

What I really like about Skills is that it respects the fact that we, as developers, have very specific ways we like things done. My code review preferences are different from yours. The SQL patterns I care about depend on my stack. The format I need for documentation depends on my team.

Previously, expressing all of that context to Claude required either a lot of manual effort or just accepting generic answers. Skills make it so that effort pays off once, and then just... works, automatically, going forward.

And honestly, the fact that you can use Claude itself to help you *write* your skills is a great touch. You describe what you want, Claude drafts the instructions, you iterate a few times, and you end up with a well-crafted skill without having to be a prompt engineer.

