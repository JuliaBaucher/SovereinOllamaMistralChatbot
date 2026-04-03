# SovereinOllamaMistralChatbot

# 🧠 Sovereign AI Local Prototype — Architecture Overview

## 📌 Overview

This project is a **local (sovereign) AI system** running entirely on a laptop.  
It implements a **RAG (Retrieval-Augmented Generation)** pipeline using:

- FastAPI (application logic)
- PostgreSQL + pgvector (knowledge base + semantic search)
- Ollama (local LLM inference)

---

## 🏗️ High-Level Architecture

User → UI → FastAPI (API + RAG) → PostgreSQL (pgvector) + Ollama → Logs/Audit


---

## 🧩 Core Components

### 1. 🗄️ System Layer (PostgreSQL + pgvector)

**Role:** Knowledge base + semantic retrieval

- Stores:
  - text chunks (documents)
  - embeddings (vectors)
  - metadata (source, chunk index)
- Enables:
  - vector similarity search
  - top-K retrieval

**pgvector adds:**
- `VECTOR(n)` column type
- similarity operators (e.g. `<=>`)
- nearest-neighbor search

👉 Important:
- Does **NOT create embeddings**
- Does **NOT understand text**
- Only performs **mathematical similarity**

---

### 2. 🐍 Python Application (.venv)

**Role:** Orchestrator (the “brain” of the system)

Implemented with FastAPI.

Handles:

1. Receive user question  
2. Generate embedding (sentence-transformers)  
3. Query PostgreSQL (pgvector similarity search)  
4. Retrieve top-K relevant chunks  
5. Build prompt (context + instructions + question)  
6. Call Ollama (LLM)  
7. Return answer  

👉 This is where **all logic lives**

---

### 3. 🤖 Ollama Runtime (LLM)

**Role:** Local inference engine

- Runs Mistral (7B) locally
- Exposes HTTP API (`http://localhost:11434`)
- Generates answers from prompt

👉 Important:
- No knowledge by itself  
- Relies on provided context (RAG)

---

## 🔄 End-to-End Flow

User question
↓
FastAPI
↓
Embedding model (MiniLM)
↓
PostgreSQL + pgvector
→ similarity search (top-K)
↓
Relevant context retrieved
↓
Prompt built (context + question)
↓
Ollama (LLM)
↓
Generated answer

---

---

## 🧠 Key Concepts

### Embeddings
- Numerical representation of text
- Similar meaning → similar vectors

### Semantic Search
- Based on vector similarity (not keywords)
- Uses distance metrics (e.g. cosine distance)

### pgvector
- Stores embeddings
- Computes similarity
- Returns top-K nearest chunks

---

## ⚠️ Important Clarifications

- pgvector:
  - ✅ stores vectors
  - ✅ computes similarity
  - ❌ does NOT create embeddings
  - ❌ does NOT understand language

- Python app:
  - ✅ creates embeddings
  - ✅ orchestrates the pipeline

- Ollama:
  - ✅ generates answers
  - ❌ does not retrieve knowledge

---

## 🎯 One-Line Summary

**PostgreSQL = memory**  
**Python = orchestrator**  
**Ollama = AI brain**

---

## 🚀 Purpose of This Architecture

- Fully local (data privacy)
- Simple but production-like design
- Ideal for learning:
  - RAG pipelines
  - embeddings
  - LLM orchestration

## Demonstration (start your server)


```uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload ```

👉 This starts your local server at: http://127.0.0.1:8000

test quesiton 'What is the remote work policy'

```ngrok http 8000```

👉 ngrok creates a public URL like: https://abc123.ngrok.io

Internet → ngrok URL → your laptop:8000 → FastAPI

Access Postgre and pgvector (from bash) : psql "postgresql://postgres:postgres@127.0.0.1:5432/sovereign_ai"


