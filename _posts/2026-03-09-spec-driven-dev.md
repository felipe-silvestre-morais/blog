---
title: "Spec-Driven Development: How to Actually Get AI to Build What You Want"
date: 2026-03-09 13:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [ai-coding, spec-driven-development, llm, claude-code, cursor]
---

I have been doing a lot of AI-assisted coding lately. And if you are like me, at some point you hit a wall with vibe coding. You know what I mean — you start with a great idea, throw a prompt at Claude Code or Cursor, and it generates something. Then you tweak it. Then it breaks something else. Then you tweak again. Two hours later you have a pile of code that kind of works but you're not totally sure why. And you have zero idea how you'd explain it to someone else.

I went through this frustration and started looking for a better way. That's when I found spec-driven development, and it genuinely changed how I think about building with AI.

So let me explain what it is, why it matters, and how you can actually apply it — including the tools that make it practical today.

---

## The Problem With Vibe Coding at Scale

Vibe coding is fantastic. I want to say that clearly before anything else. For prototyping, for exploring ideas, for building quick scripts — it's magic. You describe what you want, the model writes it, you iterate. Done.

But here's the thing: the magic starts to break down once your project grows past a certain point.

When you give an AI model a prompt like "build a login page for my users," you get *a* solution. But there are maybe 30 different valid ways to implement a login page. Which session management approach? Which hashing algorithm? Where does the token go — cookie or localStorage? The model makes these decisions based on its training data, its mood (okay, not really, but it feels that way sometimes), and the subtle implications of your prompt. You might go back and forth three, four, five times before you get something that matches what you actually had in mind.

And the deeper problem: when something breaks later, you're debugging code whose decisions you never consciously made. That's a strange and kind of uncomfortable position to be in as an engineer.

This is the fundamental gap that **spec-driven development** closes.

---

## What Is Spec-Driven Development?

Spec-driven development is the idea that before the AI writes a single line of code, you define a **specification** — a structured document that describes what the system should do, how it should behave, and what constraints it needs to follow.

That specification then becomes the primary artifact. Everything else — the implementation, the tests, the documentation — flows from it.

It's not a new concept in software engineering. We've always had PRDs (Product Requirements Documents) and technical specs. But in the AI-assisted coding era, the spec takes on a different role: it's not just a document for humans to read. **It's the input you give to the AI agent instead of an imperative prompt.** You're not telling the model *how* to build something. You're describing *what* you want it to do, and letting the model figure out the how within those constraints.

The difference sounds small. The results are very different.

---

## How It Compares to What Came Before

Let me put it in context, because I think the evolution is actually really elegant.

**Traditional development** was code-first. You start with intuition, you write the implementation, and documentation (if it happens at all) comes at the end. This works fine when you are the one writing every line.

**Test-driven development (TDD)** was a step forward. You start by writing tests — defining what the correct behavior looks like — and then you write the code that makes those tests pass. This forces clarity about intent before implementation.

**Spec-driven development** goes even further. You start with a specification that describes the behavior, the constraints, the expected inputs and outputs, the failure cases. From that spec, you can generate requirements. From requirements, you generate a design document. And *then* the code gets written — by the AI — against that design. Tests get generated from the same spec too.

Think of it as TDD on steroids, but for the AI era. You're not writing the tests yourself. You're writing the *intent*, and the AI derives both the tests and the implementation from that.

```
Traditional:  intuition → code → (docs, eventually)
TDD:          tests → code → docs
Spec-Driven:  spec → requirements → design → code + tests (all AI-generated)
```

---

## The Workflow in Practice

Let me walk through a concrete example. Say you're building a user authentication feature.

**Vibe coding approach:** You tell the model "add a /login endpoint for user authentication." It generates something. Maybe it uses JWT, maybe sessions — you'll find out when you read the code. If it's not what you wanted, you start editing.

**Spec-driven approach:** You write a spec first. Something like this:

```markdown
# Feature: User Authentication

## Endpoint
POST /login

## Inputs
- `username` (string, required)
- `password` (string, required)

## Behavior
- Validate that both fields are present; return 400 if either is missing
- Look up user by username in the database
- Verify password against bcrypt hash
- On success: return 200 with a signed JWT (HS256, 24h expiry)
- On failure: return 401 with generic "invalid credentials" message
  (never distinguish between "user not found" vs "wrong password")

## Test Cases
- Valid credentials → 200 + JWT in response body
- Missing username → 400
- Missing password → 400
- Wrong password → 401
- Non-existent user → 401
```

Now you give this to the AI agent. The model doesn't have to guess about any of these decisions — they're all defined. You get exactly the behavior you described. The test cases are already written as part of the spec, so the agent can implement and verify in one pass.

