---
title: "From Vibe Coding to Agentic Engineering: The Shift Karpathy Saw Coming"
date: 2026-05-14 09:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [agentic-engineering, vibe-coding, llms, ai-tools, software-development, andrej-karpathy]
---

There's a quote that's been sitting in my head since I watched Andrej Karpathy's talk at AI Engineer Summit: *"I've never felt more behind as a programmer."*

That's Karpathy. The guy who co-founded OpenAI, got Tesla Autopilot working, and has spent the last few years teaching AI fundamentals to hundreds of thousands of people. If *he* feels behind, what does that say about the rest of us?

Honestly, the first time I read it, my reaction was somewhere between relief and panic. Relief because — okay, it's not just me. Panic because — wait, if Karpathy feels behind, that means the gap is real, and it's growing fast.

I've been sitting with that tension for a while now. And after watching this talk a few times, I think I finally understand what he's pointing at. It's not that the tools got too complicated. It's that the *paradigm* shifted, and most of us haven't fully caught up yet.

---

## The December Moment

Karpathy describes a clear inflection point in December of last year. Before that, he was using AI assistants the way most of us do — ask for a chunk of code, review it, fix the parts that are wrong, move on. The AI was a fast autocomplete, basically.

Then in December — he was on a break, had more time to just *play* — he started noticing something different. The chunks came out fine. He asked for more. That came out fine too. He kept asking, and at some point he realized: he couldn't remember the last time he actually corrected the output.

And just like that, he was vibe coding.

I had a similar moment, though I couldn't have articulated it as cleanly at the time. There's a specific session I remember where I was building something with Claude Code — I don't even remember what exactly, something with a FastAPI backend — and at some point I stopped reading the diffs. Not because I stopped caring, but because I'd developed enough trust in the system that the overhead of checking every line felt... wasteful. I just ran the tests, checked the output, and kept going.

That's the shift. And it happens quietly. One day you're reviewing every edit. The next, you're writing software at a speed that would have seemed impossible to you six months ago.

---

## Software 3.0, or: The Prompt Is The Program

Karpathy has this framing I keep coming back to. Software 1.0 is traditional code — you write explicit instructions, the computer executes them. Software 2.0 is machine learning — you don't write instructions, you arrange data and objectives and train a model to learn the behavior. Software 3.0 is this: you prompt. The LLM is the interpreter, and your context window is how you program it.

This sounds abstract until you see a concrete example — and Karpathy's best one is the OpenClaude installation story. Old paradigm: you'd write a bash script, handle all the edge cases for different platforms, maintain it as things break. New paradigm: the OpenClaude installation is literally a block of text you paste to your agent, and the agent — using its own intelligence — figures out your environment, debugs things in a loop, and makes it work.

The difference in power is enormous. The bash script has to anticipate every possible scenario. The agent reasons about the scenario it actually finds itself in.

His other example — MenuGen — is even more striking. He built a whole app to take a photo of a restaurant menu, OCR it, generate images for each dish, and display them nicely. Months of work to build something genuinely useful. Then he saw someone do the same thing by giving a photo to Gemini with a one-line prompt. Same output, no app.

His reaction: "All of my MenuGen is spurious. It's working in the old paradigm. That app shouldn't exist."

That line hit me harder than it probably should have. How many things I've built "shouldn't exist" by the same logic? Not because they're bad — they worked — but because I was still thinking in software 1.0 terms when the world had already moved on.

---

## Vibe Coding vs. Agentic Engineering

This is the crux of the talk, and Karpathy's distinction here is sharp.

**Vibe coding raises the floor.** Anyone can build anything now, regardless of technical background. That's genuinely incredible — it democratizes software creation in a way that nothing else has. But vibe coded software is also unreviewed, potentially insecure, and the person who vibe coded it probably couldn't debug it if something went wrong.

**Agentic engineering preserves the quality bar.** You're still responsible for your software. You're still accountable for security. You're still expected to ship something that won't break production. But — and this is the key — you can go much faster doing it. The question is: how?

Karpathy frames agentic engineering as an actual engineering discipline. The agents are powerful but stochastic — they're fallible, they make weird mistakes, they don't share your implicit understanding of your system's constraints. Coordinating them well, without sacrificing quality, is the skill.

He puts the performance ceiling much higher than 10x. I believe him. I've had days where I've shipped more working code in four hours than I used to in a week. I've also had days where I spent six hours debugging something an agent introduced that I would've caught immediately if I'd been writing the code myself. That variance is real, and learning to work *with* it rather than against it is the whole game right now.

---

## The Jagged Intelligence Problem

One of the more interesting threads in the talk is Karpathy's concept of "jagged intelligence." These models are genuinely astonishing at some things — refactoring a 100,000-line codebase, finding zero-day vulnerabilities — and genuinely terrible at others that seem much simpler.

His example: Opus can restructure your entire codebase, but will tell you to *walk* to a car wash 50 meters away instead of driving there.

