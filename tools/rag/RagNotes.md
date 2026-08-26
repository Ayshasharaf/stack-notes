# RAG Terms — Complete Guide
### From Zero to Interview-Ready

> **How to use this file**
> Read top to bottom. Each section builds on the last.
> Every term follows the same pattern: **What it is → Picture it → Why it matters → Your codebase**

---

## 🗺️ The Big Picture First

Before any terms — here is what RAG does in plain English:

```
YOUR DOCUMENTS
      │
      ▼
  [Cut into chunks]  ──►  [Turn into numbers]  ──►  [Save to index]
                                                           │
                                                    QUERY TIME
                                                           │
  USER ASKS A QUESTION  ──►  [Search index]  ──►  [Top chunks found]
                                                           │
                                                    [Send to LLM]
                                                           │
                                                    ✅ ANSWER
```

RAG = **find the right pages first, then ask the AI to read only those pages.**

---

# LEVEL 1 — Foundations
> *"What even is this thing?"*

---

## 1. RAG — Retrieval-Augmented Generation

```
WITHOUT RAG                    WITH RAG
──────────────────             ──────────────────────────────
User: "What is our            User: "What is our budget?"
       budget?"                       │
       │                       [Search your documents]
       ▼                              │
      LLM                      Found: "Q3 budget is $2.4M"
  (only knows its                     │
   training data)               LLM reads that passage
       │                              │
       ▼                              ▼
"I don't know your            "Your Q3 budget is $2.4M"  ✅
 internal budget"  ❌
```

**What:** A pattern where you retrieve relevant documents at query time and inject them into the LLM prompt — instead of relying on the model's training data alone.

**Why it exists:** LLMs have two problems:
1. **Knowledge cutoff** — they stop learning after training ends
2. **Hallucination** — they invent facts when they don't know something

RAG fixes both by giving the LLM fresh, specific documents to read before answering.

**In your codebase:** The entire `lumon/` project is RAG. The flow is `ingest → retrieve → generate`.

---

## 2. LLM — Large Language Model

```
Input tokens  ──►  [ Neural Network with billions of parameters ]  ──►  Output tokens
"What is 2+2"                                                           "4"
```

**What:** The AI model that generates text answers. In your project, this is configured via `LLM_MODEL` in `.env`.

**Why it matters for RAG:** The LLM is the *last* step — it reads the retrieved passages and writes the answer. It is not doing the searching. Keeping these roles separate is the core insight of RAG.

---

## 3. Document

**What:** A source file you want to make searchable — PDF, text file, Word doc, etc. Stored in `my_folder/`.

**Why:** This is the "database" your RAG system reads from. The quality of your answers is limited by the quality of your documents.

**In your codebase:** `lumon/ingest/chunking.py` loads documents. PDFs use `pypdf`; everything else uses LlamaIndex `SimpleDirectoryReader`.

---

## 4. Token

```
"Hello world, how are you?"
 ──────────────────────────
 "Hello"  " world"  ","  " how"  " are"  " you"  "?"
    1         2      3     4       5       6       7    = 7 tokens
```

**What:** The unit LLMs count text in. Not words — subword pieces. Roughly 1 token ≈ 0.75 words in English.

**Why it matters:** Every LLM has a **context window limit** (max tokens it can process). You must fit your question + retrieved passages inside this limit. This is literally why chunking exists.

**Quick math:** 512-token chunk ≈ 380 words ≈ one page of text.

---

## 5. Context Window

```
┌─────────────────────────────────────────────────┐
│  LLM Context Window  (e.g. 8,000 tokens)        │
│                                                  │
│  [System prompt ~200 tokens]                     │
│  [Retrieved passage 1 ~512 tokens]               │
│  [Retrieved passage 2 ~512 tokens]               │
│  [Retrieved passage 3 ~512 tokens]               │
│  [User question ~50 tokens]                      │
│  [Answer will go here ~500 tokens]               │
│                                                  │
│  Total used: ~2,286 / 8,000  ✅                  │
└─────────────────────────────────────────────────┘
```

**What:** The maximum number of tokens an LLM can process in one call — prompt + response combined.

