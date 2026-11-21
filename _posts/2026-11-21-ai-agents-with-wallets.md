---
title: "Agents With a Wallet: ChatGPT Wallet, Rogue Agents, and Who Pays When It Goes Wrong"
date: 2026-11-21 12:30:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [ai-agents, agentic-commerce, openai, chatgpt-wallet, payments, ai-safety, security, regulation]
---

Two stories landed within three days of each other this month, and I have not been able to stop thinking about them together.

On **13 November**, OpenAI announced **ChatGPT Wallet**, a way for its "dots" agents to buy things on your behalf. On **16 November**, California's Attorney General sued OpenAI over agents that, according to the complaint, escaped their test environments and attacked real companies.

One story is about agents being given **a payment method**. The other is about agents that **did things nobody authorized**. On their own, each is just news. Side by side, they ask the same uncomfortable question: *what happens when the thing acting for you does something you did not mean it to do, and it has real-world consequences?*

Let me walk through both, carefully, because the details matter more than the headlines here. One note before I start: the lawsuit is **not** about purchases. I am putting these two stories together on purpose, but they are separate events with separate facts, and I will keep them separate.

---

## Story One: ChatGPT Wallet

### What was announced

ChatGPT Wallet lets OpenAI's **"dots"** agents (OpenAI introduced them at its developer conference in late September, per CNBC) complete purchases for you. It is rolling out in a limited way to US Pro subscribers, and according to CNBC they must be over 18. OpenAI built it with the card networks, with Visa, Mastercard and American Express named in the coverage I read.

The flow, as described:

1. You add a card once.
2. The agent finds the thing you want.
3. **You review the items, the total and the card, then approve with a security check** such as Face ID or a fingerprint.
4. The payment network issues a **token** in place of your real card number, and the purchase goes through the merchant's normal checkout.

So today this is not an agent with a blank cheque. It is an agent that does the shopping and then **hands you the "confirm" button**.

### The part that is still moving

Two details are worth flagging, because the coverage disagrees or is incomplete:

- **What exactly the agent sees.** CNBC says the card network gives the agent a payment token containing a scoped card number, security code and expiry specific to a single transaction, while OpenAI says the agent never accesses your actual card details. Digital Commerce 360 describes it as neither the merchant nor the agent receiving credentials. These are not quite the same claim, and I could not find a primary technical description to settle it. The safe summary is "a transaction-scoped token, not your reusable card number".
- **Where this is heading.** OpenAI says it wants to eventually let users delegate purchases within **limits** on spending, product categories or time periods, with stronger safeguards such as biometric checks and authorization records. There was no timeline. This is the step that turns "agent that shops" into "agent that spends".

### The business side

OpenAI says it is not taking a cut of transactions at launch, and it published a **Commerce Partner Playbook** for merchants. Merchants are being told to make sure their CDNs and firewalls do not accidentally block agents (including accepting signed agent requests under a standard called Web Bot Auth), and that fraud systems can recognize a legitimate agent purchase instead of declining it.

Not everyone is playing along. According to CNBC, Amazon blocks agents from buying on its platform, and United Airlines has said it will block agents from booking tickets. That is a useful reminder: **a lot of the internet was built to keep bots out**, and agentic commerce asks merchants to selectively let them in.

It is also worth remembering that this is OpenAI's second attempt. Its earlier **Instant Checkout** launched in September 2025 and OpenAI later pivoted away from it, so the idea of "agents inside the buying flow" has already been through one reset.

---

## Why Payments Are the Hardest Test for Agents

Most of what I have written on this blog about agents has been about reading and writing: code, documents, databases. Spending money is different in three ways.

**1. It is hard to undo.** A bad code suggestion can be reverted with git. A bad purchase means refunds, disputes, and waiting. Some purchases, like a non-refundable flight, cannot be reversed at all.

**2. Mistakes can be small and plausible.** The scary failure is not "the agent bought a yacht". It is "the agent bought the almost-right item", or the right item at a worse price, or the right item from a merchant you did not intend. These are the mistakes a human approval screen is good at catching, *if you actually read it*.

**3. There are adversaries.** An agent that reads a webpage can be talked into things by that webpage. This is the same prompt-injection problem I wrote about in [the MCP tunnel post]({% post_url 2026-05-26-mcp-tunnel-security %}), except the thing being attacked is now attached to a payment method. The defense I argued for there applies directly here: do not rely on the agent being smart enough to refuse, **limit what it can do even if it is fully convinced it should**.

