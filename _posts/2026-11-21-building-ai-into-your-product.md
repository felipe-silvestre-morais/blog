---
title: "Not Just Building With AI, Building AI Into the Product: Models, Costs, Measurement and the Engineer's New Job"
date: 2026-11-21 23:05:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [ai-engineering, llm, product-engineering, model-selection, cost-optimization, evals, observability, ai-agents]
---

Most of what I have written here this year is about **using AI to build software**: coding agents, worktrees, specs, skills. That is one half of the story. The other half is quieter and, I would argue, bigger for most businesses: **putting AI models inside the product itself**, so that your *customers* use them, whether or not they know it.

That changes the engineering problem completely. When an agent writes your code, a bad answer costs you a review cycle. When a model sits inside your product, a bad answer reaches a customer, a good answer costs you money **every single time it runs**, and nobody is in the loop to catch the difference. You are now operating a component that is probabilistic, metered by the token, and changes when its provider ships a new version.

This post is a field guide for that world. It covers:

1. **Where AI actually fits** inside a product, with real examples and ideas.
2. **Choosing different models for different jobs**, with real prices and a worked cost example.
3. **How to track every AI request**, so you can see what you are spending and why.
4. **How to measure quality correctly** and improve it without fooling yourself.
5. **How products will change** once models are embedded in them.
6. **What the engineer's role becomes**, and what you need to know.

A note on sources before we start. For prices and the measured cost figures I use Anthropic's published [pricing page](https://platform.claude.com/docs/en/about-claude/pricing) and its cost-optimization guide, which I read on 21 November 2026. Those are the numbers I could verify, so the examples use Claude models. The principles apply to any provider, but check *your* provider's price list: prices change often, and an example that is out of date is worse than no example.

---

# Part 1: Where AI Fits Inside a Product

## Stop Asking "Where Can I Add AI?"

The weak version of this question produces a chatbot bolted onto the corner of the app. The better question is: **which jobs in my product are currently done by a human, by brittle rules, or not done at all, and involve language, judgement or messy input?** Models are good at a specific family of jobs. Here is the list I use:

| Job | What the model does | Typical model tier | Example |
|---|---|---|---|
| **Classify / route** | Put input into a category | Small, cheap | Tag a support ticket as billing, bug or feature request |
| **Extract** | Turn messy text, PDFs or images into structured fields | Small to mid | Pull amounts, dates and vendor from an invoice |
| **Summarize** | Compress long content | Small to mid | Summarize a 40-message thread for a manager |
| **Generate** | Draft text, code or content | Mid to large | Draft a reply, a product description, a lesson exercise |
| **Answer over your data (RAG)** | Find the relevant records, then answer from them | Mid | "What did we decide about pricing in Q3?" over a workspace |
| **Converse / assist** | Multi-turn help inside a workflow | Mid | An in-app assistant that knows your account |
| **Act (agents)** | Choose tools and carry out multi-step tasks | Mid to large | "Reconcile these invoices and flag the mismatches" |
| **Moderate / guard** | Check inputs and outputs against policy | Small | Block unsafe or off-topic requests before the main model sees them |
| **Rank / personalize** | Order or tailor content to a user | Small, or classic ML | Reorder a feed, tailor onboarding |

Two things stand out in that table. First, **most of the jobs are small ones**. The glamorous "agent that does everything" is the exception, not the rule. Second, notice what is *not* on the list: arithmetic, exact lookups, anything with a deterministic right answer that code can compute. Use code for those. A model that approximates a regex is slower, costlier and less reliable than the regex.

## What Existing Products Do

Companies rarely publish which models they use or how, so I will stick to what is reasonably public and flag what is secondhand:

- **Duolingo** uses language models to help generate exercises. As described in public write-ups, human experts set the theme, grammar, vocabulary and exercise types, and the model produces candidate exercises. The design lesson: **the human defines the constraints, the model fills in the volume**. (Secondary sources, treat details with care.)
- **Shopify's Sidekick** is an assistant inside the merchant admin. Shopify describes related AI features such as an inbox agent that answers customers from the product catalog, store policies and published content, and suggestions to fix weak product listings. The lesson: the assistant is valuable because it sits **next to the data and the actions** the user already has. See [Shopify's overview](https://www.shopify.com/blog/large-language-models).
- **Notion AI** answers questions in plain English over a workspace's content, which works because the content already lives in the product. The lesson: **the best AI features often come from data you already own.**
- **GitHub Copilot** put a model inside the editor, where the user already was, with no new workflow to learn.
- **OpenAI's ChatGPT Wallet**, which I covered in [a recent post]({% post_url 2026-11-21-ai-agents-with-wallets %}), is the next step: models that do not just advise but *spend money*, behind per-purchase approval and scoped payment tokens.

For a wider catalogue, Evidently's list of [55 real-world LLM applications](https://www.evidentlyai.com/blog/llm-applications) is a good starting point.

## Ideas for Your Own Product

Here are patterns that work in many kinds of software. I am proposing these as ideas, not claiming a specific company ships them:

- **Support tooling:** classify and prioritize tickets (small model), draft replies from your docs (mid model), escalate only the hard ones (larger model).
- **Onboarding and empty states:** instead of a blank screen, let the user describe their goal and generate a starter configuration for them to edit.
- **Search that understands intent:** retrieve relevant items first (see my posts on [RAG basics]({% post_url 2026-03-07-rag-basics %}) and [RAG vs long context]({% post_url 2026-03-10-rag-vs-long-context %})), then let a model rewrite the answer in context.
- **Data entry elimination:** extract fields from uploaded documents and show them pre-filled for a human to confirm.
- **Internal tools:** log summarizers, incident timelines, PR descriptions, release notes. Internal use is a forgiving place to start, because mistakes are cheap.
- **Accessibility:** alt text for images, simplified rewrites of dense text, captions.
- **Proactive monitoring:** a nightly job that reads the day's anomalies and writes a plain-English brief.

A rule of thumb for picking the first one: choose a feature where **a wrong answer is cheap and obvious**, where **a human can easily review or undo**, and where the **value of a right answer is high**. Drafts and suggestions fit that. Silent automation of irreversible actions does not.

---

# Part 2: Different Models for Different Jobs

## Why One Model for Everything Is Expensive

Here are the prices per million tokens (MTok) from Anthropic's pricing page for four current models, as of 21 November 2026:

| Model | Input | Output | Cache read | Batch (input / output) |
|---|---|---|---|---|
| **Claude Haiku 5.5** (prompts up to 100K tokens) | $0.10 | $0.50 | $0.01 | $0.05 / $0.25 |
| **Claude Sonnet 5.5** | $2.00 | $10.00 | $0.10 | $1.00 / $5.00 |
| **Claude Opus 5.5** | $4.00 | $20.00 | $0.20 | $2.00 / $10.00 |
| **Claude Fable 5.2** | $8.00 | $40.00 | $0.20 | $4.00 / $20.00 |

That is an **80x gap** in both input price and output price between the smallest and largest model. Three details matter in practice:

- **Haiku 5.5 is priced by prompt length.** Above 100,000 tokens, every token of the request costs 5x as much (input $0.50, output $2.50). Keep its prompts short, or compact conversations before they cross that line.
- **Cache reads are much cheaper than fresh input.** On Opus 5.5 and Sonnet 5.5 a cache read costs 0.05x the input price, on Fable 5.2 it is 0.025x. The first time you send a prefix, you pay a premium to write it to the cache (1.25x input for the 5-minute cache). After that, repeats are cheap.
- **Batch is 50% off every token**, including cached ones, in exchange for results within 24 hours. It is for work nobody is waiting on.

## A Worked Example: One Feature, Four Models

Say a feature runs **1 million requests a month**, each with **2,000 input tokens and 300 output tokens**. The arithmetic: 1M × 2,000 = 2,000 MTok in, 1M × 300 = 300 MTok out.

| Model | Input cost | Output cost | Monthly total |
|---|---|---|---|
| Haiku 5.5 | 2,000 × $0.10 = $200 | 300 × $0.50 = $150 | **$350** |
| Sonnet 5.5 | 2,000 × $2 = $4,000 | 300 × $10 = $3,000 | **$7,000** |
| Opus 5.5 | 2,000 × $4 = $8,000 | 300 × $20 = $6,000 | **$14,000** |
| Fable 5.2 | 2,000 × $8 = $16,000 | 300 × $40 = $12,000 | **$28,000** |

(This ignores caching, thinking tokens, retries and tool overhead, all of which move the real number. It is for intuition, not budgeting.)

The same feature costs **$350 or $28,000 a month** depending on a single parameter. If the cheap model does the job well enough, you have just saved $27,650 a month. If it does not, you have a quality problem that is invisible on a spreadsheet. That is why the rest of this post is about *measuring*.

## What Anthropic's Own Measurements Say

The cost-optimization page publishes measurements. They are Anthropic-internal and the page itself calls them directional, so read them as guidance, not guarantees. A few that I found useful:

- **A good default starting point is a mid-priced model at its default setting**, not the biggest. On a coding benchmark (SWE-bench Pro), Opus 5.5 at its default matched a Fable-class model at about **a fifth of the cost per solved task** ($0.22 vs $1.19).
- **The smallest model is for high-volume, checkable work.** On a graduate-level science question set (GPQA Diamond), Haiku 5.5 scored 85% vs 91% for Opus 5.5 and Sonnet 5.5, at about **a twentieth of Opus 5.5's cost per correct answer**. A six-point gap is irrelevant for classifying tickets and fatal for medical advice. *Your* task decides.
- **Effort is a dial inside one model.** On Opus 5.5, going from `low` to `medium` to `high` effort on a coding subset:

| Effort | Tasks solved | Cost per task |
|---|---|---|
| `low` | 87.4% | $0.11 |
| `medium` (default) | 92.8% | $0.21 |
| `high` | 95.3% | $0.29 |
| `xhigh` | 96.7% (single run) | $0.74 |

The curve is steep at the start and flat at the end: the last 1.4 points cost 2.5 times as much. On a research benchmark the curve was even flatter. The lesson is to **sweep effort on your own workload** before paying for the top.

- **Re-run only the failures at higher effort.** Run everything at `low`, and send only the tasks that fail a check (tests, a validator) to `high`. On Opus 5.5 this solved about 97% at roughly $0.17 per task, vs 95.3% at $0.29 for everything at `high`. It needs a reliable failure signal.
- **Prompt caching is the largest lever.** Without it, the same benchmark runs cost about 3.1 to 3.3 times as much on the models measured, and a triage-agent workload cost about 7.3 times as much. Healthy agent loops read a median of 84% of their input from the cache, and below about 80% the page suggests hunting for something that is breaking it (a timestamp in the system prompt, for example).

## Patterns for Using More Than One Model

**1. One model, tuned.** Start here. Tune the prompt, caching and effort of a single model before building anything fancier. Anthropic's own guide says to tune one model first and build a pairing only if it beats the single model's whole score-versus-cost curve. One model also means one cache.

**2. Cascade (small first, escalate on failure).** The small model answers, a check decides whether the answer is good enough, and only failures go to the big model. The cost arithmetic is worth knowing: if the small model costs one-twentieth of the large one, then at **20% escalation** you pay about 25% of the large-model-only cost (5% for everything, plus 20% for the escalations). At **60% escalation** you pay about 65%, because you pay for *both* models on the escalated requests. The savings erode quickly if the small model fails often. (This math is from a third-party analysis, but you can verify it with arithmetic.)

**3. Router (classify first, pick a model).** A cheap model or a rule decides which model handles each request. It can work well, but the router itself has to be built, tuned and monitored. One engineering post I read describes a case where routing *increased* total cost by 12% because the overhead exceeded the savings at low volume. Start with a two-tier cascade and add a router only when volume justifies it.

**4. Advisor (cheap model does the work, consults a stronger one).** Anthropic measured this and found it fragile: it only pays if the cheap model actually consults the advisor. In the one pairing that consulted often, the score rose by 1.7 points (at the edge of noise) for about 2.1 times the cost. Where the cheap model rarely consulted, results were no better than the cheap model alone.

**5. Orchestrator (a big model plans, small workers do the bulk).** This helped in two measured cases: routine work with an expensive tail, and work too big for one context window. For a single dependent chain of steps, the big model alone at lower effort did better.

**6. Batch for anything asynchronous.** Nightly summaries, backfills, evaluations, content generation: if nobody is waiting, take the 50% discount.

## And Sometimes the Answer Is "No Model"

Before reaching for any LLM, ask whether a **rule, a database query, a classical ML model or a plain embedding search** does the job. They are cheaper, faster and fully predictable. The mature pattern is a pipeline where code handles what code handles well and models handle the fuzzy parts.

---

# Part 3: Tracking Every AI Request

If you cannot see your AI requests, you cannot manage their cost, quality or safety. The good news is that every response tells you exactly what it consumed.

## What to Record for Each Call

For every model call, I would log:

- **Which feature** triggered it (`ticket_triage`, `reply_draft`). This is the single most useful field, and the one providers cannot give you.
- **Model and version**, and your own **prompt version**, so you can compare releases.
- **Tokens, split four ways**: regular input, cache writes, cache reads and output. Each bills at a different rate. Thinking tokens bill as output, and responses can report them separately.
- **Cost**, computed from those tokens and the price table.
- **Latency**: time to first token for interactive features, total time for everything.
- **Outcome**: `stop_reason` (a refusal or `max_tokens` cut-off is a failure, even though the HTTP call succeeded), retries, and errors.
- **A trace ID and a pseudonymous user or tenant ID**, so you can join the call to the rest of the request and spot a single customer driving cost.
- **A quality signal when you have one**: thumbs up or down, whether the user accepted or edited the output, whether the ticket was reopened.

Be deliberate about **prompts and outputs**: they are invaluable for debugging and they often contain personal data. Decide what you store, for how long and who can read it. Many tracing tools capture content by default, so check.

## A Minimal Wrapper

Here is a small Python wrapper that does the essentials. It uses the official Anthropic SDK and logs one JSON line per call:

```python
import json, logging, time
import anthropic

client = anthropic.Anthropic()
log = logging.getLogger("ai")

# USD per million tokens. These change; load them from config, not code.
PRICES = {
    "claude-haiku-5-5":  {"in": 0.10, "out": 0.50,  "cache_read": 0.01, "cache_write": 0.125},
    "claude-sonnet-5-5": {"in": 2.00, "out": 10.00, "cache_read": 0.10, "cache_write": 2.50},
    "claude-opus-5-5":   {"in": 4.00, "out": 20.00, "cache_read": 0.20, "cache_write": 5.00},
}

def cost_usd(model, usage):
    p = PRICES[model]
    return (
        usage.input_tokens * p["in"]
        + (usage.cache_read_input_tokens or 0) * p["cache_read"]
        + (usage.cache_creation_input_tokens or 0) * p["cache_write"]
        + usage.output_tokens * p["out"]
    ) / 1_000_000

def ask(feature, prompt_version, model, system, user_text, max_tokens=1024):
    start = time.monotonic()
    response = client.messages.create(
        model=model,
        max_tokens=max_tokens,
        system=system,
        messages=[{"role": "user", "content": user_text}],
    )
    log.info(json.dumps({
        "feature": feature,
        "prompt_version": prompt_version,
        "model": model,
        "response_id": response.id,
        "input_tokens": response.usage.input_tokens,
        "cache_read_tokens": response.usage.cache_read_input_tokens or 0,
        "cache_write_tokens": response.usage.cache_creation_input_tokens or 0,
        "output_tokens": response.usage.output_tokens,
        "cost_usd": round(cost_usd(model, response.usage), 6),
        "latency_ms": int((time.monotonic() - start) * 1000),
        "stop_reason": response.stop_reason,
    }))
    return response
```

Treat this as a sketch, not production code. A real one also needs error handling and retries (the official SDKs already retry transient errors like 429 and 5xx twice by default), streaming for long outputs, a check for `stop_reason == "refusal"`, and, on newer models, the provider's recommended fallback setting for refusals. Read the current docs for the model you use.

## Go From Logs to Answers

Once you have those lines in a log pipeline or a warehouse, the questions become easy to ask:

- **Cost per feature, per day, per tenant.** Which feature is eating the budget? Is one customer 40% of it?
- **Cost per *completed task*, not per call.** A cheaper request that needs three retries is not cheaper. Anthropic's guide makes the same point: judge cost per completed task.
- **Cache hit rate**, from `cache_read_tokens` over total input. A sudden drop usually means someone put a timestamp or a request ID into the prompt prefix.
- **Latency percentiles** (p50, p95, p99), not averages.
- **Refusal rate, `max_tokens` cut-off rate and escalation rate**, if you cascade.
- **Tail cost.** In one research benchmark, two problems out of twenty carried 43% of the spend. The average hides the expensive few.

## Use the Provider's Own Numbers Too

Your logs are your attribution. The provider's usage and cost reports are your **source of truth for the bill**. Anthropic offers a Usage and Cost Analytics API for organizations that can group token usage and cost by model, API key and workspace. A practical trick: **use separate API keys or workspaces per feature or environment**, so the provider's reports can answer "what did this feature cost?" without relying on your own instrumentation alone. Set **spend limits** per workspace as a last-resort backstop, so a runaway loop hits a ceiling before it hits your credit card.

## Tracing and Observability Tools

For multi-step flows (retrieval, then a model call, then a tool call, then another model call), per-call logs are not enough; you want **traces**. The ecosystem is mature: open-source and hosted tools such as **Langfuse, LangSmith, Arize Phoenix and Braintrust** capture traces, tokens and evaluations, and most build on **OpenTelemetry**. Two caveats from my reading:

- The OpenTelemetry **GenAI semantic conventions are still marked experimental ("Development")** according to the sources I found, so attribute names and metrics have been changing. Pin versions and expect to adjust dashboards.
- The standard defines **token-usage attributes but not cost**, and has no notion of *your* feature. You compute cost from a price table and add your own `feature` attribute on each span.

## Put Guardrails on the Money

- **Per-request limits:** set `max_tokens` deliberately. Too low truncates answers; too high lets one request run away.
- **Per-user and per-tenant rate limits** and quotas.
- **Budget alerts** on cost per day and cost per feature.
- **A kill switch**: a feature flag that turns an AI feature off, or swaps it to a cheaper model, without a deploy.
- **Prompt-injection and abuse protection.** If the feature reads untrusted content, treat it as hostile; I wrote about containment in the [MCP tunnel post]({% post_url 2026-05-26-mcp-tunnel-security %}) and about [building safer agents]({% post_url 2026-03-15-build-safer-agents %}).

---

# Part 4: Measuring Quality Correctly (and Improving It)

Cost dashboards are easy. The hard question is: **is it any good, and did my last change make it better?** The honest way to answer is an **evaluation set**, an "eval".

## Build a Small Eval First

Anthropic's cost guide describes a minimal recipe that I think is the right on-ramp:

- **20 to 30 real inputs**, pulled from production logs or written by hand. Freeze them.
- **A way to judge each output.** In order of preference: exact or golden answers; an automated check (valid JSON, required fields, tests pass); a short rubric a human scores; and, only if nothing else works, a model acting as the judge.
- **A runner** that executes the frozen inputs through one configuration and reports **pass rate and cost per task**.

That is enough to make per-change decisions. For a final production cutover, the same guide's bar is higher: around **50 cases and at least five trials per configuration**, because a single run can swing by a case or two for no real reason.

A tiny version, with the dataset as one JSON object per line (`{"input": "...", "expected": "billing"}`):

```python
import json

def evaluate(model, cases, classify):
    passed, costs = 0, []
    for case in cases:
        response = classify(model, case["input"])        # your feature code
        label = response.content[0].text.strip()
        passed += (label == case["expected"])
        costs.append(cost_usd(model, response.usage))
    return {
        "model": model,
        "pass_rate": passed / len(cases),
        "cost_per_task": sum(costs) / len(cases),
    }

cases = [json.loads(line) for line in open("evals/tickets.jsonl")]
for model in ["claude-haiku-5-5", "claude-sonnet-5-5"]:
    print(evaluate(model, cases, classify))
```

Run it and you have the data to answer "does the model that costs a twentieth as much clear the bar?" with a number instead of a hunch.

## Rules That Keep You Honest

- **Change one thing at a time.** Prompt, model, effort and retrieval all interact. If you change three and the score moves, you learned nothing.
- **Look at cost and quality together**, and plot them. Take the **Pareto frontier**: the configurations where nothing is both cheaper and better.
- **Respect noise.** A difference of one or two cases on a single run is not a result. Repeat the trials.
- **Check the tail, not just the average.** The cheap model looks fine on the median task and loses on the hard few that decide whether customers stay.
- **A cheaper change that gives back accuracy is not an optimization.** Revert it and write down why.
- **Keep the eval running after launch.** Models, prompts and data all drift.
- **Treat a model upgrade as a migration.** Re-run the evals, re-tune effort from the new model's default, and re-audit prompts written for an older model. Anthropic's guide notes that carried-over prompts can make a newer model do more work for no gain.

## Offline Evals, Online Evals, and Humans

The sources I read agree on a three-layer setup:

- **Offline:** your frozen dataset, run before every release, like unit tests.
- **Online:** score a **sample of live traffic** (a few percent is common, plus anything flagged by guardrails or low confidence) to catch problems the dataset missed.
- **Human review** for the small slice that the automatic checks flag as failing, and to **calibrate any model-as-judge** against human labels before you trust it.

And close the loop: **every production failure that matters becomes a new case in the offline set**, so it can never regress silently. Over a few months, your eval set becomes the most valuable asset in the whole system, because it encodes what "good" means for *your* product.

## Product Metrics Are the Final Judge

Model-quality scores are proxies. The real measures are product outcomes:

- **Acceptance and edit rate:** how often do users keep the suggestion, and how much do they change it?
- **Task success and time saved.**
- **Escalation or reopen rate**, for support features.
- **Retention and satisfaction** among users of the feature vs those who are not.
- **Guardrail metrics:** complaints, refusals, safety incidents.

Where you can, **A/B test** (or at least shadow-run the new configuration on real traffic before switching), and ship behind a flag so you can roll back.

## The Improvement Loop

Here is the loop, in the order I would apply the levers, following Anthropic's guidance of free wins first and trade-offs last:

1. **Instrument** (Part 3), so you can see the baseline.
2. **Fix the free wins:** prompt caching, trimming input (do not send the whole manual every time), `max_tokens` hygiene, batching asynchronous work. These cut cost *without* lowering quality.
3. **Sweep effort** on your current model against the eval.
4. **Only then try a cheaper model**, one step at a time, against the eval.
5. **Consider multi-model patterns** (cascade, advisor, orchestrator) only if a single tuned model is not enough.
6. **Ship behind a flag, monitor online, feed failures back into the eval set**, and repeat.

---

# Part 5: How Products Will Change

I will mark this section clearly as **my opinion**; nobody knows exactly where this goes. But some shifts look likely to me, and several are already visible.

**From forms and menus to intent.** Instead of "fill in these twelve fields", the interface asks what you want and proposes a filled-in answer for you to confirm. The UI becomes a place to review and steer, not to type.

**From tools you operate to agents that act.** Software starts doing multi-step work on your behalf: reconciling, scheduling, buying. That raises the stakes on permissions, approvals and audit trails, as in the wallet example above. The product becomes partly a *policy* system: what may the agent do alone, what needs a tap, what is forbidden.

**Your next user may be an agent.** More of your product's traffic will come from other people's agents calling your APIs and tools. That is why standards such as MCP matter (I covered the [new stateless spec]({% post_url 2026-11-21-mcp-2026-07-28-stateless-spec %})), and why merchants are deciding whether to let agents in at all: according to recent reporting, Amazon blocks agents from buying and United Airlines has said it will block agents from booking tickets. Having a deliberate **agent-facing surface** (clean APIs, signed requests, clear terms) is becoming part of product design.

**Background and proactive behaviour.** Features that run on a schedule or on a trigger, without anyone opening the app: the morning brief, the anomaly explanation, the draft ready before you ask. These are cheap to run in batch and change how a product feels.

**Adaptive interfaces.** Layouts, onboarding and help that adapt to the user's context, generated or arranged at runtime. This raises hard questions about consistency and testing.

**Economics change.** Traditional software has near-zero marginal cost per user action. AI features have a real **cost of goods sold per use**. That pushes products towards usage limits, credit systems, tiered plans and outcome-based pricing, and it makes *cost per completed task* a metric your finance team will care about.

**Trust becomes a feature.** Showing sources, letting users edit and undo, explaining what the system did and being honest when it is unsure. Regulation pushes the same way: the EU AI Act's transparency obligations began applying on 2 August 2026, with a 2 December 2026 deadline for marking AI-generated content for systems already on the market, according to law-firm summaries I read. I am not a lawyer; check the details with one if you ship in the EU.

**Models get cheaper and smaller, and run in more places.** The price of the smallest tier above is a tenth of a dollar per million input tokens. As that trend continues, features that were too expensive to run on every request will become routine, and some work will move to the device or to your own infrastructure.

**What I do not expect:** the death of deterministic software. The systems that win will be hybrids, with models handling the fuzzy parts and ordinary code, tests and permissions guarding the edges.

---

# Part 6: The Engineer's Role, and What You Must Know

I wrote about the hiring side of this in [software engineers in the AI era]({% post_url 2026-05-14-hiring-software-engineers-ai-era %}). From the product side, here is how I see the job changing.

## The Shift

You used to write the function. Now you increasingly **design a system around components you cannot fully control**: a model whose behaviour is probabilistic, whose provider can change it, and whose every call costs money. The center of gravity moves:

- from *writing logic* to **defining what "correct" means** and proving it with evals;
- from *debugging stack traces* to **debugging behaviour** through traces and examples;
- from *optimizing CPU* to **optimizing tokens, latency and quality together**;
- from *"does it work?"* to **"how often does it work, for whom, and at what cost?"**.

The code is still yours. It just does a different job: orchestrating, constraining, measuring and recovering.

## What an Engineer Needs to Know

Here is the checklist I would hand to someone starting on an AI-enabled product:

1. **How models actually work**, at a practical level: tokens, context windows, why they hallucinate, what temperature-like settings and thinking effort do. (My post on [how LLMs actually work]({% post_url 2026-03-20-how-llms-actually-work %}) is a starting point.)
2. **Prompt and context engineering.** What goes in, in what order, how to keep the stable prefix first so caching works, and how to keep it lean.
3. **Retrieval.** When to use RAG, when long context is enough, how to chunk and rank, and how to handle stale or conflicting data.
4. **Tool and API design for models.** Narrow, well-described tools with schemas the model cannot easily misuse. Structured outputs when you need a guaranteed shape.
5. **Evals.** How to build a dataset, pick graders, calibrate a judge, read noise, and gate releases on results.
6. **Observability and cost.** Instrumentation, attribution per feature, budgets, alerts and kill switches (Part 3).
7. **Model selection and routing.** The cost and quality trade-offs, caching, batching, effort, cascades.
8. **Reliability engineering.** Timeouts, retries with backoff, idempotency, graceful degradation and fallback models, rate limits, and handling refusals and truncated outputs. An AI call is a remote, flaky dependency like any other.
9. **Security.** Prompt injection, data exfiltration, least-privilege credentials, sandboxing, treating all retrieved content as untrusted. Containment, not wishful thinking.
10. **Privacy and compliance.** What data goes to the provider, retention, regional processing, user consent, and the rules where you operate.
11. **UX for uncertainty.** Showing sources, confidence, approvals and undo; designing for the case where the model is wrong.
12. **Product sense.** Knowing which problems deserve a model, and having the discipline to use a rule, a query or a human when they do not.
13. **Fundamentals.** Data modelling, distributed systems, testing, and the ability to read code. They are what make everything above work, and [they matter more, not less]({% post_url 2026-05-21-software-fundamentals-matter-more-than-ever %}).

## A Learning Path

If this list feels long, here is a sequence that builds one skill on the next:

- **Week 1:** build one small feature with a single model call and the logging wrapper above. Watch the cost per call.
- **Week 2:** write a 25-case eval for it and get a pass rate. Try a cheaper model and compare.
- **Week 3:** add caching and see the cost drop; add a cascade or a validation step and see the quality change.
- **Week 4:** put it behind a flag, add online sampling, and review the failures by hand. Add the interesting ones to the eval set.

At the end you have done the whole loop once: build, instrument, evaluate, optimize, monitor. Everything after that is repetition at larger scale.

---

# Putting It Together: A Support-Ticket Assistant

To close, a deliberately boring example, because boring is what ships. A product receives **100,000 support tickets a month**, and wants AI to help.

- **Step 1, classify every ticket** with the small model (about 1,500 input and 50 output tokens each): 150 MTok in × $0.10 plus 5 MTok out × $0.50 is **about $17.50**.
- **Step 2, draft a reply** for the 40% that need one, using the mid-tier model (3,000 in, 400 out): 120 MTok × $2 plus 16 MTok × $10 is **$400**.
- **Step 3, escalate the hard 10%** (4,000 tickets) to the larger model with more context (6,000 in, 800 out): 24 MTok × $4 plus 3.2 MTok × $20 is **$160**.

Total: about **$577 a month**. For comparison, sending *every* ticket through the larger model at 3,000 input and 400 output tokens would cost 300 MTok × $4 plus 40 MTok × $20, which is **$2,000**. (Illustrative, and ignoring caching, thinking tokens and retries, all of which would move both numbers.)

But the number that matters is not the price, it is whether the tiers *work*. So you would run the eval on each step, check that the classifier's pass rate holds at the small model, measure the human edit rate on drafts, track the escalation rate, watch the cache hit rate, and keep a thumbs-down button that feeds failures back into the dataset. The architecture is easy. The discipline is the product.

---

# The Honest Caveats

- **Prices and models move quickly.** Everything above was accurate to Anthropic's pricing page on 21 November 2026. Look at the current page before you budget.
- **The measured figures are directional.** They come from Anthropic's own benchmarks, they are model- and task-specific, and they will not transfer perfectly to your workload. Run your own evals.
- **Some of the routing and cascade guidance comes from third-party engineering blogs**, several from vendors who sell routing products. I verified the arithmetic, not their claimed savings, and I found no independently verified company case study.
- **The product examples are secondhand.** Companies do not publish their model choices, so I limited myself to publicly described behaviour and marked what I could not confirm.
- **The observability standards are still settling**, so tooling choices made today may need revisiting.
- **The "future of products" section is opinion.** It is informed by what is shipping now, but it is a forecast, not a fact.
- **I used Claude models for the numbers** because those were the prices I could verify from the source. The method, though (instrument, evaluate, tier, measure), is provider-agnostic.

---

# Final Thoughts

Using AI to write your software makes you faster. Putting AI inside your software changes **what you are building and how you operate it**. The leverage is real: a $350 feature that replaces hours of manual work, an assistant that makes a product feel alive. So are the new failure modes: a bill that grows with usage, answers that are confidently wrong, a model that changes under you.

The engineers who thrive will be the ones who treat the model as **one unreliable, metered component in a system they design carefully**: choose the right tier for each job, measure everything, keep humans in the loop where the stakes are high, and improve with evidence instead of vibes.

If you take one action this week, make it this: pick one AI feature you run (or want to run), **log its tokens and cost per call**, and write **ten real examples** of what "correct" looks like. That small step puts you ahead of most teams. Everything else in this post is how you build on it.

---

*Further reading: Anthropic's [pricing](https://platform.claude.com/docs/en/about-claude/pricing) and [cost optimization guide](https://platform.claude.com/docs/en/about-claude/models/optimizing-for-cost-and-intelligence), [Langfuse, LangSmith and Phoenix compared](https://www.ajayk.sh/blog/llm-observability-tools-compared-2026/), a [guide to model routing in production](https://tianpan.co/blog/2026/04/18/model-routing-production-when-router-costs-more), [LLM-as-a-judge explained](https://www.langchain.com/resources/llm-as-a-judge), and the [Steptoe summary of the EU AI Act amendments](https://www.steptoe.com/en/news-publications/steptechtoe-blog/eu-ai-act-amendments-enter-into-force.html).*
