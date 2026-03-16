# rag-memento
> *"I can't remember to forget you"*
`rag-memento` is a lightweight analytics library for RAG pipelines. It tracks which documents in your vector database are actually retrieved, which ones users engage with — and which ones have been silently forgotten.
---
## The Problem


Your vector database grows. Documents get added, re-chunked, updated.
Old versions linger. Near-identical content gets indexed twice.
Outdated files keep surfacing in results, quietly poisoning your answers.

Without tracking, you're building on top of noise.
rag-memento leaves notes — and reads the ones you left before.
---
## Features

- **Retrieval Tracking** — log every search result with rank, score, and query context
- **Engagement Signals** — capture when users act on a retrieved document
- **Duplicate Detection** — identify semantically near-identical chunks competing for the same queries
- **Obsolete File Flagging** — surface documents superseded by newer versions of the same content
- **Corpus Health Report** — ranked table of your documents by actual usage
- **Ghost File Detection** — surface documents never retrieved in N days
- **Zero Infrastructure** — SQLite by default, no cloud, no account, no friction
- **Integrates in 3 lines** — wraps your existing pipeline, touches nothing else

---
 ## 📦 Installation
```bash
pip install rag-memento
```
---

## 🏃 Quickstart
```python
from rag_memento import Memento

memento = Memento()  # SQLite by default, stored in ./memento.db

# Wrap your vector DB search
results = your_vectordb.search(query)
memento.log_retrieval(results, query=query)

# When a user engages with a source
memento.log_engagement(doc_id="your-doc-id", event="opened")

# See what's working (and what isn't)
memento.report()
```
---

## 📊 Report Output
```
┌─────────────────────────────────────────────────────────────────────────┐
│ rag-memento — Corpus Health Report                                      │
│ 142 documents tracked · last 30 days                                   │
├────────────────────────────┬──────────┬────────────┬────────────────────┤
│ Document                   │ Retrieved│ Engagement │ Status             │
├────────────────────────────┼──────────┼────────────┼────────────────────┤
│ onboarding-guide.md        │ 94       │ 61%        │ ✓ healthy          │
│ api-reference-v2.md        │ 78       │ 44%        │ ✓ healthy          │
│ api-reference-v1.md        │ 71       │ 9%         │ ⚠ obsolete (v2)   │
│ setup-instructions.md      │ 44       │ 38%        │ ⚠ duplicate (×2)  │
│ troubleshooting-faq.md     │ 12       │ 8%         │ ◌ low engagement  │
│ legacy-setup-2021.md       │ 0        │ —          │ 👻 ghost           │
└────────────────────────────┴──────────┴────────────┴────────────────────┘

  👻 ghosts: 12 documents · 💀 obsolete: 5 · ⚠ duplicates: 8 pairs
  Estimated corpus noise: 31% — run memento.cleanup_report() for details
```

---
## 🔍 Duplicate & Obsolete Detection
```python
# Find semantically near-duplicate chunks (cosine similarity threshold)
dupes = memento.get_duplicates(threshold=0.92)
# → [("setup-v1.md::chunk_4", "setup-v2.md::chunk_1", similarity=0.97), ...]

# Find documents likely superseded by a newer version
obsolete = memento.get_obsolete()
# → [{"file": "api-reference-v1.md", "superseded_by": "api-reference-v2.md", ...}]

# Get a full cleanup recommendation
memento.cleanup_report()
# Prints a prioritised action list: what to delete, merge, or re-chunk
```

Duplicate detection compares embedding vectors already in your vector store —
no re-embedding, no extra API calls.

---

## ⚙️ Configuration
```python
memento = Memento(
    db_path="./memento.db",          # SQLite path
    ghost_threshold_days=30,         # Days before a doc is flagged as ghost
    duplicate_threshold=0.92,        # Cosine similarity for duplicate detection
    obsolete_pattern=r"v\d+",        # Regex hint for versioned filename detection
    top_k_report=20,                 # Documents shown in report
)
```

---
## 🔌 Integrations

Works with any vector store. Built-in helpers for:

- **Chroma** — `from rag_memento.integrations import ChromaMemento`
- **Qdrant** — `from rag_memento.integrations import QdrantMemento`
- **LlamaIndex** — drop-in callback handler
- **LangChain** — retriever wrapper

Custom store? `log_retrieval()` accepts any list of dicts with `doc_id` and `score`.

---
## 🏗️ Architecture
```
Your RAG Pipeline
       ↓
  Memento Wrapper               ← the only thing you add
  ├── log_retrieval()
  └── log_engagement()
       ↓
  memento.db (SQLite)
  ├── retrievals
  ├── engagements
  └── file_stats (nightly aggregate)
       ↓
  Analysis Layer
  ├── ghost detection
  ├── duplicate detection        ← compares stored embeddings
  └── obsolete flagging          ← version pattern + retrieval displacement
       ↓
  memento.report()               ← actionable output
```

---

## 🧹 Acting on the Data
```python
# Get ghost files (never retrieved in 30 days)
ghosts = memento.get_ghosts()

# Get semantically near-duplicate document pairs
dupes = memento.get_duplicates()

# Get files likely superseded by newer versions
obsolete = memento.get_obsolete()

# Get high-retrieval, low-engagement docs (candidates for re-chunking)
suspects = memento.get_suspects()

# Export full stats as DataFrame
df = memento.to_dataframe()
```

---

## 📝 License
MPL 2.0 (Mozilla Public License)

---

*Like the movie: the clues were always there. rag-memento just helps you read them.*
