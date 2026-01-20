# ProductLens AI — KPI-Driven Competitive Intelligence (RAG)

## Problem Statement

Product managers need to continuously evaluate competitors to make roadmap decisions tied to specific business KPIs (e.g., growth, retention, revenue concentration).
However, competitive analysis today is:

* Manual and time-intensive (reading multiple product pages and docs)
* Error-prone due to ungrounded assumptions and hallucinated features
* Difficult to align explicitly to **one target KPI**
* Unreliable when competitors are outside the same market category

**ProductLens AI** addresses this by performing **context-grounded, KPI-specific competitive analysis** using only verified product documentation provided by the user, while explicitly rejecting invalid competitor or KPI inputs. 

---

## Objective

Build a **Retrieval-Augmented, Agentic AI system** that:
1. Accepts a **context company corpus** (product feature URL or file)
2. Accepts a **competitor company URL**
3. Accepts a **single target KPI**
4. Produces a **feature-level competitive comparison**
5. Generates **roadmap priorities justified by causal KPI reasoning**
6. Explicitly exits when:
   * The competitor is not in the same market
   * The KPI is not relevant to the context company

All outputs remain **strictly grounded to retrieved context** and does not hallucinate features. 

---

## System Architecture

<img width="1202" height="978" alt="ProductLens AI" src="https://github.com/user-attachments/assets/2236edb4-5b07-47ad-974a-6144bd927163" />

### 1. Corpus Ingestion & Indexing
* URL/file ingestion for **context company product features**
* Vector embeddings generated using `text-embedding-3-small`
* Indexed into **Astra DB Vector Store** with metadata preservation

### 2. Retrieval & Grounding Layer (RAG)
* Semantic retrieval using similarity search
* Context filtering to ensure:
  * Only retrieved chunks are available to the agent
  * No cross-document contamination
* Re-ranking enabled to improve feature relevance precision

### 3. Input Validation & Gating (Hard Stops)
The agent enforces **binary gates before reasoning**:
* KPI relevance validation against retrieved context
* Market alignment check between context company and competitor
* Early termination with deterministic error messages if invalid
  (Implemented directly in the system prompt logic) 

### 4. Agentic Reasoning Layer
* Single-agent execution (non-autonomous)
* Prompt-enforced reasoning constraints:
  * Feature → KPI lever mapping
  * Causal explanation of impact
  * Directional KPI effect (increase/decrease)
  * Time horizon (short-term / long-term)
* No tool hallucination; agent reasons only over retrieved context and corpus

### 5. Output Contract (Structured Enforcement)
The agent will return:
* Summary
* Feature gaps
* Roadmap priorities (Now / Next)
* KPI justification per recommendation
  Each section is bounded by **token and bullet limits** to reduce latency and drift. 

---

## Key Features (Agentic AI – Technical)

* **Context-Bound RAG Execution** (no open-world reasoning)
* **KPI-First Reasoning Graph** (feature → KPI lever → outcome)
* **Input Gating & Early Exit Mechanisms**
* **Feature-Level Semantic Comparison**
* **Prompt-Enforced Causal Reasoning**
* **Deterministic Output Schema**
* **Hallucination Prevention via Context Grounding**
* **Latency-aware response constraints**

---

## Chunking Strategy
ProductLens AI uses a **fixed-size overlapping chunking strategy** during corpus ingestion to optimize retrieval accuracy and semantic continuity.

### Design Details
* **Chunking method:** Fixed-size chunks with overlap (1000 chunck size and 200 overlap)
* **Chunk size:** Constant token/window length
* **Overlap:** Configured overlap between adjacent chunks to preserve cross-boundary semantics
* **Segmentation:** Applied after HTML/text extraction and before embedding generation

### Rationale
* Prevents loss of semantic context at chunk boundaries (e.g., feature descriptions split across sections)
* Improves recall during vector similarity search by ensuring related concepts appear in multiple chunks
* Stabilizes retrieval quality for feature-level comparisons across competitors

### Impact on RAG Performance
* Higher contextual relevance during similarity search
* Reduced partial-feature hallucination
* More consistent grounding when mapping features to KPI levers

This chunking strategy directly supports **context faithfulness** and **feature-level accuracy**, which contributed to high scores in **Context Management (5/5)** and **Accuracy (5/5)** during evaluation.

---

## Evaluation Framework (Evals)

Evaluation was performed using a **multi-dimension weighted rubric** across real user queries and competitive scenarios. 

### Evaluation Dimensions

#### 1. Performance (Weight: 40%)

* Relevance to KPI and company: **5/5**
* Accuracy grounded in context: **5/5**
* Faithfulness across runs: **5/5**

#### 2. Context Management (Weight: 30%)

* No hallucinated features: **5/5**
* Practical relevance of retrieved context: **5/5**

#### 3. Metric Reasoning (Weight: 20%)

* Causal reasoning quality: **4/5**
* Handling invalid KPI / competitor inputs: **5/5**

#### 4. UI / UX (Weight: 10%)

* Latency: **2/5** (≈40s observed)
* Output format compliance: **5/5**
* Clarity of language: **3/5**

---

### Final Weighted Score

| Category           | Weighted Score  |
| ------------------ | --------------- |
| Performance        | 200             |
| Context Management | 150             |
| Metric Reasoning   | 90              |
| UI / UX            | 33.3            |
| **Total**          | **473.3 / 500** |

**Final Score: 4.73 / 5** 

---

## Known Limitations & Improvements

Identified directly from failed eval cases:
* Metric reasoning lacked explicit causal structure (fixed via prompt updates)
* High latency due to over-generation (fixed via bullet & token caps)
* No fallback when context is insufficient (added early termination clause)

All fixes were implemented as **prompt-level guardrails**, not post-processing hacks. 

---

## Why This Matters

ProductLens AI demonstrates how **Agentic RAG systems can be evaluated like real products**:
* With hard gates
* Deterministic failure modes
* KPI-driven reasoning
* Explicit tradeoffs between depth and latency

---
## Generated Output Sample
Sample Output generated against companies: Mogli Technologies (https://www.mogli.com/product) and Twilio (https://www.twilio.com/en-us/products)

https://github.com/mrudu10/RAG-based-ProductAnalyzer/blob/main/competitor-analysis.pdf

---
## App's UI - https://productlensai.lovable.app/
<img width="1374" height="895" alt="image" src="https://github.com/user-attachments/assets/d05696c0-6e64-4b8e-b852-378db322d3f5" />

NOTE!!!!!!!
Since the corpus is deployed on AstraDB, The database might be hibernated as AstraDB databases idle for over 48 hours are automatically put into hibernation. Hence the demo link might not work. This is not because of Product Implementation Failure, rather is expected due to AstraDB hibernation!!!
