### Azure Databricks Workspace

Azure Databricks workspace names must be **unique within a region**. They are exposed via regional DNS endpoints: adb-..azuredatabricks.net. 
Important: Azure Resource Groups do not provide name isolation for Databricks workspaces.
Uniqueness is enforced at the regional control plane level, not at the resource group level.

### Databricks Unity Catalog

Unity Catalog is Databricks’ **central data governance and storage management layer**.

In practical terms, it replaces three legacy mechanisms at once:

1. The legacy Hive metastore
2. Ad-hoc DBFS storage paths
3. Cluster-local permissions

You can think of Unity Catalog as the control plane for all data and AI assets in Databricks.

With Unity Catalog:
	•	Users stop thinking in filesystem paths
	•	Data is addressed by governed names
	•	Permissions are centrally enforced
	•	Lineage and auditing become first-class
Instead of paths like: /mnt/mybucket/data/foo You work with: catalog.schema.table

### Object Model: catalog → schema → table

Unity Catalog enforces a three-level namespace: catalog.schema.table. This is not cosmetic — it defines governance, security, and storage boundaries.

#### Catalog — top-level boundary

A **catalog** is the highest-level container. It typically maps to:
- an organization
- an environment (dev / staging / prod)
- a major data domain

Catalogs define:
- storage isolation
- security boundaries
- governance scope
- ownership

In this project, the workspace automatically created the catalog: databricks_rag_demo (same name as the workspace)

#### Schema — logical grouping inside a catalog

A **schema** is a namespace inside a catalog.

Equivalent terms:
- Schema = database (Postgres terminology)
- Schema = dataset / namespace

Schemas group related tables and allow fine-grained permissions.
Example: databricks_rag_demo.default

#### Table — the actual data

A **table** is:
- a Delta Lake dataset
- managed by Unity Catalog
- transactional, versioned, and secure

Example: databricks_rag_demo.default.raw_azure_compute_docs
Tables are not files — they are governed data objects with lineage, ACLs, and metadata.

#### Why Unity Catalog is required (TODO add more)

Modern Databricks workspaces enforce governed storage:
- Public DBFS root is disabled
- Path-based Delta writes are blocked
- Storage must be centrally managed
- _delta_log must live in managed locations

Unity Catalog is what makes this possible.

Specifically, Unity Catalog:
- Automatically provisions managed ADLS Gen2 storage
  - Azure Data Lake Storage Gen2 (ADLS Gen2)
- Owns the physical storage location
- Manages _delta_log paths
- Enforces ACLs
- Enables saveAsTable(...)
- Enables cross-workspace sharing
- Enables lineage tracking and auditing

Without Unity Catalog:
- Delta tables are unreliable
- Storage permissions break
- Path-based writes fail
- Governance is impossible

This is why Unity Catalog is now the default and required mode for modern Databricks workspaces.

#### Verifying Unity Catalog

After workspace creation, some default catelogs will be created.

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

### Databricks Architecture

┌──────────────────────────────┐
│        Databricks UI         │
└─────────────┬────────────────┘
              │
        (Spark jobs)
              │
┌─────────────▼───────────────┐
│     Compute (clusters)      │  ← 1–N VMs
│       - Ephemeral           │
│       - Replaceable         │
│       - Stateless           │
└─────────────┬───────────────┘
              │
              ▼
┌──────────────────────────────┐
│   Persistent Cloud Storage   │
│      (ADLS / S3 / GCS)       │
│      - Delta tables          │
│      - _delta_log            │
│      - Unity Catalog managed │
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

Key Design Principles
	•	Storage is persistent
	•	Compute is disposable
	•	Governance is centralized
	•	Tables are objects, not files