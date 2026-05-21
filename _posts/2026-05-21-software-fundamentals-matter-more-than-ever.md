---
title: "Software Fundamentals Matter More Than Ever in the AI Age"
date: 2026-05-21 10:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [ai-coding, software-design, tdd, ddd, fundamentals, llm, claude-code]
---

I've been feeling something lately that I couldn't quite name. There's this sense of exhaustion mixed with progress. I'm shipping faster than ever, but I'm also more tired than I've ever been in my dev career. I'm using AI every day, but some days it feels like I'm fighting it more than working with it.

Then I watched Matt Pocock's talk "Software Fundamentals Matter More Than Ever" and it clicked. Not just the "yeah this makes sense" click — the kind that makes you go back and look at how you've been working and realize you've been doing it wrong.

---

## The Specs-to-Code Promise (And Why It Falls Apart)

If you've been in the AI coding space for more than five minutes, you've heard the pitch: write a spec, have AI generate the code, don't look at the code, just update the spec and regenerate when something breaks. Run the "AI compiler" again and again.

Matt calls this the *specs-to-code movement*, and he's honest about having tried it himself. The result? Each iteration produced worse code. He ran the compiler, got bad code. Ran it again, got even worse code. Kept going, ended up with garbage.

Why? Because this is just vibe coding by another name. And there's a deeper reason it fails: **software entropy**.

The Pragmatic Programmer has a whole chapter on this. Software entropy is the idea that codebases naturally tend toward chaos. Every time you make a change thinking only about that change — not about the design of the whole system — your codebase gets worse. The specs-to-code loop does exactly this, over and over, faster and faster.

And here's the brutal flip side: **bad code is now more expensive than ever**. AI in a good codebase does really, really well. In a bad one, it struggles. So if you're accumulating technical debt through careless AI-generated code, you're actively reducing the value of your AI investment.

Good code bases are easy to change. Bad code bases aren't. That's the whole game.

---

## Failure Mode 1: The AI Doesn't Do What I Want

You know this one. You have a clear idea in your head, the AI produces something entirely different. It's not that the AI is stupid — it's that you and the AI don't share a *design concept*.

Frederick P. Brooks wrote about this in *The Design of Design*. When two people build something together, there's an ephemeral idea floating between them — the shared mental model of what's being built. That's the design concept. It's not a markdown file, it's not a spec — it's the invisible theory of what you're making.

When you start a task with AI, there's no shared design concept. You have yours; the AI has none.

Matt's solution is a Claude Code skill he calls **"Grill Me"**:

```
Interview me relentlessly about every aspect of this plan until
we reach a shared understanding. Walk down each branch of the
design tree, resolving dependencies between decisions one by one.
```

This simple prompt turns the AI into an adversary — in the good sense. It asks you 40, 60, sometimes 100 questions before it's satisfied. The conversation you generate becomes a product requirements document, or gets turned directly into GitHub issues for an autonomous agent to pick up.

The result is shared understanding *before* a single line of code is written. Wildly better outcomes.

I've been using something similar. Before any non-trivial task, I ask Claude to interrogate me about the design. It's annoying sometimes — it feels slower — but the code that comes out the other side is so much more aligned with what I actually wanted.

---

## Failure Mode 2: The AI Is Too Verbose

The AI talks past you. It uses terms you don't use. You use terms it doesn't use. The implementation diverges from your mental model because you're not speaking the same language.

This is classic Domain-Driven Design territory. DDD has a concept called **ubiquitous language**: a shared vocabulary among developers, domain experts, and the code itself. Everyone uses the same terms, all derived from the same domain model.

Matt built a skill for this too — one that scans your codebase, identifies the key terminology, and generates a markdown file of shared terms. Pass this to the AI at the start of sessions. What he found by reading the AI's thinking traces is that with a ubiquitous language, the AI thinks more precisely, with less verbosity, and produces implementations more aligned with the actual plan.

It's a simple idea with a big payoff. Your codebase already has a vocabulary. Extracting and formalizing it takes ten minutes. Using it consistently makes every AI interaction sharper.

---

## Failure Mode 3: The AI Built the Right Thing But It Doesn't Work

The design was aligned, the implementation started, but the output doesn't work. This one is painful because you got the requirements right and still lost.

The solution is **TDD** — but not just for quality. For *pace control*.

The Pragmatic Programmer calls it "outrunning your headlights": the rate of feedback is your speed limit. If you're driving at night, you can only safely go as fast as you can see. Code the same way. Test as you go, take small deliberate steps.

