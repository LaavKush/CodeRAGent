<div align="center">

# CodeRAGent

### Agentic RAG for Software Engineering QA

<p>
  <strong>
    Context-aware question answering over technical documentation
    and open-source source code.
  </strong>
</p>

<p>
  <img src="https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/LangGraph-Agentic_Routing-1C3C3C?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Qdrant-Vector_Search-DC244C?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Mistral_7B-Ollama-FF7000?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Streamlit-Frontend-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white"/>
  <img src="https://img.shields.io/badge/FastAPI-Backend-009688?style=for-the-badge&logo=fastapi&logoColor=white"/>
  <img src="https://img.shields.io/badge/Docker-Containerized-2496ED?style=for-the-badge&logo=docker&logoColor=white"/>
</p>

<p>
  <a href="#overview">Overview</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#agent-flow">Agent Flow</a> ·
  <a href="#installation">Installation</a> ·
  <a href="#evaluation">Evaluation</a>
</p>

</div>

---

<div align="center">

> **Retrieve → Evaluate → Rewrite → Retry → Search → Generate**

</div>

<table>
<tr>
<td width="50%" valign="top">

## Overview

CodeRAGent is an agentic Retrieval-Augmented Generation (RAG) system designed for software engineering question answering.

It works over:

- Technical documentation
- Open-source source code
- Repository-level context
- External web knowledge

</td>

<td width="50%" valign="top">

## What makes it agentic?

Unlike a conventional RAG pipeline, CodeRAGent does not blindly retrieve once and generate.

The system can:

- Route the query
- Retrieve repository context
- Rerank candidates
- Grade retrieval quality
- Rewrite weak queries
- Retry retrieval
- Fall back to web search
- Generate the final answer

</td>
</tr>
</table>

---

## Architecture

```text
                         ┌───────────────────────┐
                         │   Streamlit Frontend  │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │      FastAPI API      │
                         └───────────┬───────────┘
                                     │
                                     ▼
                         ┌───────────────────────┐
                         │    LangGraph Agent    │
                         └───────────┬───────────┘
                                     │
                         ┌───────────┴───────────┐
                         ▼                       ▼
                  ┌──────────────┐       ┌──────────────┐
                  │ Query Router │       │ Direct Path  │
                  └──────┬───────┘       └──────────────┘
                         │
                         ▼
                  ┌──────────────┐
                  │    Qdrant    │
                  │   Retrieval  │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │ BGE Reranker │
                  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │  Retrieval   │
                  │    Grader    │
                  └──────┬───────┘
                         │
                    Relevant?
                    /       \
                  YES        NO
                   │          │
                   │          ▼
                   │   ┌──────────────┐
                   │   │ Query Rewrite│
                   │   └──────┬───────┘
                   │          │
                   │          ▼
                   │     Re-retrieval
                   │          │
                   └──────────┬┘
                              │
                       Still insufficient?
                              │
                              ▼
                       ┌──────────────┐
                       │ Tavily Search│
                       └──────┬───────┘
                              │
                              ▼
                       ┌──────────────┐
                       │  Mistral 7B  │
                       │    Ollama    │
                       └──────┬───────┘
                              │
                              ▼
                       ┌──────────────┐
                       │ Final Answer │
                       └──────────────┘
```

---

## Agent Flow

### Repository-specific question

```text
User Question
      │
      ▼
Query Router
      │
      ▼
Repository Retrieval
      │
      ▼
BGE Reranking
      │
      ▼
Retrieval Grader
      │
      ├── Relevant ─────────────► Answer Generation
      │
      └── Not Relevant
                │
                ▼
          Query Rewriting
                │
                ▼
           Re-retrieval
                │
                ├── Relevant ────► Answer Generation
                │
                └── Still Weak
                        │
                        ▼
                  Tavily Web Search
                        │
                        ▼
                  Answer Generation
```

### General question

```text
User Question
      │
      ▼
Query Router
      │
      ▼
DIRECT
      │
      ▼
Answer Generation
```

---

## Core Components

<table>
<tr>
<td align="center" width="25%" valign="top">

### Agent

**LangGraph**

**LangChain**

Agentic routing and workflow orchestration.

</td>

<td align="center" width="25%" valign="top">

### Retrieval

**Qdrant**

**Sentence Transformers**

**BGE Cross-Encoder**

Vector search and relevance reranking.

</td>

<td align="center" width="25%" valign="top">

### Generation

**Mistral 7B**

**Ollama**

Local answer generation.

</td>

<td align="center" width="25%" valign="top">

### Infrastructure

**FastAPI**

**Docker**

**Docker Compose**

Backend and deployment infrastructure.

</td>
</tr>
</table>

---

# Features

### Agentic Query Routing

The LangGraph workflow determines how a question should be handled:

- Direct path for simple or general questions
- Retrieval path for repository-related questions
- Complex retrieval flow for deeper investigation

### Retrieval-Augmented Generation

- Dense vector retrieval using Qdrant
- Repository documentation and source-code retrieval
- Sentence-transformer embeddings
- BGE cross-encoder reranking

### Corrective Retrieval

The retrieval pipeline evaluates the quality of retrieved context.

