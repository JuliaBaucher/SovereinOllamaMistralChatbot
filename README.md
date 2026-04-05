# SovereinOllamaMistralChatbot

# 🧠 Architecture Overview


## Demo


https://github.com/user-attachments/assets/a923854b-37f1-4592-97f2-c734604bf96e


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

### 4. Fast API

FastAPI = list of URLs → mapped to Python functions

/ {'GET'}
/ask {'POST'}
/health {'GET'}
/metrics {'GET'}

ask request

```
{
  "question": "string"
}
```
ask response

```
{
  "answer": "string",
  "sources": [
    {
      "source": "string",
      "chunk_index": 0,
      "content": "string",
      "score": 0
    }
  ],
  "latency_ms": 0
}
```
ask example
```
{
  "question": "What is the remote work policy?",
  "retrieved_chunks": [
    {
      "source": "company_policy.txt",
      "chunk_index": 0,
      "content": "Employees may work remotely up to three days...",
      "score": 0.12
    }
  ],
  "constructed_prompt": "Context:\n[company_policy.txt#0] ...\n\nQuestion: ...",
  "llm_input": {
    "model": "mistral",
    "prompt": "...full prompt..."
  },
  "llm_output": "Employees may work remotely up to three days per week...",
  "final_response": {
    "answer": "...",
    "sources": [...],
    "latency_ms": 842
  }
}
```

http://127.0.0.1:8000/docs

http://127.0.0.1:8000/redoc

### 5. Logs

```tail -f logs/audit.jsonl```

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

## Demonstration (start your server)

## 🚀 Run the Sovereign AI Demo

### 1. Start the application (Terminal 1)

```bash
cd ~/sovereign-ai-laptop
source .venv/bin/activate
uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

---

### 2. Check Ollama (Terminal 2)

```bash
curl http://127.0.0.1:11434
```

Optional:

```bash
ollama list
```

---

### 3. Show the knowledge base

```bash
cd ~/sovereign-ai-laptop
ls data/kb
cat data/kb/company_policy.txt
```

---

### 4. Open the UI

Open in your browser:

```
http://127.0.0.1:8000
```

Ask:

```
What is the remote work policy?
```

---

### 5. Inspect PostgreSQL + pgvector

Connect to the database:

```bash
psql "postgresql://postgres:postgres@127.0.0.1:5432/sovereign_ai"
```

Then run **each command separately** inside `psql`:

```sql
\pset pager off
```

```sql
\dt
```

```sql
SELECT source, chunk_index, content FROM documents;
```

```sql
SELECT embedding FROM documents LIMIT 1;
```

```sql
\q
```

---

### 6. Show logs

```bash
cd ~/sovereign-ai-laptop
cat logs/app.log
cat logs/audit.jsonl
```

Optional live view:

```bash
tail -f logs/app.log
```

---

### 🧠 Summary

* **FastAPI** → handles API and UI
* **PostgreSQL + pgvector** → stores and retrieves embeddings
* **Ollama (Mistral)** → generates answers locally
* **Logs** → provide full traceability of requests and responses




## Demo with Github

```ngrok http 8000```

👉 ngrok creates a public URL like: https://abc123.ngrok.io

Internet → ngrok URL → your laptop:8000 → FastAPI

Access Postgre and pgvector (from bash) : psql "postgresql://postgres:postgres@127.0.0.1:5432/sovereign_ai"


