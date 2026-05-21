---
title: "The Hidden Layer in Your Repo: A Tour of AI Agent Metadata"
date: 2026-05-21 14:30:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [claude-code, ai-coding, agents, mcp, skills, metadata, repo-structure]
---

I cloned a repo last week from a team I hadn't worked with before. Standard stuff at first glance — `src/`, `tests/`, a `README.md`, a `package.json`. But then I noticed the other files. A `CLAUDE.md` at the root. An `AGENTS.md` next to it. A `.claude/` directory with skills, commands, and a `settings.json` inside. A `.mcp.json`. A `.cursorrules` file someone hadn't deleted. A `MEMORY.md` index pointing to a folder of small markdown files.

None of this stuff existed two years ago. Now it's everywhere. And nobody really sits down and explains what all of it *is*, or how the pieces fit together. You just keep bumping into new files in new repos and quietly pattern-matching what they probably do.

So that's what this post is. A tour of the AI agent metadata layer that's quietly grown into modern codebases — what each piece is for, how they compose, and what I've learned about using them well.

---

## Why There's a Second Layer Now

Before we go through the files, it helps to name the thing they're solving.

Coding agents like Claude Code, Cursor, Copilot, and Cline are useful in inverse proportion to how much you have to re-explain every time you start a session. The first time you ask an agent "use Pydantic v2, not v1" or "don't run the database migration locally" is fine. The fiftieth time, you start wondering why this thing keeps forgetting.

The answer the ecosystem has converged on: **stop telling the agent and start telling the repo**. Drop instructions into files that the agent reads automatically. Encode preferences, conventions, workflows, and capabilities in plain markdown that lives alongside the code.

That's the whole game. Everything that follows is a variation on it — different files solving different slices of the same "give the agent context without me typing it again" problem.

---

## Project Instructions: CLAUDE.md, AGENTS.md, and Friends

This is the file you'll see first in most repos. It tells the agent how *this specific project* works.

```markdown
# My Project

This is a Django 5 app with a React frontend in /web.

## Conventions
- Python: type hints required, ruff for linting
- React: functional components only, no class components
- Tests: pytest for backend, vitest for frontend
- Never commit to main directly; always open a PR

## Common commands
- `make dev` — start backend + frontend
- `make test` — run all tests
- `make migrate` — apply Django migrations
```

The big four right now:

| File | Tool | Notes |
|------|------|-------|
| `CLAUDE.md` | Claude Code | Loaded automatically. Can live at repo root, in subdirectories, or in `~/.claude/` for global rules. |
| `AGENTS.md` | OpenAI Codex, Cursor (newer), several others | Convergent standard pushed by OpenAI. Many tools now read it. |
| `.cursorrules` | Cursor (legacy) | Older Cursor format. Being replaced by `.cursor/rules/*.mdc` and `AGENTS.md`. |
| `.github/copilot-instructions.md` | GitHub Copilot | Repository-level instructions for Copilot Chat. |

Here's the messy part: **none of these tools read all of these files**. Claude Code reads `CLAUDE.md`. Copilot reads its own file. Cursor reads its own. There's no universal standard yet, even though `AGENTS.md` is getting close.

What I do in shared repos: keep one canonical file (usually `AGENTS.md` or `CLAUDE.md`) with the real content, and have the others be one-line stubs pointing to it. Something like:

```markdown
<!-- .github/copilot-instructions.md -->
See [AGENTS.md](../AGENTS.md) for project conventions.
```

It's ugly but it works, and you don't have three drifting copies of the same rules.

**What goes in these files**: project structure, conventions, common commands, things to avoid, domain vocabulary. **What doesn't**: secrets, anything that changes constantly (current task state), or implementation details that the code itself already shows.

---

## Skills: Packaged Capabilities the Agent Can Use

This is the newer one, and the one I think is most underrated.

A **skill** is a folder with a `SKILL.md` inside it. The SKILL.md has frontmatter describing when the skill should be used, and a body explaining the steps. The agent decides to invoke a skill when its description matches what the user is asking for.

Here's a skill from this very blog:

```markdown
---
name: new-blog-post
description: Create a new blog post for fsilvestreai.github.io.
  Use when the user wants to write a new blog post, article, or
  technical content for their blog.
---

# New Blog Post Creator

You are creating a new blog post for Felipe Silvestre's technical blog...

## Voice & Personality
[detailed instructions follow]
```

When I type `/new-blog-post` (or just ask Claude to "write a new blog post"), Claude reads that SKILL.md and follows it. The skill is the difference between "Claude writes a generic blog post" and "Claude writes a post in my voice, with the right frontmatter, saved to the right folder, with the right tags."

Skills typically live in:
- `.claude/skills/<skill-name>/SKILL.md` — project-specific
- `~/.claude/skills/<skill-name>/SKILL.md` — your personal skills, available everywhere

Skills can also include supporting files in the folder — scripts, templates, examples — that the SKILL.md references. It's basically a tiny package, but written for an agent to consume instead of a human.

