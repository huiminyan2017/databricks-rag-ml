# Enterprise RAG System on Databricks

This project builds an end-to-end Retrieval-Augmented Generation (RAG)
system on Databricks using Azure Compute documentation.

## Motivation
LLMs hallucinate on enterprise infrastructure knowledge.
This system grounds answers in internal documentation.

## Stack
- Databricks (Spark, Delta Lake)
- PyTorch
- Sentence Transformers
- FAISS
- MLflow

## Data
- Azure Compute documentation (GitHub markdown)
- ~3k documents
- ~100k text chunks

## Architecture
[diagram here]

## Status
- [ ] Data ingestion
- [ ] Chunking
- [ ] Embeddings
- [ ] Retrieval
- [ ] RAG inference
- [ ] MLflow
- [ ] Serving

## Demo Script
1) In Databricks, open `07_serving_and_demo.ipynb`.
2) Run:

```python
result = ask("How do I resize an Azure virtual machine?", k=6, retriever="A", do_eval=True)
print(result["answer"])
print([s["url"] for s in result["sources"][:3]])
```

Validate logging:
    SELECT * FROM databricks_rag_demo.default.rag_query_logs ORDER BY created_at DESC LIMIT 5;

Validate evaluation:
    SELECT * FROM databricks_rag_demo.default.rag_evaluations ORDER BY created_at DESC LIMIT 5;

TODO:
---

# Two small “final polish” suggestions (optional)

1) Add `retriever` to logs (A/B) so you can compare quality.
2) Add latency metrics (embed time, retrieval time, LLM time).

If you want, I can add both in a very small patch.