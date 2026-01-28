

# 📕 README.md

## **Data Ingestion Team**

---

## 🧭 Team Overview

The **Data Ingestion Team** is responsible for designing and implementing the **entry point of the Real-Time Financial Fraud Detection & Monitoring System**. This team ensures that historical financial transaction data is **reliably converted into real-time streaming events** and ingested into the platform using **fault-tolerant, scalable streaming pipelines**.

This team lays the foundation for all downstream analytics, machine learning, and dashboards by delivering **high-quality raw streaming data** into the **Bronze Layer** of the Medallion Architecture.

---

## 🎯 Key Responsibilities

### 🔄 Streaming Data Simulation

* Convert **historical IEEE-CIS fraud data** into **continuous live-like streams**
* Emit transactions as incremental files (JSON/CSV)
* Simulate real-world transaction arrival patterns

### 📥 Streaming Ingestion

* Ingest streaming files using **Databricks Auto Loader**
* Enforce schema consistency
* Capture ingestion metadata
* Ensure **append-only**, **exactly-once** semantics

### 🥉 Bronze Layer Management

* Store **raw, unmodified streaming data**
* Preserve source fidelity for auditing and reprocessing
* Maintain ingestion lineage and metadata

---

## 📊 Data Sources

### Dataset Used

**IEEE-CIS Fraud Detection Dataset (Kaggle)**
A large-scale, industry-standard dataset widely used for fraud detection research.

### Files Handled

| File Name               | Purpose                                          |
| ----------------------- | ------------------------------------------------ |
| `train_transaction.csv` | Historical transaction records with fraud labels |
| `test_transaction.csv`  | Unlabeled transaction data                       |
| `train_identity.csv`    | Device and network metadata                      |
| `test_identity.csv`     | Identity metadata for test data                  |
| `sample_submission.csv` | Reference file (not ingested)                    |

> ⚠️ **Important:**
> `sample_submission.csv` is **not ingested**. It is used only for Kaggle submissions and has **no role in streaming ingestion**.

---

## 🧠 Dataset Characteristics

* **Total Transactions:** 590,540
* **Fraud Rate:** ~3.5%
* **Time Span:** 6 months
* **Join Key:** `TransactionID`
* **Label:** `isFraud` (binary)

### Critical Insight (Fraud Labeling Logic)

Fraud labeling is **account-based**, not transaction-based.
Once a chargeback occurs, **all subsequent transactions linked to the same account or email are labeled as fraud**.

This impacts:

* Streaming simulation realism
* ML interpretation downstream

---

## 🏗️ Ingestion Architecture

```
Historical CSV Files
        ↓
Python Streaming Generator
        ↓
Streaming Source Folder
        ↓
Databricks Auto Loader
        ↓
Bronze Layer (Delta Table)
```

✔ Fully streaming
✔ Fault-tolerant
✔ Schema-enforced
✔ Free & open-source

---

## 📂 Folder Structure (Team Scope)

```
data/
├── historical/
│   ├── train_transaction.csv
│   ├── test_transaction.csv
│   ├── train_identity.csv
│   └── test_identity.csv
│
├── streaming_source/
│   └── txn_<timestamp>.csv
```

---

## 🔁 Streaming Data Generator

### Purpose

Simulates **real-time transaction events** from historical data.

### Design Principles

* Micro-batch file generation
* Time-based emission
* Stateless generation
* Reproducible & configurable

### Generator Logic

* Randomly sample historical rows
* Attach:

  * Event ID
  * Event timestamp
* Write small batches every few seconds

### Sample Generator Code

```python
import pandas as pd
import time, uuid

src = "/dbfs/FileStore/data/historical/train_transaction.csv"
out = "/dbfs/FileStore/data/streaming_source/"

df = pd.read_csv(src)

while True:
    batch = df.sample(10)
    batch["event_id"] = [str(uuid.uuid4()) for _ in range(len(batch))]
    batch["event_timestamp"] = pd.Timestamp.utcnow()
    batch.to_csv(out + f"txn_{int(time.time())}.csv", index=False)
    time.sleep(5)
```

---

## 📥 Bronze Layer Ingestion

### Technology Used

* **Databricks Structured Streaming**
* **Auto Loader (cloudFiles)**
* **Delta Lake**

### Bronze Table Characteristics

* Append-only
* No transformations
* Schema enforced
* Ingestion metadata captured

### Bronze Table Schema Includes

* All raw transaction fields
* `event_timestamp`
* `_ingestion_timestamp`
* `_source_file`
* `_file_size`
* `_file_mod_time`

### Sample Ingestion Query

```sql
CREATE OR REPLACE TABLE stream_bronze_data
USING DELTA
AS
SELECT *
FROM STREAM read_files(
  '/FileStore/data/streaming_source',
  format => 'csv',
  header => true
);
```

---

## 🥉 Bronze Layer Design Principles

| Principle          | Reason                 |
| ------------------ | ---------------------- |
| Raw data only      | Enables reprocessing   |
| No business logic  | Separation of concerns |
| Append-only        | Streaming safety       |
| Metadata preserved | Audit & lineage        |

---

## 🔐 Data Quality & Reliability

* Schema inference + enforcement
* Corrupt records captured via `_rescued_data`
* Fault-tolerant ingestion
* Exactly-once processing semantics

---

## 🚫 What This Team Does NOT Do

❌ Data cleaning
❌ Feature engineering
❌ Aggregations
❌ ML logic
❌ Fraud scoring

These are handled by **Data Transformation** and **ML Analytics teams**.

---

## 🧠 How This Team Adds Value

> A fraud detection system is only as reliable as its ingestion layer.

This team ensures:

* Continuous data availability
* Real-time realism
* Reproducibility
* Strong architectural foundation

---

##  Summary

> “The Data Ingestion Team converts historical IEEE-CIS fraud data into real-time streaming events and ingests them using Databricks Auto Loader into a fault-tolerant Bronze layer, forming the foundation of the entire fraud detection pipeline.”

---