Less ambiguity. More predictable output. And crucially — you now have a document that explains *why* the code is the way it is.

---

## The Tools That Make This Practical

This is the part I find most exciting, because the tooling for spec-driven development has gotten really good recently.

### Claude Code + CLAUDE.md

Claude Code is Anthropic's terminal-based coding agent, and it has a feature that is perfect for spec-driven workflows: the `CLAUDE.md` file. This is a markdown file you put at the root of your project, and Claude Code reads it at the start of every session. It's your project-level specification — architecture decisions, conventions, constraints, things the model should always follow.

```markdown
# CLAUDE.md

## Project: User Auth Service

### Architecture
- FastAPI + PostgreSQL
- JWT for authentication (HS256)
- Passwords hashed with bcrypt (12 rounds)

### Conventions
- All endpoints return `{ "data": ..., "error": null }` on success
- All errors return `{ "data": null, "error": { "code": ..., "message": ... } }`
- Database queries go through repository classes, never directly in route handlers
- Every new endpoint needs a corresponding test in /tests

### Do not
- Use sessions (JWT only)
- Store tokens in localStorage on the frontend
- Expose stack traces in error responses
```

Every time you start a new Claude Code session in that project, the model reads this file and understands your architectural constraints before doing anything. You stop having to repeat yourself.

### Cursor + .cursorrules / Project Rules

Cursor has a similar mechanism. You can define `.cursorrules` at the project root (or project rules in the newer versions) that work the same way — project-wide context that gets injected into every chat and inline edit session.

The difference is Cursor is more IDE-like, so you get the spec alongside your code in the editor. It works really well for iterating on feature specs within files before handing them off to the agent.

### Windsurf + Memories

Windsurf (from Codeium) has a concept called "memories" — persistent context that the AI builds up over your sessions. It's a slightly different angle on the same idea: the model accumulates understanding of your project conventions over time, so you don't have to re-specify them constantly.

### Kiro

This one I am particularly excited about. Kiro is a new IDE from AWS that is built spec-first from the ground up. Instead of treating the spec as just a markdown file you manage manually, Kiro has first-class spec artifacts in the IDE — you define specs in structured files, and the agent uses them explicitly as the source of truth for all code generation. It's the most direct implementation of spec-driven development I've seen in a real product.

It also auto-generates "hooks" — background tasks that keep your code in sync with your specs when things change. That's a genuinely new idea.

---

## What a Good Spec File Actually Looks Like

There's no single standard format yet, which is one area where the tooling still needs to mature. But based on what works well in practice, a good spec for a feature typically has these sections:

```markdown
# [Feature Name]

## Overview
One paragraph: what this does and why it exists.

## Behavior
- What the system does in the happy path
- What it does in error cases
- Any constraints (rate limits, size limits, etc.)

## Interface
- API contract (if applicable): endpoints, inputs, outputs, status codes
- Data model changes (if applicable): new tables, fields, migrations needed

## Test Cases
- Explicitly list the scenarios you want tested
- Include edge cases you know about upfront

## Out of Scope
- Things this spec explicitly does NOT cover
- (This prevents the model from going too wide)

## Open Questions
- Things you still need to decide
- (Better to name them explicitly than let the model decide for you)
```

The "Out of Scope" and "Open Questions" sections are the ones most people skip, and they're the ones that save you the most time in practice.

---

## The Spec as a Living Document

One pattern I've started using: I keep a `specs/` directory in my projects. Each feature gets its own file. When I want to add a feature, I write the spec file first — even before opening Claude Code or Cursor. It takes maybe 15 to 20 minutes. And then I pass it to the agent.

The spec file serves three purposes at once: it's the instructions for the AI, it's documentation for my future self, and it's communication to any teammates about what is being built and why.

```
project/
├── CLAUDE.md           ← project-level spec (conventions, arch decisions)
├── specs/
│   ├── auth.md         ← authentication feature spec
│   ├── payments.md     ← payments feature spec
│   └── notifications.md
├── src/
└── tests/
```

When the spec changes — and it will — you update the file and re-run the agent against it. The code stays in sync with the intent.

---

## Spec-Driven for Bug Fixes, Extensions, and Refactors

Most of the discussion around spec-driven development focuses on greenfield features, but honestly some of the biggest wins I've had are in the other scenarios — fixing bugs, extending existing behavior, and refactoring messy code. These are the situations where vibe coding really hurts you, because the model has no clear anchor for what "correct" looks like after the change.

### Bug Fixes

The instinctive way to fix a bug with AI is to paste the error and say "fix this." Sometimes that works. More often, the model patches the symptom without understanding the intended behavior, and you end up with a different bug three days later.