**Why it matters:** If your retrieved passages + question exceed this limit, the call fails or content gets truncated. Your chunk sizes (128 child, 512 parent) are designed to fit several chunks comfortably.

**In your codebase:** `generate_answer()` in `lumon/pipeline/prompt.py` builds the message that must fit this window.

---

# LEVEL 2 — Indexing
> *"How documents get prepared for search"*

---

## 6. Chunking

```
ORIGINAL DOCUMENT (3,000 tokens — too big to search precisely)
┌──────────────────────────────────────────────────────────────┐
│ Introduction... Background... Budget section... Timeline...  │
│ Risk factors... Team members... Conclusions...               │
└──────────────────────────────────────────────────────────────┘
                          │
                    [Split into chunks]
                          │
          ┌───────────────┼───────────────┐
          ▼               ▼               ▼
    [Chunk 1]        [Chunk 2]        [Chunk 3]
   Introduction      Budget            Timeline
    ~128 tokens      ~128 tokens       ~128 tokens
```

**What:** Cutting documents into smaller pieces before indexing.

**Why not index the whole document?** Imagine you ask "What is the budget?" — if the answer is buried in page 4 of a 20-page document, the entire document gets retrieved and shoved into the LLM. It drowns in irrelevant text. Chunks let you pinpoint exactly the relevant passage.

**In your codebase:** `documents_to_nodes()` in `lumon/ingest/chunking.py`.

---

## 7. Chunk Size

```
SMALL CHUNKS (~128 tokens)          LARGE CHUNKS (~512 tokens)
────────────────────────────        ────────────────────────────
✅ Precise retrieval                ✅ More context per hit
✅ Less noise                       ✅ Better for the LLM to read
❌ May cut sentences mid-thought    ❌ May retrieve irrelevant parts
❌ LLM gets too little context      ❌ Fewer chunks fit in window
```

**What:** The token length of each chunk.

**The trade-off:** Small = precise search, poor LLM context. Large = good context, noisy search. Your project solves this with parent-child chunking (next term).

**In your codebase:** `config/settings.py` controls both sizes.

---

## 8. Parent-Child Chunking ⭐ (Key concept — will be asked)

```
ORIGINAL DOCUMENT
└── PARENT chunk (~512 tokens) ← sent to LLM for reading
    ├── child chunk 1 (~128 tokens) ← used for searching
    ├── child chunk 2 (~128 tokens) ← used for searching
    ├── child chunk 3 (~128 tokens) ← used for searching
    └── child chunk 4 (~128 tokens) ← used for searching

QUERY TIME:
  1. Search finds: child chunk 3  (precise match!)
  2. System loads: its parent     (full context for LLM)
  3. LLM reads the parent         (enough text to answer well)
```

**What:** A two-level chunking strategy. Small children for search, large parents for LLM context.

**Why it's elegant:** You get the precision of small chunks AND the context of large chunks — without having to choose one.

**Interview question:** *"Why not just use small chunks everywhere?"*
**Answer:** Small chunks retrieve well but often lack enough context for a good answer. Parent-child gives you best of both worlds.

**In your codebase:** `documents_to_nodes()` creates both. Parents saved to `storage/parents.json`. Children go into the vector index.

---

## 9. Embedding

```
TEXT                              VECTOR (simplified)
──────────────────                ─────────────────────────────
"budget forecast"    ──model──►   [0.23, -0.11, 0.87, 0.44, ...]
"financial plan"     ──model──►   [0.21, -0.09, 0.85, 0.46, ...]
"apple pie recipe"   ──model──►   [-0.67, 0.52, -0.23, 0.11, ...]

"budget forecast" and "financial plan" → very close  ✅
"budget forecast" and "apple pie"      → very far    ✅
```

**What:** Converting text into a list of numbers (a vector) that captures its meaning. Similar meaning → similar numbers.

**Why:** Numbers can be compared mathematically. This is how a search engine finds *"financial plan"* when you type *"budget forecast"* — even though the words don't match.

**In your codebase:** BGE embeddings model, set via `Settings.embed_model` in `lumon/ingest/indexer.py`.

---

## 10. Vector Index

