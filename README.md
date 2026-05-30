## databricks-usecases
#repo for exploring databricks usecases.

# Medallion Data Pipeline Architecture (S3 to BI via Unity Catalog)

This repository contains the data engineering pipeline implementing a Medallion Architecture using **AWS S3** as the landing zone, **Databricks Unity Catalog** for data governance/structuring, and downstream **BI & Analytics** integration.

---

## 🏗️ Architecture Overview

The data flows through a multi-layer refinement process designed to guarantee data quality, ACID compliance, and high performance for reporting.


```

[ Raw Data (S3) ]
│
▼
[ Ingestion & Bronze Schemas ] ──► External Volumes & Managed Tables
│
▼
[ Data Refinement (ETL) ]     ──► Cleansing, De-duplication & Transformation
│
▼
[ Silver Layer ]              ──► Conformed & Optimized Managed Tables
│
▼
[ Aggregation & Modeling ]    ──► Business Rules & Dimensional Modeling
│
▼
[ Gold Layer & BI ]           ──► Performance-Tuned Aggregate Tables ──► BI Tools

```

### 📦 Data Pipeline Layers

1. **Raw Data (S3):** Unstructured and semi-structured objects (JSON, CSV, logs, IoT streams) arrive in AWS S3 buckets.
2. **Bronze Layer (Ingestion):** * **External Volumes** provide direct, secure pointers to the raw cloud objects without exposing AWS credentials.
   * **Bronze Tables** capture raw data natively with minimal transformation, preserving history and appending ingestion timestamps.
3. **Silver Layer (Refinement):** Data is cleansed, filtered, deduplicated, and structured into validated **Managed Delta Tables** representing clean business entities.
4. **Gold Layer (Aggregation):** Data is transformed into subject-area star/snowflake schemas or highly optimized aggregate tables tuned directly for analytical consumption.
5. **BI & Analytics:** PowerBI, Tableau, Looker, and other platforms consume semantic data exclusively from the Gold Layer.

---

## 🗂️ Databricks Unity Catalog Object Hierarchy

Unity Catalog enforces a strict three-tier namespace (`catalog.schema.table_or_volume`) to manage governance and accessibility across the entire pipeline.


```

Metastore (AWS Region level)
└── 📁 dev_catalog (or prod_catalog)
├── 📂 bronze_ingest_schema
│    └── 📄 volumes/
│         └── 📁 raw_files (External Volume pointing to s3://my-bucket/raw/)
│
├── 📂 bronze_schema
│    └── 📊 tables/
│         ├── 📄 raw_transactions (Managed or External Table)
│         └── 📄 raw_user_logs
│
├── 📂 silver_schema
│    └── 📊 tables/
│         ├── 📄 clean_transactions (Managed Delta Table)
│         └── 📄 dim_users
│
└── 📂 gold_schema
└── 📊 tables/
├── 📄 daily_sales_summary (Pre-aggregated for BI)
└── 📄 user_engagement_metrics

```

### 🔍 Hierarchy Component Roles

* **Top-Level Catalog (`dev_catalog`):** Acts as the primary software-isolation layer and overarching governance hub. Data permissions are maintained here.
* **Ingest Schema (`bronze_ingest_schema`):** Contains **External Volumes** pointing securely to your landing S3 buckets. Engineers can interact with files using standard SQL/POSIX paths without managing low-level storage keys.
* **Bronze Schema (`bronze_schema`):** Hosts the initial ingestion tables. These tables mirror the structure of the source S3 objects while ensuring schema validation.
* **Silver Schema (`silver_schema`):** Houses **Managed Delta Tables** containing cleansed and conformed master and fact data. 
* **Gold Schema (`gold_schema`):** Houses optimized, pre-aggregated tables tailored to specific business logic. **BI tools are granted exclusive read access to this schema** to ensure governance and prevent raw-data performance bottlenecks.

---

## 🔐 Governance & Security Model

* **Data Engineers:** Have `CREATE` and `MODIFY` privileges across the Catalog to manage pipelines from Volume to Gold.
* **Data Analysts / BI Service Accounts:** Are granted strict `SELECT` permissions **only** on the `gold_schema`. Raw files (Volumes) and intermediate logic (Bronze/Silver) are abstracted away from reporting tools.

```