```text
Retrieve
   ↓
Grade
   ↓
Low Relevance
   ↓
Rewrite Query
   ↓
Retrieve Again
```

### Web Search Fallback

When repository context remains insufficient, CodeRAGent can use Tavily web search to retrieve information outside the indexed repository.

### Local LLM

Answer generation is performed using **Mistral 7B through Ollama**, allowing the core generation workflow to run locally.

### Observable Execution Trace

The Streamlit interface exposes:

- Selected route
- Retrieved document count
- Retrieval score
- Retry count
- Query rewriting
- Retrieved source files
- Web-search usage
- Retrieved context
- Developer/debug information

---

# Tech Stack

| Layer | Technology |
|---|---|
| LLM | Mistral 7B |
| Local Runtime | Ollama |
| Agent Framework | LangGraph |
| RAG Framework | LangChain |
| Backend | FastAPI, Uvicorn |
| Vector Database | Qdrant |
| Embeddings | Sentence Transformers |
| Reranking | BGE Cross-Encoder |
| Web Search | Tavily |
| Frontend | Streamlit |
| Evaluation | RAGAS |
| Containerization | Docker |
| Orchestration | Docker Compose |

---

# Project Structure

```text
CodeRAGent/
│
├── app/
│   ├── agents/
│   ├── api/
│   │   └── routes.py
│   ├── core/
│   │   ├── config.py
│   │   └── qdrant.py
│   ├── embeddings/
│   ├── evaluation/
│   │   ├── benchmark.py
│   │   └── ragas_evaluator.py
│   ├── graph/
│   │   ├── nodes.py
│   │   ├── query_rewriter.py
│   │   ├── retrieval_grader.py
│   │   ├── router.py
│   │   ├── state.py
│   │   └── workflow.py
│   ├── ingestion/
│   ├── llm/
│   │   └── ollama_client.py
│   ├── models/
│   ├── prompts/
│   ├── rag/
│   │   └── rag_pipeline.py
│   ├── retrieval/
│   ├── services/
│   ├── vectorstore/
│   │   └── qdrant_client.py
│   ├── websearch/
│   │   └── web_searcher.py
│   ├── main.py
│   └── streamlit_app.py
│
├── data/
│   ├── benchmark.json
│   └── raw/
│       └── repos/
│
├── docker/
├── scripts/
│   └── index_repository.py
│
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env
└── README.md
```

---

# How the System Works

## 1. Repository Ingestion

Technical documentation and source code are collected from the target repository.

## 2. Chunking

The repository is divided into useful chunks so individual sections of documentation and code can be retrieved independently.

## 3. Embeddings

Each chunk is converted into a vector representation using the configured embedding model.

## 4. Vector Storage

Embeddings and their associated metadata are stored in **Qdrant**.

## 5. Query Routing

The LangGraph router decides whether the question should:

- Take the direct path
- Use repository retrieval

## 6. Retrieval

For retrieval-based questions, relevant repository chunks are retrieved from Qdrant.

## 7. Reranking

Retrieved candidates can be reranked using a **BGE cross-encoder** to improve relevance.

## 8. Retrieval Grading

The retrieval grader evaluates whether the retrieved context is useful for answering the question.

## 9. Query Rewriting

If retrieval quality is insufficient, the agent rewrites the query and retries retrieval.

## 10. Web Search

If repository context is still insufficient, the system can use **Tavily** as an external knowledge source.

## 11. Answer Generation

The collected context is passed to **Mistral 7B through Ollama**, which generates the final answer.

---

# Streamlit Frontend

The Streamlit frontend is designed to make the agent's execution observable.

```text
Question
   ↓
Query Router
   ↓
Vector Retrieval
   ↓
BGE Reranking
   ↓
Retrieval Grader
   ↓
Query Rewrite / Retry
   ↓
Web Search (when needed)
   ↓
Final Answer
```

The interface exposes:

| Trace Information | Purpose |
|---|---|
| Selected Route | Shows the agent's routing decision |
| Retrieved Documents | Number of retrieved repository chunks |
| Retrieval Score | Indicates retrieval quality |
| Retry Count | Shows corrective retrieval attempts |
| Rewritten Query | Displays query transformation |
| Repository Sources | Shows relevant source files |
| Web Search | Indicates whether external search was used |
| Retrieved Context | Provides optional context inspection |
| Debug Information | Helps inspect the execution flow |

---

# Installation

## Clone the Repository

```bash
git clone https://github.com/LaavKush/CodeRAGent.git
cd CodeRAGent
```

## Create a Virtual Environment

```bash
python -m venv .venv
```

### Windows

```powershell
.\.venv\Scripts\Activate.ps1
```

### Linux / macOS

```bash
source .venv/bin/activate
```

## Install Dependencies

```bash
python -m pip install -r requirements.txt
```

---

# Environment Configuration

Create a `.env` file in the project root.

```env
QDRANT_COLLECTION=software_docs

QDRANT_URL=http://localhost:6333
QDRANT_API_KEY=

OLLAMA_BASE_URL=http://localhost:11434
OLLAMA_MODEL=mistral:7b

TAVILY_API_KEY=
```

