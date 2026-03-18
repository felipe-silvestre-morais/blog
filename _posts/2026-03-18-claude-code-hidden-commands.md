---
title: "Claude Code's Hidden Superpowers: /btw, /fork, /rewind, and the Context Hygiene Toolkit"
date: 2026-03-18 15:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [claude-code, developer-tools, terminal, productivity, anthropic]
---

If you've been using Claude Code for a while, you probably know the basics: type a prompt, watch it work through your files, be amazed. But there's a whole layer of commands that most people just don't know about, and honestly, they make a huge difference in how much you can get done.

I've been using Claude Code as part of my daily workflow for a while now, and recently I went deep on a specific category of commands that I think are the most underappreciated feature of the whole tool: **context hygiene**. The idea is simple but powerful — you can control exactly what's in the AI's "memory" at any point, which turns out to be critical for getting good results on complex tasks.

Let me walk you through the commands that actually changed how I work.

---

## Why Context Hygiene Matters

Before getting into specific commands, I want to explain why this concept even matters.

Claude Code maintains a conversation history as you work. That history is the context window — everything Claude "knows" about your session. As sessions get longer, a few things start to happen: context gets polluted with dead ends and abandoned ideas, you start approaching the context window limit, and Claude starts making decisions based on old information that's no longer relevant.

The instinct is to just start a new session when things get messy. But that's throwing away valuable context! The smarter move is to use the tools Claude Code gives you to *shape* what's in context, not just nuke it.

That's what this post is about.

---

## /btw — The Newest and Most Exciting One

I have to start here because `/btw` is brand new. It was released in Claude Code v2.1.72 on March 10, 2026, built by Erik Schluntz on the Claude Code team. When Thariq Shihipar (the Claude Code lead) announced it on March 11, the tweet hit 2.2 million views in a few days. That kind of attention tells you the command is solving a real pain point.

Here's the problem it solves: you're deep in a complex refactoring task. Claude is executing it. Then suddenly you need to check something — "wait, which file has the error handler for this service?" or "what was the exact type signature we used earlier?"

Before `/btw`, you had two bad options: interrupt the whole task to ask your question (which pollutes the conversation context), or wait until it finishes and hope you remember what you wanted to ask.

`/btw` is a third option that is much better:

```bash
# Ask a side question while Claude is already mid-response
/btw which test file covers the auth middleware?

# Quick clarification without polluting context
/btw what's the difference between useEffect and useLayoutEffect?

# Check something without derailing the current task
/btw did we already update the error types in types.ts?
```

The answer comes back immediately, and — this is the key part — **it is not added to the conversation history**. The main task keeps running, and your context stays clean.

The mental model I find helpful: `/btw` is the inverse of a sub-agent. A sub-agent has full tool access but starts with an empty context. `/btw` has full visibility into your current conversation but has no tools. It can only answer from what's already in context — no file reads, no web searches, no code execution. Just a quick answer based on everything Claude already knows about your session.

A few things to know about how it works:

- Single response only — you can't have a back-and-forth `/btw` conversation
- No tool access — it reads from context, it cannot look things up
- Very low cost — it reuses the prompt cache, so tokens are minimal
- Dismiss the answer with Space, Enter, or Escape and continue working

Use `/btw` liberally. Side questions are essentially free, context-wise. Don't pollute your main working context with "wait, how does X work again?" tangents.

---

## /fork — The Git Branch for Your Conversation

This one is genuinely underused, and I think it's because the concept needs a moment to click.

`/fork` creates a branch of your current conversation at its current state. You can explore one approach in the fork, then use `/resume` to come back to the fork point and try a completely different approach. It's the conversational equivalent of `git checkout -b experiment`.

Here's a concrete example: imagine you're trying to refactor a complex service and you have two ideas about how to structure it. Instead of picking one and committing, you can:

```bash
# You're at a decision point. Fork here.
/fork

# Now try approach A — maybe a class-based refactor
> Refactor the UserService using a repository pattern

# Hmm, that's getting complicated. Let me try approach B.
/resume   # back to the fork point

# Now try approach B — simpler functional approach
> Refactor the UserService by extracting pure functions
```

This is especially powerful when you're exploring approaches you're not sure about. Instead of making a mess of your working session, you fork, experiment, and come back if it didn't work. No manual `git stash`, no "undo everything I just did."

`/fork` is also available as a context option in Skills (more on that in a future post), where you can run sub-tasks in a forked context so they don't pollute your main session.

---

## /rewind — The Safety Net

Every time you work with Claude Code, it creates implicit checkpoints as you go. `/rewind` lets you step back through those checkpoints when things go wrong.

The basic use case is obvious: Claude misunderstood something and made a bunch of changes you don't want. Instead of manually reverting files one by one and trying to explain what went wrong, you just rewind:

```bash
# Roll back conversation and file changes to a previous checkpoint
/rewind

# Or use the keyboard shortcut (double-tap Escape)
Esc Esc
```

