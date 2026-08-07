# Azure Databricks Batch ETL Pipeline

## Project Overview

This project implements a batch ETL pipeline using Azure Databricks to process daily transaction files. The pipeline follows the Medallion Architecture (Bronze, Silver, and Gold) to ingest raw transaction data, apply business transformations, and generate reporting tables for analytics. The processed data is stored in Delta tables and visualized using Databricks SQL Dashboards.

---

# Architecture

```
                 Daily Transaction CSV
                         │
                         ▼
        Unity Catalog Volume / ADLS Gen2
                         │
                         ▼
              Bronze Layer (Raw Data)
                         │
                         ▼
       Silver Layer (Business Transformations)
                         │
                         ▼
        Gold Layer (Reporting & Aggregation)
                         │
                         ▼
          Databricks SQL Dashboard
```

---

# Technology Stack

- Azure Databricks
- PySpark
- Delta Lake
- Unity Catalog
- Databricks SQL
- GitHub

---

# Dataset

Input File:

```
inputDataTest.csv
```

The dataset contains daily transaction records with approximately 100,000 records.

---

# Project Structure

```
Azure-Databricks-ETL-Pipeline/

│
├── notebooks/
│      01_Bronze_Ingestion.py
│      02_Silver_Transformation.py
│      03_Gold_Transformation.py
│      Dashboard_SQL.sql
│
├── data/
│      inputDataTest.csv
│
├── screenshots/
│      bronze_table.png
│      silver_table.png
│      gold_tables.png
│      dashboard.png
│
├── README.md
│
└── architecture.png
```

---

# ETL Pipeline

## Bronze Layer

Purpose

- Read the daily CSV file.
- Preserve the raw data.
- Add ingestion metadata.

Metadata Columns

- FILE_DATE
- LOAD_TIMESTAMP
- SOURCE_FILE

Output Table

```
bronze_transactions
```

---

## Silver Layer

Purpose

Clean and transform the data according to business rules.

Business Rules

### CUSTOMER_TYPE

If Age_Band is **70-79** or **80+**, or Gender is **Unknown**, set CUSTOMER_TYPE to **Unspecified**. Otherwise retain the original customer type.

### POSTING_DATE

Use the latest date between transaction_date and posting_date.

### DESCRIPTION_TEXT

Remove:

- #####
- Trailing country code when it matches the COUNTRY_CODE column.

Example

```
SHELL ##### GBR

↓

SHELL
```

### COUNTRY_CODE

Copy the country column.

### EU_FLAG

True if:

- Country belongs to the European Union list.
- Country = GBR and transaction_date <= 31-Jan-2020.

Otherwise False.

### GENDER_CODE

| Gender | Code |
|---------|------|
| Male | M |
| Female | F |
| Unknown | X |

### POST_CODE

Mask UK postcodes.

Example

```
SE23 0AA

↓

SE230**
```

### NOTE_TEXT

Remove:

- New lines
- Tabs
- Hidden characters

Output Table

```
silver_transactions
```

---

## Gold Layer

Purpose

Generate reporting tables.

Gold Tables

- gold_daily_summary
- gold_country_summary
- gold_customer_summary
- gold_eu_summary
- gold_gender_summary
- gold_age_summary
- gold_top_customers

---

# Dashboard

The Databricks SQL dashboard contains:

- Daily Transaction Trend
- Country-wise Transaction Amount
- Customer Type Distribution
- Gender Distribution
- Age Band Summary
- EU vs Non-EU Transactions
- Top 20 Customers

---

# Execution Steps

1. Upload the daily transaction CSV file into the Unity Catalog Volume.

2. Run Notebook

```
01_Bronze_Ingestion
```

3. Run Notebook

```
02_Silver_Transformation
```

4. Run Notebook

```
03_Gold_Transformation
```

5. Execute SQL queries to create dashboard visualizations.

---

# Tables Created

Bronze

```
bronze_transactions
```

Silver

```
silver_transactions
```

Gold

```
gold_daily_summary
gold_country_summary
gold_customer_summary
gold_eu_summary
gold_gender_summary
gold_age_summary
gold_top_customers
```

---

# Performance Optimizations

- Delta Lake storage
- Medallion Architecture
- Single-pass transformations
- Delta tables for ACID compliance
- Business logic separated into Silver layer
- Reporting separated into Gold layer

---

# Monitoring

The pipeline validates:

- Total records
- Invalid record IDs
- Duplicate record IDs
- Successful table creation

---

# Assumptions

- One CSV file is received every day.
- The dataset contains Age_Band instead of Age.
- Age_Band values 70-79 and 80+ represent customers aged 70 years or above.
- Invalid record IDs are removed during Silver processing.
- UK postcode validation is performed using a regular expression.

---

# Future Improvements

- Auto Loader for incremental file ingestion.
- Databricks Workflows for orchestration.
- Audit logging.
- Quarantine table for invalid records.
- Data quality framework.
- CI/CD pipeline using Azure DevOps or GitHub Actions.

---

# Author

Name: Saitajes

Project:
Azure Databricks Batch ETL Pipeline
