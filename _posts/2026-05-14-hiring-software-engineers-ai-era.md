---
title: "The LeetCode Interview Is Dying. What Should Replace It — And What You Should Actually Study"
date: 2026-05-14 11:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [software-engineering, hiring, ai-tools, career, agentic-engineering, leetcode, jobs]
---

A friend of mine went through a technical interview a few months ago. He'd been using Claude Code daily at his previous job — orchestrating multi-step agents, reviewing generated code, making architectural calls on systems he'd never touched before. Shipping faster than he ever had.

And then he sat down for a technical screen where someone asked him to reverse a linked list on a whiteboard.

Not because it was relevant to the role. Not because he'd ever need to do it by hand. Because that's just what you do.

He got through it fine. Got the job. But he told me it felt like being asked to start a car with a hand crank because the interviewer wanted to make sure you *really* understand combustion engines. The signal was real once. It just doesn't mean what it used to.

---

## Where We Are: The System Is Cracking But Not Broken

Here's the honest picture: most companies still do it the old way. Around 80% of interviews at Google, Amazon, and Meta still featured algorithmic puzzles as of mid-2025. LeetCode, data structures, whiteboard problems — the classic playbook.

But cracks are forming.

**Meta** started piloting an AI-enabled coding interview in October 2025. One of the two traditional coding rounds at the onsite was replaced with a 60-minute session in a modified CoderPad environment *with an AI assistant built in*. Candidates could use GPT-4o, GPT-5, Claude Sonnet 4.5, Gemini 2.5 Pro — whatever they'd actually use on the job. The shift was explicit: they're no longer testing "can you code quickly without help." They're testing "can you think, prompt, and collaborate with AI while exercising good engineering judgment."

Meta planned to extend this to all back-end roles throughout 2026.

**Google** wrapped up a pilot of new in-person SWE interview formats at its biggest engineering hubs — Bay Area, Seattle, NYC, Bangalore — and was rolling them out to all SWE roles by late 2025.

**CodeSignal** launched what they're calling industry-first *agentic coding assessments*. Instead of solving a data structure problem, candidates are given requirements, use agentic AI tools to build a working solution, and then explain their decisions to a human reviewer. About a third of their customers adopted some version of AI-assisted assessment in 2025. **HackerRank** launched AI-Assisted Interviews in July 2025 as well.

And then there are companies like Stripe, Coinbase, and Airbnb that have quietly moved toward more realistic, open-ended challenges that reflect actual work.

So the shift is happening. It's just happening *slowly*, and unevenly, and not nearly as fast as the tools themselves are changing.

---

## Why Companies Are Slow to Change (And Why That's Not Entirely Stupid)

The defensiveness of traditional hiring practices isn't just laziness. There are real reasons these companies haven't updated yet, and they're worth acknowledging honestly.

**Scale.** A single entry-level position at Google can receive 10,000 applications. LeetCode-style filters eliminate 90% of candidates before a human ever reviews a resume. There's no elegant replacement for that at volume. Project-based assessments take time — time to design, time to grade, time to standardize.

**The AI impostor problem.** Companies are terrified of interviewing candidates whose skills are actually the AI's skills. If you can use Claude during your take-home, how do I know what *you* can do? LeetCode, love it or hate it, is at least proof of work that can't be easily outsourced — or at least couldn't be until recently.

**Institutional inertia.** Recruiting pipelines are built on top of established processes. Changing them requires buy-in across teams, coordination with external tools, and someone willing to own the experiment.

But here's the thing: these are reasons the change is slow, not reasons the change is wrong. The signal that LeetCode was providing is eroding. "Can you write an algorithm under pressure without any tools" maps less and less onto "can you build and maintain production software with the tools available in 2026."

And some companies have made moves that are, charitably, not how you'd want to do this.

---

## The Cautionary Tale: Klarna and What Happens When You Move Too Fast

Klarna is the clearest case study in what happens when a company overcorrects.

Between 2022 and 2023, CEO Sebastian Siemiatkowski reduced headcount from 5,527 to about 3,400 — crediting AI. Developer hiring froze. The pitch was that AI had made this possible, that the company could do more with far fewer people.

Then the CTO admitted publicly that "AI cannot work everywhere," service quality had suffered, and Klarna started rehiring.

This is the reality check the AI-replaces-all-engineers crowd doesn't talk about much. Yes, AI is dramatically increasing what individual engineers can do. No, that doesn't mean you can cut 40% of your engineering team and expect the same output.