But the really interesting part is the **selective rollback** feature added in 2026. When you open the rewind menu, you get two options:

- **Rewind everything** — reverts both the conversation history and all file changes
- **Rewind code only** — reverts all file changes while *keeping* the conversation history intact

That second option is surprisingly useful. Imagine you tried an aggressive refactoring approach. You discuss the results with Claude, decide together that the approach didn't work, but you want to keep that analysis conversation because it's useful context for the next attempt. With "rewind code only," you get exactly that — files reverted, conversation kept.

A nice pattern that works well: pair `/diff` with `/rewind`. Use `/diff` to review what Claude actually changed across your files, and if you don't like what you see, `/rewind` to undo it. Review, decide, keep or discard.

---

## /compact — The Token Lifesaver

This one is not as glamorous as `/btw` or `/fork` but I use it probably more than any other context command.

When a long session fills the context window, `/compact` compresses the conversation history into a dense summary. The session stays alive, Claude keeps the important context, but the token count drops significantly.

```bash
# Basic compact — Claude decides what to summarize
/compact

# Compact with focus instructions — you guide what survives
/compact Focus on the auth module and the failing test patterns
/compact Retain the database schema decisions and API contract
```

The second form is much more useful. When you pass instructions, you can tell Claude what matters to keep and what can be condensed. This is basically manual context engineering — you're shaping what the AI "remembers" going into the next part of the session.

A practical rule: when `/context` shows you're above 80% context usage, run `/compact` with focus instructions before continuing. If you just let it hit the ceiling, Claude will either do a forced compact on its own (with less guidance) or you'll hit errors.

---

## /clear — When You Actually Want to Start Fresh

Not everything needs a nuanced approach. Sometimes you're just done with a task and want to start something completely different. That's what `/clear` is for — it wipes the conversation history entirely and frees all context.

```bash
/clear    # also works as /reset or /new
```

The distinction from `/compact` is important: `/compact` is for staying in the same task with reduced context. `/clear` is for task boundaries when the old context would only confuse Claude.

If you closed a session yesterday and want to pick up where you left off, `/resume` is what you want — not `/clear`. Claude Code saves session history and you can resume it even after closing the terminal.

---

## /diff and /copy — The Little Helpers

Two smaller commands that round out the workflow nicely.

**`/diff`** shows you what file changes Claude has made during the current session. Before you accept a big refactor, before you commit, before you `/rewind` — just run `/diff` and see exactly what happened. It's like `git diff` but scoped to your Claude Code session.

**`/copy`** copies Claude's last response to your clipboard. Sounds simple, but there's a nice detail: if the response contains multiple code blocks, it opens an interactive picker so you can choose which one to copy. Much faster than manually selecting text in the terminal.

---

## Tracking Usage: /cost, /usage, and /context

These three commands are about awareness — knowing where you stand before things go sideways.

```bash
/context   # How full is my context window?
/usage     # How much of my plan quota have I consumed?
/cost      # How many tokens and dollars for this session? (API users only)
```

`/context` is the one I run most often. It shows your current context window usage and gives a warning if skills/commands are being excluded because of the budget. If you see that warning, run `/compact` immediately.

`/usage` is for subscription users (Pro/Max) — it shows your plan-level limits and rate limit status. `/cost` is the equivalent for API users, showing actual token cost in dollars. You'll only see the relevant one depending on how you access Claude Code.

---

## A Quick Reference

Here's the mental model I use to pick the right command:

| Situation | Command |
|---|---|
| Need to ask a quick question without breaking flow | `/btw` |
| At a decision point, want to try two approaches | `/fork` |
| Claude made a mess, I want to undo everything | `/rewind` |
| Claude made a mess, but I want to keep the conversation | `/rewind` → "Rewind code only" |
| Context is getting full, still on the same task | `/compact [focus instructions]` |
| Switching to a completely different task | `/clear` |
| Want to see what changed in my files | `/diff` |
| Need to copy a code block from Claude's last response | `/copy` |
| Want to check how much context I have left | `/context` |

---

## Final Thoughts

When I first started using Claude Code, my mental model was basically: "big powerful AI that does stuff with my code." And that's true! But the thing that actually made me more productive was learning to think about sessions more deliberately — what's in context, what should stay, what should go.

The commands in this post are the tools for that. `/btw` for side questions, `/fork` for exploration, `/rewind` for safety, `/compact` for longevity. Together they give you real control over the AI's working memory, which makes a surprisingly large difference on complex, multi-step tasks.

If I had to pick just one to start with: try `/btw` next time you're in a long Claude Code session and feel the urge to interrupt with a quick question. That little change alone might be the one that sticks.

---

*For the official command reference, run `/help` inside any Claude Code session. Anthropic also maintains documentation at [docs.anthropic.com/en/docs/claude-code/overview](https://docs.anthropic.com/en/docs/claude-code/overview).*