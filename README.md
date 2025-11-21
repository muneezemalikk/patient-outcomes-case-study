# 🏥 MediPredict.AI - Patient Outcomes Dashboard

### 🚀 https://patient-dashboard-nine-orcin.vercel.app/
*(This is the interactive frontend for the case study below)*

---

# 🏥 Big Data Analytics: Case Study Report

| **Project Details** | **Information** |
| :--- | :--- |
| **Team Members** | Muneezé Malik, Madiha Shahab |
| **Instructor** | Sir Shayan Shah |
| **Submission Date** | 21st November 2025 |
| **Topic** | Patient Experience & Outcomes (Using MIMIC-IV) |

---

# Title: Enhancing Patient Journeys & Reducing ICU Readmissions via Big Data Analytics

## Phase 1: Problem Identification

### Selected Use Case
**Patient Experience & Outcomes (Healthcare)**

### Business Problem
Healthcare organizations aim to enhance patient care quality and minimize unnecessary hospital visits. One major challenge is **unplanned hospital readmissions**, which increase costs and negatively affect patient experience.

### Project Objective (Scope)
I will build a machine learning model that predicts whether a patient is likely to be **readmitted within 30 days** after discharge. This prediction will help hospitals identify high-risk patients early and provide timely follow-up care.

### Dataset Selected
**MIMIC-IV (v3.1)**—A large, publicly available real-world hospital dataset containing admissions, lab tests, diagnoses, ICU stays, and outcomes.
* **Source:** [PhysioNet MIMIC-IV](https://physionet.org/content/mimiciv/3.1/)

### Why This Dataset Fits the Use Case
* Contains detailed **patient journeys** across emergency, inpatient, and ICU departments.
* Supports outcome-based analysis, such as 30-day readmission.
* Includes multi-structured data (labs, notes, vitals), matching the case study requirement.
* **Real-world big data scale:** over 546,000 hospital admissions.

### Boundaries
* Focus on adult inpatient admissions.
* Use only data available *up to the moment of discharge* (to avoid data leakage).
* Only unplanned readmissions within 30 days will be considered.

---

## Phase 2: Data Sourcing

### Dataset Metadata
* **Name:** MIMIC-IV (Medical Information Mart for Intensive Care), Version 3.1
* **Source:** PhysioNet — MIT Laboratory for Computational Physiology
* **Scale:**
    * 364,627 unique patients
    * 546,028 hospital admissions
    * 94,458 ICU stays
    * Over **50 GB** of structured + semi-structured data

### Modules & Structure
MIMIC-IV is divided into two main modules:

**1. hosp Module (Hospital-Wide EHR Data)**
* Demographics, Admissions & Discharges
* Transfers between departments
* Laboratory results & Microbiology
* Medications, Diagnoses & Billing (ICD codes)

**2. icu Module (ICU High-Granularity Data)**
* Vitals signs & Inputs/outputs
* Treatments, Interventions & Clinical events
* Charted nurse observations

### Why This Fits "Big Data"
* Contains complete patient journeys from admission to discharge.
* Includes multi-structured data (labs, vitals, notes).
* Provides a **360° view** of each patient.
* File Formats: Comma-separated values (CSV), Deidentified (HIPAA-compliant).

---

## Phase 3: Pipeline Design

To process multimillion-row clinical datasets, a distributed **Data Lakehouse architecture** is used.


<img width="2816" height="1536" alt="pipeline" src="https://github.com/user-attachments/assets/dae2581a-6ce9-4102-bc77-df9cb7e485ce" />


### A. Data Ingestion & Storage
* **Ingestion:** Batch processing via **Airflow / PySpark** to move CSVs from PhysioNet into the data lake.
* **Storage Layers:**
    * **Raw Zone:** Immutable CSVs (gzipped) in S3/HDFS.
    * **Bronze:** Cleaned Parquet tables with standardized types.
    * **Silver:** Joined/Normalized tables (derived fields like Length of Stay, Comorbidities).
    * **Gold:** Curated feature tables ready for ML models.
* **Partitioning:** By `anchor_year_group` and hashed `subject_id` to optimize Spark processing.

### B. Processing & Patient Journey Graph Analysis
* **Engine:** Apache Spark (PySpark) for large tables (`chartevents` >300M rows).
* **Graph Modeling:** Using **GraphFrames / GraphX** to model transfers.
    * *Nodes:* Hospital units (ER, MICU, CCU, Ward).
    * *Edges:* Transfers with `in-time`, `out-time`, and duration.

**Graph Analysis Goals:**
1.  **PageRank:** Identify bottleneck units.
2.  **Path Analysis:** Detect complex transfer paths linked to readmissions.
3.  **Community Detection:** Cluster patient flows to discover patterns.

### C. Feature Engineering
* **Demographics:** Age, gender.
* **Clinical History:** Charlson/Elixhauser scores.
* **Admission Snapshot:** LOS, discharge disposition.
* **Journey Features:** Number of transfers, path cluster ID.

### D. Visualization & Reporting
* **Tableau/Superset:** Patient journey dashboards & Sankey maps.
* **Python:** ROC/PR curves, SHAP summaries for explainability.

![workflow](https://github.com/user-attachments/assets/49509ccb-cf42-459e-9427-c186a0bf38a7)



---

## Phase 4: Machine Learning Methodology

### A. Pre-Processing Strategy
* **Missing Values:** Median imputation for vitals; Categorical features encoded as "Unknown".
* **Categorical Variables:** One-Hot Encoding for nominal (Admission Type); Label Encoding for ordinal (Severity Scores).
* **Outlier Handling:** Clip physiological measurements to clinically plausible ranges.

### B. Algorithm Recommendation

| Algorithm | Reason for Selection |
| :--- | :--- |
| **XGBoost / LightGBM** | **(Selected)** Handles high-dimensional data, robust to missing values, and provides feature importance. Ideal for binary classification (Readmission). |
| **Logistic Regression** | Baseline for interpretability; allows clinical insight into coefficients. |
| **LSTM / GRU** | Optional for sequential time-series modeling (vitals trends). |

### C. Dataset Analysis
* **Imbalance:** Readmission labels are imbalanced (fewer positive cases). Strategies: Class weighting or SMOTE.
* **High-Dimensionality:** Hundreds of features require regularization.

---

## Phase 5: Implementation Plan

### A. Tech Stack
* **ETL:** Pandas, NumPy, PySpark
* **Graph Analysis:** NetworkX, GraphFrames
* **ML:** XGBoost, Scikit-Learn, SHAP
* **Viz:** Matplotlib, Seaborn, Plotly

### B. Logic Flow
1.  **Load Data:** Ingest `admissions` and `transfers` tables.
2.  **Clean:** Handle nulls, convert timestamps.
3.  **Feature Build:** Calculate LOS, count prior admits, aggregate vitals.
4.  **Train:** Split data (80/20), train XGBoost Classifier.
5.  **Evaluate:** Check AUC-ROC and Recall.

### C. Evaluation Metrics
* **Recall (Sensitivity):** *Crucial.* We must prioritize identifying at-risk patients. Missing a readmission (False Negative) is dangerous.
* **ROC-AUC:** Measures discriminatory power across all thresholds.
* **Calibration Plots:** Ensures predicted probabilities align with real-world risk.
