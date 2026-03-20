---
title: "Claude Code Custom Commands: Turn Repetitive Prompts into One-Word Superpowers"
date: 2026-03-20 13:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [claude-code, developer-tools, productivity, custom-commands, slash-commands]
---

I've been using Claude Code almost every day for a while now, and if there's one feature that changed the most how I actually work with it, it's custom commands. Not the built-in slash commands like `/clear` or `/compact` — those are great but they come out of the box. I'm talking about the ones *you build yourself*, the ones that encode your own workflows and your own taste into a single word.

This post is a practical guide on how to create custom commands, how to make them genuinely powerful, and some concrete examples I use that I haven't seen covered elsewhere. If you already know the basics of Claude Code's built-in commands from my earlier guide, this is the natural next step.

---

## What Custom Commands Actually Are

Think of a custom command as a prompt you've saved with a name. Instead of typing the same 10-line prompt every time you want Claude to do something, you write it once in a Markdown file, give it a name, and from that moment on you just type `/command-name` and it runs.

At their core, custom slash commands are Markdown files that contain instructions for Claude Code. When you invoke a slash command, its content becomes the prompt sent to Claude.

The moment I understood that — that the whole file contents become the prompt — things clicked. You can write a really detailed, careful, opinionated prompt in there and Claude will follow it every single time. No more hoping you phrased the request well enough. You phrased it once, correctly, and now it's reusable.

---

## Two Types of Commands: Personal vs Project

Before creating anything, you need to decide where to put the file. There are two locations and they serve different purposes.

**Personal commands** go in `~/.claude/commands/`. They are available in every project you work on. These are for your own preferences and workflows that don't belong to any specific codebase.

**Project commands** go in `.claude/commands/` inside your project folder. They are version-controlled and shared with anyone else working on the repository, ensuring consistent procedures and prompts across the team. This is actually very powerful for teams — you commit the `.claude/commands/` folder to git and everyone gets the same commands automatically.

My personal rule: if it's about my coding style or something only I care about, it goes in `~/.claude/commands/`. If it's about the project — how to run tests, how to create a new migration, how PRs should look — it goes in the project folder.

---

## Creating Your First Command

The mechanics are simple. Create a Markdown file, write your prompt, done.

```bash
# Personal command available in all projects
mkdir -p ~/.claude/commands
cat > ~/.claude/commands/explain.md << 'EOF'
Explain this code clearly. Assume I understand programming but not necessarily this codebase. 
Use an analogy if it helps. Highlight any non-obvious decisions or gotchas.
EOF
```

Now in any Claude Code session you can type `/explain` and it will use that prompt. The file name becomes the command name — `explain.md` becomes `/explain`.

---

## Making Commands Dynamic with Arguments

Static prompts are useful but arguments are where things get interesting. You can use `$ARGUMENTS` to grab everything typed after the command, or get more specific with positional arguments like `$1`, `$2`, and so on.

Here's a practical example. Let me say you work with GitHub Issues. Instead of explaining the whole context every time, you write this once in `.claude/commands/fix-issue.md`:

```markdown
Please analyze and fix the GitHub issue: $ARGUMENTS

Follow these steps:
1. Use `gh issue view $ARGUMENTS` to read the full issue description and comments
2. Search the codebase to understand which parts are affected
3. Implement the fix with tests
4. Write a clear commit message referencing the issue number
```

And now `/fix-issue 247` just works. Claude reads the issue, finds the relevant code, fixes it, and commits — all from two words in your terminal.

You can also use positional arguments when you need more control:

```markdown
---
argument-hint: [issue-number] [priority]
description: Fix a GitHub issue with priority context
---
Fix issue #$1 with priority $2. If priority is "high", include extra test coverage.
Check the issue description and implement the necessary changes.
```

Then `/fix-issue 247 high` passes `247` as `$1` and `high` as `$2`. Much more expressive.

---

## Frontmatter: Where Commands Get Really Powerful

Plain Markdown works fine for simple prompts, but for even more fine-grained control, you can add a block of YAML frontmatter to the very top of your command's Markdown file to control how the command behaves.

