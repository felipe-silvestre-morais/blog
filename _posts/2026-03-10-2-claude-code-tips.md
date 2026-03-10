---
title: "Claude Code: The Tips I Wish I Had on Day One"
date: 2026-03-10 14:00:00
author: Felipe Silvestre
categories: [ai, dev-tools]
tags: [claude-code, productivity, ai, workflow]
---

I've been using Claude Code almost every day for the past few months, and my relationship with it went through some very specific phases. First: wow, this is incredible. Second: why does it keep asking me for permission to do anything. Third: okay I think I understand how this actually works. Fourth: I cannot imagine going back.

If you're somewhere between phase one and three, this post is for you. I want to share the things that actually made a difference in how I use it — from basic setup to some patterns that genuinely changed my workflow.

Let me start with one important thing that took me a while to internalize: Claude Code is not a faster Copilot. It's something different. It's an agent that can read your whole codebase, run commands, write files, execute tests, and reason about multi-step problems. To get the most out of it, you have to think about it differently.

---

## First Things First: Where You Run It Matters

This sounds obvious but I missed it at first. Always run Claude Code from the **root of your project**. Not from your home directory, not from a subdirectory. The root.

```bash
cd ~/my-project
claude
```

Claude Code uses your current directory to understand the context of your project. It reads file structure, existing code, configuration files, and more. If you start it from the wrong place, it's like asking a senior engineer to review your PR without giving them access to the repo.

Once you're in the right directory, the first thing you should run is:

```bash
/init
```

This generates a `CLAUDE.md` file at your project root. And this file — I promise — is one of the most impactful things you can set up.

---

## CLAUDE.md: Your Persistent Memory for the Project

Every new Claude Code session starts fresh. It doesn't remember that you always use `pnpm` instead of `npm`, or that your backend is Python 3.12, or that you have a specific pattern for error handling that you don't want it to change.

`CLAUDE.md` solves this. It's a Markdown file that Claude Code reads at the beginning of every session, before it does anything. Think of it like an onboarding document you wrote for a new engineer on your team.

Here's what a useful `CLAUDE.md` looks like:

```markdown
# Project: E-commerce API

## Tech Stack
- Python 3.12, FastAPI, SQLAlchemy (async), PostgreSQL
- Dependency management: uv (NOT pip or poetry)
- Testing: pytest with pytest-asyncio

## Code Standards
- All new endpoints must have type hints
- Use Pydantic v2 models for request/response schemas
- Never use `SELECT *` in raw SQL queries
- Prefer async functions for all I/O operations

## Project Structure
- `api/` - FastAPI routers
- `services/` - Business logic layer
- `models/` - SQLAlchemy models
- `schemas/` - Pydantic schemas

## Commands
- Run tests: `pytest tests/ -v`
- Start dev server: `uvicorn api.main:app --reload`
- Database migrations: `alembic upgrade head`

## Rules
- Before implementing anything non-trivial, write a plan and ask me to confirm
- Always run the test suite after any change to the models layer
- Don't change pyproject.toml without asking first
```

Now every session, Claude Code knows your stack, your conventions, and your rules — without you having to explain it again. You can also put a `CLAUDE.md` inside subdirectories, and it will be loaded when Claude is working in that folder. Useful for monorepos where different parts of the codebase have different rules.

---

## The Slash Commands You Actually Need

Claude Code has a bunch of slash commands. Let me focus on the ones I use every single day.

### `/clear` — Use it more than you think

Every time you start a new, unrelated task, clear the context:

```
/clear
```

This resets the conversation without exiting Claude Code. Why does this matter? Context is expensive. Not just in money — in quality. A long, messy context with half-finished discussions and old code snippets makes Claude less accurate on your new task. Fresh context = better results.

I use `/clear` between features, between debugging sessions, basically any time I'm switching what I'm doing.

### `/compact` — When you don't want to start over

Sometimes you've been working on something for a while, and the context is getting large, but you're not done. `/compact` summarizes the current conversation into a shorter representation. You keep the relevant history without the bloat.

```
/compact
```

It's not perfect — sometimes important details get dropped in the summary. But it's often better than starting over with `/clear` in the middle of a complex task.