The problem is that AI by default is terrible at this. It produces huge amounts of code and then thinks "I should probably type-check that." It outpaces its own feedback loops.

TDD forces the AI to take small steps. Write a failing test, make it pass, refactor. The discipline of TDD isn't just about correctness — it's about keeping the AI from generating 500 lines before checking if any of it works.

If you're not using TypeScript, or not giving front-end AI access to the browser to verify output, you're leaving massive feedback loop value on the table.

---

## Failure Mode 4: The Code Is Hard to Test (and Hard for AI to Navigate)

This one connects everything else. Testability and good architecture turn out to be the same problem.

John Ousterhout's *A Philosophy of Software Design* has a concept called **deep modules**: relatively few large modules with lots of functionality hidden behind *simple interfaces*. The opposite — lots of small shallow modules with complex interfaces — is what AI naturally produces.

Here's the thing about shallow codebases: they're hard for AI to explore. The AI has to walk through a maze of tiny blobs just to understand what's going on. It can't get to the right module in time, misses dependencies, and produces bad changes. Shallow module codebases — which AI is excellent at *creating* — are also codebases where AI performs worst when *modifying*.

Deep module codebases look different: clear boundaries, interfaces at the top, complexity hidden inside. And here's the beautiful part: **you test at the interface**. The AI fills in the implementation; you verify it from the outside. You don't need to hold all the interior complexity in your head.

Matt has a skill for this too: "Improve Codebase Architecture" — a reusable set of steps that scans for related code, identifies refactoring opportunities, and proposes wrapping them in deep modules.

---

## Failure Mode 5: Your Brain Can't Keep Up

This was the one that hit hardest. You're shipping more code than ever, but you feel more exhausted than you've ever been. Something is wrong.

The culprit is shallow, sprawling codebases. You — like the AI — have to keep all that complexity in your head. Deep modules solve this for you too. Treat modules as gray boxes: design the interface yourself, delegate the implementation to AI, and stop reviewing the internals of every non-critical module.

Kent Beck: *"Invest in the design of the system every day."*

Specs-to-code is the opposite of this. It's divesting from system design — throwing away the one thing that makes AI useful: a codebase that's easy to change, easy to test, and easy to reason about.

---

## What This Means for How I Work

I came away from this talk with a clearer picture of my actual job in the AI age:

**I am the strategist. AI is the tactical programmer.**

The AI is the sergeant on the ground, making code changes. I need to be thinking at the strategic level: architecture, interfaces, domain modeling, module design. The classics — DDD, TDD, clean architecture — weren't made obsolete by AI. They became *prerequisites* for using AI well.

The books I should be reading aren't about AI. They're the same books I should have been reading anyway:
- *A Philosophy of Software Design* — John Ousterhout
- *The Pragmatic Programmer* — Hunt & Thomas
- *The Design of Design* — Frederick P. Brooks

My practical changes since watching this:
1. I start non-trivial tasks with a "grill me" phase — AI questions me before writing code
2. I'm maintaining a ubiquitous language document per project, updated whenever new domain concepts emerge
3. I write the test (or at least the test skeleton) before asking AI to implement anything
4. When reviewing AI-generated code, I ask: "Is this adding shallow modules, or deep ones?" If shallow, I refactor before moving on

---

## Final Thoughts

The narrative that AI will replace developers by letting non-engineers "just write specs" misunderstands what makes software hard. It's not the typing. It's the design decisions, the architecture, the accumulated understanding of what the system is and should become.

AI speeds up the execution layer enormously. But that makes the design layer *more* important, not less. If you can generate 10x more code, you'd better be generating 10x better code. Otherwise you're just producing a much bigger mess, much faster.

Your years of experience understanding software design aren't liabilities in the AI age. They're multipliers. The developers who will do the best with these tools are the ones who understand the fundamentals deeply — and use that understanding to direct the AI strategically.

Code is not cheap. Code matters. Software fundamentals matter more than ever.

---

*Watch the full talk: [Software Fundamentals Matter More Than Ever — Matt Pocock](https://www.youtube.com/watch?v=v4F1gFy-hqg)*

*Books referenced: [A Philosophy of Software Design](https://www.amazon.com/Philosophy-Software-Design-John-Ousterhout/dp/1732102201) · [The Pragmatic Programmer](https://www.amazon.com/Pragmatic-Programmer-journey-mastery-Anniversary/dp/0135957052) · [The Design of Design](https://www.amazon.com/Design-Essays-Computer-Scientist/dp/0201362988)*

*Matt Pocock's skills repo: [mattpocock/skills](https://github.com/mattpocock/skills)*
