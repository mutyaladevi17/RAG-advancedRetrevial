# Insurellm RAG: Advanced Retrieval Pipeline

![RAG Pipeline](https://img.shields.io/badge/Architecture-Advanced_RAG-purple)
![Litellm](https://img.shields.io/badge/Framework-Litellm-blue)
![Python](https://img.shields.io/badge/Language-Python_3.10+-yellow)

## Overview
This repository implements an **Advanced Retrieval-Augmented Generation (RAG)** system for a mock insurance company named "Insurellm". Moving beyond native, off-the-shelf framework abstractions, this custom-built pipeline focuses on maximizing context retrieval metrics through highly sophisticated techniques: intelligent document preprocessing, query rewriting, multi-query retrieval, and LLM-powered reranking.

---

## Features
* **LLM-Generated Summaries:** Enriches standard document chunks by dynamically generating a targeted *headline* and *summary* for every chunk prior to vectorization.
* **Semantic Document Segmentation:** Avoids arbitrary text splitting by using an LLM to smartly chunk documents based on semantic boundaries and semantic meaning.
* **LLM-Based Reranking:** Dynamically reorders the retrieved chunks prior to generation by prompting an LLM to rank chunks strictly by relevance to the query.
* **Query Rewriting:** Utilizes an LLM to analyze the conversational history and rewrite the user's query into a highly specific, optimized search term.
* **Multi-Query Merging:** Executes vector searches for both the original query and the rewritten query concurrently, merging the results to ensure comprehensive coverage.
* **Custom Orchestration:** Built primarily with the `openai` SDK and `litellm` for direct, lightweight LLM API interactions.
* **Robust Error Handling:** Employs the `tenacity` library to provide exponential backoff and retry capabilities for API resilience.
* **Gradio Interfaces:** Includes an interactive chat application and an automated, visual evaluation dashboard.

---

## Architecture & Tech Stack
* **LLM Abstraction Layer:** Litellm & OpenAI SDK
* **Primary Models:** `gpt-4.1-nano` (Customizable)
* **Embeddings:** `text-embedding-3-large`
* **Vector Database:** ChromaDB (Persistent)
* **Resilience:** Tenacity
* **UI & Evaluation:** Gradio, Pandas

---

## Pipeline Step-by-Step

This advanced RAG implementation operates through a multi-stage pipeline designed to solve common retrieval failures like vocabulary mismatch and context fragmentation:

1. **Intelligent Ingestion & Semantic Segmentation (`ingest.py`):** 
   Instead of using naive character or token-based splitters, documents are fed to an LLM which performs intelligent semantic segmentation. The LLM splits the text logically while maintaining desired overlap.
2. **Metadata Enrichment & Summarization:**
   For every segmented chunk, the LLM generates a concise **Headline** and a **Summary**. The final string embedded and stored in ChromaDB is a concatenation of `Headline + Summary + Original Text`. This drastically increases the semantic density of the embeddings, making them highly discoverable.
3. **Query Rewriting (`answer.py`):** 
   Given a user's raw query and previous chat history, an LLM rewrites the prompt into a precise, context-infused search query optimized for vector databases.
4. **Multi-Query Retrieval:**
   The pipeline queries ChromaDB twice: once with the *original* query and once with the *rewritten* query. Results are merged to maximize recall and keyword coverage.
5. **LLM-Based Reranking:**
   The merged context chunks are fed back into a powerful LLM acting as a zero-shot reranker. The LLM evaluates the raw chunks against the user's intent and returns a perfectly ordered list of chunks by relevance.
6. **Final Generation:**
   The top `K` reranked chunks form the definitive context window, which is sent to the LLM to generate the final, accurate response for the user.

---

## Prerequisites

Ensure you have the following installed on your local machine:
* Python 3.10+
* `uv` or `pip` package manager

---

## ⚙️ Installation & Setup

1. **Clone the repository and enter the directory:**
   ```bash
   cd RAG-advancedRetrevial
   ```

2. **Install the dependencies:**
   ```bash
   uv pip install -r requirements.txt
   # OR
   pip install -r requirements.txt
   ```

3. **Configure Environment Variables:**
   Create a `.env` file in the root directory and add your API credentials:
   ```env
   OPENAI_API_KEY=your_openai_api_key_here
   ```

4. **Prepare the Vector Database:**
   To parse, semantically segment, embed, and load the knowledge base documents into ChromaDB:
   ```bash
   uv run adv_implementation/ingest.py
   ```

---

## 💻 Usage

### 1. Chat Application
Start the conversational AI agent by running:
```bash
uv run app_gradio.py
```
This launches an interactive web UI (typically at `http://127.0.0.1:7860`) where you can ask Insurellm questions and observe the retrieved chunks.

### 2. Evaluation Dashboard
Evaluate the sophisticated retrieval pipeline against 150 benchmark tests:
```bash
uv run evaluator.py
```
This Gradio dashboard visualizes MRR, nDCG, Keyword Coverage, Accuracy, Completeness, and Relevance.

---

## Evaluation Results

The advanced retrieval system significantly enhances context discovery compared to baseline RAG architectures:

| Metric | Score | 
| :--- | :---: | 
| **Mean Reciprocal Rank (MRR)** | **0.8800* | 
| **Normalized DCG (nDCG)** | 0.8581 | 
| **Keyword Coverage** | **95.5%** | 
| **Answer Accuracy** | 4.51/5 |
| **Answer Completeness** | 4.31/5 |
| **Answer Relevance** | 4.81/5 | 

*Insight: The advanced query rewriting, combined with semantic summarization and LLM reranking, is incredibly effective at fixing vocabulary mismatches and surfacing hard-to-find keywords, achieving a massive **95.5% keyword coverage rating**.*

---

## Project Structure

```text
RAG-advancedRetrevial/
├── app_gradio.py                 # Main interactive Gradio Chat Application
├── evaluator.py                  # Evaluation dashboard for the advanced pipeline
├── adv_implementation/
│   ├── answer.py                 # Core logic: Rewriting, Reranking, and Merging
│   └── ingest.py                 # Ingestion script (Semantic Segmentation + Summaries)
├── evaluation/
│   ├── eval.py                   # Scoring logic (LLM-as-a-judge, MRR math)
│   └── test.py                   # Test dataset loader
└── preprocessed_db/              # Persistent local database directory
```

---

## Limitations & Future Work

* **Inference Latency:** Processing a single message currently requires sequential LLM calls (Rewrite $\to$ 1st Stage Retrieval $\to$ LLM Reranker $\to$ Final Generation), introducing higher latency than single-shot RAG.
* **Operational Costs:** The recursive use of large language models for document segmentation, intermediate ranking, and rewriting can scale costs significantly over large corpora.
* **Future Optimization:** 
  * Replace the heavy LLM reranker with a dedicated, lightweight cross-encoder model (e.g., `BAAI/bge-reranker-large`).
  * Introduce conversational caching using tools like Redis or GPTCache to return answers to common questions without re-triggering the full pipeline.
