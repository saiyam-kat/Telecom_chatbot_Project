# 📡 RAG Telecom Support Chatbot

A Retrieval-Augmented Generation (RAG) customer care chatbot for telecom support. It answers questions about mobile connectivity, billing, SIM issues, and roaming by retrieving relevant context from three separate knowledge sources and generating grounded responses with Qwen3-32B via Groq.

**Live demo:** _add your deployed Streamlit link here once deployed_

---

## Why this project

Most chatbot demos answer from a single document. This one merges **three independent knowledge sources** — FAQs, real resolved support tickets, and a full PDF guide — into a single retrieval step, so answers can be grounded in policy text, real precedent, and long-form documentation at once.

---

## Architecture

```
User question
     │
     ▼
Merged Retriever (top-3 from each store)
  ├── ChromaDB · faq        (FAQ entries from CSV)
  ├── ChromaDB · tickets    (resolved support tickets from SQLite)
  └── ChromaDB · guides     (PDF guide, chunked)
     │
     ▼
ChatPromptTemplate → Qwen3-32B (Groq) → Streamed Answer
```

**Embedding model:** `sentence-transformers/all-MiniLM-L6-v2` (runs locally via HuggingFace, no API cost)
**LLM:** `qwen/qwen3-32b` served by [Groq](https://groq.com) (low-latency inference)
**Orchestration:** LangChain Runnables (`retriever | prompt | llm | parser`)

---

## Features

- 🔍 **Multi-source retrieval** — pulls context from FAQs, past tickets, and a PDF guide simultaneously, tagging each result by source
- ⚡ **Streaming responses** — answers appear token-by-token instead of waiting for the full generation
- 💬 **Two interfaces** — a Streamlit web UI and a CLI, both built on the same core RAG chain
- 🧩 **Modular ingestion** — each data source has its own ingestion script, so re-indexing one source doesn't require rebuilding the others

---

## Project Structure

```
Telecom_chatbot_Project/
├── app.py              # Streamlit web UI
├── main.py             # CLI entry point
├── rag_chain.py        # Builds the LangChain RAG chain
├── retriever.py         # Merges the three Chroma retrievers
├── ingest_faq.py        # Loads data/faq.csv → Chroma 'faq' collection
├── ingest_tickets.py    # Loads data/tickets.db → Chroma 'tickets' collection
├── ingest_pdf.py         # Loads data/telecom_guide.pdf → Chroma 'guides' collection
├── data/
│   ├── faq.csv
│   ├── tickets.db
│   ├── telecom_guide.pdf
│   ├── seed_tickets.py
│   └── generate_pdf.py
├── chroma_store/        # Persisted vector database (built at ingest time)
├── pyproject.toml
└── .env.example
```

---

## Setup

### Prerequisites
- Python 3.11+
- [uv](https://docs.astral.sh/uv/) (recommended) or pip
- A free [Groq API key](https://console.groq.com)
- A free [HuggingFace token](https://huggingface.co/settings/tokens) (read access only)

### 1. Clone and install
```bash
git clone https://github.com/saiyam-kat/Telecom_chatbot_Project.git
cd Telecom_chatbot_Project
uv sync          # or: pip install -e .
```

### 2. Configure environment variables
```bash
cp .env.example .env
```
Fill in `.env`:
```
GROQ_API_KEY=your_groq_api_key_here
HF_TOKEN=your_huggingface_token_here
```

### 3. Build the vector store
```bash
python ingest_faq.py
python ingest_tickets.py
python ingest_pdf.py
```

### 4. Run it
**Web UI:**
```bash
streamlit run app.py
```
Opens at `http://localhost:8501`.

**CLI:**
```bash
python main.py
```

---

## Data Sources

| Collection | Source file | Granularity |
|---|---|---|
| `faq` | `data/faq.csv` | 1 document per FAQ row |
| `tickets` | `data/tickets.db` | 1 document per resolved ticket |
| `guides` | `data/telecom_guide.pdf` | Chunks of 600 chars, 100-char overlap |

Every query retrieves the top 3 results from each collection — 9 context documents total — before generation.

---

## Regenerating Seed Data
```bash
python data/seed_tickets.py     # reseed the SQLite ticket database
python data/generate_pdf.py     # regenerate the PDF guide
```
Re-run the matching ingest script after regenerating.

---

## Tech Stack

`Python` · `LangChain` · `ChromaDB` · `Groq (Qwen3-32B)` · `HuggingFace sentence-transformers` · `Streamlit` · `SQLite`