### `/context` — Understand what Claude actually sees

This one is underused. `/context` shows you a breakdown of what's currently in Claude's context window, including token counts. I check this when Claude starts giving responses that feel off — usually it's because the context is too full or has too much irrelevant stuff.

```
/context
```

### Plan Mode — The most important workflow change

When you ask Claude to plan before acting, it will describe what it intends to do and wait for your approval before making any changes. You can trigger this by starting your message with the word "plan":

```
plan: refactor the authentication module to use JWT instead of sessions
```

This is especially useful for large changes spanning multiple files, anything touching the database or infrastructure, or features you haven't fully thought through yet.

I learned this the hard way after Claude confidently rewrote a bunch of stuff in a way that was technically correct but completely wrong for my architecture. Now I plan first on anything non-trivial. The plan often reveals misunderstandings early, which is exactly when you want to catch them.

---

## Stop Fighting the Permission System (But Do It Smartly)

One of the most frustrating things about Claude Code when you first use it is that it asks permission for everything. Edit a file? Permission. Run a command? Permission. It's like having an assistant who needs approval to use the stapler.

There are two ways to handle this.

**Option 1: Trust mode for specific commands.** You can configure Claude Code to automatically approve certain tools in your project settings. Add a `.claude/settings.json` file:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm run test)",
      "Bash(npm run lint)",
      "Bash(git diff)",
      "Bash(git status)"
    ]
  }
}
```

Now it runs tests and linting without asking. Still asks for file edits and other commands. This is the safer approach.

**Option 2: Skip permissions entirely.** If you know what you're doing and want maximum speed:

```bash
claude --dangerously-skip-permissions
```

Yes, the name is a bit dramatic. In practice, if Claude makes a destructive mistake, it's rarely something you can't undo with `git checkout` or `git reset`. The risk is real but manageable if you're working on a project with good git hygiene. I use this often during exploration phases, and I'm more careful when working on production code.

---

## Context is Like Milk: Keep it Fresh

Here's a mental model that changed how I use Claude Code: **context is a resource that degrades over time**.

The longer your session, the more context Claude has to carry. After a while, it's holding the full conversation history, every file it's read, every command it ran. This makes it slower to reason, more likely to contradict earlier decisions, and more expensive.

Some things that help:

**Start each session with a clear goal.** Before you type anything, know what you want to accomplish in this session. Don't use Claude Code as a general-purpose "let me explore" tool without boundaries.

**Be specific about files.** Instead of asking Claude to "look at the codebase and figure out how authentication works", try "@src/auth/middleware.py — explain how the JWT validation works here". You're controlling what enters the context.

**Use `@` to reference files explicitly.** Claude Code supports `@filename` syntax to include specific files in context. This is much better than hoping Claude will find the right file on its own.

```
How should I extend @src/models/user.py to support OAuth providers?
```

**Don't load the whole repo when you don't need it.** If you're working on the frontend, you probably don't need Claude to read all your backend code too. Be deliberate about what goes in.

---

## Running Multiple Sessions in Parallel

This one feels a bit advanced but it's actually very practical.

There's nothing stopping you from opening multiple terminal windows and running Claude Code in each of them. One session works on a new feature, another debugs a different issue, another writes tests for something you just built.

The key rule: **don't let parallel sessions touch the same files**. If they do, you'll end up with conflicts and confusion. Use separate branches for parallel work, or make sure the sessions are working on truly independent parts of the codebase.

I usually run two sessions. One for the actual development task, and one where I ask questions, explore options, and think through approaches without worrying about it making changes. That second session is kind of like rubber duck debugging, but the duck is very smart.

---

## MCP Integration: Give Claude the Tools It Needs

If you've read my post about MCP, you know what this is. If not, the short version: MCP (Model Context Protocol) lets you connect Claude to external tools and data sources in a standardized way.

Claude Code has first-class support for MCP servers. You can add them with:

```bash
claude mcp add -s user playwright npx @playwright/mcp@latest
```

This adds Playwright as a user-level MCP server, meaning it's available across all your Claude Code sessions. Now Claude can open browsers, interact with your web app, take screenshots, and inspect what's actually rendered — which is extremely useful for building and testing UI.

Some MCP servers that are genuinely useful for development:
- **Playwright** — Browser automation. Claude can test UI interactions for real.
- **GitHub MCP** — Create and review PRs, manage issues, all from Claude Code.
- **Filesystem servers** — Give Claude access to specific directories outside your project.

One setup I really like: run `/install-github-app` and Claude will automatically review your PRs. As you ship code faster with AI tools, PR volume increases naturally. Having automatic reviews before merging is a nice quality check.

---

## Write Better Prompts for Code Tasks

The quality of what Claude Code does is directly proportional to how clearly you explain what you want. I know this is obvious, but I see people giving very vague instructions and then being surprised when the result is vague.

Some patterns that consistently work better:

**Describe the goal, not the implementation.** Instead of "add a cache decorator to this function", try "this endpoint is slow because it queries the database on every request — the data changes once per hour at most, what's the best way to cache it?" Claude will figure out the right approach, and it might be better than what you were thinking.

**Give it the context it needs.** "Fix the bug" is not enough. "The `/users/profile` endpoint returns 500 when the user has no profile picture set — the error is a NullPointerException in `ProfileSerializer.to_dict()`. Fix it." That's something Claude can actually work with.

**Tell it what constraints exist.** "Refactor this to be more readable, but don't change the public interface — this module is imported by 15 other files and I can't break backward compatibility."

**Ask for explanation alongside the code.** "Implement this and explain the tradeoffs you considered." This is more expensive in tokens, but it catches misunderstandings before they become commits.

---

## Custom Commands: Automate Your Repetitive Workflows

Claude Code supports custom slash commands. You create a Markdown file inside `.claude/commands/` and it becomes available as `/command-name`.

For example, I have a `create-pr` command:

```markdown
# .claude/commands/create-pr.md

