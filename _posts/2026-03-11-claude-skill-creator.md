---
title: "Building Claude Code Skills the Right Way with the Skill Creator"
date: 2026-03-11 10:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [claude-code, skills, developer-tools, automation, ai-engineering]
---

If you've been using Claude Code skills for a while, you probably went through the same phase I did: writing a `SKILL.md` by hand, eyeballing a few outputs, thinking "yeah this looks fine," and moving on. It works okay. But there's a better way, and once you try it you won't go back.

The **skill-creator** is a skill for building skills. It gives you a real engineering workflow — draft, test, benchmark, iterate — instead of just vibing and hoping for consistent results.

---

## Installing the Skill Creator

The skill-creator is an example skill from Anthropic. How you install it depends on where you're working.

### On Claude.ai

Go to **Customize > Skills**, scroll through the example skills, and toggle on **skill-creator**. That's it. No files, no terminal.

If you want to install it from a `.skill` file (e.g., a packaged version someone shared), click the **"+"** button and choose **"Upload a skill"**. Upload the ZIP file and it will appear in your skills list.

### On Claude Code

The cleanest way is through the plugin marketplace. In any Claude Code session, run:

```
/plugin marketplace add anthropics/skills
```

Then browse and install the `example-skills` plugin, which includes the skill-creator.

Alternatively, you can install it manually by dropping it directly into Claude Code's skills directory:

```bash
# For global install (available in all your projects)
mkdir -p ~/.claude/skills/skill-creator
cp -r /path/to/skill-creator/* ~/.claude/skills/skill-creator/

# For project-scoped install (committed to a specific repo)
mkdir -p .claude/skills/skill-creator
cp -r /path/to/skill-creator/* .claude/skills/skill-creator/
```

Claude Code picks up skills from `~/.claude/skills/` (global) and `.claude/skills/` (project-level). Project-level skills take priority if there's a conflict.

### Verifying it's active

In Claude Code, run `claude skills list` or just ask Claude directly: *"What skills do you have available?"* You should see skill-creator in the list. The skill only activates when you're clearly asking to build or improve a skill — it won't interfere with regular coding sessions.

---

## Using the Skill Creator

Once installed, you don't invoke it with a slash command — you just describe what you want to build and the skill-creator kicks in automatically:

```
I want to create a skill that helps me write better git commit messages 
following the Conventional Commits format.
```

Or if you want to improve an existing skill:

```
I have a skill for SQL formatting but it's inconsistent on edge cases. 
Can you help me improve it with the skill-creator?
```

From there, Claude walks you through the full workflow.

---

## The Workflow, Step by Step

### 1. Intent Capture

Before writing anything, the skill-creator interviews you. Not just "what should this skill do" — it digs into: what would you actually type to trigger this? What format should the output be? Are there edge cases that need special handling?

The last question it always asks is whether to set up automated test cases. For skills with well-defined outputs — commit messages, SQL formatting, PR checklists — assertions make sense. For something like "write in my personal writing style," you can't assert "the style is correct" programmatically, so qualitative review is better. The skill-creator makes this call based on what you describe.

Don't rush through this part. Time spent here saves iterations later.

### 2. Draft and Review

After the interview, Claude generates the full `SKILL.md`. Read it carefully, and pay special attention to the **description field** in the frontmatter — it's the only thing Claude reads when deciding whether to load the skill. If the description doesn't match how you'd naturally phrase requests, the skill will fail to trigger even if the instructions are perfect.

This is actually the most common silent failure mode: you write great instructions, but the description is phrased in a way you'd never type, and Claude just never uses the skill. We'll fix this properly in the optimization step, but catching it early is better.

### 3. Test Cases and Assertions

The skill-creator proposes realistic test prompts — the kind of thing you'd actually type, not a perfectly crafted demo. For our commit message skill:

```
1. "I added caching to the user preferences endpoint to reduce DB load"
2. "Fixed a crash when the config file doesn't exist at startup"
3. "I was refactoring the auth module and accidentally fixed a small bug along the way"
```

That third one is intentionally ambiguous — two things happening at once, unclear which type should win. Good test cases include these edge cases, not just the happy path.