The spec-driven approach: before you touch any code, write down what the *correct* behavior should be. You're defining the expected outcome, not describing the bug itself.

Say you have a payment processing service and you discover that a coupon code is being applied twice if the user double-clicks the checkout button. A vibe coding prompt looks like this:

> "There's a bug where the coupon is applied twice. Fix it."

A spec-driven prompt looks like this:

```markdown
# Bug Fix: Coupon Double-Application on Checkout

## Observed behavior (the bug)
If the user clicks "Place Order" twice quickly, the coupon discount is applied 
twice to the total, resulting in a negative or incorrect order total.

## Expected behavior (the spec)
- A coupon code can be applied to an order exactly once
- Subsequent calls to `apply_coupon(order_id, code)` with the same order_id 
  must be idempotent — same result, no error, no double application
- The checkout button must be disabled immediately on first click and remain 
  disabled until the request completes (success or failure)

## Root cause hypothesis
The `apply_coupon` function does not check if a coupon has already been 
applied before inserting a new discount record.

## Fix constraints
- Do not change the coupon data model
- The idempotency check must happen at the service layer, not just the UI
- Add a database-level unique constraint on (order_id, coupon_code) as a safety net

## Test cases to add
- Calling apply_coupon twice with the same order_id + code → second call is a no-op
- Calling apply_coupon with two different codes on the same order → both applied (allowed)
- Rapid double-click simulation in the UI → only one request completes
```

The difference is huge. The model now knows what "correct" means, what layer to fix it at, what constraints to respect, and what tests to write. You're not just patching the symptom — you're defining the contract and making it enforced.

### Extending Features

Extending an existing feature is where spec-driven really shines, because you already have a spec for the original feature. You don't start from scratch — you *amend* it.

Let's say you built the `/login` endpoint from the earlier example. Now you need to add OAuth2 support with Google. The wrong way to do this is to just prompt "add Google login to the auth system." The model will make a dozen decisions you didn't think about: where does the callback go? How does the session merge with existing accounts? What happens if the Google email is already registered with a password?

Instead, you open `specs/auth.md` and add a new section:

```markdown
## Extension: Google OAuth2 Login

### New endpoints
- GET /auth/google → redirects to Google consent screen
- GET /auth/google/callback → handles the OAuth callback from Google

### Behavior
- After successful Google auth, check if a user with that email already exists:
  - If yes: log them in (create session/JWT for that existing user)
  - If no: create a new user with `provider = "google"`, no password field
- Users created via Google cannot use the password login endpoint
  - If they try: return 400 with message "This account uses Google login. 
    Please sign in with Google."
- Store `google_id` and `google_email` in the users table

### Data model changes
Add to `users` table:
- `provider` (string, default "local") — values: "local", "google"
- `google_id` (string, nullable, unique)

### Out of scope
- Linking existing password accounts to Google (future feature)
- Other OAuth providers (GitHub, Apple, etc.)

### Test cases
- New user signs in with Google → user record created with provider="google"
- Existing Google user signs in again → no duplicate record created
- Google user tries password login → 400 with correct error message
- New Google user's email already exists as local user → logged into existing account
```

Now you pass the *entire* `specs/auth.md` file to the agent — the original section plus the new extension — and say "implement the Google OAuth extension according to the spec." The agent has full context about the existing system and the new behavior. It knows exactly what to add, what to not touch, and what tests to write.

You also keep a history of *why* the system is the way it is. Three months from now when someone asks "why can't Google users reset their password?" — the answer is right there in the spec.

### Refactoring

Refactoring is probably the scariest thing to do with an AI agent if you're not careful. "Refactor this module" is one of the most dangerous prompts you can write, because the model might change behavior while reorganizing code and you won't immediately notice.

The spec-driven approach to refactoring is: **write the spec first, then refactor, then verify the spec still holds**.

Here's a real example. Say you have a notification service that grew organically and now it's a 600-line file mixing email sending, SMS, push notifications, database logging, and retry logic all together. You want to break it apart.

```markdown
# Refactor: Notification Service Decomposition

## Goal
Split `notification_service.py` into focused modules without changing 
any external behavior.

## Current state
Single file: `services/notification_service.py`
- Handles email (via SendGrid), SMS (via Twilio), push (via FCM)
- Logs all notifications to `notification_log` table
- Has retry logic (3 attempts, exponential backoff) mixed in

## Target structure
```
services/notifications/
├── __init__.py          ← public API (same interface as today)
├── email.py             ← SendGrid logic only
├── sms.py               ← Twilio logic only  
├── push.py              ← FCM logic only
├── logger.py            ← DB logging logic
└── retry.py             ← retry decorator, reusable by all channels
```

## Strict behavioral constraints (must not change)
- All public function signatures stay identical: 
  `send_email(to, subject, body)`, `send_sms(to, message)`, `send_push(user_id, title, body)`
- Retry behavior: 3 attempts, 2^n second backoff (2s, 4s, 8s)
- Every send attempt (success or failure) must still be logged to `notification_log`
- All existing tests in `tests/test_notifications.py` must pass without modification

## What this refactor does NOT include
- No new features
- No changes to retry count or backoff timing
- No changes to the database schema
- No changes to the public API
```

