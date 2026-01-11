### Databricks Unity Catalog

#### Azure Databricks Workspace

Azure Databricks workspace names must be **unique within a region**.

They are exposed via regional DNS endpoints: adb-..azuredatabricks.net

Resource groups do NOT provide name isolation.

#### What is Unity Catalog?

Unity Catalog is Databricks’ **central data governance and storage management layer**.

In practical terms, it replaces three legacy mechanisms at once:

1. The legacy Hive metastore
2. Ad-hoc DBFS storage paths
3. Cluster-local permissions

Think of Unity Catalog as: The control plane for all data and AI assets in Databricks.

With Unity Catalog, users stop thinking in filesystem paths and instead work with **governed data objects**.

#### Object Model: catalog → schema → table

Unity Catalog enforces a three-level namespace: catalog.schema.table

#### Catalog — top-level boundary

A **catalog** is the highest-level container. It typically maps to:
- an organization
- an environment (dev / prod)
- a major data domain

Catalogs define:
- storage boundaries
- security boundaries
- governance scope

In my workspace databricks_rag_demo, the default catalog name is: databricks_rag_demo

### Schema — logical grouping inside a catalog

A **schema** is a namespace inside a catalog.

Equivalent terms:
- Schema = database (Postgres terminology)
- Schema = dataset / namespace

Schemas group related tables and allow fine-grained permissions.

Example: databricks_rag_demo.default

### Table — the actual data

A **table** is:
- a Delta Lake dataset
- managed by Unity Catalog
- transactional, versioned, and secure

Example: databricks_rag_demo.default.raw_azure_compute_docs

#### Why Unity Catalog is required (TODO add more)

Modern Databricks workspaces:
- disable public DBFS root
- block path-based Delta writes
- require managed storage

Unity Catalog:
- provisions managed ADLS storage automatically
- manages `_delta_log`
- enables `saveAsTable(...)`

Without UC, Delta tables cannot be created reliably.

#### Verifying Unity Catalog

After workspace creation:

```sql
SHOW CATALOGS;
```

Observed catalogs:
	•	databricks_rag_demo
	•	samples
	•	system

```sql
SHOW SCHEMAS IN databricks_rag_demo;
```

Observed schemas:
	•	default
	•	information_schema

This confirms Unity Catalog is active and the metastore is attached.

## 3️⃣ Notebook: `01_ingest_azure_compute_docs.ipynb`

This notebook **should be exactly what you ran**, with minimal markdown commentary.

### What to include in the notebook

#### Notebook title (Markdown cell)

```markdown
# 01 – Ingest Azure Compute Docs into Unity Catalog

This notebook downloads Azure Compute documentation from GitHub,
cleans Markdown content, and writes the data into a Unity Catalog–managed
Delta table.



┌──────────────────────────────┐
│        Databricks UI         │
└─────────────┬────────────────┘
              │
      (Spark jobs)
              │
┌─────────────▼───────────────┐
│     Compute (clusters)       │  ← 1–N VMs
│  - Ephemeral                 │
│  - Replaceable               │
│  - Stateless                 │
└─────────────┬───────────────┘
              │
              ▼
┌──────────────────────────────┐
│   Persistent Cloud Storage   │
│  (ADLS / S3 / GCS)           │
│  - Delta tables              │
│  - _delta_log                │
│  - Unity Catalog managed     │
└──────────────────────────────┘

Azure Subscription
 └─ Azure Databricks Workspace
     ├─ Unity Catalog Metastore
     │   ├─ Catalogs
     │   │   ├─ Schemas
     │   │   │   └─ Tables (Delta)
     │   │   └─ ...
     │   └─ Managed Storage (ADLS Gen2)
     └─ Compute (Clusters, ephemeral)

• Catalog + storage are bound to the workspace/metastore
• Clusters are ephemeral and disposable