Do not commit `.env` or API keys to GitHub.

For a Dockerized FastAPI container connecting to Qdrant and a locally running Ollama instance:

```text
QDRANT_URL=http://qdrant:6333
OLLAMA_BASE_URL=http://host.docker.internal:11434
```

---

# Start Qdrant

The project includes a Docker Compose configuration for Qdrant.

```powershell
docker compose up -d qdrant
```

Check running containers:

```powershell
docker ps
```

Qdrant is exposed on:

```text
http://localhost:6333
```

---

# Index a Repository

Run the repository indexing script:

```powershell
python .\scripts\index_repository.py
```

The indexed documents are stored in the configured Qdrant collection.

---

# Run FastAPI

Start the backend locally:

```powershell
python -m uvicorn app.main:app --reload --port 10000
```

API:

```text
http://localhost:10000
```

Swagger documentation:

```text
http://localhost:10000/docs
```

---

# Run Streamlit

From the project root:

```powershell
python -m streamlit run .\app\streamlit_app.py --server.port 8502
```

Open:

```text
http://localhost:8502
```

---

# Docker

Build the application image:

```powershell
docker build -t coderagent .
```

Run the FastAPI container:

```powershell
docker run --rm `
  --network coderagent_default `
  -p 10000:8000 `
  --env-file .env `
  -e QDRANT_URL=http://qdrant:6333 `
  -e OLLAMA_BASE_URL=http://host.docker.internal:11434 `
  coderagent
```

The API is then available at:

```text
http://localhost:10000
```

---

# API

The primary API exposes the RAG pipeline through a query endpoint.

## POST `/query`

Example request:

```json
{
  "question": "How does FastAPI define an API route?"
}
```

Example response:

```json
{
  "answer": "FastAPI defines routes using path operation decorators...",
  "route": "retrieve",
  "retrieval_score": 1.0,
  "web_search_used": false,
  "retry_count": 0
}
```

---

# Evaluation

The project contains two evaluation components:

```text
app/evaluation/
├── benchmark.py
└── ragas_evaluator.py
```

## Benchmark

The benchmark runner records:

- Question
- Ground truth
- Generated answer
- Retrieved contexts
- Latency
- Route
- Retrieval score
- Retry count
- Web-search usage

Results are written to:

```text
data/benchmark_results.json
```

## RAGAS

The RAGAS evaluator currently uses:

- Answer Relevancy
- Faithfulness
- Context Precision

---

# Results

Benchmark results will be added after the current benchmark run is finalized.

| Metric | Result |
|---|---:|
| Answer Relevancy | TBD |
| Faithfulness | TBD |
| Context Precision | TBD |
| Average Latency | TBD |

---

# Example Questions

## Repository Retrieval

```text
How does FastAPI define an API route?
```

Expected behavior:

```text
Router
  ↓
Retrieve
  ↓
Qdrant
  ↓
BGE Reranker
  ↓
Retrieval Grader
  ↓
Mistral
```

## Direct Question

```text
What is 2 + 2?
```

Expected behavior:

```text
Router
  ↓
Direct
  ↓
Mistral
```

## Difficult / Insufficient Retrieval

A question that cannot be sufficiently answered from the indexed repository can trigger:

```text
Retrieve
   ↓
Low Relevance
   ↓
Query Rewrite
   ↓
Retry
   ↓
Web Search
   ↓
Answer
```

---

# Why Agentic RAG?

A conventional RAG pipeline generally follows:

```text
Question
   ↓
Retrieve
   ↓
Generate
```

CodeRAGent introduces decision-making around retrieval:

```text
Question
   ↓
Should retrieval be used?
   ↓
Retrieve
   ↓
Is the context relevant?
   ↓
No ──► Rewrite ──► Retry
                  ↓
              Still weak?
                  ↓
              Web Search
                  ↓
               Generate
```

This allows the system to adapt its retrieval strategy instead of blindly generating an answer from the first retrieved documents.

---

# Resume Highlights

- Developed an **Agentic RAG system for software engineering question answering** over technical documentation and open-source code repositories.
- Implemented **LangGraph-based query routing, retrieval grading, query rewriting, and retry workflows**.
- Integrated **Qdrant vector search, embedding-based retrieval, and BGE cross-encoder reranking** for repository-level context retrieval.
- Added **Tavily web-search fallback** when repository context is insufficient.
- Integrated **Mistral 7B through Ollama** for local answer generation.
- Built a **FastAPI REST API and Streamlit frontend** for interactive question answering and agent execution visualization.
- Containerized the backend using **Docker** and configured Qdrant using Docker Compose.
- Implemented **benchmark and RAGAS evaluation** for answer relevancy, faithfulness, context precision, and latency.

---

# Future Improvements

- Multi-repository indexing
- Incremental repository indexing
- GitHub repository integration
- Streaming responses
- Support for additional LLMs
- Improved observability and tracing
- Production cloud deployment
- Expanded benchmark evaluation

---

# License

This project is intended for educational and research purposes.

---

<div align="center">

### CodeRAGent

**Agentic retrieval for software engineering knowledge.**

</div>
