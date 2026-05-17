# 🧠 The Complete LLM × RAG × Agents Knowledge Base

---

## 📐 How This Guide Is Structured

```
LAYER 0 → What is an LLM?
LAYER 1 → How LLMs "know" things (Training vs. Retrieval)
LAYER 2 → Embeddings & Vector Search
LAYER 3 → RAG (Retrieval-Augmented Generation)
LAYER 4 → Advanced RAG (Chunking, Reranking, HyDE, etc.)
LAYER 5 → Agents & Tool Use
LAYER 6 → MCP (Model Context Protocol)
LAYER 7 → Agentic RAG (Agents + RAG combined)
LAYER 8 → Interview Cheat Sheet
```

---

# LAYER 0 — What Is an LLM?

## Definition
A **Large Language Model (LLM)** is a deep neural network trained on massive text corpora to **predict the next token** (word piece). Through this simple objective, it learns grammar, facts, reasoning, and world knowledge.

## Key Concepts

| Term | Plain Meaning |
|------|--------------|
| **Token** | A chunk of text (~¾ of a word on average). "Hello world" = 2 tokens |
| **Context window** | Max tokens the model can "see" at once (e.g., 128K for GPT-4o) |
| **Inference** | Running the model to generate output (not training) |
| **Temperature** | Controls randomness. 0 = deterministic, 1+ = creative/random |
| **Parameters** | The model's "weights" — numbers learned during training. GPT-3 has 175B |
| **Prompt** | The input you send to the model |
| **Completion / Response** | The model's output |

## How Text Generation Works (Simplified)

```
Input Prompt
     │
     ▼
[Tokenizer] → converts text to token IDs
     │
     ▼
[Transformer Layers] → attention mechanisms process relationships
     │
     ▼
[Logits] → probability score for every possible next token
     │
     ▼
[Sampling] → pick next token (greedy / top-k / top-p / temperature)
     │
     ▼
[Repeat] → auto-regressively generate token by token until stop
```

