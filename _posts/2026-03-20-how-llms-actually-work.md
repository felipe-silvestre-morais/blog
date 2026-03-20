---
title: "How LLMs Actually Work: From Autocomplete to Intelligence"
date: 2026-03-20 15:00:00
author: Felipe Silvestre
categories: [ai, tech]
tags: [llm, machine-learning, transformers, neural-networks, ai-fundamentals]
---

I've been using LLMs every day for months now. Claude writes code with me, ChatGPT helps me debug, and I've built applications on top of these models. But here's something that made me uncomfortable: if someone asked me to *really* explain how they work, I'd stumble.

Sure, I could say "neural networks" and "transformers" and wave my hands about "training on massive datasets." But the honest truth is, I didn't deeply understand the mechanics. And I think a lot of us are in the same boat—we're building on top of something we treat like magic.

So I sat down and actually learned how these things work. Not at the PhD level, but enough to have a proper mental model. And here's what I found: **it's both simpler and weirder than I expected.**

Let me walk you through it. I'll start with the simplest possible explanation, then we'll layer in complexity until you have a real understanding of what's happening when you type a prompt into Claude or GPT-4.

---

## The Simplest Explanation: It's Fancy Autocomplete

Here's the core idea that unlocks everything else: **an LLM is predicting the next word.**

That's it. Really. When you type "The capital of France is" and the model responds with "Paris," it's not looking up facts in a database. It's predicting that "Paris" is the most likely next word based on patterns it learned from billions of examples.

Think about your phone's autocomplete. You type "I'm on my" and it suggests "way." It's learned that "way" often follows "I'm on my" from patterns in your texts and common phrases. An LLM is the same idea, but *vastly* more sophisticated.

The phone keyboard looks at a few words. An LLM can consider thousands of words of context. The phone has simple rules. An LLM has learned intricate patterns about grammar, facts, reasoning, style—all from examples.

But at the core: **next word prediction.** Everything else is about how we make that prediction incredibly good.

---

## Wait, But How Does It "Know" Anything?

Good question. This is where it starts to get interesting.

The model doesn't "know" things the way you and I know things. It doesn't have a fact database. Instead, during training, it reads billions of pages of text from the internet, books, papers, code—you name it. And in that text, patterns emerge.

If the model sees "The capital of France is Paris" thousands of times in different contexts, it learns a statistical relationship between those words. Not a *fact*, but a *pattern*. When you later ask "What's the capital of France?", it predicts the next words based on those learned patterns.

This is why LLMs can be confidently wrong—they're predicting likely continuations, not retrieving truth. If the training data had many examples saying "The capital of France is Lyon" (which is wrong), the model might predict that instead.

But here's what surprised me: **this simple mechanism of predicting the next word, when scaled up with enough data and computing power, starts to look like reasoning.**

The model learns grammar not because someone taught it grammar rules, but because grammatically correct continuations are more common in the training data. It learns to write code because it saw millions of code examples. It learns to reason through problems because it saw examples of people reasoning through problems.

Okay, so that's the concept. Now let's look at how this actually works mechanically.

---

## Tokens: How Text Becomes Numbers

Computers can't work with words directly. They need numbers. So the first thing that happens when you send a prompt to an LLM is that your text gets broken into **tokens**.

A token is roughly a word or part of a word. For example:
- "Hello" → one token
- "unusual" → one token
- "ChatGPT" → might be two tokens: "Chat" + "GPT"
- "🚀" → one token

The exact breakdown depends on the specific tokenizer the model uses, but the idea is consistent: text gets chopped into these atomic units.

Each token gets converted to a unique number (an ID). So "Hello world" might become `[15496, 995]`—just a list of integers the model can process.

This is important because **models have a token limit**, not a character limit. When Claude says it can handle "200K tokens," that's roughly 150K words of English text. But code or text with lots of special characters uses more tokens per word.

---

## Embeddings: Turning Tokens Into Meaning

Now we have a list of numbers representing tokens. But these numbers are just arbitrary IDs—token 15496 could be anything. The model needs to understand what these tokens *mean* and how they relate to each other.

This is where **embeddings** come in, and this is where things get fascinating.

An embedding converts each token into a long list of numbers (a vector). For models like GPT-4, this might be a list of 12,000+ numbers per token. But let's imagine a tiny 3-number embedding to understand the concept:

```
"king"   → [0.8, 0.3, 0.1]
"queen"  → [0.7, 0.3, 0.1]
"man"    → [0.9, 0.5, 0.0]
"woman"  → [0.8, 0.5, 0.0]
"cat"    → [0.1, 0.9, 0.7]
"dog"    → [0.1, 0.8, 0.7]
```

Notice the pattern? Words with similar meanings have similar numbers. "King" and "queen" are close. "Cat" and "dog" are close. But "king" and "cat" are far apart.

In the real embeddings with thousands of dimensions, these vectors capture incredibly subtle relationships. The famous example: if you take the vector for "king," subtract the vector for "man," add the vector for "woman," you get something very close to the vector for "queen."

**This is how the model "understands" meaning**—not through definitions, but through learned geometric relationships between words in a high-dimensional space.

The embedding values are learned during training. The model adjusts them so that tokens that appear in similar contexts end up with similar vectors.

---

## The Transformer: Where the Magic Happens

Now we have our input converted to embeddings—a sequence of high-dimensional vectors, one per token. The question is: how do we predict the next token?

This is where the **transformer architecture** comes in. I'm not going to dive into all the math (there are great deep-dives elsewhere), but here's the intuitive idea.

### Attention Is All You Need

The key innovation of transformers is the **attention mechanism**. Here's what it does:

For each token, the model looks at all the other tokens in the sequence and asks: "Which other tokens are most relevant for understanding *this* token in *this* context?"