Alongside the prompts, the skill-creator writes **assertions**: specific things the output should satisfy that can be checked mechanically. For commit messages: does the output start with a valid type (`feat:`, `fix:`, `refactor:`)? Is the subject line under 72 characters? Is the verb imperative ("add" not "added")? These become your quantitative benchmark.

### 4. Running the Evaluation

Here's the part that changes everything. The skill-creator runs your test prompts against two versions in parallel:

- **With skill** — Claude has your skill loaded
- **Without skill** (baseline) — same prompt, no skill

The baseline comparison is important. If the outputs look basically the same, your skill isn't adding anything. You want meaningful difference, in the direction of your intent.

Results land in a structured workspace, and then the skill-creator generates a viewer — an HTML interface where you go through outputs side by side, see assertion pass rates, and leave qualitative notes. Looking at outputs side by side is not the same as looking at them one at a time. The comparison surfaces differences you'd completely miss otherwise.

One thing to watch: assertions that pass at 100% on the baseline run. Those aren't testing anything the skill actually contributes — they're just things Claude does by default. Either tighten the assertion or drop it.

### 5. Iterate

You review the outputs, tell the skill-creator what you observed, and it updates the skill and reruns. This loop typically converges in 2-3 rounds. The first iteration always finds something — that's not a sign the skill is bad, it's the evaluation working correctly.

### 6. Description Optimization (Claude Code only)

This step requires the `claude` CLI tool, so it's only available in Claude Code, not Claude.ai. But if you're in Claude Code, it's the most valuable step.

An automated loop tests multiple description variants against a set of trigger queries, iterates up to 5 times, and returns the version with the best trigger rate — evaluated on a held-out test split (60/40) to avoid overfitting.

```bash
python -m scripts.run_loop \
  --eval-set path/to/trigger-eval.json \
  --skill-path path/to/my-skill \
  --model claude-sonnet-4-6 \
  --max-iterations 5
```

The before/after difference can be significant. My instinct was to write descriptions like "Generates commit messages following Conventional Commits format." The optimizer rewrites it to something like: "Use whenever the user wants to write or improve a commit message, mentions what they changed, or pastes a diff and asks what to call it." That phrasing matches how humans actually talk. Trigger rate triples.

### 7. Packaging

```bash
python -m scripts.package_skill path/to/my-skill/
```

You get a `.skill` file — self-contained with any bundled scripts, reference files, or assets. Ready to install, share, or commit to version control.

---

## Things That Will Trip You Up

**Test prompts that are too simple.** A prompt like "write a commit message for adding a file" is so easy that Claude handles it fine without any skill. So skill and baseline look the same, and you've proven nothing. Make your test cases complex enough that structured instructions actually change the output.

**Subjective assertions.** "The output is clear and professional" can't be verified mechanically — it'll produce noise, not signal. Stick to things you can check: format patterns, length limits, presence of specific strings, structural requirements.

**Skill scope too broad.** A skill that tries to do code review *and* write commit messages *and* generate changelogs is a mess to write and test. Split it. Multiple focused skills active at the same time is better than one sprawling one.

---

## One More Use Case: Improving Existing Skills

If you already have a skill that works most of the time but has a rough edge, the skill-creator handles this too. Tell it you want to improve an existing skill, and it snapshots your current version as the baseline. The comparison becomes old version vs. new version instead of skill vs. no-skill — which is exactly the right comparison for an improvement session.

I've used this on two skills that felt "fine" for months. Running them through the evaluation loop revealed they were producing nearly identical output to the baseline on edge cases. The skills were adding almost nothing on those inputs, and I had no idea. That's the honest case for having proper evals: you find things you weren't looking for.

---

## Final Thought

The skill-creator exists because building good skills is a small software engineering problem: write, test, measure, iterate. Those steps exist whether you do them informally (eyeballing outputs and hoping) or formally (evals, benchmarks, optimization loop). The skill-creator makes the formal path easy enough that there's no good excuse to skip it.

Pick one of your existing skills, run it through the evaluation loop, and see what you find. I'd bet you find at least one thing you didn't expect.

---

*Links: [Claude Code docs](https://docs.anthropic.com/en/docs/claude-code/overview) · [Anthropic skills repo](https://github.com/anthropics/skills) · [Conventional Commits](https://www.conventionalcommits.org)*