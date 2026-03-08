---
title: "RAG Explained: How to Give AI Access to Your Own Documents"
date: 2026-03-07
categories: [ai, tech]
tags: [rag, llm, vector-search, embeddings]
---

If you've ever tried to ask an AI about something specific to your company — an internal policy, a product manual, a technical spec — and got a vague or completely wrong answer, you've hit the core limitation that RAG was designed to fix.

Large language models are trained on a huge slice of the internet, but they have no idea about your Confluence pages, your customer contracts, your internal runbooks. And they can't be retrained every time something changes. So how do you bridge that gap?

That's exactly what **Retrieval-Augmented Generation** does.

---

## The Idea in Plain English

Instead of asking the model to *remember* your data (which it can't), you give it the relevant information *at the moment it needs it*. Before generating a response, the system finds the most relevant chunks from your knowledge base and injects them directly into the prompt. The model then reads that context and generates an answer grounded in your actual data.

It's like the difference between asking someone to answer from memory versus handing them the document and saying "read this first, then answer."

The name gives it away: you **retrieve** relevant content, **augment** the prompt with it, and then let the model **generate** an answer.

---

## How RAG Works: The Five Steps

Let's walk through the full pipeline. I'll use a concrete example throughout: an employee asking a question about the company's vacation policy.

### Step 1 — Ask

The user submits their question:

> "How many vacation days do I get after my first year?"

### Step 2 — Retrieve

The system converts the question into a search query and looks for the most relevant passages in the knowledge base. This is not a keyword search — it's a **semantic search** powered by embeddings. The system finds content that *means* the same thing, not just content that shares the same words.

Under the hood, the question gets converted into a vector (a list of numbers that represents its meaning), and the system finds the chunks in the vector store whose vectors are closest to it.

### Step 3 — Return

The most relevant passages come back from the vector store. In this example, the system retrieves a section from the employee handbook:

> *"Full-time employees accrue 15 days of paid vacation per year after completing 12 months of employment. Accrual begins on the employee's start date."*

### Step 4 — Augment

The system builds an enhanced prompt that combines the original question with the retrieved passages:

```
Context from the employee handbook:
"Full-time employees accrue 15 days of paid vacation per year after completing 
12 months of employment. Accrual begins on the employee's start date."

Question: How many vacation days do I get after my first year?
```

### Step 5 — Generate

The model reads the augmented prompt and produces a grounded answer. Something like: "After completing your first year, you're entitled to 15 days of paid vacation. Accrual starts from your hire date."

No hallucination. No generic answer. The model is reasoning directly from your document.

---

## What Makes RAG Work: Embeddings and Vector Stores

The magic of step 2 deserves a bit more explanation because it's the heart of the whole system.

**Embeddings** are numerical representations of text. When you index a document, you break it into small chunks (usually a few hundred tokens each) and run each chunk through an embedding model. The result is a vector — a point in a high-dimensional space where semantically similar texts end up close together.

"How many vacation days do I have?" and "What is the PTO accrual policy?" will have very similar vectors even though they share almost no words. This is what makes semantic search so much more powerful than keyword matching.

**Vector stores** are databases optimized for storing and searching these vectors. When you query, they find the nearest neighbors to your query vector incredibly fast. Popular options include Pinecone, Weaviate, Qdrant, pgvector (if you're already on Postgres), and ChromaDB for local development.

A minimal indexing pipeline looks like this:

```python
import chromadb
from openai import AsyncOpenAI

client = AsyncOpenAI()
vector_store = chromadb.Client()
collection = vector_store.create_collection("company_docs")

async def index_document(doc_id: str, text: str):
    # 1. chunk the text
    chunks = chunk_text(text, chunk_size=512, overlap=64)
    
    # 2. embed each chunk
    response = await client.embeddings.create(
        model="text-embedding-3-small",
        input=chunks
    )
    embeddings = [item.embedding for item in response.data]
    
    # 3. store in vector DB with metadata
    collection.add(
        ids=[f"{doc_id}-{i}" for i in range(len(chunks))],
        embeddings=embeddings,
        documents=chunks,
        metadatas=[{"document_id": doc_id, "chunk_index": i} for i in range(len(chunks))]
    )
```

And a retrieval query:

```python
async def retrieve(query: str, top_k: int = 5) -> list[str]:
    # embed the query
    response = await client.embeddings.create(
        model="text-embedding-3-small",
        input=[query]
    )
    query_embedding = response.data[0].embedding
    
    # find nearest chunks
    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=top_k
    )
    return results["documents"][0]  # list of relevant passages
```

---

## Real Cases Where RAG Shines

RAG is not the right tool for everything. But there's a category of problems where it's genuinely the best approach, and once you recognize the pattern, you'll see it everywhere.

**Internal company knowledge** — Your team has hundreds of Confluence pages, Notion docs, onboarding guides, process documentation. Instead of people digging through folders or pinging colleagues on Slack, a RAG-powered assistant can answer "what's our incident response process?" or "what are the rules around expensing software?" with references to the actual source.

**Customer support** — You have a product with a large FAQ, release notes, and support documentation. RAG lets a support bot answer customer questions accurately based on your real docs. The model can even return the specific article, so the customer can verify — and trust the answer more.

**Legal and compliance** — A legal team has thousands of contracts, regulatory documents, and internal policies. RAG lets them search semantically across all of it. "Find all clauses related to data retention in our vendor agreements" becomes actually answerable, with references to the exact sections. This is one of the most common enterprise RAG use cases right now.

**Research and analysis** — You're working with a large collection of academic papers, earnings reports, or technical specs. RAG lets an analyst ask questions across all of them without reading everything manually. The system finds what's relevant and the model synthesizes it.

**Developer documentation** — A large codebase with design documents, architecture decision records, and API docs is a perfect RAG use case. Developers can ask "why did we choose this database?" or "what are the retry semantics for this service?" and get answers grounded in the actual ADRs and specs.

The common thread across all of these: **a large body of text-based knowledge that changes occasionally, where you need the model to reason over it accurately with references**.

---

## What Happens When a Document Changes?

This is the question people forget to ask when they first build a RAG system. You index your documents, everything works great, and then... someone updates the employee handbook. Or the product team rewrites the API docs. Or a new compliance policy replaces the old one.

If you don't handle this, your RAG pipeline keeps serving stale answers based on outdated chunks. And the scary part is that the model will do it confidently, with no indication that the source has changed.

### The core operation: re-chunking and re-embedding

When a document is updated, you can't just append new content to the vector store. The old chunks are still there and will compete with the new ones during retrieval. You need to **delete the old chunks and re-index the updated document from scratch**.

```
Document updated
        ↓
Detect the change (webhook, polling, manual trigger)
        ↓
Delete all existing chunks for that document from the vector store
        ↓
Re-chunk the updated document
        ↓
Re-embed the new chunks
        ↓
Store in the vector store with updated metadata
```

The key is having a `document_id` in your chunk metadata from day one. If you track which chunks belong to which document, deleting all chunks for a given source becomes a single filter operation:

```python
async def reindex_document(doc_id: str, new_content: str):
    # Step 1: remove all old chunks for this document
    collection.delete(where={"document_id": doc_id})

    # Step 2: re-chunk the new content
    chunks = chunk_text(new_content, chunk_size=512, overlap=64)

    # Step 3: embed and store
    response = await client.embeddings.create(
        model="text-embedding-3-small",
        input=chunks
    )
    embeddings = [item.embedding for item in response.data]

    collection.add(
        ids=[f"{doc_id}-{i}" for i in range(len(chunks))],
        embeddings=embeddings,
        documents=chunks,
        metadatas=[{"document_id": doc_id, "chunk_index": i} for i in range(len(chunks))]
    )
```

### How to detect that a document changed

This depends on where your documents live. A few patterns:

**Webhooks** — Notion, Confluence, and Google Drive all send webhook events when content is updated. This is the most reactive approach: a document changes, you get notified immediately, you re-index right away. Minimal staleness.

**Polling** — A scheduled job (cron) checks documents for changes by comparing a hash of the content or a `last_modified` timestamp against what you stored at index time. Simple to implement, but you have a staleness window equal to your polling interval. A nightly run is usually fine for internal documentation.

**Manual trigger** — Your team explicitly triggers re-indexing when they publish an update. Works fine for small knowledge bases where updates are intentional and infrequent. Doesn't scale, and it depends on people remembering.

For most internal tools, a combination of webhooks (when available) plus a nightly full re-index sweep works well. The sweep catches anything the webhooks missed.

### Partial updates: keep it simple

Sometimes only a small part of a document changes — a single paragraph, a table, a version number. Should you re-index just the changed section?

Practically, **re-indexing the whole document is almost always the right call**. Partial updates are tricky to get right (chunk boundaries shift, context between chunks changes), and the cost of re-embedding a few hundred chunks is negligible. Keep it simple.

The one exception is very large documents — a 500-page manual, for example. In that case, split the document into logical sections at index time (chapters, major headings) and track each section as an independent unit. Then a change to chapter 3 only triggers a re-index of chapter 3, not the whole book.

### Versioning for compliance use cases

One thing that comes up in legal and compliance contexts: what if you need to answer a question about what the policy *was* at a specific point in time?

This requires a versioned index. Instead of deleting old chunks on re-index, you keep them tagged with a version or timestamp, and at query time you filter by version. Most vector stores support metadata filtering, so this is doable — but it adds storage cost and query complexity. Only go down this path if you genuinely need historical queries.

For most use cases, keep the index in sync with the latest version and rely on your CMS for history.

### The practical checklist

If you're building or maintaining a RAG system, make sure you have these in place:

- Every chunk has a `document_id` in its metadata
- There's a way to re-index a single document without rebuilding everything
- Some form of change detection exists — even a nightly cron is better than nothing
- Re-indexing runs are logged so you can audit knowledge base freshness
- If the use case is compliance-sensitive, chunks are versioned and timestamped

The worst failure mode is an AI confidently answering from an outdated policy document, with no one noticing. A little plumbing around document lifecycle goes a long way.

---

## RAG Is Not a Silver Bullet

A few things RAG doesn't do well, so you go in with realistic expectations:

**It doesn't handle questions that require aggregating many documents equally** — If the answer requires synthesizing information from 50 different sources with equal weight, retrieval will naturally favor some and miss others. RAG is better at finding the right passage than at reasoning across an entire corpus uniformly.

**It can still hallucinate if the retrieved context is ambiguous or incomplete** — RAG reduces hallucinations a lot, but doesn't eliminate them. If the retrieved chunk is misleading or the question doesn't have a good match in the knowledge base, the model can still produce a wrong answer.

**Chunking strategy matters more than people think** — Too small, and chunks lose context. Too large, and you lose precision in retrieval. The right chunking strategy is often domain-specific. A code file wants different chunking logic than a legal contract.

**Retrieval quality degrades with very large knowledge bases** — At some point, the signal-to-noise ratio in retrieval starts to hurt. At large scale, you need techniques like hybrid search (combining semantic and keyword), re-ranking, or query expansion to maintain quality.

---

## Final Thoughts

RAG is one of those patterns that feels simple at first but has real depth once you get into production. The basic pipeline is straightforward to build. Getting it to stay accurate, fresh, and reliable over time is where the actual engineering work is.

If you're starting out, my recommendation is to build the simplest possible version first: chunk your documents, embed them, store them in a vector DB, retrieve at query time, augment the prompt. Get that working end to end. Then worry about optimizing chunk size, adding re-ranking, handling document updates, and everything else.

The foundation is solid. And once you understand how RAG works, you'll also have a much clearer picture of when you need something more — like MCP — to go beyond reading documents and actually interact with live systems.

---

*LlamaIndex and LangChain both have excellent documentation for building RAG pipelines in Python. For vector stores, ChromaDB is great for local development, and Qdrant or pgvector are solid choices for production. OpenAI's `text-embedding-3-small` is a good default embedding model for most use cases.*