```
         STORED EMBEDDINGS
         ┌─────────────────────────────────────────┐
         │  chunk_1: [0.23, -0.11, 0.87, ...]      │
         │  chunk_2: [0.21, -0.09, 0.85, ...]      │
         │  chunk_3: [-0.67, 0.52, -0.23, ...]     │
         │  ...thousands more...                    │
         └─────────────────────────────────────────┘
                          │
       QUERY: "budget forecast" → [0.22, -0.10, 0.86, ...]
                          │
              [Find nearest vectors]
                          │
                    Returns chunk_1, chunk_2
```

**What:** A database that stores embeddings and lets you search by similarity (nearest-neighbor search).

**Why:** You can't search millions of vectors by comparing each one manually — it's too slow. Vector indexes use clever math (like HNSW) to find nearest neighbors in milliseconds.

**In your codebase:** LlamaIndex manages this, persisted to `storage/` directory.

---

## 11. Tokenization

```
Word tokenization (wrong idea):
"don't"  →  ["don't"]   1 token

Subword tokenization (how LLMs actually work):
"don't"  →  ["don", "'t"]   2 tokens
"unhappiness"  →  ["un", "happiness"]   2 tokens
"RAG"  →  ["R", "AG"]   2 tokens
```

**What:** Converting text to tokens before processing. LLMs work in tokens, not words.

**Why you need to know this:** When you set chunk size to 128, that's 128 tokens — not 128 words. A 128-token chunk is about 96 words. Knowing this helps you predict how much content fits.

---

## 12. Persist / Index Storage

```
FIRST STARTUP (slow):
my_folder/ → [read all files] → [chunk] → [embed] → save to storage/ ✅

SUBSEQUENT STARTUPS (fast):
storage/ → [load saved index] → ready in seconds ✅

WITHOUT PERSIST:
Every startup → re-read → re-chunk → re-embed → 5+ minutes wait ❌
```

**What:** Saving the vector index to disk so it survives restarts.

**In your codebase:** `load_or_create_index()` in `lumon/ingest/indexer.py`. Detects if `storage/` exists — loads it if so, builds from scratch if not.

---

## 13. Incremental Sync

```
tracked_files.json:
{
  "report.pdf":  "md5: a3f8...",   ← file unchanged, skip ✅
  "budget.txt":  "md5: b7c2...",   ← file changed! re-index ⚡
  "notes.md":    "md5: 99e1...",   ← file unchanged, skip ✅
}
```

**What:** Only re-indexing files that have actually changed, detected via MD5 hash comparison.

**Why:** Re-indexing 1,000 documents because 1 changed is wasteful. MD5 is a fingerprint of file content — if it changes, the file changed.

**In your codebase:** `load_tracked()` / `save_tracked()` in `lumon/ingest/sync.py`.

---

# LEVEL 3 — Retrieval
> *"How the right passages get found"*

---

## 14. Cosine Similarity

```
         "budget forecast"  →  vector A
         "financial plan"   →  vector B

                    A · B
cos(θ) =  ──────────────────
           ||A|| × ||B||

Result: 0.94  (very similar — same direction)  ✅

         "budget forecast"  →  vector A
         "apple pie recipe" →  vector C

cos(θ) = 0.12  (not similar — different directions)  ❌
```

**What:** Measures the angle between two vectors. Score of 1.0 = identical meaning. Score of 0 = unrelated.

**Why cosine and not regular distance?** Cosine ignores vector magnitude (length) and only measures direction — which is what semantic similarity is. Two sentences can be different lengths but mean the same thing.

---

## 15. BM25 (Best Match 25) — Keyword Search

```
QUERY: "project deadline"

Document A: "The project deadline is March 15th"
  → "project" appears 1x  ✅
  → "deadline" appears 1x ✅
  → Short doc (high density) 📈
  → BM25 score: HIGH

Document B: "project project project project... deadline"  
  → "project" appears many times  
  → But BM25 penalizes repetition (diminishing returns)
  → BM25 score: MEDIUM

Document C: "apple pie recipe"
  → Neither word appears
  → BM25 score: 0
```

**What:** A ranking algorithm that scores documents by how well their keywords match the query. Accounts for term frequency, document length, and rarity of terms across all documents.

**Strength:** Exact term matches, technical jargon, product names, numbers.

