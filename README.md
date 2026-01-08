# AI Act Semantic Search

Semantic search system to retrieve the most relevant passages of the **European AI Act** from a user query.  
The goal is **not** to generate answers, but to surface the best-matching text chunks to help navigate a long legal document.

---

## Project Overview

An end-to-end **semantic retrieval pipeline** is implemented, including:
- extracting the AI Act text from a PDF and cleaning it
- splitting the document into **chunks** suitable for retrieval
- embedding all chunks and indexing them into a vector store (separate index per model)
- querying the index with user questions and returning the **top-k most relevant chunks**
- benchmarking retrieval quality using **Hit@k** and **MRR**

---

##  Dataset (AI Act document)

Source: official **EU AI Act** text (PDF).  
Preprocessing included:
- text extraction and normalization (PDF artifacts, whitespace)
- segmentation into smaller **overlapping chunks** to preserve legal context
- metadata tracking (e.g., article number) for traceability

---

##  Methods & Technical Approach

### 1) Chunking + Indexing
The AI Act is split into chunks, then embedded and stored in a vector database.  
Each embedding model is indexed in its **own ChromaDB collection** to ensure fair comparison.

### 2) Embedding models compared

Two Sentence-Transformers embedding models are compared:

- **MiniLM (Small/EN)**: `all-MiniLM-L6-v2`  
  Lightweight model optimized for speed and general English semantic similarity.

- **MPNet (Medium/Multi)**: `paraphrase-multilingual-mpnet-base-v2`  
  Larger multilingual model that captures deeper semantics and performs better on formal/legal phrasing.

Retrieval uses **cosine similarity** to rank chunks by relevance.

### 3) Evaluation protocol (Hit@k + MRR)
A small evaluation set was manually defined: each question is associated with the expected article(s).
- **Hit@k**: success if at least one expected article appears in the top-k retrieved results
- **MRR**: rewards systems that rank the correct article as close to rank #1 as possible

---

## 📈 Results (Retrieval Benchmark)

| Model | Hit@1 | MRR@1 | Hit@3 | MRR@3 | Hit@5 | MRR@5 |
|------|------:|------:|------:|------:|------:|------:|
| MiniLM (`all-MiniLM-L6-v2`) | 29.7% | 0.297 | 40.5% | 0.342 | 48.6% | 0.363 |
| MPNet (`paraphrase-multilingual-mpnet-base-v2`) | 51.4% | 0.514 | 73.0% | 0.604 | 75.7% | 0.610 |

>> MPNet clearly outperforms MiniLM on both Hit@k and MRR, meaning the correct legal articles are retrieved **more often** and **ranked higher**.

---

##  Tech Stack

Python, pdfplumber, sentence-transformers, ChromaDB, cosine similarity

---

##  Next steps

- Improve chunking strategy (overlap / boundaries by article structure)
- Increase the evaluation set size and add more diverse legal queries
- Optional: connect retrieval to a language model for answer generation (RAG)
