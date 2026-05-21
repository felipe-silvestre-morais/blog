---
title: "Claude Code in Your Pocket: Remote Control, Scheduled Agents, and What That Unlocks"
date: 2026-05-21 15:30:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [claude-code, remote-agents, mobile, productivity, anthropic, schedule, workflows]
---

I was at the playground with my daughter on Saturday morning when I got a push notification from the Claude app. A PR I had asked Claude Code to triage the night before had finished its review, opened a couple of follow-up tasks in the repo, and was now sitting there waiting for me to look at it. I opened the app, scrolled through the diff on my phone, approved one suggestion, asked it to redo another, and put my phone back in my pocket. She was still negotiating turns on the slide.

That whole interaction took maybe ninety seconds. And it made me realize something obvious in hindsight: **Claude Code stopped being a terminal tool a while ago, and I had been treating it like one out of habit.**

If you're still SSH-ing into your laptop from a coffee shop because "that's where Claude Code lives", this post is for you. Let me walk through what remote-controlling Claude Code from your phone actually looks like in 2026, what it unlocks, and where it still falls short.

---

## Wait, What Are We Actually Talking About?

Quick orientation, because "Claude Code on your phone" can mean a few different things and people mix them up.

There are basically three surfaces you can drive Claude Code from today:

| Surface | What it is | Where it runs |
|---|---|---|
| **Terminal CLI** | The classic `claude` command on your machine | Your laptop |
| **claude.ai/code** | Web app, mobile-friendly | Anthropic's cloud, sandboxed env per session |
| **Claude mobile app** | iOS/Android app with a Code tab | Same cloud env as the web app |

The CLI is what most of us started with. The other two are the interesting ones for this post, because they run **in a sandboxed cloud environment**, not on your laptop. That's the unlock. You don't need your machine to be on. You don't need to be on your home network. You don't need a tailscale tunnel. You just need a phone.

The mobile app and the web app share the same backend. Anything you start in one shows up in the other. So "remote control by phone" in practice means: kick off work in the cloud, then steer it from whichever screen happens to be in your hand.

---

## How It Actually Works

Here's the rough mental model I've built for it.

When you start a Claude Code session from the web or mobile app, Anthropic spins up an ephemeral container with a clone of the repo you pointed it at (via GitHub auth). Claude works inside that container — running tests, editing files, hitting the network if you let it. When it's done, it pushes a branch and opens a PR back to your repo. You never see the container; you just see the conversation and the resulting PR.

The phone, in this setup, is a **dumb terminal for a smart backend**. You're not running anything locally. The phone is just where the conversation happens — typing prompts, reading diffs, approving steps.

A typical loop looks like this:

1. Open the Claude app, tap the Code tab.
2. Pick a repo (you've already linked GitHub).
3. Type a task: *"Fix the failing test in `test_payments.py` — it broke after the Stripe SDK bump."*
4. Lock your phone. Go do something else.
5. Get a push notification when Claude finishes (or stops to ask a question).
6. Open the app, review the PR, approve or redirect.

That's it. No terminal, no IDE, no laptop.

---

## The Two Things That Make This Actually Useful

I don't want to oversell this. Typing on a phone is still typing on a phone, and you're not going to write a 600-line refactor prompt on a screen the size of a credit card. But two specific features have made remote-controlling Claude Code from my phone genuinely valuable, not just a party trick.

### 1. Two flavors of automation: `/loop` and `/schedule`

Claude Code gives you two distinct ways to make work happen on a cadence, and people mix them up constantly. Both matter for the phone story, so let me untangle them properly — because once you get the mental model, you'll start using both.

**`/loop` — keep doing this, while I'm here.**

`/loop` is a local primitive. You're in a Claude Code session, and you want the same prompt to fire at a regular interval until you tell it to stop. Example:

```text
/loop 10m check open PRs on `repo/foo`, post a one-line status
of each in this chat, and tell me if anything looks stuck
```

Every 10 minutes, in your current session, Claude reruns that prompt. You can also omit the interval and let the model self-pace:

```text
/loop keep an eye on the deploy in #releases, ping me as soon
as it's green or red
```

Without an interval, Claude picks its own cadence based on what it's watching — tighter when something's about to change, looser when nothing's happening. This is great for polling external state (a CI build, a deploy, a queue depth) without you having to guess the right timing.

What `/loop` *isn't*: persistent. Close the session, the loop stops. The Claude Code process needs to be running. It's a "while I'm here" tool, not a "while I'm asleep" tool. Think of it as a babysitter for the current session, not a cron job.

**`/schedule` — wake up and do this, whether or not I'm here.**

`/schedule` creates a **routine**: a remote, cron-based agent that lives on Anthropic's servers and runs whether your machine is on or off. Same kind of task, reframed:

```text
/schedule every weekday at 8am, check open PRs on `repo/foo`
and post a digest as a comment on the tracking issue
```

That gets stored as a routine, fires every weekday at 8am, runs in a sandboxed cloud container, and the output shows up wherever you told it to land (a GitHub issue, a Slack message, the app inbox). You can manage them with `/schedule list`, `/schedule run <id>` to fire one early, or `/schedule delete <id>` to retire it.

You can also schedule **one-shot** runs:

```text
/schedule tomorrow at 9am, check whether the auth migration ran
overnight, run the validation script if it did, and notify me
```

That fires once and disappears. Genuinely useful for "remind me to verify X tomorrow morning" without cluttering your real calendar.

A few routines I have running right now:

- **Morning PR triage** — every weekday 7:30am, scan open PRs older than 48 hours, summarize what's blocking each one, drop a comment if it's me. I read the summary on my phone over coffee.
- **Dependency drift** — once a week, check for outdated dependencies in three of my repos, open a PR with the safe upgrades (patch + minor only).
- **Backlog gardener** — once a week, look at issues with no activity in 30 days, suggest which ones could be closed.

The thing that took me a while to internalize: **these aren't running on my laptop**. They run whether my laptop is on or off, whether I'm in a meeting or asleep. The phone is just where I read the results.

**When to use which.**

The cleanest framing I've landed on: **`/loop` lives in your terminal. `/schedule` lives in the cloud.** If the work should outlive the session, it's a routine. If it should die when you close the laptop, it's a loop.

| You want to... | Use |
|---|---|
| Poll something during this session | `/loop <interval>` or `/loop` (self-paced) |
| Babysit a long-running thing right now | `/loop` |
| Have something happen at 8am every day | `/schedule` |
| Run a one-time check tomorrow morning | `/schedule` |
| Keep a job running while the laptop is closed | `/schedule` |
| Stop a recurring task when you walk away from the desk | `/loop` |

One more practical note: routines burn tokens every time they fire. A routine that runs hourly across three repos can add up fast. Set them up deliberately, prune them quarterly, and prefer daily over hourly unless you really need the latency.

### 2. Push notifications + the long-running task pattern

The second unlock is more subtle. Once you trust Claude Code to actually finish a task without you watching, you stop watching. And once you stop watching, you can kick off long tasks (30+ minutes of model work) and just walk away.

This is the pattern I've settled into for "real" work tasks:

```text
[me, from phone on the train]
Take a look at the `feat/new-checkout` branch, run the full test
suite, and if anything fails, fix it without changing the public
API of CheckoutService. If you finish without failures, run the
typecheck. Notify me when you're done or if you get stuck.
```

Lock phone. Read a book. Forty minutes later, ding — PR's ready or there's a question. This used to require either babysitting from a laptop or hoping the SSH tunnel didn't drop. Now it just works.

I want to be honest about one thing: this only works because the cloud environment is **sandboxed and authenticated against my GitHub**, so I can let it run without supervision in a way I wouldn't let a process run on my actual laptop. The blast radius is "a branch on a repo I own", not "my entire filesystem".

---

## Real Cases Where This Shines

I want to be concrete about the cases I keep reaching for. These are the ones that actually pay off — not the ones that sound cool in demos.

**Case 1: Reviewing PRs on the move.** Honestly the killer app for me. I tap a notification, read the diff, leave comments, approve. No laptop. Better than the GitHub mobile app for this because I can ask follow-up questions: *"Why did you change this regex?"* and get an actual answer from Claude in the same thread.

**Case 2: Kicking off the long task before the meeting.** Big task that'll take 30+ minutes? I now start it from my phone at the top of the meeting and check the result at the end. The old version of me would have started it at my laptop, then waited, then context-switched to the meeting late.

**Case 3: The "fix it while I sleep" task.** Set up a flaky test investigation as a scheduled overnight task. Wake up to either a fix or a clear writeup of why it's flaky. This is luxurious in a way that still feels slightly illegal.

**Case 4: Capturing ideas at 11pm.** I get an idea about a project while in bed. Before, I'd either lose it or get up and open the laptop. Now I open the Claude app, dump the idea into a session, and tell it to draft a design doc or sketch the code. Half the time I delete it in the morning. The other half it's a great starting point.

**Case 5: Triaging issues from the couch.** "Open the three oldest issues in repo X, read them, write a one-paragraph summary of each, and tell me which one I should pick up first." This is the kind of work I used to *not do* because the activation energy was too high. From the couch on a Sunday, it's two thumbs.

---

## The Patterns I've Actually Settled Into

A few things I've learned the hard way that I'd tell past-me.

**Write the prompt like you're briefing someone, not chatting.** Phone-typed prompts tend to be short and vague. That's fine for "summarize this PR", but for anything that involves code changes, take the extra fifteen seconds to specify the scope, the constraint, and what "done" looks like. The cloud agent is fast but it's not a mind reader, and you don't want to come back to an unrelated refactor.

**Default to automation for anything you've typed twice.** If you find yourself running the same prompt in the same session, reach for `/loop`. If you find yourself running it across sessions, reach for `/schedule`. Future-you will thank present-you for not retyping it a fiftieth time.

**Set up `CLAUDE.md` carefully.** Since the cloud agent is starting from a fresh container every time, your `CLAUDE.md` (and any [agent metadata files](https://felipe-silvestre-morais.github.io/blog/posts/the-hidden-layer-in-your-repo-a-tour-of-ai-agent-metadata/)) are doing more work than they do locally. There's no shared session history to fall back on. Repo conventions, "don't touch X", commands to run — put them in the repo, not in your head.

**Be deliberate about which repos get cloud access.** I link my personal stuff and my side projects. I don't link client code or anything with secrets I haven't audited. The sandbox is sandboxed, but secrets in repos are still secrets in repos, and I want to know exactly what surface a cloud agent has access to.

**Don't try to write the prompt in the elevator.** Some tasks are still better at a keyboard. Phone is for kicking things off, steering, reviewing — not for architecting.

---

## Where It Still Falls Short

I promised balance, so here's the honest list of things that still don't work well.

**Typing complex prompts on a phone is bad.** Voice dictation helps a lot. But anything where you need to paste a chunk of code or write a precise multi-step plan, you're better off waiting until you're at a keyboard.

**No local context.** The cloud agent can't see files on your laptop, your scratch notes, the half-written branch you didn't push. If your context is local, the cloud is the wrong tool.

**Network-restricted dependencies.** If your project needs internal package registries, VPN-only services, or local databases to actually run tests — the cloud env can't reach them. There are workarounds (secrets, allowlisted hosts) but they're work.

**Cost isn't free.** Cloud sessions and scheduled agents burn tokens like any other Claude usage. A scheduled agent that runs hourly across three repos adds up. Watch your usage; don't set up cron jobs you'll forget about.

**It's still a young surface.** The mobile app is good but not as feature-complete as the CLI. There are commands and flags that only really work in the terminal. Expect some "wait, I can't do that from here?" moments.

---

## What I Think Happens Next

This is speculative, but I've been thinking about it a lot.

The shift from "AI coding tool on my laptop" to "AI coding agent running in a cloud sandbox I drive from my phone" is bigger than it sounds. It's the same shift that turned "Photoshop on my machine" into "Figma in a browser" — except for the unit of work being a *PR* instead of a *file*.

What that means in practice: more of the coding work in my week is becoming **asynchronous**. I queue tasks, I review results, I steer between them. The synchronous "type code, see code" loop is still there for hard problems and design work, but for the long tail of maintenance work, I'm increasingly the *editor* and Claude is the *author*. The phone is the editing surface because the phone is what I have on me.

I don't think this fully replaces sitting down at a keyboard. I do think it changes the ratio. A year ago I probably did 95% of my coding work at the desk. Now it's maybe 70%, and a meaningful chunk of the rest happens on a phone in pockets of time I previously couldn't have used at all.

---

## Final Thoughts

If you've been using Claude Code only from your terminal, I'd nudge you to try this. Not as a replacement — as an additional surface. Specifically:

1. **Link a personal repo to claude.ai/code today.** Just one. See what it feels like.
2. **Set up one `/schedule` routine and one `/loop`.** A weekly dependency check is a low-stakes routine. A self-paced `/loop` watching your next CI run is a low-stakes loop. Run both once and the distinction will click.
3. **Install the Claude mobile app and enable push notifications.** Not for the dopamine hit — for the "task done" signal that lets you stop watching.
4. **Try one long task end-to-end from your phone.** Kick it off, lock the screen, do something else, come back to review.

The mental model shift is the hard part. You're not "using Claude Code on your phone" — you're letting Claude Code run somewhere else, and using your phone to talk to it. Once that clicks, you start finding pockets of time you didn't realize were usable.

I still do most of my real work at the desk. But the work-shaped objects I now ship from a playground, a train, or the couch — those add up. Quietly. And on a Saturday morning, that's a strange and good thing.

---

*Resources: [Claude Code on the web](https://claude.ai/code), [Anthropic's Claude mobile app](https://claude.ai/download), and the built-in `/loop` (in-session, self-paced or timed) and `/schedule` (remote routines, cron-based) skills in your local Claude Code install.*