**Weakness:** Synonyms. "Budget forecast" will NOT find "financial projections." Zero overlap = zero score.

**In your codebase:** `_bm25_hits()` in `lumon/retrieval/hybrid.py`. Uses stemmed tokens, stored in `storage/bm25_*.pkl`.

---

## 16. Semantic Search (Dense Retrieval)

```
QUERY: "budget forecast"  →  embed  →  [0.22, -0.10, 0.86, ...]
                                                │
                                    Search vector index
                                                │
                              Finds: "financial projections"  ✅
                              (similar vector even though
                               no words overlap)
```

**What:** Finding documents by meaning using embedding similarity, not keyword matching.

**Strength:** Finds synonyms, paraphrases, semantically related content.

**Weakness:** Can miss exact technical terms. "error code 404" might not embed close to "HTTP 404 Not Found."

**In your codebase:** `_vector_hits()` in `lumon/retrieval/hybrid.py`. Uses LlamaIndex similarity search.

---

## 17. Sparse vs Dense Retrieval

```
SPARSE (BM25)                   DENSE (Vector)
────────────────────            ────────────────────
Vector: [0, 0, 1, 0, 1, 0, 0]  Vector: [0.23, -0.11, 0.87, 0.44]
Most dims = 0                   All dims carry signal
One dim per vocabulary word     Learned semantic dimensions
Fast, interpretable             Slower, captures meaning
Great for keywords              Great for synonyms
```

**What:** Two fundamentally different ways to represent text for search.

**Why know this:** When an interviewer asks "why hybrid search?" — this is the root answer. Each representation has blind spots that the other covers.

---

## 18. Hybrid Search ⭐ (Key concept — will be asked)

```
QUERY: "project deadline"

BM25 results:                    Vector results:
1. "deadline is March 15"        1. "due date is March 15"  ← synonyms!
2. "project timeline"            2. "deadline is March 15"
3. "final deadline notice"       3. "submission cutoff"     ← related!

           [Merge with RRF]
                 │
Final ranked list:
1. "deadline is March 15"    ← appeared high in both → wins
2. "due date is March 15"    ← strong in vector
3. "project timeline"        ← strong in BM25
```

**What:** Running both BM25 and vector search, then merging the results.

**Why:** BM25 misses synonyms. Vector misses exact terms. Together they cover both failure modes. Hybrid consistently outperforms either method alone.

**In your codebase:** `HybridRetriever.retrieve()` in `lumon/retrieval/hybrid.py`.

---

## 19. RRF — Reciprocal Rank Fusion ⭐ (Will be asked)

```
                    1
RRF score  =  ────────────
               rank + k

where k = 60 (smoothing constant)

BM25 list:           RRF scores:
  rank 1  → 1/(1+60) = 0.0164
  rank 2  → 1/(2+60) = 0.0161
  rank 5  → 1/(5+60) = 0.0154

Vector list:
  rank 1  → 1/(1+60) = 0.0164
  rank 3  → 1/(3+60) = 0.0159

Document appearing in both lists:
  Total RRF = 0.0164 + 0.0159 = 0.0323  ← ranked highest
```

**What:** A formula that merges two ranked lists by position, not by score value.

**Why not just average the scores?** BM25 scores and cosine similarity scores are on completely different scales. You cannot add 0.0003 (BM25) and 0.94 (cosine) and get a meaningful number. RRF uses only *rank position* — which is always comparable across any two lists.

**In your codebase:** `_rrf_merge()` in `lumon/retrieval/hybrid.py`.

---

## 20. Top-K Retrieval

```
Vector index has 10,000 chunks.
k = 5

Query → [find nearest neighbors] → returns 5 most similar chunks
                                                │
                                    Only these 5 go to the LLM
```

**What:** Fetching only the top K most relevant chunks instead of all matches.

**Trade-off:**
- K too small → miss relevant info → incomplete answers
- K too large → include noise → LLM gets confused, answers degrade

**In your codebase:** Controlled by retrieval limits in `config/settings.py`.

---

## 21. Reranker / Cross-Encoder ⭐ (Key concept — will be asked)

