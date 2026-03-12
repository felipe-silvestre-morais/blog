---
title: "I Shipped a Feature Today. I'm Not Sure I Built It."
date: 2026-03-12 20:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [ai-coding, software-engineering, identity, craftsmanship, developer-experience]
---

I closed a pull request yesterday. It was a solid one — a new integration with a third-party payments API, a bit of middleware refactoring, some tests. My manager looked at it and said "great work." My teammates approved it without much comment. It shipped to production and worked perfectly.

And then I sat there for a moment feeling... weird. Because I had written maybe 5% of that code, the rest was the prompt to the model and boom!

This is the thing nobody is talking about loudly enough. Not the productivity gains. Not the model benchmarks. Not which tool has the best autocomplete. It's the strange, uncomfortable feeling that creeps in when you close a PR that Claude Code or Cursor Agent basically wrote for you, and you try to answer the question: *did I actually build this?*

I don't think I'm alone in this. I've talked to a lot of engineers lately and this guilt — or grief, or whatever you want to call it — is quietly everywhere.

---

## The Shift Happened Faster Than We Could Process It

Two years ago, AI tools were assistants. They filled in the next line, suggested a function signature, saved you from googling the same Stack Overflow answer for the fifth time. You were still firmly in control. You were still the one *building*.

Then 2025 happened. After a brief prompt, Claude Code can build features, run tests, fix bugs, and check its own work, all without direct human supervision. I remember reading that and thinking "sure, theoretically." Then I watched it actually do it to a real problem in my real codebase and the feeling was genuinely unsettling.

One junior engineer, anonymous because he isn't authorized to speak to the press, described his predominant feeling as grief. "The skill you spent years developing is now just commoditized to the general public. It makes you feel kind of empty."

That word — grief — is more accurate than guilt, actually. It's the feeling of losing something you didn't expect to lose so fast.

---

## The Identity Was Always Tied to the Act of Building

Here's the thing about software engineering as a profession: we built our identity around making things.

At its core, writing code is about mastery and control — skills we've spent years perfecting. Programming today is much more about stitching together APIs, frameworks, and libraries in creative ways to build something meaningful.

The act of thinking through a problem, designing a solution, and expressing it precisely in a language that makes a machine do exactly what you intended — that is what drew most of us to this profession. It is a creative act, a form of craftsmanship, and for many engineers, the most satisfying part of their day.

Now that act is being partially delegated. And we don't quite know how to feel about it.

One engineer captured the shift perfectly in a widely shared essay, describing how AI transformed the engineering role from builder to reviewer. "Every day felt like being a judge on an assembly line that never stops. You just keep stamping those pull requests."

I relate to that more than I'd like to admit. There's something different about reviewing AI-generated code versus writing code yourself, even when the outcome is the same. The sense of ownership isn't quite the same.

---

## The Research Doesn't Make It Simpler

The data on this is honestly a bit confusing, which doesn't help with the emotional side.

On one hand, the productivity narrative: early studies from GitHub, Google, and Microsoft found developers completing tasks 20% to 55% faster. And Google's Sundar Pichai noted on the Lex Fridman Podcast that engineering velocity at Google had increased roughly 10%.

But then there's the uncomfortable counterpoint from the METR randomized controlled trial — one of the most rigorous studies done on this: when experienced developers working on their own open-source repositories used AI tools, they took 19% longer than without them.

A Multitudes report assessing more than 500 developers found that engineers merged about 27% more pull requests with AI — but they also experienced nearly a 20% rise in out-of-hours commits, a potential signal of burnout ahead.

So we are doing more. We might be shipping more. But we're also working more hours, and the work feels less like ours. That's a strange trade.

And there's the trust problem. In 2025, only 29% of developers reported trusting the accuracy of AI-generated code — a sharp decline from 40% in prior years. The number-one frustration for 45% of respondents was dealing with "AI solutions that are almost right, but not quite."

So even the productivity gains are partially spent on verification. We review code we didn't write, looking for errors we didn't make, in systems we don't fully understand.

---

## The Skill Erosion Question Is Real

There's something else worth naming honestly: the atrophy risk.

One engineer stopped using AI tools for a side project when free access ran out, and found himself struggling with tasks that previously came naturally. "I was feeling so stupid because things that used to be instinct became manual, sometimes even cumbersome."

An international AI safety report cited a study finding that clinicians' rate of detecting tumors during colonoscopy was 6% lower after several months of performing the procedure with AI assistance — a warning about how over-reliance on AI can quietly degrade the underlying human skill.

I think about this a lot when I reach for Claude Code reflexively instead of thinking through a problem first. The concern isn't dramatic — it's subtle. Like going from cooking every night to ordering delivery, and then one day realizing you're not sure you still know how to cook.