The honest version is something more like: the best engineers with great AI tooling are now significantly more productive than they were. But you still need engineers. You still need people who understand systems, who exercise judgment, who own decisions. The leverage went up. The human requirement didn't disappear.

Shopify took a different approach — and I think it's more interesting. Tobi Lütke issued an internal directive that teams must *demonstrate why AI cannot do the job* before requesting additional headcount. That's not "fire everyone." That's "prove AI can't handle this before adding a person." That's actually a reasonable forcing function, even if it sounds harsh in headline form.

---

## The Number That Surprised Me

I expected research to surface data showing AI tools massively accelerate software engineers. And some data shows that.

But the most methodologically rigorous study I found — a randomized controlled trial by METR in July 2025 — found something more complicated.

16 experienced developers. 246 tasks. Mature open-source projects where the developers had roughly five years of prior experience. With AI tools vs. without AI tools.

Result: **developers using AI tools were 19% *slower*.**

That's not a typo. Slower.

Here's what makes it even more interesting: the developers *thought* they were 20% faster. They perceived a speedup that wasn't there in the data.

The researchers are careful to note AI is "likely useful in other contexts" — unfamiliar codebases, less experienced developers, different kinds of tasks. The study was specifically about experienced engineers working in projects they knew deeply.

I think the honest interpretation is this: the productivity gains from AI tools are real, but they're not evenly distributed. You get the most leverage when you're entering unfamiliar territory — a new codebase, a new language, a domain you don't know well. You might get less (or negative) leverage when you're an expert working in something you've maintained for years and the AI keeps suggesting things that don't fit the local context.

The implication for hiring: the "10x productivity with AI" narrative is partially real, but the gains are uneven and situational. Hiring for raw AI-tool fluency without also evaluating engineering judgment is going to miss important signals.

---

## What the New Hiring Process Should Actually Look Like

If I were designing an engineering interview process from scratch today — and more companies should be — it would look nothing like LeetCode.

**Replace algorithmic puzzles with real projects.** Andrej Karpathy suggested something like: give a candidate a substantial project, have them build it with agents, then have agents try to break it. Did the result hold up under adversarial pressure? That's much closer to what the job actually requires.

**Assess judgment, not just execution.** Can the candidate recognize when the AI output is wrong? Can they articulate *why* a given architecture decision was made, not just that it was? Can they spot the bug the agent introduced? These are harder to test than "did the algorithm run in O(n log n)," but they're the actual skills that matter now.

**Test tool fluency explicitly.** Not "do you know the LLM exists," but: show me how you'd approach this problem using Claude Code. Let me watch you work. How do you decompose the task? How do you review the output? When do you override what the agent suggests?

**Give more time, not less.** The skill is orchestration over time — not sprint coding in 45 minutes. A take-home over a few days with a clear spec and open-ended latitude tells you much more about how someone actually builds things.

Some companies are beginning to do versions of this. Most aren't.

---

## What Engineers Should Actually Study Now

This is the part I'm most opinionated about.

The entry-level job market is genuinely difficult right now. Software developer postings are down roughly 35% from pre-2020 levels and about 70% from their 2022 peak. Salesforce froze junior developer hiring for 2025. Entry-level postings dropped ~60% between 2022 and 2024. CS graduate unemployment is sitting at 6.1% — among the highest across majors, which would have been unthinkable three years ago.

There's a real structural shift happening where AI tools are absorbing what used to be entry-level work — the boilerplate, the first drafts, the simple CRUD, the basic automation. If you were going to spend two years writing junior code to build up to senior work, that ladder just got harder to climb.

Here's what I think actually matters to focus on, in order of importance:

**1. Systems thinking and architecture.** This is the one skill AI genuinely cannot replace right now. Understanding how systems behave under load, how data flows, where the consistency boundaries are, what happens when a service fails — this is still deeply human territory. The more an AI can handle implementation, the more the architectural thinking you bring becomes the actual value. Invest heavily here. Read the good books (*Designing Data-Intensive Applications* is still essential). Study distributed systems. Understand databases at a deeper level than "SQL goes in, data comes out."

**2. Security and reliability.** Vibe coded software is a security disaster waiting to happen. Agents introduce dependencies you didn't ask for, skip validation, and make assumptions about trust boundaries. The engineer who understands what's actually dangerous and can review AI-generated code for vulnerabilities is worth a lot right now. OWASP, threat modeling, how auth actually works — this is underrated.