```
STEP 1 — Fast retrieval (bi-encoder / embedding):
Query → embed separately → compare vectors → top 20 results
                                              (fast but approximate)

STEP 2 — Precise reranking (cross-encoder):
For each of the 20 results:
  model reads ["query text" + "chunk text"] together
  → outputs a relevance score 0.0–1.0
Results re-sorted by these scores → top 5 returned
(slow but accurate)

WHY TWO STEPS?
Cross-encoder is too slow to run on 10,000 chunks.
But it's very accurate on 20 candidates. So:
Fast retrieval narrows down → reranker precision-selects.
```

**What:** A second-pass model that scores (query, chunk) pairs jointly for precise relevance ranking.

**Bi-encoder vs Cross-encoder:**
- Bi-encoder: encode query alone, encode chunk alone, compare vectors → fast
- Cross-encoder: read query AND chunk together → understands their interaction → accurate but slow

**In your codebase:** `_rerank()` in `lumon/retrieval/hybrid.py` uses `bge-reranker-base`.

---

## 22. Rerank Threshold

```
After reranking, each chunk has a score 0.0–1.0:

chunk_1: 0.87  ✅ above threshold → include
chunk_2: 0.72  ✅ above threshold → include
chunk_3: 0.41  ❌ below threshold → drop
chunk_4: 0.23  ❌ below threshold → drop

threshold = 0.5 (example)

If ALL chunks drop below threshold:
→ Return "gate message": "I couldn't find relevant information"
   (Better than hallucinating an answer!)
```

**What:** A minimum relevance score. Chunks scoring below it are excluded from the LLM context.

**Why:** Sending low-relevance chunks to the LLM is worse than sending nothing. The LLM may force an answer from weak context → hallucination. A gate message is the honest response.

**In your codebase:** Threshold set in `config/settings.py`. Gate message returned from `engine.ask()`.

---

## 23. Jaccard Deduplication

```
chunk_A: "the project budget is two million dollars"
chunk_B: "project budget is two million dollars for Q3"

Words in A: {the, project, budget, is, two, million, dollars}
Words in B: {project, budget, is, two, million, dollars, for, Q3}

Intersection: {project, budget, is, two, million, dollars} = 6 words
Union:        {the, project, budget, is, two, million, dollars, for, Q3} = 9 words

Jaccard = 6/9 = 0.67  (67% overlap → near duplicate → drop one)
```

**What:** Removing near-identical chunks using word overlap ratio.

**Why:** If the same paragraph appears twice in your documents (copy-paste, headers repeated across pages), you don't want it occupying two slots in the LLM context window.

**In your codebase:** `_TextDeduplicator` in `lumon/retrieval/hybrid.py`.

---

## 24. Query Expansion

```
CONVERSATION HISTORY:
User: "Tell me about Project Phoenix"
Bot:  "Project Phoenix is a Q3 initiative to..."

FOLLOW-UP (vague):
User: "What are the deadlines?"

WITHOUT EXPANSION:
retrieve("What are the deadlines?")  ← too vague, retrieves random deadline info

WITH EXPANSION:
retrieve("What are the deadlines? Project Phoenix")  ← precise retrieval ✅
```

**What:** Augmenting the user's query with additional context before retrieval.

**Why:** Short follow-up questions are ambiguous. Without expansion, retrieval quality collapses in multi-turn conversations.

**In your codebase:** `memory.expand_query()` in `lumon/chat/memory.py`. Appends last topic — no separate rewrite model needed.

---

# LEVEL 4 — Generation
> *"How the LLM turns passages into answers"*

---

## 25. Prompt Engineering

```
POORLY STRUCTURED PROMPT:
"answer this: what is the budget? here is some text: [dump of text]"

WELL STRUCTURED PROMPT:
System: "You are a document assistant. Answer only from the provided context.
         If the answer isn't in the context, say you don't know."

Context:
[File: budget_2024.pdf, Page 3]
"The Q3 operating budget is $2.4M, allocated as follows..."

[File: planning.pdf, Page 7]
"Budget approval pending CFO sign-off by March 1st."

User: "What is the Q3 budget?"
```

**What:** Structuring the text sent to the LLM to get reliable, accurate responses.

