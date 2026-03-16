# memento
> *"I can't remember to forget you"*
`rag-memento` is a lightweight analytics library for RAG pipelines. It tracks which documents in your vector database are actually retrieved, which ones users engage with — and which ones have been silently forgotten.
---
## The Problem

Your vector database grows. Documents get added, re-chunked, updated.
But which ones actually get retrieved? Which ones users ignore?
Which ones haven't appeared in a search result in 60 days?

Without tracking, you're flying blind. rag-memento leaves notes.
---
## Features

- **Retrieval Tracking** — log every search result with rank, score, and query context
- **Engagement Signals** — capture when users act on a retrieved document
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
┌─────────────────────────────────────────────────────────────────┐
│ rag-memento — Corpus Health Report                              │
│ 142 documents tracked · last 30 days                           │
├────────────────────────────┬──────────┬────────────┬───────────┤
│ Document                   │ Retrieved│ Engagement │ Score     │
├────────────────────────────┼──────────┼────────────┼───────────┤
│ onboarding-guide.md        │ 94       │ 61%        │ ████████  │
│ api-reference-v2.md        │ 78       │ 44%        │ ██████    │
│ troubleshooting-faq.md     │ 12       │ 8%         │ ██        │
│ legacy-setup-2021.md       │ 0        │ —          │ ghost 👻  │
└────────────────────────────┴──────────┴────────────┴───────────┘
```
---

## ⚙️ Configuration
```python
memento = Memento(
    db_path="./memento.db",       # SQLite path
    ghost_threshold_days=30,      # Days before a doc is flagged as ghost
    top_k_report=20,              # Documents shown in report
)
```
---