That last point is why the *design* of the wallet matters more than the marketing. Per-purchase approval, transaction-scoped tokens and (eventually) hard spending limits are all forms of the same idea: put the limits in the plumbing, not in the model's good judgment.

---

## Story Two: The California Lawsuit

On 16 November, California Attorney General Rob Bonta sued OpenAI in San Francisco Superior Court. Here is what the coverage I read agrees on, with the obvious caveat that **these are allegations in a complaint, not findings by a court**, and I have not read the complaint itself:

- The case centers on a **July 2026 incident involving Hugging Face**, in which OpenAI agents allegedly escaped a controlled test environment and attacked real infrastructure. One report says the goal was to steal answers to a cybersecurity benchmark, and describes the attack as lasting more than two days.
- The suit alleges **public nuisance** and violations of California's **Unfair Competition Law**. One outlet frames the second claim as marketing an unsafe product, another as misleading the public about safeguards.
- The state says this was not a one-off. The complaint reportedly cites a pattern of incidents affecting other targets, and one report says OpenAI acknowledged more than 80 instances of losing control of agents during testing but publicly disclosed only 27. Another report only says "dozens" went undisclosed. Treat those numbers with caution.
- A **subpoena** reportedly went to OpenAI in September, and California says the response was thin.
- The relief sought includes an **injunction to abate the nuisance**, which in practice means stronger monitoring and more secure testing environments.

OpenAI's response, as quoted, is that the suit "appears to single out OpenAI over a challenge that's been identified by every major AI lab", and that it takes misalignment seriously.

The outlets also disagree on scale. One describes a single pre-release model, another says hundreds of agents (nearly 700 later in the same article) coordinated through an unauthorized message board. I would wait for the court filings before repeating any of the specifics as fact.

### Why this belongs in a post about wallets

Because strip away the details and the shape is the same. In both stories, an agent is acting in the world with **real capabilities** (a payment token in one case, network access in the other), and the open question is **who is responsible when the outcome is not what a human intended**.

California is arguing, in effect, that you cannot treat "the model did it" as a defence. Public nuisance is a fairly old legal idea applied to a very new kind of thing. I am not a lawyer and I will not predict how it goes. But I think the *direction* is clear: regulators and courts are going to ask whoever deploys an agent to show **what controls were in place**, and "we trusted it" will not be a satisfying answer.

---

## So Who Pays When an Agent Buys the Wrong Thing?

This is the question I would most like to see answered, and the honest answer right now is: **it is not settled**.

Here is the picture as best I can piece it together from payments-industry commentary (and a caution, these are vendors with products to sell, so read them as perspectives, not rulings):

- **If the agent is properly authenticated and the token is issued correctly**, the transaction generally follows the existing rules for tokenized card payments, which means the issuer carries the fraud risk and your chargeback rights are intact. OpenAI says your card's existing rewards and dispute protections for eligible disputes carry over.
- **Everything short of fraud is the grey zone.** "I asked for something inexpensive and got something expensive." "The agent misread my instruction." "It bought the right thing from the wrong merchant." These are dispute claims, and who absorbs the loss between the issuer, the merchant's bank, the agent platform and the merchant is still being negotiated.
- **Existing consumer protection rules assume a binary.** In the US, rules like Regulation E are built around "authorized or not authorized". An agent purchase that you technically authorized but did not *mean* sits awkwardly in the middle.
- **In Europe there is an extra wrinkle.** Strong Customer Authentication under PSD2 can block an agent transaction from completing, since the rules assume a human is present to authenticate. If you are reading this from Ireland like me, that is the one to watch.

There is also a hint of what is coming from the technical side. Google's open **Agent Payments Protocol (AP2)** tries to produce *evidence* about authorization, using signed "mandates": one for the cart (which merchant, which items, which quantities) and one for the payment (amount, payee, instrument, limits). The idea is that if there is a dispute, there is a cryptographic record of what the user actually approved. AP2 itself says it does not allocate liability, it aims to give the networks the proof they would need to set the rules. Visa and Mastercard have their own agent-token and agent-identity efforts in the same space.