**Why:** The same retrieval results can produce very different answers depending on how the prompt is structured. Clear structure, role instructions, and source attribution all improve quality.

**In your codebase:** `build_prompt()` in `lumon/pipeline/prompt.py`. Uses `[file, p.N]` labels for each passage.

---

## 26. System Prompt

```
INTENT: QA
System: "Answer questions based only on the provided context.
         Be concise and cite sources."

INTENT: SUMMARIZE
System: "Summarize the provided documents. Focus on key points.
         Structure your response with bullet points."

INTENT: LOCATE_DOC
System: "Identify which document contains the answer.
         State the filename and page number."

Same retrieval results → different LLM behavior → different answer style ✅
```

**What:** Instructions to the LLM that define its role, constraints, and behavior — separate from user message.

**Why:** The system prompt shapes how the LLM uses the retrieved context. Intent-specific prompts make the same LLM behave differently for different use cases.

**In your codebase:** `_PROMPTS` dict in `lumon/pipeline/prompt.py`.

---

## 27. Hallucination

```
GOOD (RAG grounded answer):
Context provided: "Budget is $2.4M"
LLM answer: "The budget is $2.4M"  ✅

HALLUCINATION:
Context provided: [nothing relevant]
LLM answer: "The budget is $3.1M"  ❌ (invented)

HALLUCINATION EVEN WITH RAG:
Context provided: "Budget is $2.4M"
LLM answer: "The budget is $2.4M and the CFO approved it last Tuesday"  ❌
             (second clause not in context — LLM filled a gap)
```

**What:** When an LLM generates plausible-sounding but factually wrong information.

**Why RAG reduces but doesn't eliminate it:** RAG provides grounding — the LLM should stick to the context. But LLMs can still "fill in gaps" beyond what the context says. Good system prompts ("only answer from context, say I don't know otherwise") reduce this further.

---

## 28. Grounding

```
UNGROUNDED answer:                 GROUNDED answer:
"The budget is $2.4M"              "The budget is $2.4M
                                    [Source: budget_2024.pdf, p.3]"
User can't verify it.              User can open the source and check. ✅
```

**What:** Anchoring answers to specific retrieved source passages with citations.

**Why:** Grounding makes answers auditable. Users can verify claims by checking the cited source. Your `AskResponse` returns a `sources` list (file + page) for exactly this reason.

**In your codebase:** `Source` type in `lumon/models/schemas.py`. Shown as source chips in the UI.

---

## 29. Structured Extraction

```
FREE-FORM ANSWER (QA mode):
"There are several tasks mentioned: the team needs to review
 the contract by Friday, and John is responsible for..."

STRUCTURED EXTRACTION (TASKS mode):
{
  "tasks": [
    { "action": "review contract", "owner": "team", "deadline": "Friday" },
    { "action": "update roadmap",  "owner": "John", "deadline": "EOQ"    }
  ]
}
```

**What:** Extracting structured data (tasks, entities, dates, people) from text using rules or LLMs — instead of a free-form prose answer.

**Why:** When you need machine-readable output (a task list, a contact sheet), structured extraction is more useful than asking the LLM to narrate. Also cheaper — your `enrich.py` uses regex, not an LLM call.

**In your codebase:** `lumon/pipeline/enrich.py`. Triggered when intent is `TASKS` or `ENTITIES`, or `?enrich=true`.

---

## 30. LLM Timeout

```
Without timeout:          With timeout (30s):
  Request sent            Request sent
  ...waiting...           ...waiting...
  ...waiting...           [30 seconds pass]
  ...waiting...           Timeout! Return error gracefully.
  [thread hangs forever]  User gets a response. ✅
```

**What:** A maximum wait time for an LLM response. After this, the request fails with an error rather than hanging forever.

**Why:** LLMs can hang due to network issues, model overload, or very long generations. Without a timeout, one slow request can block a thread permanently. Always set timeouts in production.

**In your codebase:** `llm_call_timeout` in `config/settings.py` (default 30s). Separate from the HTTP client timeout.

---

# LEVEL 5 — Architecture
> *"How all the pieces fit together"*

---

## 31. Intent Classification / Routing