Review all my recent changes with `git diff main`, write a clear PR description 
that includes:
- What changed and why
- How to test it
- Any migration steps needed

Then create a draft PR with `gh pr create --draft` using this description.
```

Now I just type `/create-pr` and Claude handles the whole PR creation. It reads my diffs, writes a description that's actually informative, and opens the draft PR. This saves me maybe 10 minutes per PR, and it's done consistently every time.

Other custom commands I use regularly:
- `/review-changes` — Do a self-review of everything I changed in this session before I commit
- `/explain-tests` — Run the test suite and explain any failures in plain language
- `/update-docs` — After I change an API endpoint, update the relevant documentation files

Custom commands are small automation investment that compound over time. You set them up once and they just work.

---

## A Few Honest Caveats

Claude Code is genuinely good, but it's not perfect and I want to be honest about that.

It makes mistakes. Not constantly, but enough that you need to review what it does. Don't let it run for 20 minutes on a complex task and then commit everything without reading the diff. The changes are usually directionally correct but sometimes wrong in the details.

It can get confused in very large codebases. Context limitations are real. If your project has hundreds of files and complex interdependencies, Claude Code sometimes loses track of things. This is where a well-written `CLAUDE.md` and disciplined use of `/clear` and explicit `@file` references becomes really important.

It's expensive if you're careless. Long sessions with lots of file reads add up. Being intentional about context management is not just about quality — it also affects cost.

But honestly? Even with all of that, it's still one of the most productive tools I've ever used for software development. The ability to say "I need to add rate limiting to all our API endpoints" and have it plan the approach, implement it across all the relevant files, and run the tests — in one conversation — is remarkable.

---

## The Summary, If You Want to Start Right Now

1. Install: `npm install -g @anthropic-ai/claude-code`
2. Run from your project root: `cd your-project && claude`
3. Run `/init` to create your `CLAUDE.md` and fill it with context about your project
4. Use `/clear` aggressively when switching tasks
5. Use `plan:` prefix for any non-trivial change
6. Reference files with `@` to control what goes into context
7. Add custom commands for the things you do repeatedly

Start with those and you'll already be ahead of most people using this tool.

---

*The official docs are at [code.claude.com/docs](https://code.claude.com/docs). If you want me to go deeper on any specific part — MCP integration, multi-agent workflows, or scripting Claude Code for CI/CD automation — let me know in the comments.*