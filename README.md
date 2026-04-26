# century-health-data-engineer-assignment
End-to-end healthcare data pipeline using Databricks, PySpark, Delta Tables, and Medallion Architecture with ETL, data quality checks, SQL analytics, and Gold reporting tables.

## Project Overview

This project was completed as part of the Century Health Data Engineer Take-Home Assignment.

The objective was to design and implement a modern healthcare data pipeline using Medallion Architecture principles. The raw datasets contained multiple file formats and real-world data quality issues such as missing values, malformed dates, duplicate records, inconsistent key naming, and packed symptom attributes.

The final solution was built in Databricks Community Edition using PySpark and Spark SQL.

---

# Tech Stack

- Databricks Community Edition
- PySpark
- Spark SQL
- Delta Tables
- Python
- Pandas (limited export support)
- GitHub

---

# Source Datasets

The assignment included five datasets:

1. patients.csv  
2. symptoms.csv  
3. medications.csv  
4. conditions.xlsx *(converted to CSV for Databricks compatibility)*  
5. encounters.parquet

---

# Project Architecture

The solution follows the Medallion Architecture:

Raw Files → Bronze Layer → Silver Layer → Gold Layer

## Bronze Layer
Stores raw ingested source data with minimal transformation.

## Silver Layer
Performs cleansing, datatype correction, null handling, deduplication, and schema standardization.

## Gold Layer
Creates analytics-ready business tables:

- gold.symptoms_observation
- gold.master_health

---
## Pipeline Orchestration Architecture

The project was operationalized using Databricks Jobs workflow orchestration.

### Task Dependency Flow

1. setup_schema_and_load  
2. 02_bronze_layer  
3. 03_silver_layer  
4. 04_gold_layer  
5. 05_validation  

This ensures automated sequential execution of the Medallion pipeline from ingestion through validation.

# Folder Structure

```text
century-health-data-engineer-assignment/
│── README.md
│── requirements.txt
│── .gitignore
│
├── notebooks/
│   ├── 01_setup_schema_and_load.py
│   ├── 02_bronze_layer.py
│   ├── 03_silver_layer.py
│   ├── 04_gold_layer.py
│   └── 05_validation.py
│
├── sql/
│   └── part5_sql_challenge.sql
│
├── docs/
│   ├── er_diagram.png
│   ├── architecture.png
│   └── data_quality_audit.md
│
└── sample_output/
    └── gold_master_sample.csv