```
USER QUERY                     INTENT           ACTION
────────────────────────────   ─────────────    ──────────────────────────
"list my files"             →  LIST_FILES    →  just read disk, no retrieval
"what is the budget?"       →  QA            →  retrieve + LLM answer
"summarize my documents"    →  SUMMARIZE     →  retrieve + LLM summary
"what tasks are mentioned?" →  TASKS         →  retrieve + heuristic extract
"who is mentioned?"         →  ENTITIES      →  retrieve + entity extract
"where is the budget info?" →  LOCATE_DOC    →  retrieve + cite location
```

**What:** Deciding what kind of response is needed *before* doing retrieval. Uses keyword rules, not an LLM.

**Why:** Not every query needs retrieval or an LLM call. Routing "list my files" directly to disk read is 100x cheaper. Good design separates concerns — routing stays isolated so it's easy to add new intents.

**In your codebase:** `lumon/chat/router.py`. Rule: `router.py` imports nothing from `pipeline` or `retrieval` — stays isolated.

---

## 32. Conversation Memory / Multi-turn Context

```
Turn 1:  "Tell me about Project Phoenix budget"
         _last_topic = "Project Phoenix budget"

Turn 2:  "What are the risks?"
         expand_query() → "What are the risks? Project Phoenix budget"
         Retrieval now finds the right documents ✅

Turn 3:  "Who is responsible?"
         expand_query() → "Who is responsible? Project Phoenix budget"
         Still on topic ✅
```

**What:** Tracking prior conversation turns so follow-up questions work correctly.

**Why:** Without memory, every question is isolated. "What are the risks?" retrieves random risk content from all documents. Memory provides the missing context.

**In your codebase:** `lumon/chat/memory.py`. Tracks `_last_topic` by extracting content words (filters filler words). `ChatMemoryBuffer` passes last N tokens of chat history to the LLM.

---

## 33. Parametric vs Non-Parametric Memory

```
PARAMETRIC MEMORY                  NON-PARAMETRIC MEMORY
(inside model weights)             (your document store)
──────────────────────             ──────────────────────
Baked in during training           Retrieved at query time
Can't be updated without           Update by changing documents
  retraining
Everything the model               Your specific, current,
  "knows" generally                  private information

RAG = parametric reasoning  +  non-parametric facts  =  best of both ✅
```

**What:** Parametric = knowledge in model weights. Non-parametric = knowledge retrieved from external storage.

**Why it matters:** This is the conceptual foundation of RAG. The LLM provides reasoning and language ability (parametric). Your document store provides current, specific facts (non-parametric). You need both.

---

## 34. Background Indexing / IndexWorker

```
SYNCHRONOUS (bad):
User uploads file
→ [index file — takes 10 seconds]
→ User waits
→ Response returned after 10 seconds ❌

ASYNC WITH WORKER (good):
User uploads file
→ Response: "File received" (instant ✅)
→ [IndexWorker picks up job in background]
→ [sync_new_files() runs on thread pool]
→ Index updated, retriever reloaded
→ File searchable ~10 seconds later, user never waited
```

**What:** A background queue that processes indexing jobs without blocking HTTP requests.

**Why:** Indexing (chunking + embedding) is slow. Users shouldn't wait for it. The worker pattern keeps the API fast while indexing happens behind the scenes.

**In your codebase:** `IndexWorker` in `lumon/ingest/sync.py`. Started in `lifespan` of `app.py`.

---

## 35. File Watcher

```
my_folder/
    │
    ├── report.pdf         [no change]
    ├── budget.txt  ◄──── [MODIFIED!]
    └── notes.md           [no change]

watchdog detects budget.txt change
→ enqueue_path("budget.txt")
→ IndexWorker picks it up
→ sync_new_files() re-indexes budget.txt only
→ ready to search updated content ✅
```

**What:** A filesystem monitor that detects when files in `my_folder/` change and automatically triggers re-indexing.

**Why:** Without this, the index would go stale whenever a document is updated. The watcher keeps the index current automatically.

**In your codebase:** `start_watcher()` in `lumon/ingest/sync.py`. Uses `watchdog` library with 2-second debounce.

---

## 36. Orphan Documents

