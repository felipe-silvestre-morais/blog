---
title: "Beyond the Prompt: Parallel Coding Agents with Git Worktrees, and the Engineering Skills That Matter More Now"
date: 2026-11-21 18:30:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [git-worktrees, claude-code, codex, ai-agents, parallel-agents, testing, productivity, software-engineering, developer-workflow]
---

There is a pattern I keep noticing among engineers who adopted AI coding tools over the last year. They got very good at one thing: **writing a prompt and waiting for Claude Code or Codex to finish**. And they stopped there.

That is a perfectly good way to start. But it is also a ceiling. The engineers I see pulling ahead are not the ones with the cleverest prompts. They are the ones who treat the agent as **one tool in a much bigger toolbox**: they run several agents at once without them trampling each other, they let tests and linters steer the agent instead of their own eyeballs, they reach for a debugger when the agent is stuck in a loop, and they know enough Git to untangle things when it goes sideways.

This post is for the engineer who has been living in the chat box. It has two halves:

1. **Parallel agents with git worktrees.** What they are, what Claude Code and Codex actually do with them, and the problems worktrees do *not* solve (this is where most people get burned).
2. **Ten skills and techniques beyond "ask the agent"** that make you more effective in this new era, most of which are older than AI and still underrated.

Everything about the tools below comes from their official docs where I could read them, and where I could test a command locally I did. I will say which is which.

---

# Part 1: Parallel Agents with Git Worktrees

## Why Parallelism, and Why It Is Not Free

A coding agent spends most of its time *waiting*: on the model, on your tests, on a build. While it does, you are idle too, unless you have something else running. The obvious productivity move is to start a second task, then a third.

The obvious problem: **two agents editing the same folder will destroy each other's work**. One checks out a branch while the other is mid-edit. One runs the formatter across files the other is changing. Files appear and vanish underneath them.

The classic fix is to give each agent its own copy of the repository. You could `git clone` the repo three times, but then you have three separate histories to keep in sync. There is a built-in Git feature designed for exactly this.

## Git Worktrees in Five Minutes

A **worktree** is an additional working directory attached to the same repository. Think of one repository (one `.git` database, one history) with several folders you can work in at the same time, each with its own checked-out branch.

```bash
# from your main checkout
git worktree add ../myapp-fix-login -b fix-login   # new folder, new branch
git worktree add ../myapp-search    -b add-search

git worktree list
# /code/myapp               f842753 [main]
# /code/myapp-fix-login     f842753 [fix-login]
# /code/myapp-search        f842753 [add-search]
```

Each folder is a full checkout. Commit in one and it shows up in the shared history immediately, so there is nothing to sync. When you are done:

```bash
git worktree remove ../myapp-fix-login
```

One rule worth knowing, because it is a feature, not a bug: **a branch can only be checked out in one worktree at a time.** If you try to add a second worktree on a branch that is already checked out somewhere, Git refuses with a "already used by worktree" error. I tried it in a scratch repo to be sure. This is exactly the guardrail you want when several agents are running.

## What Claude Code Does With Worktrees