An example. In the sentence:
> "The animal didn't cross the street because it was too tired."

When processing the word "it," the attention mechanism learns to pay strong attention to "animal"—not "street"—because in this context, "it" refers to the animal. The model learns this pattern from seeing millions of similar examples.

Now consider:
> "The animal didn't cross the street because it was too wide."

Now "it" refers to "street." Same word, different attention. The model learns to figure this out from context.

**Attention allows the model to dynamically decide which parts of the input are relevant for predicting the next word.** This is why LLMs can handle long-range dependencies and complex context in ways that older models couldn't.

### Multiple Layers

The transformer doesn't just do this once. It has many layers (GPT-4 reportedly has 120+ layers), each one building a progressively more sophisticated understanding.

- Early layers might capture simple patterns like "this word is a noun"
- Middle layers might identify relationships: "this noun is the subject of this verb"
- Later layers might capture abstract concepts: "this sentence is expressing skepticism about a claim"

Each layer refines the representation, passing its output to the next layer.

### The Output

After all these layers, we have a final representation for each position in the sequence. For the last position (where we want to predict the next token), the model outputs a probability distribution over all possible tokens:

```
"Paris":      0.32
"London":     0.08
"the":        0.05
"France":     0.03
...
[50,000 other tokens with lower probabilities]
```

The model typically picks the highest probability token, though it can also sample randomly based on these probabilities (controlled by "temperature" settings—higher temperature = more random/creative).

---

## Training: How It Learns These Patterns

So how does the model learn to do all this? Through **training** on massive amounts of text.

The process is surprisingly simple in concept:

1. **Show the model a sentence** from the training data, but hide the last word
2. **Ask it to predict** what the last word is
3. **Check if it got it right**—compare its prediction to the actual next word
4. **Update the model's weights** (all those numbers in the embeddings, attention mechanisms, etc.) to make the correct answer slightly more likely next time
5. **Repeat billions of times** with different examples

This is called "self-supervised learning" because the model generates its own training examples—every piece of text becomes many training examples (predict word 2 from word 1, predict word 3 from words 1-2, etc.).

After seeing enough examples, the model learns patterns:
- Grammatical structures appear again and again
- Factual relationships appear in consistent ways
- Reasoning patterns emerge from seeing people reason through problems
- Code patterns emerge from seeing millions of code examples

The model with the best predictions—the one that minimizes the difference between its predictions and the actual next words across billions of examples—is the one that has learned the richest patterns about language, knowledge, and reasoning.

### Fine-Tuning and RLHF

The base model trained this way is pretty good at predicting text, but it's not necessarily helpful or safe. It might complete "How do I..." with whatever it saw on the internet, good or bad.

So companies do additional training:

**Fine-tuning**: Train on carefully curated examples of good assistant behavior. Show it examples of helpful, harmless, and honest responses.

**RLHF (Reinforcement Learning from Human Feedback)**: Have humans rate different model responses, then use those ratings to train the model to prefer responses humans rate highly.

This is why Claude and ChatGPT feel like helpful assistants rather than just text prediction engines.

---

## What This Means In Practice

Understanding how LLMs work changes how I use them. A few things that clicked for me:

**1. They're pattern matchers, not reasoning engines** (mostly)

When Claude writes code, it's not *thinking* through the logic step-by-step like I do. It's generating a continuation that matches patterns it learned from millions of code examples. This is why it sometimes makes mistakes that seem "stupid"—it's predicting likely continuations, not checking correctness.

Though—here's the weird part—this pattern matching starts to look like reasoning at scale. The model learns patterns of how to reason through problems because it saw examples of reasoning.

**2. Context is everything**

Since the model is always predicting based on what came before, the way you structure your prompt massively impacts the output. If you include an example of the format you want, the model's pattern matching will likely continue in that format. This is why "few-shot prompting" (giving examples) works so well.

**3. Hallucinations make sense**

The model isn't retrieving facts—it's predicting likely continuations. If the pattern suggests a certain answer, it will generate that even if it's wrong. It's confidently generating what *should* come next based on its training, not checking if it's true.

This is why retrieval-augmented generation (RAG) helps—you explicitly put correct information in the context so the model's predictions are grounded in facts, not just patterns.

**4. Tokens matter for cost and performance**

Since models are priced per token and have token limits, being concise in your prompts isn't just good practice—it literally costs less and lets you fit more context. Likewise, asking for concise responses saves tokens in the output.

---

## Final Thoughts

I'll be honest: even after understanding all this, there's still something magical about it. The fact that "predict the next word" at massive scale produces something that can write poetry, debug code, reason through problems, and have coherent conversations—that's still remarkable.

We're not entirely sure why scaling up this simple mechanism produces such sophisticated capabilities. There are theories about "emergent abilities" appearing at certain scales, but it's still an active research area.

What I *do* know is that treating LLMs like magic is limiting. Understanding the mechanics—tokens, embeddings, attention, pattern matching—gives me better intuition for:
- How to structure prompts effectively
- When to trust the output and when to verify
- What tasks LLMs will excel at vs. struggle with
- How techniques like RAG and fine-tuning actually help

You don't need to understand every equation in the transformer paper. But having a mental model of "it's predicting the next token based on learned patterns, using attention to focus on relevant context, across many layers of abstraction"—that's enough to use these tools more effectively.

And that's what I wanted to share with you. Not a research paper, but a practical understanding that makes these tools less like magic and more like... very sophisticated pattern-matching machines that happen to be incredibly useful.

*Further reading: If you want to go deeper, I recommend Andrej Karpathy's "Let's build GPT" video for a code-level walkthrough, and "Attention Is All You Need" (the original transformer paper) if you want the real math. Jay Alammar's "The Illustrated Transformer" is also excellent for visual learners.*