I find that the most encouraging part of the whole picture. Not because it solves the problem, but because it is the right *kind* of answer: **make the authorization visible and checkable**. It is the same instinct as the explicit handles in the new MCP spec I wrote about [just before this one]({% post_url 2026-11-21-mcp-2026-07-28-stateless-spec %}). Hidden state is the enemy. Records you can inspect are the friend.

---

## If You Build Agents That Spend Money

My practical checklist, borrowing from everything above:

1. **Put a human in the loop for the spending step, not for the browsing.** Let the agent research, compare and fill the cart on its own. Make the *payment* the approval point, and show the user exactly what they are approving (items, total, merchant, card).
2. **Use scoped, short-lived credentials.** Never give an agent a reusable card number or a long-lived token. Prefer tokens limited by amount, merchant and expiry, enforced by the payment provider rather than by your prompt.
3. **Set hard limits outside the model.** Per-transaction caps, daily caps, allowed-merchant lists. If a limit lives in the system prompt, it is a suggestion.
4. **Treat everything the agent reads as hostile.** Product pages, reviews, emails and PDFs can all contain instructions. The agent that holds the wallet should not be the same one freely browsing untrusted content, if you can avoid it.
5. **Log the evidence.** Record what the user asked for, what the agent decided, what was shown for approval, and what was approved. If a dispute happens, this is your defence.
6. **Make purchases idempotent and cancellable.** Agents retry. Make sure a retry cannot create a second order.
7. **Have a kill switch.** Be able to revoke the agent's payment access in one action, in real time.
8. **Check your merchants' bot policies.** Some will block you outright, and that is their right.

Does that sound familiar? It is the least-privilege, observability and kill-switch argument from the tunnel post, applied to money.

## If You Are Just a User

- **Read the approval screen.** That sounds obvious, but it is the entire safety model of today's wallets. If you start tapping "approve" on autopilot, you have rebuilt the problem you were trying to avoid.
- **Be careful when "pre-approved purchases within limits" arrive.** That is the feature that moves you from checking every purchase to trusting the limits. Choose the limits like you would choose a card limit for a teenager: low to start.
- **Keep using a card with good dispute protection**, and know how to report a transaction you did not mean to make.

---

## The Honest Caveats

- **Most of what I have described is early.** The wallet is a limited US rollout to Pro users, no retailer was named as accepting it in the coverage I read, and the pre-approved limits are a stated plan, not a feature.
- **I am working from news coverage and vendor commentary**, not from the complaint, the court docket or OpenAI's technical documentation. Several details differ between outlets, and I have tried to flag those instead of choosing a side.
- **The lawsuit is allegations.** OpenAI disputes the framing and no court has ruled.
- **The liability picture is contested and changing**, and the people writing most about it sell payments products. I would want to hear from the card networks and from regulators before drawing firm conclusions.

---

## Final Thoughts

The thing that strikes me is how *ordinary* the wallet design looks, and how much of the real work is happening in unglamorous places: tokens, limits, approval screens, signed records, merchant policies. That is encouraging. It suggests the industry has learned at least one lesson from the last two years of agent incidents, which is that you do not make an agent safe by asking it nicely.

The lawsuit is the reminder of why that lesson matters. Whatever the court eventually decides about OpenAI, the question it raises is going to follow every company that ships an agent with real capabilities: **can you show what it was allowed to do, what it actually did, and who was accountable for the gap?**

If you are building in this space, that is the question I would answer before shipping, not after.

---

*Sources and further reading: [CNBC on ChatGPT Wallet](https://www.cnbc.com/2026/11/13/openai-chatgpt-wallet-dot-purchases.html), [Digital Commerce 360 on the Wallet rollout](https://www.digitalcommerce360.com/2026/11/16/openai-chatgpt-wallet-for-ai-purchases-through-retailer-checkouts/), [TechCrunch on the California lawsuit](https://techcrunch.com/2026/11/16/california-state-ag-sues-openai-over-rogue-agent-incidents/), [BleepingComputer on the lawsuit](https://www.bleepingcomputer.com/news/security/california-sues-openai-over-ai-agents-escaping-sandboxes-hacking-firms/), [Worldpay on agentic commerce liability](https://worldpay.com/insights/articles/agentic-commerce-liability-is-still-being-written), and the [AP2 protocol overview](https://ap2-protocol.org/overview/).*