Claude Code has first-class support. From its [official worktree docs](https://code.claude.com/docs/en/worktrees.md):

```bash
claude --worktree feature-auth      # or: claude -w feature-auth
```

This creates a worktree under `.claude/worktrees/feature-auth/` on a new branch called `worktree-feature-auth`, and starts the session inside it. Run it in a second terminal with a different name and you have two isolated sessions. A few details that are worth knowing:

- **Add `.claude/worktrees/` to your `.gitignore`.** The docs recommend it, otherwise the worktree folders show up as untracked noise.
- **Fresh worktrees have no untracked files.** That means no `.env`, no `node_modules`. A `.worktreeinclude` file at the repo root (gitignore syntax) tells Claude which *ignored* files to copy into new worktrees, for example `.env` and `.env.local`. You still run your install step yourself.
- **Start from a pull request.** `claude --worktree "#1234"` (or a PR URL) creates a worktree for that PR, which is handy for reviewing or fixing someone else's branch without touching yours.
- **Base branch.** The `worktree.baseRef` setting controls whether new worktrees branch from the remote default branch (`"fresh"`, the default) or from your current local HEAD (`"head"`).
- **Cleanup.** When you exit, an unnamed worktree with no changes is removed automatically along with its branch. If there is work in it, Claude asks whether to keep or remove it. Non-interactive `-p` runs are not cleaned up, so remove those yourself with `git worktree remove`.
- **Subagents can be isolated too.** In a custom subagent definition under `.claude/agents/`, add `isolation: worktree` to the frontmatter and each run gets its own temporary worktree:

```markdown
---
name: refactorer
description: Performs large mechanical refactors in isolation
isolation: worktree
---
Refactor the code as requested. Run the tests before you finish.
```

A few caveats from the same docs that will save you a confusing afternoon. Worktrees still **share the `.git` directory** with the main checkout. "Don't ask again" permission approvals are saved to the *main* checkout's `.claude/settings.local.json`. And a subagent running in a worktree uses the main conversation's instruction files rather than loading the worktree's own `CLAUDE.md`. If you rely on per-branch instructions, test that it behaves the way you expect.

## What Codex Does With Worktrees

The Codex app manages worktrees *for* you, and the model is a little different. From the [official docs](https://learn.chatgpt.com/docs/environments/git-worktrees):

- You pick **Worktree** under the composer, choose a starting branch, and send your prompt. Codex starts from that branch's HEAD.
- Managed worktrees live under `$CODEX_HOME/worktrees` and use a **detached HEAD by default**, so no branch is created until you want one. When you do, "Create branch here" in the chat header turns the worktree into a branch.
- It keeps the **15 most recent** managed worktrees by default (configurable in settings), skipping any attached to pinned or in-progress chats. A snapshot is saved before deletion so reopening an old chat can restore it.
- A **Hand off** button moves a chat and its code between Local and a worktree, with Codex doing the Git operations.
- It has the same `.worktreeinclude` idea for copying ignored files like `.env`, but this applies only to worktrees the app manages locally, not ones you create yourself with `git worktree add`.
- Scheduled tasks (automations) in Git repositories can run on dedicated background worktrees so they do not collide with what you are doing.

The practical summary: Claude Code gives you **named branches in a known folder** that you control from the terminal; Codex gives you **disposable, app-managed workspaces** that you promote to a branch when the work is good. Both rest on the same Git feature, and both are worth understanding underneath, because the interesting problems begin where the tools stop.

---

## What Worktrees Do *Not* Isolate

This is the part most tutorials skip. A worktree isolates **files and branches**. It does not isolate anything that *runs*.

Three agents in three worktrees will still happily collide on:

| Shared thing | What goes wrong |
|---|---|
| **Ports** | Every dev server wants `3000`. The second one fails to start, or worse, the agent tests against *another agent's* server and thinks its code works. |
| **Databases** | Three agents running migrations against the same local Postgres will break each other's schema mid-run. |
| **Environment files** | A fresh worktree has no `.env` unless you copy it. The agent then "fixes" the missing config in creative ways. |
| **Dependencies and caches** | No `node_modules` or virtualenv. Shared build caches and lockfiles can also be mutated by two agents at once. |
| **Containers and background services** | `docker compose` project names, Redis instances and message queues are global unless you make them local. |

My answer is a small script that makes "new isolated workspace" a single command. Here is a version I tested in a scratch repo (the install line is commented out; put your own there):

```bash
#!/usr/bin/env bash
# usage: wt-new <name> [base-branch]
set -euo pipefail
name="$1"; base="${2:-main}"
root="$(git rev-parse --show-toplevel)"
dir="$(dirname "$root")/$(basename "$root")-$name"

git worktree add -b "$name" "$dir" "$base"

# copy the secrets/config the worktree won't have
[ -f "$root/.env" ] && cp "$root/.env" "$dir/.env"

# stable, per-worktree port and database name derived from the branch name
port=$(( 3000 + $(printf '%s' "$name" | cksum | cut -d' ' -f1) % 1000 ))
db="app_${name//[^a-zA-Z0-9]/_}"
printf 'PORT=%s\nDATABASE_URL=postgres://localhost/%s\n' "$port" "$db" >> "$dir/.env"

# (cd "$dir" && npm ci --silent)   # install dependencies here
echo "worktree: $dir  port: $port  db: $db"
```

Run it twice and each worktree gets its own folder, its own branch, a copy of your `.env`, a deterministic port, and its own database name. Two things to notice:

- **A hash-based port can collide** (it is a number from 3000 to 3999, and two names can land on the same one). It is fine for a handful of worktrees. If you run many, keep a small registry or let the OS pick a free port.
- **You still have to create the database** (or use SQLite or a throwaway container per worktree). The script only gives each one a *name*.

If your stack is heavier, the same idea scales up: a `docker compose -p <name>` project per worktree, and a seed step that creates the isolated database. The principle is the one to remember: **isolation has to be designed at every layer, not just the file system.**

---

## Merge Conflicts and the Review Bottleneck

Parallel work does not remove conflicts. It **postpones them to merge time**, and there are two kinds.

**Textual conflicts** are the ones Git catches. You can check for them *before* you merge, without touching any working tree, using `git merge-tree` (it needs Git 2.38 or newer):

```bash
# does feat-a conflict with feat-b? exit code 1 means yes, 0 means no
git merge-tree --write-tree feat-a feat-b
echo $?
```

I ran this in a scratch repo with two branches editing the same line: it printed the conflicted file and exited with `1`, and exited with `0` for a clean merge. It makes a nice pre-flight check before you start combining branches.

**Semantic conflicts** are the ones Git cannot see. Two branches can merge cleanly and still be broken together, because agent A renamed a function's meaning while agent B started calling it the old way. The fix is boring and effective: **merge one branch at a time, run the full test suite after each**, and never batch-merge five agent branches and hope.

The real constraint is not Git, though. It is **you**. Each agent produces something you must review, and your attention is the scarce resource. Some rules that have helped:

- **Limit work in progress.** Two or three parallel tasks is where most people stay productive. Beyond that you are context-switching between reviews, not accelerating.
- **Split by area, not by whim.** Tasks that touch different modules parallelize well. Tasks that touch the same files will conflict, so run those one after the other.
- **Demand evidence.** Ask each agent to finish with what it ran, what passed, and what it could *not* verify. Review that first.
- **Keep changes small.** A 3,000-line agent PR is not reviewable. Ask for smaller, independently mergeable steps.

---

# Part 2: Ten Skills Beyond "Ask the Agent"

Parallel agents multiply whatever your process already is. If your process is "prompt and pray", you will get more of that faster. The techniques below are what make the output trustworthy, and most of them do not involve an AI at all.

## 1. Make Tests the Steering Wheel

The single biggest upgrade is to stop judging the agent's work by reading it and start judging it by **tests you trust**. A few levels, in increasing strength:

- **Write the failing test first**, run it, see it fail for the right reason, *then* let the agent make it pass. This stops the agent from writing a test that was shaped to its own code.
- **Property-based tests** (Hypothesis for Python, fast-check for JavaScript) check rules like "sorting twice equals sorting once" over hundreds of generated inputs. Agents are good at writing examples and bad at imagining the weird input, and this covers exactly that gap.
- **Mutation testing** (mutmut for Python, Stryker for JavaScript and others) deliberately breaks your code in small ways and checks that your tests notice. If the agent's tests still pass when the logic is mutated, they are decorative. This is the best defence I know against green tests that prove nothing.

## 2. Make the Repo Easy for an Agent to Verify

Agents do far better in codebases where "is this right?" has a fast, automated answer. That means strict types, a formatter and a linter that run in seconds, a one-command test target, and pre-commit hooks. Each of those is a feedback loop the agent can run on itself.

If a check takes ten minutes, the agent will skip it or you will. If it takes ten seconds, it becomes part of every iteration. Investing in a faster test suite is now a direct multiplier on agent quality. I wrote about the instruction side of this in [AI agent metadata in codebases]({% post_url 2026-05-21-ai-agent-metadata-in-codebases %}); the short version is a small, accurate `CLAUDE.md` or `AGENTS.md` that tells the agent the exact commands to build, test and lint.

## 3. Use Deterministic Guardrails, Not Polite Requests

"Please always run the formatter" in a prompt is a request. A **hook** is a rule. Claude Code can run a command automatically after the agent edits a file, for example:

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npm run -s lint:fix" }
        ]
      }
    ]
  }
}
```

The same principle applies one level up: CI, branch protection and required reviews are guardrails that do not depend on anyone remembering. Add **static analysis** (Semgrep, CodeQL, your language's own tooling) and **dependency scanning** to that stack, because agents will happily add a package you have never heard of. Put your limits in the plumbing, not in the prompt.

## 4. Plan First, Then Execute

Most bad agent output starts with an ambiguous task. Ask for a **plan** before any code: which files, which approach, what could go wrong. Read the plan, correct it, *then* let it execute. A two-minute correction at the plan stage saves a twenty-minute rewrite after. This is the core of what I described in [spec-driven development]({% post_url 2026-03-09-spec-driven-dev %}), and it pairs well with parallel agents: write three small, clear specs, then launch three worktrees.

## 5. Learn the Debugging Tools the Agent Cannot Hold For You

When an agent gets stuck, it usually loops: guess, run, fail, guess again. You can break that loop with tools that give **facts instead of guesses**.

- **A real debugger** (breakpoints, inspecting live state) beats adding `print` statements for most hard bugs.
- **A profiler** tells you where time actually goes, so you do not optimize the wrong function.
- **`EXPLAIN ANALYZE`** on a slow query tells you what the database is doing, which no amount of rewording the prompt will.
- **`git bisect`** finds the commit that broke something by binary search, and it can be automated:

```bash
git bisect start
git bisect bad                 # current commit is broken
git bisect good v1.4.0         # this older tag worked
git bisect run npm test        # let Git find the first bad commit for you
```

Paste the results of these tools into the agent's context. Handing it the profiler output or the bisect result turns a vague "it is slow" into a precise task.

## 6. Get Fluent in the Git Features Agents Make You Need

Agents generate many commits and many branches, so Git skills now pay off more than before:

- `git add -p` to stage only the hunks you want from a messy working tree.
- `git rebase -i` with `--autosquash` to turn an agent's twelve "fix" commits into two meaningful ones.
- `git reflog` to recover work after a bad reset. The reflog remembers where HEAD was, so "I lost my changes" is rarely true.
- `git range-diff` to see what changed between two versions of a rebased branch.
- `git rerere` to remember how you resolved a conflict so the next identical one resolves itself, which is handy when you merge many parallel branches.

## 7. Treat the Terminal as an Instrument

Parallel agents live in terminals, so terminal fluency is leverage. A multiplexer like **tmux** or **Zellij** lets you keep a session per worktree and detach and reattach without losing anything. **`ripgrep`** and **`fd`** find things across a large repo in milliseconds, **`fzf`** gives you fuzzy search over anything, and **`jq`** lets you slice JSON from logs and APIs. When you can answer "where is this used?" in two seconds yourself, you stop paying for an agent run to find out.

## 8. Use a Second Opinion on Purpose

One of the best habits is to have **a different model or a fresh session review the work**, with a prompt like "find bugs and missing tests in this diff; do not praise it". A reviewer with no memory of writing the code has no sunk-cost bias. Use it as a filter before *you* spend attention, not as a replacement for your attention. With Claude Code you can even open the PR in its own worktree (`claude --worktree "#1234"`) so your own checkout stays untouched.

## 9. Contain the Blast Radius

The more autonomy you give agents, the more the surroundings matter. Run them in a **dev container or sandbox**, give them least-privilege credentials, never put production secrets in their environment, and keep the permission allow-lists tight. I wrote about the reasoning in [building safer agents]({% post_url 2026-03-15-build-safer-agents %}) and about the credentials problem in [the MCP tunnel post]({% post_url 2026-05-26-mcp-tunnel-security %}). Parallelism makes this more important, not less: three unattended agents have three times the chance of one doing something you did not expect.

## 10. Keep Your Fundamentals Sharp

This is the unglamorous one, and the most important. The agent writes the code, but *you* decide whether the design is right, whether the abstraction will hurt in six months, whether that O(n²) loop will matter at scale, and whether the data model can survive the next feature. Those judgments come from understanding systems, not from prompting. I made the longer case in [software fundamentals matter more than ever]({% post_url 2026-05-21-software-fundamentals-matter-more-than-ever %}) and in [from vibe coding to agentic engineering]({% post_url 2026-05-14-from-vibe-coding-to-agentic-engineering %}). If you only have time to sharpen one thing, read code, lots of it, including the code your agents write.

---

# Putting It Together: A Workflow You Can Try This Week

Here is a concrete loop that combines the two halves:

1. **Pick two or three tasks in different areas** of the codebase. Write each as a short spec with an acceptance test.
2. **Create a worktree per task** with your setup script, so each has its own branch, port, database and environment.
3. **Launch one agent per worktree** (in tmux panes if you like). Tell each the exact commands to build and test.
4. **While they run, review the previous result**, not the one in progress. Your attention is the bottleneck, so keep it busy with the next review rather than watching.
5. **Gate each result**: tests green, linters green, mutation or property tests where it matters, and a second-opinion review of the diff.
6. **Merge one branch at a time**, pre-flighting with `git merge-tree`, running the full suite after each.
7. **Remove the worktrees** and delete the branches when you are done, so you do not accumulate clutter.

---

# The Honest Caveats

- **Parallel agents are not always faster.** If your tasks overlap, your tests are slow, or your reviews pile up, parallelism adds overhead without adding throughput. Start with two.
- **Tooling details move quickly.** I read the Claude Code and Codex worktree pages for this post, but flags, defaults and limits change often. Check the current docs before depending on a specific behaviour.
- **Review is still the hard part.** A pipeline that produces more code than you can responsibly read is a liability, not a productivity gain. Sizing the work to your review capacity matters more than any tool.
- **Some of the advice above is opinion.** The WIP limit of two or three, the suggestion to use hooks, and the specific tools I named come from experience and conventional practice, not from controlled studies.

---

# Final Thoughts

The surprising thing about this era is how much of the advantage comes from **old, boring engineering practice** meeting a new kind of worker. Branches, tests, types, linters, debuggers, version control, review: these were always how good teams stayed sane. Agents just raise the stakes, because a fast, tireless colleague who occasionally misunderstands you needs *more* structure around it, not less.

If you only ever type prompts into one agent, you are using maybe a quarter of what is available. Learn worktrees so you can run several in parallel. Learn tests, hooks and static analysis so you can trust what comes back. Learn the debugger, the profiler and Git well enough to take over when the agent is stuck. That combination is what turns the tools from a clever autocomplete into real leverage.

Pick one thing from this post and try it this week. I would start with `git worktree add` and a second agent on a small, separate task. It takes ten minutes, and it changes how you think about the work.

---

*Sources and further reading: the official [Claude Code worktrees documentation](https://code.claude.com/docs/en/worktrees.md) and [Codex worktrees documentation](https://learn.chatgpt.com/docs/environments/git-worktrees), plus the [Git worktree reference](https://git-scm.com/docs/git-worktree) and the [`git merge-tree` reference](https://git-scm.com/docs/git-merge-tree). For the parallel-agent pitfalls (ports, databases, semantic conflicts), see community write-ups like [Upsun's guide to worktrees for parallel AI agents](https://developer.upsun.com/posts/ai/git-worktrees-for-parallel-ai-coding-agents).*