The reason, he argues, is verifiability. The labs train these models using reinforcement learning on verifiable domains — math, code, formal logic — where you can definitively say whether an answer is correct. So the models get spectacular at those things. But everyday common-sense reasoning doesn't have clean verification signals, so the models stay jagged there.

This has a practical implication for how you build with them: you need to stay in the loop, especially for decisions that fall outside the domains where the model was heavily trained. Don't just trust the output blindly. The model will confidently tell you to walk to the car wash. You need the judgment to catch it.

This also matters for what you build. If you're trying to automate something and wondering whether the current generation of LLMs is good enough, the most useful question isn't "is this task hard?" It's "is this task verifiable?" If there's a clean signal for whether the output is correct, you're in luck. If you're relying on aesthetic judgment or common sense alone, you'll get jaggedness.

---

## What You Still Own

Despite all this, there's a lot that Karpathy thinks humans remain genuinely responsible for — at least for now.

**Taste and aesthetics.** He mentions looking at agent-generated code and having "a little bit of a heart attack" — it works, but it's bloated, full of copy-paste, with awkward abstractions that are technically fine but feel wrong. The agents don't have internalized aesthetic standards yet. They're not being rewarded for elegant code, so they don't produce it.

**Judgment on the big decisions.** His MenuGen agent tried to match Stripe and Google accounts by email address instead of maintaining a persistent user ID. That's a fundamental design error — email addresses can be different across services, so the correlation breaks. The agent didn't catch it because it wasn't in the spec. Karpathy had to be the one to realize the spec needed that constraint.

**Understanding, not just thinking.** This is the one that stuck with me most. Someone tweeted something that he can't get out of his head: *"You can outsource your thinking, but you can't outsource your understanding."* 

That's exactly right. I can have Claude Code write a whole authentication system, and if I don't understand how it works, I can't review it, can't debug it, can't extend it, can't explain it to anyone else. The thinking — the code generation — is outsourced. But the understanding has to live somewhere in a human brain, and right now that means mine.

The answer isn't to go back to writing everything by hand. It's to be intentional about what you actually internalize versus what you just let the agent handle. Knowing the PyTorch API details? Let the agent handle that. Understanding that there's an underlying tensor view that can be more or less efficient depending on how you access memory? That you still need to know.

---

## The Agent-Native World We Haven't Built Yet

One frustration Karpathy mentions that I feel deeply: almost everything is still built for humans.

Documentation is written for humans to read and follow. Services require humans to navigate GUIs and configure settings. Deployment workflows involve clicking through menus. Even the "developer experience" stuff — which is supposed to be streamlined — is still fundamentally designed around a human executing steps.

He describes deploying MenuGen on Vercel as being more painful than actually building the app. Setting up DNS, configuring services, connecting things together — all of it requiring manual human navigation. He imagines a world where you can just give a prompt to an agent and have the whole thing deployed without touching anything.

We're not there yet. But that's the direction everything is moving. The infrastructure that survives is going to be the stuff that's legible to agents — clear APIs, machine-readable configs, responses that make sense to an LLM parsing them.

The question I keep asking: what am I building right now that I should actually be building agent-native? Where am I writing human-readable documentation that should really be instructions an agent can execute directly? That reframe matters a lot.

---

## On Hiring and the New Bar

Karpathy makes a point about hiring that I think is underappreciated. Most companies haven't updated their hiring process for the agentic era. They're still giving people coding puzzles — small, well-defined problems with obvious correct answers.

That's completely the wrong signal now.

He suggests something like: give candidates a big project, let them build it with agents, then have agents try to break it. Did they ship something secure? Did it hold up? That's the actual skill you're evaluating — not whether they can solve a sorting problem from memory, but whether they can coordinate agents effectively and maintain quality across a complex system.

I don't know many companies actually doing this. Most are still pattern-matching on credentials or leetcode scores. But if the 10x engineer of two years ago is now getting 50x leverage from agents, the variance in output between a strong agentic engineer and a mediocre one is enormous. Getting the hiring filter right matters a lot more now.

---

## Final Thoughts

Karpathy's talk is worth watching in full. But the thing I keep coming back to is the reframe he offers on what it means to be a good engineer right now.

It's not about writing more code. It's not about knowing more APIs. It's about being a good *director* — staying in charge of the spec, the taste, the architecture, the judgment calls — while letting agents handle the implementation details. The human is still in the room. Just playing a different role.

The engineers who figure this out early are going to compound that advantage significantly. The ones who stay in the old paradigm — either refusing to use agents, or vibe coding with no quality bar — are going to struggle.

Karpathy was right to feel behind. But I think what that feeling actually points to is the size of the opportunity, not the impossibility of catching up. The tools are better than they've ever been. The ceiling just got a lot higher.

*Watch the full talk: [Andrej Karpathy: From Vibe Coding to Agentic Engineering](https://www.youtube.com/watch?v=96jN2OCOfLs) — it's about an hour and worth the time if you build anything with LLMs.*
