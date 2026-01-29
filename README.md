## 📌 Project Overview

This project implements a production-grade **Data Engineering Pipeline** designed to handle a real-world M&A (Merger & Acquisition) data integration scenario.

**The Business Problem:**
"AtliQ" (a large FMCG parent company) acquired "Sports Bar" (a nutritional startup). The startup stored daily sales transactions in disparate CSV files with inconsistent schemas, typos, and missing data. The parent company needed this data merged into their centralized Data Warehouse for monthly executive reporting.

**The Solution:**
I built a scalable ETL pipeline using **Databricks** and **Medallion Architecture** to ingest, clean, transform, and aggregate the high-volume transactional data, bridging the gap between the startup's raw daily logs and the parent company's monthly analytical requirements.

---

## 🏗 Architecture

The pipeline follows the **Medallion Architecture** (Bronze $\rightarrow$ Silver $\rightarrow$ Gold) to ensure data quality and governance.

![Architecture Diagram](Diagrams/architecture_diagram.png)
*(Note: Ensure your diagram is uploaded to the 'Diagrams' folder in your repo)*

### Data Flow
1.  **Source (AWS S3):** Raw CSV files (Orders, Customers, Products) land in S3 buckets.
2.  **Bronze Layer (Raw Ingestion):**
    * Data is ingested "as-is" using **Schema-on-Read**.
    * Metadata columns (filename, timestamp) are added for lineage tracking.
3.  **Silver Layer (Cleansed & Enriched):**
    * **Data Quality Checks:** Deduplication, null handling, and type casting.
    * **Normalization:** Categorical standardization (e.g., fixing "Bangalore" vs "Bengaluru") using Dictionary Mappings.
    * **Linkage:** Generating **Surrogate Keys (SHA-256 Hashing)** to link child products to parent master data.
4.  **Gold Layer (Aggregated for BI):**
    * **Aggregation:** Rolling up **Daily** transaction grains to **Monthly** reporting grains.
    * **Merging:** Performing **Upserts (SCD Type 1)** into the final Fact tables.

---

## 🛠 Tech Stack

* **Cloud Storage:** AWS S3 (Data Lake)
* **Compute Engine:** Azure Databricks (Apache Spark / PySpark)
* **Storage Format:** Delta Lake (for ACID transactions & Time Travel)
* **Orchestration:** Databricks Workflows (simulated via Notebooks)
* **Language:** Python (PySpark), SQL

---

## 📂 Repository Structure

| File | Description |
| :--- | :--- |
| `0_setup_catalogs.py` | Initializes the Unity Catalog structure (Creating Bronze, Silver, Gold schemas). |
| `1_dim_date_table_creation.py` | Programmatically generates a **Date Dimension** table with fiscal quarters and month attributes. |
| `2_customers_data_processing.py` | Cleans Customer data, fixes city typos using mapping dictionaries, and handles NULLs via lookup tables. |
| `3_products_data_processing.py` | Normalizes Product names using **Regex**, extracts variants (e.g., "60g"), and generates **Hash Keys**. |
| `4_pricing_data_processing.py` | Handles price history using **Window Functions** to determine the valid price per month. |
| `5_full_load_fact.py` | **Historical Load:** Ingests the entire history of sales data and aggregates it for the initial warehouse setup. |
| `6_incremental_load_fact.py` | **CDC Pipeline:** Processes *only* new daily files, re-calculates monthly totals, and merges updates into the Gold layer. |

---

## 🚀 Key Technical Features

### 1. Incremental Data Loading (CDC)
Instead of reprocessing the entire history every day, the `6_incremental_load_fact.py` script identifies only the new files arriving in S3. It uses a **Staging Table** approach to process these deltas and updates the main warehouse efficiently.

### 2. Schema Evolution & Normalization
* **Problem:** The child company used integer IDs (`101`, `102`), while the parent used UUIDs.
* **Solution:** Implemented **SHA-256 Hashing** on product names to generate consistent, collision-free Surrogate Keys that work across distributed systems.

### 3. Advanced Data Cleaning (PySpark)
* **Regex Extraction:** Extracted product weights (e.g., `(60g)`) from unstructured text strings to create structured features.
* **Window Functions:** Used `row_number()` over partitions to handle duplicate pricing logs and identify the "latest valid price" for a given period.

### 4. ACID Transactions (Delta Lake)
Used Delta Lake's `MERGE INTO` (Upsert) syntax to enforce **SCD Type 1** (Slowly Changing Dimensions). This ensures that if a customer updates their address, the record is updated in place without creating duplicates.

---

## 💻 How to Run

1.  **Prerequisites:** Access to a Databricks Workspace and an AWS S3 bucket mounted/configured with access keys.
2.  **Setup:** Run `0_setup_catalogs.py` to create the database schemas.
3.  **Dimensions:** Run the dimension notebooks (`1` through `4`) to populate the static data (Customers, Products, Pricing).
4.  **History:** Run `5_full_load_fact.py` to load historical data.
5.  **Daily Routine:** Schedule `6_incremental_load_fact.py` to run daily for ongoing updates.

---

## 👤 Author

**Shubham Kokare**
* [LinkedIn](https://www.linkedin.com/in/shubham-kokare-350196322)
* [GitHub](https://github.com/Shubhamrkokare)

---
*This project was built to demonstrate advanced Data Engineering capabilities including Data Warehousing, ETL Optimization, and Big Data Processing.*