The fields I use most:

**`description`** — Shows up in `/help`. Without this, your command is just a filename with no context. Always add a description.

**`allowed-tools`** — This one is the most important. The "allowed-tools" list lets you grant the command permission to run specific shell scripts without requiring repeated approval. You know that annoying thing where Claude keeps asking permission to run git commands? With `allowed-tools`, you pre-approve the specific operations the command needs.

**`argument-hint`** — Shows autocomplete hints when you start typing the command. Helpful for complex commands you'll use rarely and forget the argument format.

**`model`** — Force a specific model for the command. Useful for quick, cheap tasks where you don't need Opus.

**`disable-model-invocation`** — This one is subtle but important. If you set `disable-model-invocation: true`, only you can invoke the skill. Use this for workflows with side effects or that you want to control timing, like `/commit`, `/deploy`, or `/send-slack-message`. You don't want Claude deciding to deploy because your code looks ready.

Here's a real git commit command that combines all of this:

```markdown
---
allowed-tools: Bash(git add:*), Bash(git status:*), Bash(git diff:*), Bash(git commit:*)
argument-hint: [optional commit message]
description: Stage all changes and create a smart commit message
disable-model-invocation: true
---

## Context
- Current status: !`git status`
- Current diff: !`git diff HEAD`
- Current branch: !`git branch --show-current`

## Task
Stage all changes and create a commit. If $ARGUMENTS is provided, use it as the commit message. 
Otherwise, write a clear, concise commit message based on the diff above.

Follow conventional commits format: feat/fix/refactor/docs/chore.
Check for any TODO comments, debug logs, or hardcoded secrets before committing — if found, warn me instead of committing.
```

Notice the `!` prefix on bash commands inside the content. That's how you execute bash commands and embed their output directly in the prompt. When you run `/commit`, Claude already has the current diff and status injected into the prompt — it doesn't need to go fetch them.

---

## Commands vs Skills: What Changed in 2026

If you look at the current documentation, you'll notice that Anthropic now recommends `.claude/skills/` over `.claude/commands/`. In 2026, the traditional custom commands and Skills have been unified. Existing `.claude/commands/` files continue to work. Skills add frontmatter-based auto-invocation control and support file management on top of commands.

The practical difference: commands are things *you* invoke manually with `/command-name`. Skills can also be invoked by Claude automatically when it detects the task matches the skill's description.

For most custom workflows — the things you want to run deliberately — the `.claude/commands/` pattern still works perfectly and I keep using it. For things that benefit from automatic invocation (like a "fetch library docs" skill that Claude should use whenever you ask about that library), the Skills format is better.



---

## Commands I Use That You Probably Don't Have

Here are some commands from my own setup that go a bit beyond the typical examples you see:

### `/catchup` — Reload context after `/clear`

One of my most used commands. The idea is that you can call `/clear` as the context window gets full, then reload the necessary work-in-progress into your new conversation.

```markdown
---
description: Reload work-in-progress context after a /clear
allowed-tools: Bash(git status:*), Bash(git diff:*)
---

Read all uncommitted git changes and load them into context:
- !`git status`
- !`git diff HEAD`

Summarize what work is in progress so we can continue where we left off.
```

### `/deploy-check` — Pre-flight before shipping

Before I push anything significant I run this:

```markdown
---
description: Pre-deployment quality check
allowed-tools: Bash
disable-model-invocation: true
---

Run a pre-deployment checklist:
1. Run the full test suite and report results
2. Run the linter and show any errors (not warnings)
3. Build for production and check for errors
4. Search for TODO, FIXME, console.log, and hardcoded secrets in staged files
5. Check that environment variable references are consistent

Do NOT auto-fix anything. Only report. I want to review before proceeding.
```

The `Do NOT auto-fix anything` line is intentional — the power of custom commands is you can encode your preferences very precisely.

### `/new-post` — For this exact blog

Yes, I use Claude Code to bootstrap new blog posts:

```markdown
---
description: Create a new Jekyll blog post with frontmatter
argument-hint: [post title]
allowed-tools: Bash(date:*), Write
---

Create a new Jekyll blog post for my blog at _posts/:
- File name format: !`date +%Y-%m-%d`-$ARGUMENTS with spaces replaced by hyphens, all lowercase
- Include full frontmatter: title, date, author (Felipe Silvestre), categories, tags
- Add a placeholder introduction paragraph
- Leave the rest empty with section headings as comments

Title: $ARGUMENTS
```

### `/review-pr` — Focused code review

Different from the GitHub Actions integration. This is for when I'm the author and want to catch my own mistakes before asking others:

```markdown
---
description: Self-review before opening a PR
allowed-tools: Bash(git diff:*), Bash(git log:*), Read
---

Review the changes in this branch as if you're a senior engineer doing a code review.
Current changes: !`git diff main...HEAD`
Recent commits: !`git log main..HEAD --oneline`

Look specifically for:
- Logic errors and edge cases not covered by tests
- Security issues (injection, auth gaps, exposed data)
- Performance problems that would only show up at scale
- Missing error handling
- Public API changes that break backwards compatibility

Be direct. Don't praise things, just flag problems. If it looks fine, say so briefly.
```

### `/lib-context $1` — Fetch current docs for a library

Inspired by the Dexie.js example I found in the community, this one solves a real problem: Claude's training data for third-party libraries gets stale fast.

```markdown
---
description: Fetch current documentation for a library before working with it
argument-hint: [library name or docs URL]
allowed-tools: WebFetch
---

Fetch the current documentation for: $ARGUMENTS

1. Try fetching https://[library].dev/llms.txt or similar LLM-friendly docs if the URL is not provided
2. Read the key concepts and current API surface
3. Summarize what you learned and flag any patterns that might differ from what you already know

After this, you'll have current context to help me with $ARGUMENTS code.
```

---

## Tips for Writing Good Commands

A few things I learned the hard way:

**Be specific about what you don't want.** Vague prompts give vague results, even as commands. "Review this code" is weaker than "Review this code for security issues only, do not comment on style."

**Use `disable-model-invocation: true` for anything with side effects.** Anything that writes files, commits, deploys, or sends messages should only run when you explicitly type it. Don't let Claude decide to deploy your code because it thought the task was complete.

**Add `description` to every command.** Run `/help` in a session. Commands without descriptions are just a list of meaningless names. A one-line description takes 10 seconds and makes your command actually discoverable.

**Keep project commands in version control.** The `.claude/commands/` folder should be committed to your repo. This is team knowledge, not personal config. When a new engineer joins, they get all your workflows for free.

**Use `!` for bash output injection liberally.** Commands that inject live context (current branch, test results, git diff) are so much more powerful than commands that just send static text. The `!` syntax is underused.

---

## Starting Point: Let Claude Build Your Commands

Here's a meta-trick: if you're not sure how to structure a command for your workflow, just ask Claude Code directly. Describe what you want to automate and ask it to write the command file for you. Claude Code supports custom hooks, slash commands, and project-specific configuration — and you can have Claude build these for you.

I've done this several times. I describe my workflow verbally, Claude writes the command, I tweak it a bit, and now it's part of my setup forever. The whole thing takes five minutes.

---

## Final Thoughts

Custom commands are one of those things that look like a small quality-of-life feature until you actually start using them seriously. Then they start to feel less like shortcuts and more like encoding your own expertise into the tool.

The prompts you write once, carefully, tend to be better than the ones you type quickly under pressure. And every time you run that command, you get the careful version — not the rushed version. That's kind of the whole point.

Start with one command for something you do every day. Get the frontmatter right, add the bash context injection, test it a few times. Once you see it work, you'll want to build five more.

---

*Resources: [Claude Code custom commands docs](https://docs.claude.com/en/docs/claude-code/overview), [awesome-claude-code on GitHub](https://github.com/hesreallyhim/awesome-claude-code), [community commands collection](https://github.com/wshobson/commands)*