I treat skills as **reusable workflows I'd otherwise have to re-explain every time**. "Review this PR." "Verify this change actually works in the browser." "Create a new database migration following our pattern." Each of those is a skill in one of my repos, because each one used to be a long prompt I'd half-remember and have to rewrite.

---

## Custom Commands: Shortcuts for Repeated Prompts

I wrote a [whole post](/posts/claude-code-custom-commands/) on these, so I'll keep it brief.

Custom commands live in `.claude/commands/<name>.md` and are invoked with `/name`. They're simpler than skills — just a markdown file containing a prompt that gets injected when you type the command.

```markdown
<!-- .claude/commands/ship.md -->
Run the test suite, fix any failures, then create a PR with a
clear title and description summarizing the changes on this branch.
```

Type `/ship` and you don't have to remember any of that — the command does.

**Skills vs commands** is the question I get most often. My rule of thumb:
- **Command** when it's a fixed prompt I run often.
- **Skill** when there are decisions to make, steps to follow, supporting files to read, or when I want the agent to invoke it *automatically* based on context.

Commands are basically "save this prompt." Skills are closer to "package this capability."

---

## Subagents: Delegated Personalities

Subagents live in `.claude/agents/<name>.md` (or globally in `~/.claude/agents/`). Each one defines a specialized agent that the main Claude can delegate to.

```markdown
---
name: code-reviewer
description: Reviews pull requests for correctness, style, and
  security issues. Use proactively after non-trivial changes.
tools: Read, Grep, Glob, Bash
---

You are a thorough code reviewer. Focus on:
- Correctness and edge cases
- Security (especially OWASP top 10)
- Adherence to project conventions in CLAUDE.md
- Readability and maintainability
[...]
```

The main agent decides "this is a code review task" and spawns the subagent with its own fresh context window, its own restricted toolset, and its own system prompt. The subagent runs, returns a summary, and the main agent continues.

Why bother? Two reasons:

1. **Context isolation**. A long research task can return a 200-word summary instead of dumping 50 file reads into your main conversation. Your main context stays clean.
2. **Specialization**. A code-reviewer agent that only has read tools and a security-focused prompt will behave differently than your general-purpose agent. You can build a fleet of these.

The cost: each subagent starts cold. No memory of your conversation, no preloaded context. So the prompt to a subagent has to be self-contained, which is more work upfront.

I use them sparingly. Most tasks don't need delegation. But for "review this whole PR" or "search across the codebase for everywhere we touch X" — yes, every time.

---

## Memory: Persistent Across Sessions

Here's something newer that I'm still figuring out: agents are starting to ship with **persistent memory systems** that survive across sessions.

In Claude Code, this lives in a path like `~/.claude/projects/<encoded-project-path>/memory/`. It's a folder of small markdown files plus a `MEMORY.md` index. Each memory file has frontmatter:

```markdown
---
name: prefers-deep-modules
description: User prefers deep modules over many shallow ones
metadata:
  type: feedback
---

Refactor toward deep modules (Ousterhout style) rather than splitting
into many small ones.

**Why:** User has had bad experiences with sprawling shallow codebases
and finds them hard for both humans and AI to navigate.
**How to apply:** When AI-generated code creates many small files,
suggest consolidating into fewer, deeper modules with simple interfaces.
```

`MEMORY.md` is an index that's always loaded into the agent's context. The individual memory files get loaded when relevant.

The categories I've seen converge:
- **User memories**: who you are, what you do, how you think
- **Feedback memories**: rules you've corrected the agent on
- **Project memories**: ongoing initiatives, decisions, who's doing what
- **Reference memories**: pointers to external systems (Linear projects, Slack channels, dashboards)

This is different from `CLAUDE.md`. CLAUDE.md is *project context*, checked into git, shared with the team. Memory is *your personal accumulated knowledge*, lives in your home directory, follows you across projects.

I've been letting mine grow for about a month now and the difference is noticeable. The agent stops re-asking the same questions. It stops re-explaining the same things. It actually remembers I'm me.

---

## MCP Servers: External Tools Through a Standard Port

**MCP** (Model Context Protocol) is the standard for plugging external tools into an agent. Think of it like USB for AI — a single protocol that lets the agent talk to your filesystem, your database, your Linear account, your Gmail, whatever has an MCP server.

