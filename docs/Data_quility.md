# Data Quality Audit

## Overview

This document summarizes the data quality assessment performed on the five raw datasets provided for the Century Health Data Engineer Take-Home Assignment.

The source data contained several common real-world issues such as:

- Missing values
- Malformed dates
- Duplicate rows
- Inconsistent column naming
- Mixed casing in identifiers
- Incorrect datatypes
- Packed multi-value attributes
- Outlier numeric values
- Join key mismatches across tables

All issues were reviewed and addressed during the Silver transformation layer to create clean, analytics-ready datasets.

---

# Datasets Audited

1. patients.csv  
2. symptoms.csv  
3. medications.csv  
4. conditions.xlsx *(converted to CSV for Databricks compatibility)*  
5. encounters.parquet

---

# Audit Methodology

Each dataset was profiled using PySpark to inspect:

- Schema and datatypes
- Null counts
- Duplicate records
- Unique key consistency
- Invalid formats
- Join key relationships
- Outlier values
- Text normalization needs

---

# Detailed Findings

---

## 1. patients Dataset

### Expected Purpose
One row per patient containing demographics and location details.

### Issues Found

| Column | Issue | Impact | Resolution |
|--------|------|--------|-----------|
| birthdate | Stored as string | Cannot perform age/date analysis | Converted to DATE |
| deathdate | Invalid values like `9999-99-99` | Breaks casting logic | Converted to NULL using try_cast |
| gender | Missing values | Incomplete demographics | Filled with `unknown` |
| marital | Missing values | Null dimension values | Filled with `unknown` |
| lat | Some values abnormally small | Bad geo reporting | Corrected scaling issue |
| duplicate rows | Duplicate patient records | Double counting risk | Removed duplicates |
| mixed text casing | Inconsistent text values | Grouping issues | Standardized |

### Actions Applied

- Safe date conversion
- Null handling
- Latitude correction
- Deduplication

---

## 2. encounters Dataset

### Expected Purpose
One row per healthcare visit / encounter.

### Issues Found

| Column | Issue | Impact | Resolution |
|--------|------|--------|-----------|
| start | String timestamp | No time analysis possible | Converted to TIMESTAMP |
| stop | String timestamp | No duration analysis possible | Converted to TIMESTAMP |
| id | Needed as encounter primary key | Join dependency | Preserved as encounter_id |
| duplicate rows | Possible repeated encounters | Inflated counts | Removed duplicates |
| schema inconsistency | Mixed naming style | Harder joins | Standardized |

### Actions Applied

- Timestamp conversion
- Column standardization
- Duplicate removal

---

## 3. conditions Dataset

### Expected Purpose
Patient diagnosis / condition history.

### Issues Found

| Column | Issue | Impact | Resolution |
|--------|------|--------|-----------|
| patient | Mixed casing / spaces | Join mismatch | Trimmed + lowercase |
| encounter | Mixed casing / spaces | Join mismatch | Trimmed + lowercase |
| start | Invalid string dates | Casting errors | Safe cast to DATE |
| stop | Missing / malformed dates | Partial history | Converted to NULL |
| duplicate rows | Repeated conditions | Duplicate analytics | Removed duplicates |

### Actions Applied

- Identifier normalization
- Date correction
- Deduplication

---

## 4. medications Dataset

### Expected Purpose
Medication prescriptions and costs.

### Issues Found

| Column | Issue | Impact | Resolution |
|--------|------|--------|-----------|
| start | String timestamp | No timeline analysis | Converted to TIMESTAMP |
| stop | Null values | Unclear active medication status | Treated as active medication |
| totalcost | String datatype | Numeric analysis blocked | Cast to DOUBLE |
| totalcost | Extreme high values | Distorted averages | Flagged / cleansed |
| duplicate rows | Repeated rows | Cost inflation | Removed duplicates |

### Actions Applied

- Timestamp conversion
- Numeric casting
- Active medication logic
- Duplicate removal

---

## 5. symptoms Dataset

### Expected Purpose
Patient symptom observations.

### Issues Found

| Column | Issue | Impact | Resolution |
|--------|------|--------|-----------|
| symptoms | Packed multi-value field | Not queryable | Unpivoted into long format |
| patient | Formatting inconsistencies | Join issues | Standardized |
| duplicate rows | Duplicate observations | Double counting | Removed duplicates |

### Example Raw Value

```text
Rash:34;Joint Pain:39;Fatigue:9;Fever:1

| patient | symptom_name | symptom_value |
| ------- | ------------ | ------------- |
| P001    | Rash         | 34            |
| P001    | Joint Pain   | 39            |
| P001    | Fatigue      | 9             |
| P001    | Fever        | 12            |

Actions Applied
Split values
Exploded rows
Standardized identifiers

| Validation Check                          | Status |
| ----------------------------------------- | ------ |
| encounters.patient exists in patients     | Passed |
| conditions.encounter exists in encounters | Passed |
| medications.patient exists in patients    | Passed |
| symptoms.patient exists in patients       | Passed |

Standardization Rules Applied in Silver Layer
Converted all column names to snake_case
Trimmed leading/trailing whitespace
Lowercased join identifiers
Corrected datatypes
Removed duplicates
Replaced selected null dimension values
Preserved business keys


Gold Layer Readiness

After cleansing, the data was transformed into:

gold.symptoms_observation

One row per patient per symptom.

gold.master_health

Integrated analytical table containing:

patient demographics
encounters
diagnoses
medications
symptom scores

Final Summary
The raw datasets reflected realistic healthcare data challenges.
Through structured cleansing and transformation in the Silver layer, all critical quality issues were remediated and reliable Gold-layer analytical tables were produced for downstream reporting and SQL analysis.