**3. Genuine AI-tool fluency, not surface-level.** Not just "I have GitHub Copilot" — everyone has GitHub Copilot. I mean: can you actually orchestrate Claude Code or Cursor to implement a real feature end-to-end while maintaining code quality? Do you know when to override the agent? Do you understand what makes a good system prompt for your CLAUDE.md? Can you review AI-generated code critically? The difference between someone who uses AI superficially and someone who uses it well is enormous, and it's becoming the most visible split in engineering productivity.

**4. Reading and reviewing generated code.** This sounds obvious but a lot of people skip it. If AI writes 22-27% of production code (which is roughly where we are as of late 2025), humans need to be good at reviewing that code, not just generating it. Reading is a different skill from writing. Invest in it.

**5. Fundamentals — selectively.** The argument that "fundamentals don't matter anymore because AI can handle everything" is wrong. But the *which* fundamentals to invest in has changed. You don't need to memorize sorting algorithms. You do need to understand memory, I/O, network latency, and concurrency. You need to understand data structures conceptually even if you never implement a B-tree by hand. The level of abstraction has risen; the need for underlying understanding hasn't disappeared.

**6. Communication and accountability.** As agents do more of the execution, the human's job increasingly involves writing clear specs, communicating what was built and why, and owning outcomes when things go wrong. The engineer who can talk to product managers, explain tradeoffs to stakeholders, and write a good technical design doc is more valuable than they were before — not less.

What I'd deprioritize: grinding LeetCode for its own sake. Know your data structures, understand algorithmic complexity, but spending 200 hours on competitive programming in 2026 is a poor use of time relative to the above. If you need to clear a hiring bar that requires it, do the minimum to clear that bar — then invest the rest of your time elsewhere.

---

## A Note on the Junior Engineer Situation

I want to be honest about this because I think a lot of the optimistic narratives gloss over it.

The entry-level engineer experience is harder than it's been in 20 years, by some measures since the dot-com bust. The tools that make senior engineers more productive also reduce the demand for the kind of work that used to be how junior engineers learned.

A 2024 survey found 70% of hiring managers believe AI can do the jobs of interns. 57% of hiring managers trust AI's work more than interns or recent grads. That's a brutal data environment if you're just starting out.

But I don't think the career path is closed. I think it's changed. The engineers who break through are going to be the ones who skip straight to working at a higher level — not trying to compete with AI on code generation, but developing systems thinking and judgment faster than the previous generation had to. The ramp is steeper, but the ceiling is also higher.

The risk is spending years trying to get good at the things AI is already better at you, instead of getting good at the things AI isn't.

---

## Final Thoughts

The hiring process for software engineers is in this weird transitional state where the world has changed faster than the systems we use to evaluate people. Most companies are still running interviews calibrated for a world that existed three years ago, measuring skills that matter less now and skipping skills that matter more.

The companies moving fastest — Meta's AI-enabled interview, CodeSignal's agentic assessments — are pointing at what this should look like. Real projects. Judgment under realistic conditions. Tool fluency tested explicitly, not assumed.

The rest will catch up eventually. They always do.

In the meantime: if you're an engineer trying to figure out where to put your energy, the answer isn't "learn to prompt better." It's "develop judgment that AI doesn't have yet." Systems thinking, security awareness, the ability to review and own AI-generated code, communication that makes you a good director of agents rather than just a user of them.

You can outsource your thinking. You can't outsource your understanding. The engineers who internalize that distinction early are the ones who are going to be fine.

*Sources and further reading:*
- *[Meta's AI-Enabled Coding Interview Guide](https://www.coditioning.com/blog/13/meta-ai-enabled-coding-interview-guide)*
- *[METR Study: AI Makes Experienced Developers 19% Slower (arxiv 2507.09089)](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)*
- *[Stack Overflow 2025 Developer Survey — AI](https://survey.stackoverflow.co/2025/ai)*
- *[CodeSignal Agentic Coding Assessments](https://www.prnewswire.com/news-releases/codesignal-launches-industry-first-agentic-coding-assessments-for-ai-era-engineering-hiring-302732265.html)*
- *[How Tech Coding Assessments Are Splintering in 2025](https://hellointerview.substack.com/p/how-tech-coding-assessments-are-splintering-2025)*
- *[The impact of AI on software engineers in 2026 — Pragmatic Engineer](https://newsletter.pragmaticengineer.com/p/the-impact-of-ai-on-software-engineers-2026)*
- *[AI vs Gen Z — Stack Overflow Blog](https://stackoverflow.blog/2025/12/26/ai-vs-gen-z/)*