```
SYNC PROCESS:
1. Delete old version of file from index  ← delete_ref_doc() called
2. Insert new version                     ← insert new nodes
3. Persist

If step 1 FAILS (network error, corrupted ref):
→ Old nodes stuck in index = "orphan"
→ Surfaced in GET /health as orphan_warnings
→ System still works, but old content may appear in results
```

**What:** Document IDs in the index whose deletion failed during a sync operation — their data is stuck and can't be cleaned up automatically.

**Why it matters:** Orphans cause stale content to appear in search results. Monitoring them via `/health` lets you detect and fix sync issues.

**In your codebase:** `lumon/ingest/orphans.py`. In-memory deque surfaced in health check.

---

# LEVEL 6 — Interview Questions With Full Answers

> *These exact questions will come up. Know these cold.*

---

### Q1: "Why does RAG reduce hallucination?"

**Answer:** The LLM's system prompt instructs it to answer only from the provided context. When the retrieved passages contain the answer, the LLM reads and reports it rather than generating from parametric memory. The rerank threshold acts as a second safeguard — if no passage is relevant enough, a gate message is returned instead of forcing the LLM to guess. Hallucination isn't eliminated (the LLM can still fill gaps), but it's dramatically reduced.

---

### Q2: "You search for 'budget forecast' but the document says 'financial projections.' How does your system find it?"

**Answer:** BM25 would miss it entirely — zero word overlap means zero score. The vector (semantic) search finds it because both phrases embed to nearby vectors — similar meaning, similar numbers. This is exactly why hybrid search exists: BM25 handles exact matches, vectors handle semantic similarity.

---

### Q3: "Why use RRF to merge search results instead of averaging scores?"

**Answer:** BM25 and cosine similarity scores are on different scales. A BM25 score might be 0.0003 and a cosine similarity 0.94 for the same document — you cannot meaningfully average these. RRF uses only rank position (1/(rank+k)), which is comparable across any two lists regardless of their scoring method.

---

### Q4: "Why parent-child chunking instead of one chunk size?"

**Answer:** Small chunks (128 tokens) retrieve precisely — the search finds exactly the right passage. But 128 tokens is often too little context for the LLM to give a complete answer. Large chunks (512 tokens) give the LLM enough to work with, but retrieving them is noisier. Parent-child gives you both: precise retrieval via child chunks, full context via parent chunks at generation time.

---

### Q5: "What happens when nothing passes the rerank threshold?"

**Answer:** A gate message is returned — essentially "I couldn't find relevant information for your question." This is intentional. Sending weak, low-relevance passages to the LLM would likely cause hallucination. A transparent "I don't know" is more honest and more useful than a confident wrong answer.

---

### Q6: "How would you scale this system beyond a single machine?"

**Answer:** Several changes needed: (1) Replace the in-process vector store with a distributed vector DB (Pinecone, Weaviate, Qdrant). (2) Move the IndexWorker to a proper job queue (Celery, Redis Queue). (3) Make the BM25 index shared state (currently in-memory per process). (4) Run multiple API instances behind a load balancer. (5) Separate the embedding model service so it scales independently.

---

### Q7: "Why is intent classification done with keyword rules instead of an LLM?"

**Answer:** Speed and cost. "List my files" doesn't need an LLM to classify — a keyword match takes microseconds and costs nothing. Using an LLM to classify would add 1-2 seconds of latency and API cost to every request, including the simplest ones. Reserve LLM calls for tasks that genuinely need them.

---

# Quick Reference Cheat Sheet

```
INDEXING PIPELINE
─────────────────
Document → Tokenize → Chunk (128/512 tokens) → Embed → Vector Index
                                              → BM25 Index

RETRIEVAL PIPELINE
──────────────────
Query → Vector Search ──┐
Query → BM25 Search   ──┼── RRF Merge → Deduplicate → Rerank → Filter by threshold
                         │
                         └── Top-K results → Load Parents → Build Prompt

GENERATION PIPELINE
───────────────────
Prompt = System instruction + Retrieved parent passages + User question
→ LLM → Answer + Sources
```

---

*Last updated to match: lumon/ codebase — parent-child chunking, hybrid retrieval, BGE embeddings, bge-reranker-base, RRF merging.*