## The Problem With LLMs Alone
- Knowledge is **frozen at training cutoff** (e.g., doesn't know events after Sept 2024)
- Can **hallucinate** — confidently make up facts
- Cannot access **private/proprietary data** (your company's docs, your DB)
- Cannot take **actions** in the world

> This is exactly why RAG, Agents, and MCP exist.

---

# LAYER 1 — Training vs. Retrieval

## Two Ways an LLM Can "Know" Something

```
┌─────────────────────────────────┐    ┌──────────────────────────────────────┐
│     PARAMETRIC KNOWLEDGE        │    │       NON-PARAMETRIC KNOWLEDGE        │
│  (baked into model weights)     │    │   (fetched at runtime, not trained)   │
│                                 │    │                                        │
│  ✅ Fast                        │    │  ✅ Fresh, up-to-date                  │
│  ✅ No lookup needed            │    │  ✅ Can use private/custom data        │
│  ❌ Stale (training cutoff)     │    │  ✅ Source-citable, less hallucination │
│  ❌ Can hallucinate             │    │  ❌ Requires retrieval infrastructure  │
│  ❌ Can't use private data      │    │  ❌ Retrieval quality matters a lot    │
└─────────────────────────────────┘    └──────────────────────────────────────┘
         Fine-tuning lives here                    RAG lives here
```

## Fine-Tuning vs. RAG (Common Interview Question!)

| | Fine-Tuning | RAG |
|--|-------------|-----|
| **What it does** | Updates model weights on new data | Retrieves relevant context at query time |
| **Data freshness** | Static (requires retraining to update) | Dynamic (update DB, no retraining) |
| **Cost** | High (GPU hours) | Lower (indexing + inference) |
| **Hallucination** | Reduces domain errors but still hallucinates | Reduces hallucination with grounding |
| **Use case** | Teaching style/format/domain behavior | Answering questions from private docs |
| **Best for** | "Sound like a doctor" | "Answer from this hospital's manual" |

---

# LAYER 2 — Embeddings & Vector Search

## What Is an Embedding?

An **embedding** is a list of floating-point numbers (a vector) that represents the *meaning* of a piece of text in a high-dimensional space.

```
Text: "The cat sat on the mat"
            │
         [Embedding Model]
            │
            ▼
Vector: [0.21, -0.83, 0.54, 0.09, ..., 0.37]   ← 384 to 3072 numbers long
```

**Key insight**: Semantically similar texts have vectors that are **close together** in that space.

```
"dog" vector ────── close ─────── "puppy" vector
"dog" vector ─────────────── far ─────────────── "spaceship" vector
```

## How Similarity Is Measured

### Cosine Similarity (most common)
Measures the **angle** between two vectors. Range: -1 to 1 (1 = identical meaning).

```
          Vector B
         /
        / ← small angle = similar
       /
      /───────── Vector A

cos(θ) = (A · B) / (|A| × |B|)
```

### Other Metrics
- **Dot Product** — fast, but sensitive to vector magnitude
- **Euclidean Distance** — measures straight-line distance (L2)

## Popular Embedding Models

| Model | Dims | Notes |
|-------|------|-------|
| `text-embedding-3-small` (OpenAI) | 1536 | Fast, cheap |
| `text-embedding-3-large` (OpenAI) | 3072 | More accurate |
| `all-MiniLM-L6-v2` (HuggingFace) | 384 | Open-source, fast |
| `bge-large-en-v1.5` (BAAI) | 1024 | Strong open-source |
| `nomic-embed-text` | 768 | Good open alternative |

## Vector Databases

A **vector database** stores embeddings and allows fast similarity search (ANN = Approximate Nearest Neighbor).

```
Query: "What is the refund policy?"
   │
   ▼ embed query → [0.12, 0.87, ...]
   │
   ▼ search vector DB for nearest vectors
   │
   ▼ return top-k most similar documents
```

### Popular Vector DBs

| DB | Type | Notes |
|----|------|-------|
| **Pinecone** | Cloud-managed | Easy to use, scalable |
| **Weaviate** | Open-source / Cloud | Feature-rich, hybrid search |
| **Qdrant** | Open-source | Rust-based, fast |
| **Chroma** | Open-source | Great for local dev/prototyping |
| **Milvus** | Open-source | Enterprise-grade |
| **pgvector** | PostgreSQL extension | If you already use Postgres |
| **FAISS** | Library (Facebook) | In-memory, no persistence |

---

# LAYER 3 — RAG (Retrieval-Augmented Generation)

## What Is RAG?

**RAG = Retrieval-Augmented Generation**

Instead of asking an LLM to answer from memory, you first **retrieve relevant documents** and then give them to the LLM as context.

```
User Query: "What's our company's parental leave policy?"
                │
                ▼
         [Embed the query]
                │
                ▼
     [Search vector DB for similar docs]
                │
                ▼
    [Get top-k relevant document chunks]
                │
                ▼
  [Inject into LLM prompt as context]
                │
                ▼
    [LLM answers grounded in real docs]
                │
                ▼
     "Employees get 16 weeks paid leave..."
```

## RAG Pipeline — Full Flow

```
=== INDEXING PHASE (offline, done once) ===

[Raw Documents: PDFs, Word, HTML, DB rows]
          │
          ▼
     [Document Loader]
          │
          ▼
     [Text Splitter / Chunker]
          │
          ▼
     [Embedding Model]  →  [Vector DB]
                              (stores chunks + their embeddings)


=== QUERY PHASE (online, per user question) ===

[User Question]
          │
          ▼
     [Embed Question]
          │
          ▼
    [Vector DB Search]
          │
          ▼
    [Top-K Chunks Retrieved]
          │
          ▼
    [Prompt Construction]
    "Answer this question using the context below:
     Context: {chunk1} {chunk2} {chunk3}
     Question: {user_question}"
          │
          ▼
       [LLM]
          │
          ▼
     [Final Answer]
```

## Chunking Strategies

Chunking = splitting long documents into smaller pieces that fit in context.

| Strategy | Description | Best For |
|----------|-------------|----------|
| **Fixed-size** | Split every N characters | Simple, fast |
| **Sentence-based** | Split on sentence boundaries | General text |
| **Recursive** | Try paragraph → sentence → word until fits | Default choice (LangChain default) |
| **Semantic** | Group sentences by meaning shift | High-quality retrieval |
| **Document-aware** | Respect headers, sections, markdown | Structured docs |

### Chunk Size Trade-offs

```
Small chunks (100-300 tokens)
  ✅ More precise retrieval
  ❌ Less context per chunk (loses surrounding meaning)
  ❌ More chunks = more storage + slower search

Large chunks (500-1500 tokens)
  ✅ More context preserved
  ❌ May retrieve irrelevant parts of a big chunk
  ❌ Uses more context window in LLM
```

**Rule of thumb**: Start with 512 tokens, 10-15% overlap between chunks.

---

# LAYER 4 — Advanced RAG

## The Problem With Naive RAG

```
User: "Compare Q3 and Q4 revenue performance"

Naive RAG retrieves: Just Q3 doc OR just Q4 doc
Result: Incomplete answer ❌

Better: Retrieve BOTH, then synthesize ✓
```

## B25 / BM25 — Keyword Search (Lexical Retrieval)

**BM25** (Best Match 25) is a classic **keyword-based ranking algorithm** — it's NOT neural.

```
How BM25 scores a document for a query:
- Counts term frequency (TF) in the document
- Penalizes very long documents (length normalization)
- Rewards rare words across the corpus (IDF — Inverse Document Frequency)
```

### BM25 vs. Vector Search

| | BM25 | Vector Search |
|--|------|---------------|
| **Type** | Keyword / lexical | Semantic / neural |
| **Needs** | Exact/close word matches | Meaning matches |
| **Example** | Finds "SQL injection" if those words exist | Finds "database attack" even without the words |
| **Speed** | Very fast (inverted index) | Slower (ANN search) |
| **Best for** | Precise technical terms, IDs, codes | Conceptual/paraphrased queries |

### Hybrid Search = BM25 + Vector Search

```
Query → BM25 Search ─────────┐
                              ├──► [Fusion / RRF] ──► Top results
Query → Vector Search ────────┘
```

**Reciprocal Rank Fusion (RRF)**: Combines rankings from both. A document ranked #2 in both methods scores higher than one ranked #1 in only one.

## Reranker

After retrieval gives you top-20 candidates, a **reranker** re-scores them with a more powerful (but slower) model and returns the truly best top-3 or top-5.

```
RETRIEVAL (fast, approximate)
   Initial query → vector search → top 20 candidates
                      │
                      ▼
RERANKING (slow, accurate)
   Cross-encoder model scores each (query, document) pair
                      │
                      ▼
   Reordered: truly most relevant docs bubble to top
                      │
                      ▼
   Top 3–5 → LLM context
```

### Why Reranking Matters

```
Without reranker: [doc3, doc7, doc1, doc15, ...]  ← approximate order
With reranker:    [doc7, doc1, doc3, ...]          ← true relevance order

LLM only sees top 3 → if those 3 are wrong, answer is wrong
```

### Bi-encoder vs. Cross-encoder

| | Bi-encoder (retrieval) | Cross-encoder (reranking) |
|--|------------------------|--------------------------|
| **Process** | Embed query and doc separately | Process query + doc together |
| **Speed** | Fast (pre-computed doc vectors) | Slow (computed per pair) |
| **Accuracy** | Good | Excellent |
| **Use** | First-stage retrieval | Second-stage reranking |

### Popular Reranker Models

- `cross-encoder/ms-marco-MiniLM-L-6-v2` (HuggingFace)
- `bge-reranker-large` (BAAI)
- Cohere Rerank API
- `mixedbread-ai/mxbai-rerank-large-v1`

## HyDE (Hypothetical Document Embeddings)

```
User asks: "What causes inflation?"
               │
               ▼
  LLM generates a FAKE answer first:
  "Inflation is caused by..."
               │
               ▼
  Embed that fake answer (not the question)
               │
               ▼
  Search for real docs similar to the fake answer
               │
               ▼
  Retrieve more relevant docs than searching with question alone
```

**Why**: The embedding of a hypothetical answer is closer to real answers than the embedding of a question.

## Multi-Query RAG

```
User: "Tell me about the health effects of coffee"
               │
               ▼
  LLM rewrites query into 3-5 different queries:
  - "cardiovascular effects of caffeine"
  - "coffee and sleep quality research"
  - "antioxidants in coffee benefits"
               │
               ▼
  Run ALL queries → combine unique results → rerank → answer
```

## Parent-Child Chunking (Small-to-Big Retrieval)

```
Retrieve small chunks (precise match)
         │
         ▼
But send the PARENT (larger chunk) to LLM (more context)

[Parent Doc]
  [Child Chunk A] ← retrieved (precise)
  [Child Chunk B]
  [Child Chunk C]

LLM receives: [Parent Doc]  ← more context, better answer
```

## Contextual Compression

Instead of sending raw chunks, ask an LLM to extract only the relevant sentences first:

```
Retrieved chunk: 500 tokens of a legal document
                    │
                    ▼
          [Compression LLM]
          "Extract only the sentences relevant to: {query}"
                    │
                    ▼
          Compressed: 50 tokens of the exact relevant part
                    │
                    ▼
          Fits 10x more context in LLM window!
```

## RAG Evaluation Metrics

| Metric | What It Measures |
|--------|-----------------|
| **Faithfulness** | Does the answer contradict the retrieved context? |
| **Answer Relevance** | Does the answer address the question? |
| **Context Precision** | Are retrieved chunks actually relevant to the question? |
| **Context Recall** | Did retrieval find all necessary information? |

**Frameworks**: RAGAs, TruLens, ARES

---

# LAYER 5 — Agents & Tool Use

## What Is an LLM Agent?

An **agent** is an LLM that can:
1. **Decide** what action to take
2. **Use tools** (search web, run code, query DB, call APIs)
3. **Observe** the result
4. **Loop** until the task is complete

```
User: "Find the top 5 AI companies by valuation, put them in a spreadsheet"

  ┌─── Agent Loop ────────────────────────────────┐
  │                                               │
  │  [LLM thinks] → "I need to search the web"   │
  │       │                                       │
  │  [Tool: web_search("top AI companies 2024")]  │
  │       │                                       │
  │  [Observe result] → got list                  │
  │       │                                       │
  │  [LLM thinks] → "Now create spreadsheet"      │
  │       │                                       │
  │  [Tool: create_file("companies.xlsx")]        │
  │       │                                       │
  │  [Observe] → "File created" → DONE ✓         │
  └───────────────────────────────────────────────┘
```

## ReAct Pattern (Reason + Act)

The most common agent architecture:

```
Thought: "I need to find the current price of AAPL"
Action: search("AAPL stock price today")
Observation: "AAPL is trading at $189.23"
Thought: "Now I can answer the question"
Action: finish("AAPL is currently $189.23")
```

**Repeat Thought → Action → Observation until done.**

## Tool Calling (Function Calling)

Modern LLMs (GPT-4, Claude, Gemini) support structured tool calling:

```json
// You give the LLM a list of available tools:
{
  "tools": [
    {
      "name": "get_weather",
      "description": "Get current weather for a city",
      "parameters": {
        "city": { "type": "string" }
      }
    }
  ]
}

// LLM returns a structured call:
{
  "tool": "get_weather",
  "arguments": { "city": "Dubai" }
}

// You execute it, return result, LLM continues
```

## Types of Memory in Agents

```
┌──────────────────────────────────────────────────┐
│                  AGENT MEMORY                    │
│                                                  │
│  IN-CONTEXT (short-term)                         │
│  └── Conversation history in the prompt          │
│      Limited by context window size              │
│                                                  │
│  EXTERNAL (long-term)                            │
│  └── Vector DB storing past interactions         │
│      Summarized conversation logs                │
│      Entity memory (facts about the user)        │
│                                                  │
│  EPISODIC                                        │
│  └── "Last time user asked about X, they meant Y"│
│                                                  │
│  SEMANTIC                                        │
│  └── General world knowledge (from training)     │
└──────────────────────────────────────────────────┘
```

## Multi-Agent Systems

Multiple specialized agents collaborating:

```
User Request
     │
     ▼
[Orchestrator Agent]
  ├──► [Research Agent] → searches web, papers
  ├──► [Code Agent]     → writes & runs code
  ├──► [Analysis Agent] → interprets results
  └──► [Writer Agent]   → formats final output
     │
     ▼
Final Response
```

**Frameworks**: LangGraph, CrewAI, AutoGen, LlamaIndex Workflows

---

# LAYER 6 — MCP (Model Context Protocol)

## What Is MCP?

**Model Context Protocol** is an open standard (by Anthropic) that defines a **universal way for LLMs/agents to connect to external tools and data sources**.

Think of it as **USB for AI** — a standardized connector so any AI can plug into any tool without custom integration code.

## The Problem MCP Solves

```
Before MCP:
  Claude ──── custom code ────► GitHub
  Claude ──── custom code ────► Google Drive
  Claude ──── custom code ────► Slack
  Claude ──── custom code ────► Database
  (Each integration is bespoke, duplicated effort)

After MCP:
  Claude ──── MCP protocol ────► [MCP Server: GitHub]
  Claude ──── MCP protocol ────► [MCP Server: Google Drive]
  Claude ──── MCP protocol ────► [MCP Server: Slack]
  Claude ──── MCP protocol ────► [MCP Server: Database]
  (One standard to rule them all)
```

## MCP Architecture

```
┌─────────────────────────────────────────────────────────┐
│                     MCP HOST                            │
│  (Claude Desktop, Claude.ai, your AI app)               │
│                                                         │
│  ┌──────────┐    MCP Protocol     ┌──────────────────┐  │
│  │ LLM/AI   │◄───────────────────►│   MCP Client     │  │
│  │ (Claude) │                     │  (connector)     │  │
│  └──────────┘                     └────────┬─────────┘  │
│                                            │            │
└────────────────────────────────────────────┼────────────┘
                                             │ MCP Protocol
                                    ┌────────▼─────────┐
                                    │   MCP Server     │
                                    │  (e.g., GitHub)  │
                                    └────────┬─────────┘
                                             │
                                    ┌────────▼─────────┐
                                    │   Actual Tool    │
                                    │  (GitHub API)    │
                                    └──────────────────┘
```

## What MCP Servers Expose

MCP servers can expose three types of things:

| Type | Description | Example |
|------|-------------|---------|
| **Tools** | Functions the LLM can call | `create_issue()`, `send_email()` |
| **Resources** | Data/files the LLM can read | A file, a database row, a URL |
| **Prompts** | Reusable prompt templates | "Summarize this PR" template |

## MCP Transport Types

| Transport | Use Case |
|-----------|---------|
| **stdio** | Local processes (runs on your machine) |
| **SSE (HTTP)** | Remote servers over the internet |

## Why MCP Matters

- **Ecosystem**: Any app that supports MCP can use any MCP server (100+ available)
- **Security**: You control what tools the AI can access
- **Composability**: Chain multiple MCP servers in one agent
- **Portability**: Build once, works with Claude, GPT, any MCP-compatible model

---

# LAYER 7 — Agentic RAG

## What Is Agentic RAG?

**Agentic RAG** = RAG + Agent capabilities. Instead of a fixed "embed → retrieve → answer" pipeline, the agent **decides how and when to retrieve**, can **use multiple retrieval strategies**, and **iterates until confident**.

```
Naive RAG:
  Question → Retrieve → Answer  (one shot, fixed pipeline)

Agentic RAG:
  Question → [Agent thinks] → Decide retrieval strategy
                            → Retrieve from source A
                            → [Not enough info] → Retrieve from source B
                            → [Still uncertain] → Search web
                            → [Enough context] → Generate answer
                            → [Check answer quality] → Refine if needed
                            → Final Answer
```

## Key Patterns in Agentic RAG

### 1. Adaptive Retrieval
Agent decides *whether* to retrieve at all:
```
Question: "What is 2+2?"  → No retrieval needed, answer directly
Question: "What's in our Q4 report?"  → Retrieval needed
```

### 2. Multi-Step Retrieval (Iterative RAG)
```
Step 1: Retrieve overview docs
Step 2: Based on overview, retrieve specific details
Step 3: Synthesize across both sets
```

### 3. Self-Reflective RAG (SELF-RAG)
The model critiques its own output:
```
Answer generated → "Is this supported by the retrieved context?"
                → "Is this relevant to the question?"
                → If NO → retrieve again → regenerate
```

### 4. Corrective RAG (CRAG)
Evaluate retrieved docs quality:
```
Retrieved docs → [Relevance Evaluator]
    │
    ├── RELEVANT → use them
    ├── AMBIGUOUS → supplement with web search
    └── IRRELEVANT → discard, do web search only
```

### 5. Graph RAG
Instead of flat chunks, build a knowledge graph:
```
[Entity: "Apple Inc."] ──[makes]──► [Entity: "iPhone"]
                       ──[CEO]───► [Entity: "Tim Cook"]
                       ──[rival]──► [Entity: "Samsung"]

Query: "What phone does the iPhone's maker's CEO talk about?"
Graph traversal finds answer across connected entities
```

## Agentic RAG Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AGENTIC RAG SYSTEM                       │
│                                                             │
│   User Query                                                │
│       │                                                     │
│       ▼                                                     │
│  ┌──────────────┐     ┌────────────────────────────────┐   │
│  │  Planner /   │────►│         Tool Belt               │   │
│  │  Orchestrator│     │  ┌────────────────────────┐    │   │
│  │  (LLM)       │◄────│  │ vector_search(query)   │    │   │
│  └──────────────┘     │  │ bm25_search(keywords)  │    │   │
│         │             │  │ web_search(query)       │    │   │
│         │             │  │ sql_query(statement)    │    │   │
│         ▼             │  │ rerank(docs, query)     │    │   │
│  ┌──────────────┐     │  │ summarize(docs)         │    │   │
│  │   Memory     │     │  └────────────────────────┘    │   │
│  │  (context +  │     └────────────────────────────────┘   │
│  │  past steps) │                                           │
│  └──────────────┘                                           │
│         │                                                   │
│         ▼                                                   │
│     Final Answer                                            │
└─────────────────────────────────────────────────────────────┘
```

---

# LAYER 8 — Complete Concept Map

## Everything Connected

```
                        ┌───────────────┐
                        │    USER       │
                        └───────┬───────┘
                                │
                                ▼
                        ┌───────────────┐
                        │    AGENT      │◄─────────────────┐
                        │ (LLM + logic) │                  │
                        └───────┬───────┘                  │
                                │ decides                  │
               ┌────────────────┼────────────────┐         │
               ▼                ▼                ▼         │
        ┌─────────┐      ┌─────────────┐  ┌──────────┐   │
        │ Direct  │      │     RAG     │  │  Tools   │   │
        │ Answer  │      │  Pipeline   │  │ via MCP  │   │
        └─────────┘      └──────┬──────┘  └────┬─────┘   │
                                │               │         │
                         ┌──────▼──────┐        │  result │
                         │  Retrieval  │        │◄────────┘
                         │             │
                    ┌────┴────┐   ┌────┴────┐
                    │ Vector  │   │  BM25   │
                    │ Search  │   │ Search  │
                    └────┬────┘   └────┬────┘
                         └──────┬──────┘
                                │ top-20
                                ▼
                         ┌─────────────┐
                         │  Reranker   │  ← cross-encoder scores each pair
                         └──────┬──────┘
                                │ top-3/5
                                ▼
                         ┌─────────────┐
                         │   Context   │  ← injected into LLM prompt
                         │   Window    │
                         └──────┬──────┘
                                │
                                ▼
                         ┌─────────────┐
                         │    LLM      │  ← generates grounded answer
                         └─────────────┘
```

---

# 📋 Interview Cheat Sheet

## Core Definitions (1-liner each)

| Term | One-Line Definition |
|------|---------------------|
| **LLM** | Neural net trained to predict next token on massive text data |
| **Token** | Smallest unit of text (~¾ word); LLMs process sequences of these |
| **Embedding** | Numerical vector representing meaning of text in high-dimensional space |
| **Vector DB** | Database optimized for similarity search on embeddings |
| **RAG** | Retrieve relevant docs at query time and give them to LLM as context |
| **Chunking** | Splitting documents into smaller pieces for indexing/retrieval |
| **BM25** | Classic keyword-based ranking algorithm using TF-IDF weighting |
| **Hybrid Search** | Combining BM25 (lexical) + vector (semantic) search with score fusion |
| **Reranker** | Cross-encoder model that reorders retrieved docs by true relevance |
| **HyDE** | Generate fake answer first, embed that to find better real matches |
| **Agent** | LLM that can plan, use tools, observe results, and loop to completion |
| **ReAct** | Agent loop pattern: Reason → Act → Observe → Repeat |
| **MCP** | Anthropic's open protocol: standard interface for LLMs to connect to tools/data |
| **Agentic RAG** | Agent that decides how/when to retrieve, using multiple strategies |
| **Fine-tuning** | Training a pre-existing model on new data to update its weights |
| **Cosine similarity** | Angle-based measure of vector closeness; 1 = identical meaning |
| **Cross-encoder** | Reranker model that processes query+doc together for precise scoring |
| **Bi-encoder** | Embedding model that encodes query and doc separately; used in retrieval |
| **Context window** | Max tokens an LLM can process in one call |
| **Hallucination** | LLM confidently generating false information |
| **Temperature** | Controls output randomness; 0 = deterministic, higher = more creative |

## Common Interview Questions & Answers

### Q: What's the difference between RAG and fine-tuning?
**A**: Fine-tuning updates model weights to internalize new knowledge — expensive, static, requires retraining to update. RAG retrieves fresh knowledge at runtime — cheaper, dynamic, update the DB and it's immediately available. Use fine-tuning for style/behavior changes, RAG for knowledge questions.

### Q: Why use a reranker? Isn't vector search good enough?
**A**: Vector search (bi-encoder) encodes query and documents independently for speed, making it approximate. A reranker (cross-encoder) sees query and document together, giving a much more accurate relevance score. The typical pattern: vector search gets top-20 fast, reranker sorts top-5 accurately — best of both worlds.

### Q: What is MCP and why does it matter?
**A**: Model Context Protocol is Anthropic's open standard for connecting AI models to external tools and data sources. Like USB standardized device connections, MCP standardizes how AI connects to tools — so you build an MCP server once and any compatible AI can use it, without custom integration code per model.

### Q: What are the main failure modes of RAG?
**A**: (1) Bad chunking — chunks split mid-thought, lose context. (2) Retrieval failure — relevant doc not in top-k. (3) Reranking failure — wrong docs promoted. (4) Context stuffing — LLM ignores relevant context buried in long prompt. (5) Out-of-corpus questions — answer isn't in the documents at all. Each needs its own fix.

### Q: What is agentic RAG vs. regular RAG?
**A**: Regular RAG is a fixed pipeline: embed → retrieve → answer, one shot. Agentic RAG gives an LLM control over the retrieval process — it can query multiple sources, retry on failure, evaluate retrieved quality, and iterate until it has enough info, making it far more robust for complex multi-hop questions.

### Q: What is BM25 and when do you use it over vector search?
**A**: BM25 is a keyword ranking algorithm based on term frequency and inverse document frequency. Use it when exact keyword matching matters — product codes, technical terms, proper nouns, legal language. Use vector search when meaning matters more than exact words. Best practice: use both in hybrid search with RRF fusion.

---

# 🗺️ Tech Stack Map (For Builders)

```
Task                     Popular Choices
─────────────────────────────────────────────────────
Document loading         LangChain, LlamaIndex, Unstructured
Chunking                 LangChain RecursiveTextSplitter, LlamaIndex
Embedding models         OpenAI, Cohere, HuggingFace sentence-transformers
Vector databases         Pinecone, Chroma, Qdrant, pgvector, Weaviate
BM25 search              Elasticsearch, BM25s (Python), Weaviate BM25
Hybrid fusion            RRF (manual), Weaviate, Qdrant
Rerankers                Cohere Rerank, bge-reranker, cross-encoder (HF)
LLM inference            OpenAI, Anthropic, HuggingFace, Ollama (local)
Agent frameworks         LangChain Agents, LangGraph, LlamaIndex, CrewAI
MCP servers              Claude Desktop + MCP ecosystem (100+ servers)
RAG evaluation           RAGAs, TruLens, ARES
Orchestration            LangGraph (stateful), Prefect, Airflow
```

---

