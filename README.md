### Enterprise RAG System on Databricks (Azure)

This project implements an end-to-end Retrieval-Augmented Generation (RAG) system on Databricks using Azure Compute documentation as the knowledge base. The system grounds LLM responses in authoritative internal documentation to reduce hallucinations and improve factual accuracy.

It demonstrates how to build a production-style RAG pipeline with ingestion, chunking, embedding, retrieval, generation, logging, and evaluation — all with enterprise-grade data governance using Unity Catalog.

### Motivation

Large Language Models (LLMs) hallucinate when asked about specialized or enterprise-specific knowledge such as:
- Cloud infrastructure details
- Internal documentation
- Configuration guides
- Platform-specific behaviors

This system mitigates hallucinations by:
1.	Retrieving relevant documentation chunks
2.	Injecting them into the prompt
3.	Forcing the model to ground its answer in retrieved content
4.	Logging and evaluating answer quality

### Tech Stack

Data & Compute
- Databricks
- Apache Spark
- Delta Lake
- Unity Catalog

LLM & Embeddings
- Azure OpenAI (chat + embeddings)

Retrieval
- Brute-force cosine similarity (Option A)
- Databricks Vector Search (Option B)

Evaluation & Observability
- LLM-as-judge
- Structured logging
- Delta-based metrics tables

### Data
- Source: Azure Compute documentation (Markdown from GitHub)
- Format: Raw .md files → cleaned text → chunked → embedded
- Size:
  - ~XXX documents (TODO update)
  - ~XXX chunks (TODO update)

### Architecture
User Query
   ↓
Embedding (Azure OpenAI)
   ↓
Retriever (Brute-force or Vector Search)
   ↓
Top-K Chunks
   ↓
Prompt Assembly
   ↓
LLM (Azure OpenAI)
   ↓
Answer
   ↓
Logging + Evaluation (Delta Tables)

### Project Structure
00_install_deps_and_restart.ipynb  # Install dependent packages and restart kernel
00_constants.ipynb                 # Centralized table names, config
00_utils.ipynb                     # Reusable functions (retrieval, logging, eval, etc.)
00_init_openai_client.ipynb        # Azure OpenAI initialization

01_ingest.ipynb                    # Data ingestion
02_chunk.ipynb                     # Chunking
03_embed.ipynb                     # Embeddings
04_retrieve_and_generate.ipynb     # Retrieve and LLM inference

05_rag_logging.ipynb               # Logging schema + demo
06_rag_evaluation.ipynb            # Evaluation schema + demo
07_serving_and_demo.ipynb          # Demo + UI

99_cleanup_environment.ipynb       # Environment reset

### Features
- Deterministic chunk IDs
- Category-balanced sampling
- Stable embeddings
- Dual retrieval backends (A/B)
- Full query logging
- Automated evaluation
- Schema-safe Delta writes
- Unity Catalog governance
- Reproducible pipelines

### Status
- Data ingestion
- Chunking
- Embeddings
- Retrieval
- RAG inference
- Logging
- Evaluation
- Demo UI
- MLflow integration (optional)
- Serving endpoint (optional)

### Prerequisites

Azure
You need to create below resources in Azure portal:
1. Azure OpenAI resource
2. Databricks workspace (Premium tier for Unity Catalog)
3. Deployed Azure OpenAI models by naviagting to Azure Foundry portal:
    - Chat model (e.g. gpt-4o-mini)
    - Embedding model (e.g. text-embedding-3-small)
    change 00_constants.ipynb if the model deployment name is not above

Secrets: (see 00_init_openai-client.ipynb)
    AZURE_OPENAI_API_KEY=
    AZURE_OPENAI_ENDPOINT=

### How to Run
Run notebooks in order:
1. 01_ingest.ipynb
2. 02_chunk.ipynb
3. 03_embed.ipynb
4. 04_retrieve_and_generate.ipynb
5. (optional) 05_rag_logging.ipynb
6. (optional) 06_rag_evaluation.ipynb
7. 07_serving_and_demo.ipynb

### Demo Script

    In Databricks, open 07_serving_and_demo.ipynb.
    Run:
    ```python
    result = ask(
        "How do I resize an Azure virtual machine?",
        k=6,
        retriever="A",
        do_eval=True
    )

    print(result["answer"])
    print([s["url"] for s in result["sources"][:3]])
    ```

    Inspect log:
    ```python
    spark.sql(f"""
        SELECT *
        FROM {RAG_LOG_TABLE}
        ORDER BY created_at DESC
        LIMIT 5
        """).display()
    ```

    Inspect Evaluation:
    ```python
    spark.sql(f"""
        SELECT *
        FROM {RAG_EVAL_TABLE}
        ORDER BY created_at DESC
        LIMIT 5
        """).display()
        ```

### Observability Design

Query Logs

Tracks:
- Question
- Retrieved chunks
- Prompt
- Answer
- Models used
- Timestamp

Evaluation Table

Scores:
- Retrieval relevance
- Answer relevance
- Faithfulness

This enables:
- Regression detection
- Prompt iteration
- Retriever comparison
- Hallucination analysis

### Cleanup

To reset the environment: 
    99_cleanup_environment.ipynb
Drops all tables and deletes local temp data.

### Future Improvements
- Add retriever type (A/B) to logs
- Add latency metrics
- Add MLflow experiment tracking
- Add feedback loop
- Add reranking
- Add hybrid search
- Add REST API