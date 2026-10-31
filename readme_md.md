# Smart Data Quality Sentinel 🛡️

**Smart Data Quality Sentinel** is an end-to-end inventory data quality and intelligence pipeline built with Databricks, PySpark, and Delta Lake. 

The project validates inventory data, detects common data-quality issues, processes newly ingested files incrementally, and generates actionable analytics to support inventory monitoring and business decision-making.

---

## 📌 Problem Statement

Inventory datasets in enterprise environments often suffer from missing fields, duplicate entries, outdated timestamps, and pricing inconsistencies. If left unmanaged, these data defects propagate to downstream reporting, leading to unreliable business intelligence, incorrect inventory replenishment, and lost revenue.

This project addresses those challenges by building an automated, scalable pipeline that:
* **Identifies** data-quality anomalies in incoming inventory records.
* **Classifies** records based on validation rules and assigns severity levels (`CRITICAL`, `WARNING`, `NONE`).
* **Processes** incoming data incrementally without needing to rebuild the entire Silver dataset.
* **Produces** actionable insights, including restock candidates and negative-margin alerts.
* **Tracks** historical data-quality scores and metrics across execution runs.

---

## 🏗️ System Architecture

```
                  Inventory CSV Files
                          |
                          v
                 Databricks Volumes
                          |
                          v
                 Ingestion Pipeline
                          |
                          v
                  Bronze Layer
              (Raw Inventory Data)
                          |
                          v
                Data Quality Engine
                          |
                          v
                  Silver Layer
          (Validated Inventory Records)
                          |
                          v
                Business Intelligence
                          |
             +------------+------------+
             |            |            |
             v            v            v
          Restock     Negative       Quality
        Candidates     Margins      Metrics
             |            |            |
             +------------+------------+
                          |
                          v
                 Gold Layer & Dashboard
```

### Data Medallion Layers

* **Bronze Layer:** Raw data storage capturing ingested inventory records along with file lineage and ingestion metadata.
* **Silver Layer:** Validated, enriched, and deduplicated inventory records containing quality tags and severity indicators.
* **Gold Layer:** Business-level aggregated tables providing metrics for restock planning, negative-margin detection, and quality trends.

---

## ✨ Key Features

### 1. Automated Data Quality Validation Engine
A PySpark validation engine scans incoming records against five key defect rules:

| Data Quality Issue | Description | Default Severity |
| :--- | :--- | :--- |
| `MISSING_SUPPLIER_ID` | Supplier information is missing or null | `WARNING` |
| `DUPLICATE_PRODUCT_ID` | Multiple records share the same product ID | `CRITICAL` |
| `SELLING_PRICE_BELOW_COST` | Selling price is lower than cost price (Negative Margin) | `WARNING` |
| `NEGATIVE_QUANTITY` | Inventory stock count is below zero | `CRITICAL` |
| `STALE_INVENTORY` | Record has not been updated within the threshold period | `WARNING` |

Each record is stamped with an overall status (`VALID` or `INVALID`) and assigned a cumulative severity score.

### 2. Incremental Data Processing & MERGE Logic
Instead of performing full table overwrites:
* The engine detects newly uploaded CSV files in Databricks Volumes.
* It identifies records affected by new or updated inventory data.
* It revalidates affected records (including existing cross-record duplicates sharing the same `product_id`).
* Delta Lake `MERGE` operations incrementally update the Silver layer and downstream Gold models.

### 3. Inventory Intelligence & Analytics
* **Restock Candidates:** Pinpoints items requiring reorder based on current stock vs. reorder levels.
* **Negative Margin Alerts:** Identifies pricing errors where selling price falls below unit cost.
* **Data Quality Scorecard:** Aggregates valid/invalid counts, issue distributions, and overall health percentages.

### 4. Databricks Monitoring Dashboard
Visualizes key operational metrics:
* Overall Data Quality Score (%)
* Valid vs. Invalid Record Breakdown
* Quality Issue Type Distribution
* High-Priority Inventory Restock Candidates
* Historical Quality Trend Analysis

---

## 🛠️ Technology Stack

| Component | Technology | Role |
| :--- | :--- | :--- |
| **Platform** | Databricks | Unified Analytics & Execution |
| **Language** | Python / SQL | Pipeline Logic & Queries |
| **Processing** | PySpark | Distributed Data Transformation |
| **Storage** | Delta Lake / Databricks Volumes | Acid Storage & File Management |
| **Orchestration**| Databricks Workflows | Job Scheduling & Pipeline Automation |
| **Visualization**| Databricks Dashboards | Interactive Quality & Risk Monitoring |

---

## 📊 Pipeline Test Results

The pipeline was validated using an inventory dataset containing **50,806** records:

| Metric | Observed Result |
| :--- | :--- |
| **Total Records Processed** | `50,806` |
| **Valid Records** | `47,915` |
| **Invalid Records** | `2,891` |
| **Critical Severity Records** | `1,465` |
| **Warning Severity Records** | `1,426` |
| **Data Quality Score** | **`94.31%`** |
| **Defect Categories Detected** | `5` |

---

## 🛡️ Reliability & Processing Features

* **Ingestion Guardrails:** File-level duplicate check prevents re-processing identical files.
* **Processing Keys & Idempotency:** Guarantees idempotent writes during retries or re-runs.
* **Delta MERGE Optimization:** Updates only changed records to minimize write amplification.
* **No-Pending-Data Check:** Gracefully skips processing when no new files are found in Databricks Volumes.

---

## 🚀 How to Run the Project

1. **Import Notebooks:** Import the pipeline notebooks into your Databricks workspace.
2. **Configure Storage:** Set up your Databricks Volume target paths for incoming CSV files.
3. **Database Setup:** Verify Unity Catalog / Hive Metastore permissions for target schemas.
4. **Bronze Ingestion:** Execute the Ingestion Notebook to process source CSVs into `sentinel_bronze_inventory`.
5. **Quality Engine:** Run the Processing Notebook to validate records and update Silver and Gold tables.
6. **Orchestrate:** Combine notebooks into a Databricks Workflow job.
7. **Monitor:** Connect the Databricks Dashboard to the Gold analytical tables.

---

## 👤 Author & Project Info

* **Project:** Smart Data Quality Sentinel
* **Core Stack:** Databricks | PySpark | Delta Lake | Python | SQL