Chris Lattner, the creator of LLVM and Swift, described the concern clearly in a conversation with Jeremy Howard: "What you want to get to, particularly as your career evolves, is mastery. The concern I have is this culture of, 'Well, I'm not even going to try to understand what's going on.'"

---

## But Here's Where I Land — And It's Actually Positive

Okay, I've spent a lot of words on the discomfort. Let me be honest about the other side, because I think it's the more important side.

When I'm not in my feelings about ownership, I notice something real: **I'm doing things that used to be impossible for me alone.**

That payment integration I mentioned? Two years ago it would have taken me three days. It took me half a day — and the half-day I spent was on understanding the API semantics, validating the business logic, thinking about edge cases, and reviewing the security model. Not on syntax. Not on boilerplate. Not on copy-pasting from the documentation.

That's actually the work I *want* to do. The part I care about. The parts where judgment matters.

The skill of 2026 is not writing a QuickSort algorithm; it is looking at an AI-generated QuickSort and instantly spotting that it uses an unstable pivot. This requires higher expertise, not lower.

That reframing matters. The bar isn't lower. In some ways it's higher, because now what's left for humans is exactly the hard stuff — system design, security reasoning, business context, judgment calls that AI doesn't have enough context to make correctly.

According to Google's DORA team, 90% of surveyed technology professionals said they were using AI at work, with more than 80% reporting it had boosted their productivity. "We see a large majority of folks that are relying on AI to get their job done," noted Nathen Harvey, who leads the DORA research team.

The engineers who are thriving right now aren't the ones who refuse to use these tools or feel guilty every time they do. They're the ones who figured out where their judgment is irreplaceable and planted their flag there.

---

## What Actually Changes About Our Job

Let me try to be concrete about what I think the new job actually looks like, because "focus on higher-level thinking" is advice that sounds good and means nothing.

**You become a code reviewer at a higher volume.** The craft doesn't disappear, it shifts. You need to read AI code critically — not accept it because it compiles, but understand it well enough to own it. That requires more skill than people give it credit for. When researchers analyzed 112,000 programs generated by ChatGPT, they found that over half contained at least one security vulnerability. Someone has to catch those. That's you.

**You become the keeper of context.** AI doesn't know your team's history, your product's quirks, why a particular decision was made three years ago. You do. That context is expensive to build and impossible to fully transfer into a prompt. The engineers who have it are valuable in ways that aren't going away.

**You become a better architect by necessity.** When implementation is fast, the bottleneck moves upstream to design. The quality of your system now depends more on the decisions you make before the code is written than on the execution of writing it. That was always true in theory. Now it's true in practice too.

**You get to build things you wouldn't have had the capacity to build before.** This is the part I find genuinely exciting. Projects that felt out of reach — not technically, just effort-prohibitive — are now real. Side projects that would have taken months take weeks. New ideas get prototyped in hours instead of days.

---

## The Historical Pattern Is Reassuring

I want to end with something that helps me when the existential anxiety kicks in.

There was a time when people wrote in absolute binary using absolute machine addresses, and many programmers of that time resisted using symbolic approaches like FORTRAN. Perhaps we're not replacing craftsmanship but rather shifting it to a higher level of abstraction.

Every generation of engineers has had this moment. Assembly to C. C to Java. Manual memory management to garbage collection. Every time the abstraction level went up, someone felt like the old craft was being lost. And every time, the engineers who learned to work at the new level became more powerful, not less.

We are not the last craftspeople. We are craftspeople being handed better tools, and also being asked to care about different things. The judgment, the taste, the system thinking, the ability to know when the AI is confidently wrong — those skills matter now more than ever, because someone has to hold the line.

There's going to be a bifurcation, as Chris Lattner put it, between people who develop learned helplessness and a smaller group who use AI to learn faster and grow faster than the tools themselves improve.

I want to be in the second group. And I think you probably do too, otherwise you wouldn't be reading this.

---

## My Practical Answer to the Guilt

So what do I actually do when I close that PR and feel weird about it?

First, I remind myself that **what I shipped is mine** — not because I typed every character, but because I made every decision that mattered. I chose the approach. I validated the behavior. I understood the tradeoffs. The AI wrote the syntax. I wrote the intent.

Second, I deliberately **keep the muscle working**. I don't use AI for everything. Some days I write a whole module by hand, even when I could delegate it. Not out of nostalgia, but because I don't want to lose the instinct. Think of it like doing the long run even when you have a car.

Third, I've stopped measuring my value by **lines of code** and started measuring it by **decisions made and problems solved**. Those are things I actually own, no matter which tool wrote the implementation.

The transition is real, and it's not entirely comfortable. But I think engineers who learn to navigate it — who hold onto their judgment while letting go of the guilt — are going to be able to do things that previous generations of software engineers could never have imagined.

That's worth something. That's actually worth a lot.

---

*Have you felt this? I'd love to know how other engineers are processing the shift. Find me on [GitHub](https://github.com/felipe-silvestre-morais)*