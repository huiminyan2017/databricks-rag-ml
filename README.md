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