This is the key insight about spec-driven refactoring: **you're writing a spec for the behavior that must be preserved**, not for the new structure itself. The structure is up to the model to figure out. You're defining the invariants that must hold before and after.

When the model is done, you run the existing test suite. If everything passes, the refactor is correct by definition — because the tests were written against the spec you just formalized.

---

### A Quick Pattern Summary

The common thread across all three scenarios is the same: **before you touch code, define what correct looks like in writing**. The format changes slightly depending on the case:

- **Bug fix spec:** observed behavior + expected behavior + fix constraints + new test cases
- **Extension spec:** new behavior + data model changes + edge cases + what's out of scope  
- **Refactor spec:** target structure + behavioral invariants that must not change + explicit "do not change" list

Once you internalize this habit, you start to notice that most AI coding mistakes — wrong fixes, unexpected behavior changes, feature creep during refactors — happen because the model was working without a written definition of "correct." The spec is that definition.

---

## Where Spec-Driven Development Is Going

I think we're very early in this. The tools are good, but they're not fully there yet. Here's what I expect to happen in the next couple of years:

**Specs will become first-class citizens in IDEs.** Right now they're mostly markdown files. Future tools (like Kiro is already starting to do) will have proper spec editors, schema validation, and explicit links between spec sections and the code they generated.

**Automated spec-to-test generation will get much better.** Today you can already use the spec to generate tests, but it still requires manual back-and-forth. Soon this will be more automated — the agent will continuously verify that the implementation matches the spec, like a type checker but for behavior.

**Specs will become the unit of collaboration.** Pull requests today contain code diffs. In the near future, they might contain spec diffs — you review the intent change, and the code is just a downstream artifact. This would be a massive shift in how code review works.

**Multi-agent systems will use specs as coordination contracts.** If you have multiple AI agents working on different parts of a system, the spec becomes the interface between them — one agent writes to a spec, another reads from it to implement a compatible counterpart.

The pattern I see is this: as AI gets better at writing code, the thing that differentiates a good engineer from a bad one stops being "can you write code?" and starts being "can you write clear, complete, unambiguous specifications?" That's the skill that's going to matter most.

---

## Should You Adopt This Now?

Yes — but gradually. You don't need to write a 10-page spec for every small task. That would be overkill and you'd hate it.

Here's the approach I'd suggest:

Start with a `CLAUDE.md` (or `.cursorrules`) in your project. Just document your tech stack, your conventions, and two or three "don't do this" rules. That alone will already make your AI interactions noticeably more consistent.

Then, for any feature that has more than two or three behaviors to define, write a short spec before prompting the agent. Keep it simple at first — even just the happy path and two error cases. You'll find it goes much faster than you expect, and the agent output is dramatically better.

Gradually you'll develop a feel for when the spec overhead is worth it (complex features, anything touching security or data integrity, anything multiple people will work on) versus when vibe coding is fine (UI tweaks, small scripts, exploration).

---

## Final Thoughts

What I love about spec-driven development is that it doesn't fight against AI — it embraces it more fully. You're not constraining the AI by giving it a spec. You're giving it better instructions. You're removing ambiguity, which is the number one thing that makes AI agents go sideways.

The best analogy I can think of: a senior engineer doesn't just tell a junior dev "build a login page." They sit down, clarify the requirements, align on the approach, and then say "okay, go build it." Spec-driven development is bringing that same discipline to your AI agent workflow.

Write the spec. Give it to the model. Review the output against the spec. It sounds simple, but it genuinely changes the experience from "chaotic and sometimes magical" to "predictable and consistently useful." And honestly, predictable and consistently useful is what you need when you're shipping real software.

Give it a try on your next feature. I think you'll feel the difference immediately.

---

*The tools mentioned in this post: [Claude Code](https://claude.ai/code), [Cursor](https://cursor.com), [Windsurf](https://codeium.com/windsurf), and [Kiro](https://kiro.dev). For more on how to write effective specs for AI agents, Anthropic's prompt engineering docs at [docs.anthropic.com](https://docs.anthropic.com) are a great starting point.*