Configuration lives in a few places depending on the tool, but for Claude Code the pattern is `.mcp.json` at the repo root or in `settings.json`:

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "${GITHUB_TOKEN}"
      }
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres",
               "postgresql://localhost/mydb"]
    }
  }
}
```

Once that's wired up, the agent has new tools available. It can query your database directly, read GitHub issues, send Slack messages — whatever the server exposes.

I wrote about MCP [in more detail elsewhere](/posts/mcp-explained/), but for this post the key point is: **MCP server configs are metadata too**. The list of servers a repo declares is a list of capabilities the agent will have available when working in that repo.

Be deliberate. Don't add servers you don't need. Every MCP server is more tools the agent has to consider on every decision, and they all eat context.

---

## Settings, Permissions, and Hooks: The Guardrails

The `.claude/settings.json` (and `settings.local.json` for personal overrides not checked into git) is where the boring-but-important stuff lives:

```json
{
  "permissions": {
    "allow": [
      "Bash(npm test:*)",
      "Bash(git status)",
      "Bash(git diff:*)",
      "Read(./**)"
    ],
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(git push --force:*)"
    ]
  },
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          { "type": "command", "command": "npm run lint:fix" }
        ]
      }
    ]
  }
}
```

Three pieces here:

- **Permissions**: which tool calls happen without prompting you. This is the file you tweak when you're tired of approving `git status` for the 200th time.
- **Hooks**: shell commands the harness runs on specific events. Above, every time the agent edits a file, lint:fix runs automatically. Hooks are how you make the agent's behavior *automatic* — memory and instructions are advice, hooks are enforcement.
- **Env vars and config**: model overrides, default editor, telemetry settings.

Hooks specifically are underused. If you've ever asked an agent "remember to always run the formatter after editing" — that's a hook, not a memory. The memory is hopeful. The hook actually happens, every time, because the harness runs it.

---

## How It All Composes

Here's a mental model that helped me. Picture a Claude Code session starting up. Roughly, this is what gets loaded:

```
1. System prompt (Claude's built-in instructions)
2. ~/.claude/CLAUDE.md         ← your global rules
3. <repo>/CLAUDE.md             ← project rules
4. ~/.claude/memory/MEMORY.md   ← your memory index
5. MCP server tool definitions  ← from .mcp.json
6. Available skills (descriptions only, loaded on demand)
7. Available subagents (descriptions only)
8. Available custom commands
9. Settings: permissions, hooks
10. Your actual prompt
```

The full bodies of skills, subagents, and individual memories are *not* all loaded upfront — only their descriptions. The agent pulls in the full file when it decides one is relevant. That's why those `description:` fields in the frontmatter matter so much. Bad descriptions = the right skill never gets invoked.

When you understand this loading order, a lot of weird agent behavior makes sense. "Why does it keep doing X?" Probably something in a CLAUDE.md you forgot about. "Why won't it use this skill?" Probably the description is too vague. "Why does it keep asking permission for the same command?" Add it to settings allow-list.

---

## My Recommendations

A year into living with this stuff:

1. **Start with one `CLAUDE.md` (or `AGENTS.md`) at the repo root.** Write it like you'd brief a new hire on day one. Conventions, commands, gotchas. Keep it under 200 lines.

2. **Make a personal global CLAUDE.md or memory.** Stuff you'd say to *any* agent on *any* project. "I prefer terse responses." "Use type hints." "Never run destructive commands without asking." This is where the compounding really starts.

3. **Build skills for things you do more than three times.** The "rule of three" applies. The first time, just type the prompt. The third time, save it as a skill or command.

4. **Use hooks for things that should be automatic, not optional.** Linting, formatting, running tests on save. Don't put these in CLAUDE.md as instructions — put them in `settings.json` as hooks. Instructions get forgotten; hooks fire every time.

5. **Be ruthless about settings.json permissions.** If you've approved the same Bash command 10 times, add it to the allow-list. Your future self will thank you.

6. **Don't try to make every tool happy.** If your team uses Claude Code, write CLAUDE.md. If you have Copilot users too, write a stub in `.github/copilot-instructions.md` pointing to it. Trying to maintain four parallel rule files is how you end up with four drifting versions of the truth.

7. **Check this stuff into git, mostly.** CLAUDE.md, AGENTS.md, skills, custom commands, subagents, `.mcp.json` — all belong in git. `settings.local.json` and your personal memory do not.

---

## Final Thoughts

What's interesting to me is that this layer didn't get *designed* into existence. There was no spec, no committee. People started dropping markdown files into their repos to make their agents work better, and the conventions emerged. CLAUDE.md, AGENTS.md, `.claude/skills/`, `.mcp.json` — all bottom-up, all converged from real use.

I think we're in the early-Ruby-on-Rails era of this. The conventions are forming, the tools are still rough, and a lot of teams haven't even noticed there's a metadata layer to invest in yet. But the gap between teams that invest in this stuff and teams that don't is already big, and it's growing.

If your repo doesn't have a CLAUDE.md yet, that's the single highest-leverage change you can make this week. Half a day of writing one good CLAUDE.md will save you and your team thousands of "remember to..." messages over the next year.

The agent is only as smart as the context you give it. The context lives in these files. Treat them like first-class artifacts.

---

*Related posts: [Claude Code Custom Commands](/posts/claude-code-custom-commands/) · [Claude Code Hidden Commands](/posts/claude-code-hidden-commands/) · [From Vibe Coding to Agentic Engineering](/posts/from-vibe-coding-to-agentic-engineering/)*

*Useful references: [Claude Code documentation](https://docs.claude.com/en/docs/claude-code) · [AGENTS.md spec](https://agents.md/) · [Model Context Protocol](https://modelcontextprotocol.io/)*
