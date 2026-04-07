# Healthcare Data Engineering Pipeline — Azure Databricks

[![Azure Databricks](https://img.shields.io/badge/Azure-Databricks-0078D4?style=flat-square&logo=microsoftazure&logoColor=white)](https://azure.microsoft.com/en-us/products/databricks)
[![PySpark](https://img.shields.io/badge/PySpark-3.x-E25A1C?style=flat-square&logo=apachespark&logoColor=white)](https://spark.apache.org/)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Enabled-00ADD8?style=flat-square)](https://delta.io/)
[![Architecture](https://img.shields.io/badge/Architecture-Medallion-6C3483?style=flat-square)](https://www.databricks.com/glossary/medallion-architecture)
[![Status](https://img.shields.io/badge/Status-Completed-2ECC71?style=flat-square)]()

---

## Overview

A production-style, end-to-end healthcare data engineering pipeline built on **Azure Databricks** using **PySpark** and **Delta Lake**. The pipeline processes **3.9M+ synthetic healthcare records** across six interconnected datasets and implements a full **Medallion Architecture (Bronze → Silver → Gold)** with incremental ingestion, schema drift handling, data quality validation, and query performance optimization.

Synthetic data was generated using [Synthea](https://synthea.mitre.org/) — an open-source patient population simulator — making this a realistic, privacy-safe simulation of a real-world healthcare data engineering workflow.

---

## Key Results

| Metric | Value |
|--------|-------|
| Total records processed | 3,907,541 |
| Dataset scale increase (run_1 → run_2) | ~900% |
| Query performance improvement (Bronze → Gold) | ~55% faster |
| Data quality checks implemented | Null, duplicate, referential integrity, invalid cost |
| Deployment environment | Azure Databricks cluster |

---

## Architecture

The pipeline follows the **Medallion Architecture** pattern — a layered approach that progressively refines raw data into analytics-ready outputs.

```
Raw CSV Data (Synthea)
run_1 / run_2
        │
        ▼
┌───────────────────────────────────┐
│           BRONZE LAYER            │
│  Raw ingestion with metadata      │
│  - Append-only from each run      │
│  - run_id + ingestion_timestamp   │
│  - Schema alignment               │
└───────────────┬───────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│           SILVER LAYER            │
│  Cleaned and validated data       │
│  - Standardized schema            │
│  - Deduplication                  │
│  - Referential integrity checks   │
│  - Null and invalid value removal │
└───────────────┬───────────────────┘
                │
                ▼
┌───────────────────────────────────┐
│            GOLD LAYER             │
│  Business-ready analytics tables  │
│  - Patient journey summaries      │
│  - Provider performance rankings  │
│  - Cost and utilization trends    │
│  - Time-based analytics           │
└───────────────────────────────────┘
```

---

## Dataset Scale

| Dataset | Records |
|---------|---------|
| Patients | 12,736 |
| Encounters | 742,364 |
| Conditions | 461,418 |
| Procedures | 2,065,888 |
| Medications | 623,172 |
| Providers | 1,963 |
| **Total** | **3,907,541** |

---

## Pipeline Design

### Bronze — Raw Ingestion

- Loads raw CSV files from versioned run folders (`run_1`, `run_2`)
- Appends new data with metadata fields: `run_id` and `ingestion_timestamp`
- Handles schema drift and column type mismatches across runs
- Preserves full ingestion history for auditability

### Silver — Transformation and Validation

- Standardizes schema across all six datasets
- Deduplicates records using primary key logic
- Enforces referential integrity between patients, encounters, and clinical tables
- Removes null and invalid values (e.g., negative cost fields)

### Gold — Analytics Layer

- Pre-aggregated tables optimized for downstream consumption
- Patient journey analysis: encounter timelines, condition progressions
- Provider performance rankings: encounter volume, patient outcomes
- Cost and utilization trends: procedure costs, medication frequency
- Time-based analytics: monthly and quarterly aggregations

---

## Incremental Pipeline Design

Each new data run follows this sequence:

```
1. New CSV files uploaded to run_n/ folder
2. Bronze layer  →  Append new records with run metadata
3. Silver layer  →  Rebuild with deduplication applied
4. Gold layer    →  Refresh aggregated analytics tables
```

This design supports continuous data growth without reprocessing historical records from scratch.

---

## Performance Optimization

Query benchmarks measured on the same analytical query across layers:

| Layer | Execution Time |
|-------|---------------|
| Bronze (raw) | ~2.00 sec |
| Gold (optimized) | ~0.91 sec |
| **Improvement** | **~55% faster** |

Optimization techniques applied:
- Pre-aggregated data models in the Gold layer
- Partition pruning using ingestion metadata
- Leveraged Spark DAG and lazy evaluation for efficient execution planning
- Delta Lake columnar storage for faster analytical scans

---

## Data Quality Checks

Implemented in a dedicated validation notebook (`04_data_quality_checks.py`):

| Check | Description |
|-------|-------------|
| Null validation | Flags missing values in critical fields |
| Duplicate detection | Identifies repeated records by primary key |
| Referential integrity | Validates foreign key relationships across datasets |
| Invalid cost values | Catches negative or zero-cost procedure records |

All checks run post-Silver transformation to gate data before it reaches Gold.

---

## Project Structure

```
healthcare-databricks-pipeline/
│
├── notebooks/
│   ├── 01_bronze_ingestion.py          # Raw CSV ingestion with metadata
│   ├── 02_silver_transformation.py     # Cleaning, deduplication, validation
│   ├── 03_gold_analytics.py            # Business-ready aggregation layer
│   ├── 04_data_quality_checks.py       # Automated quality validation
│   └── 05_incremental_bronze_merge.py  # Incremental run ingestion logic
│
├── data/
│   ├── run_1/                          # Initial dataset (Synthea batch 1)
│   └── run_2/                          # Incremental dataset (Synthea batch 2)
│
└── README.md
```

---

## Setup and How to Run

### Prerequisites

- Azure Databricks workspace (Standard or Premium tier)
- Databricks cluster with Spark 3.x and Delta Lake enabled
- Synthea synthetic data files (or any compatible CSV healthcare datasets)

### Step 1 — Upload Data

Upload the `run_1/` and `run_2/` folders to DBFS:

```
/FileStore/healthcare_pipeline/data/run_1/
/FileStore/healthcare_pipeline/data/run_2/
```

### Step 2 — Import Notebooks

Import all notebooks from the `notebooks/` folder into your Databricks workspace.

### Step 3 — Run in Order

Execute notebooks in sequence:

```
01_bronze_ingestion.py
02_silver_transformation.py
03_gold_analytics.py
04_data_quality_checks.py
```

For incremental runs (run_2 onwards):

```
05_incremental_bronze_merge.py
02_silver_transformation.py
03_gold_analytics.py
```

### Step 4 — Verify Quality

Run `04_data_quality_checks.py` after Silver transformation to validate the dataset before Gold layer refresh.

---

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Compute | Azure Databricks |
| Processing | PySpark 3.x |
| Storage format | Delta Lake (DBFS) |
| Data source | Synthea synthetic healthcare data |
| Architecture pattern | Medallion (Bronze / Silver / Gold) |

---

## Key Engineering Concepts Demonstrated

- Incremental ETL pipeline design with append-only Bronze ingestion
- Schema drift handling in distributed PySpark environments
- Multi-layer data quality validation gating production analytics
- Query performance optimization using pre-aggregated Gold models
- Spark DAG understanding and lazy evaluation optimization
- Production-style project structure with modular notebooks

---

## Planned Enhancements

- Streaming ingestion via Apache Kafka for real-time Bronze updates
- Delta Lake MERGE operations for upsert-based Silver refresh
- Pipeline orchestration using Databricks Workflows
- Power BI dashboard connected to Gold layer via DirectQuery

---

## About the Data

All data is synthetically generated using [Synthea](https://synthea.mitre.org/), an open-source patient population simulator maintained by MITRE Corporation. No real patient data is used. Synthea generates clinically realistic but entirely fictional patient records — making it the industry standard for healthcare data engineering projects and research.
