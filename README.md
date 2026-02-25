# AML Detection System

**2025-2026 IMI Big Data & Artificial Intelligence Competition hosted by the University of Toronto in collaboration with Scotiabank**

Hybrid Anti-Money Laundering detection pipeline combining unsupervised anomaly detection, domain-driven rule-based scoring, and LLM-powered explainability to identify and explain suspicious financial activity across 61,000+ customers and 5.9 million transactions.

---

## Table of Contents

- [Overview](#overview)
- [Pipeline Architecture](#pipeline-architecture)
- [Repository Structure](#repository-structure)
- [Pipeline Stages](#pipeline-stages)
- [Key Results](#key-results)
- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [Documentation](#documentation)
- [License](#license)

---

## Overview

The project tackles the challenge of identifying suspicious customers from synthetic banking data provided for the IMI Big Data & AI Competition.

The approach uses a multi-layered detection pipeline:

1. **Unsupervised ML** (Isolation Forest) to surface statistical anomalies
2. **Expert-driven rules** grounded in a curated AML Red Flag Indicator Library
3. **A hybrid scoring model** that fuses both signals for maximum detection power
4. **A generative AI explanation engine** that produces plain-language risk summaries for compliance investigators

---

## Pipeline Architecture

```
Raw Data (7 channels + KYC)
        |
        v
Data Unification & Feature Engineering (95+ features)
        |
        +---> Isolation Forest (anomaly detection)
        |                           |
        +---> Rule-Based Scoring    |
                     |              |
                     v              v
                  Hybrid Model (50/50 balanced)
                         |
                         v
                  LLM Explanation Engine
                         |
                         v
                  Risk Reports for Investigators
```

---

## Repository Structure

```
├── scripts/
│   ├── AML-load-unify-data              # Data loading & feature engineering
│   ├── AML-add-advanced-features.ipynb   # Advanced AML-specific features
│   ├── AML-isolation-forest             # Unsupervised anomaly detection
│   ├── AML-rule-based-scoring.ipynb     # Expert rule-based scoring
│   ├── AML-hybrid-model.ipynb           # Hybrid model fusion
│   └── AML-Explanation_Model.ipynb      # LLM-powered explanations
├── models/                               # Saved models, predictions & results
├── features/                             # Engineered feature datasets
├── documentation/                        # AML Knowledge Library & docs
├── other/                                # Supplementary explanation outputs
└── AML_RedFlags_Database.xlsx            # AML Red Flag Indicator Library
```

---

## Pipeline

### 1. Data Loading & Unification

**Notebook:** `scripts/AML-load-unify-data`

Ingests and normalizes transaction data from 7 banking channels into a unified master transaction table, then engineers an initial set of customer-level features.

| Channel       | Records     |
|---------------|-------------|
| Card          | 3,553,303   |
| EFT           | 1,070,698   |
| EMT           | 845,996     |
| Cheque        | 240,548     |
| ABM           | 185,690     |
| Wire          | 4,956       |
| Western Union | 2,142       |
| **Total**     | **5,903,333** |

- 61,410 unique customers, 1,000 labeled (10 confirmed suspicious)
- KYC data merged for individuals (53K) and businesses (8K)
- Date range: Nov 2024 - Jan 2025
- 95 initial features across volume/velocity, structuring, channel, geographic, temporal, and KYC categories

### 2. Advanced AML Feature Engineering

**Notebook:** `scripts/AML-add-advanced-features.ipynb`

Adds 15 domain-specific AML scores: structuring score, rapid churn, income/business mismatch, international & cash risk, composite AML risk, and interaction features (e.g. high-value x high-frequency).

### 3. Isolation Forest - Anomaly Detection

**Notebook:** `scripts/AML-isolation-forest`

Unsupervised Isolation Forest trained on 102 numeric features. Configuration: 200 trees, 1% contamination, 80% max features.

- ROC-AUC on labeled data: **0.794**
- 615 customers flagged

### 4. Rule-Based AML Scoring

**Notebook:** `scripts/AML-rule-based-scoring.ipynb`

Implements 7 rule categories derived from the AML Red Flag Indicator Library: structuring, profile mismatch, layering, geographic risk, account activity, cash intensive, and wire transfers.

Mean rule-based score: 0.263 (suspicious) vs 0.208 (clean) - successfully separates classes.

### 5. Hybrid Model

**Notebook:** `scripts/AML-hybrid-model.ipynb`

Fuses Isolation Forest and rule-based scores. The balanced strategy (50/50 weighting) performed best:

| Strategy   | Weights (ML / Rules) | ROC-AUC | Capture@5% |
|------------|----------------------|---------|------------|
| ML Heavy   | 70% / 30%            | 0.831   | 10%        |
| **Balanced** | **50% / 50%**      | **0.844** | **30%**  |
| Rule Heavy | 30% / 70%            | 0.799   | 20%        |
| Adaptive   | Dynamic              | 0.820   | 0%         |

ROC-AUC improvement over Isolation Forest alone: +5.0%.

### 6. LLM-Powered Explanation Engine

**Notebook:** `scripts/AML-Explanation_Model.ipynb`

Generates plain-language AML risk summaries for every customer, designed for compliance investigators with no ML background.

Two LLM options:
- **Local:** Llama 3.2 3B (Q4_K_M GGUF) via llama-cpp-python on GPU
- **API:** Gemini 2.5 Flash via Google AI API

Each explanation maps behavioral patterns to specific AML Red Flag Indicator IDs (e.g. `[STRUCT-003]`, `[PROF-001]`), states the risk level, and provides a recommended next step for the investigator.

---

## Key Results

| Metric                       | Value       |
|------------------------------|-------------|
| Customers analyzed           | 61,410      |
| Transactions processed       | 5,903,333   |
| Features engineered          | 110+        |
| Final model ROC-AUC          | 0.844       |
| Capture@5%                   | 30%         |
| Improvement over IF baseline | +5.0%       |
| Customers flagged            | 615 (top 1%)|

---

## Tech Stack

- **Data Processing:** Python, Pandas, NumPy
- **ML / Anomaly Detection:** scikit-learn (Isolation Forest, StandardScaler)
- **Explanation (Local):** llama-cpp-python, Llama 3.2 3B GGUF
- **Explanation (API):** Google Generative AI (Gemini 2.5 Flash)
- **Environment:** Google Colab (GPU T4)

---

## Getting Started

Run the notebooks in order:

```
1. AML-load-unify-data              -> Unified transaction table & initial features
2. AML-add-advanced-features.ipynb  -> Enhanced AML-specific features
3. AML-isolation-forest             -> Anomaly detection model
4. AML-rule-based-scoring.ipynb     -> Rule-based scores
5. AML-hybrid-model.ipynb           -> Final hybrid predictions
6. AML-Explanation_Model.ipynb      -> LLM-generated risk summaries
```

The raw transaction data is sourced from the competition's private dataset and is not included in this repository. The pipeline assumes the data is accessible via Google Drive at the paths specified in each notebook.

---

## Documentation

- [`documentation/AML_Knowledge_Library.xlsx`](documentation/AML_Knowledge_Library.xlsx) - Full AML indicator reference
- [`documentation/AML_Knowledge_Library_Documentation.pdf`](documentation/AML_Knowledge_Library_Documentation.pdf) - Knowledge library documentation
- [`AML_RedFlags_Database.xlsx`](AML_RedFlags_Database.xlsx) - Red flag indicators used by the scoring & explanation engines

Only a subset of the indicators in the knowledge library are used in the final model. Indicators belonging to specific OCG-related typology clusters are excluded; only general money laundering indicators are used to limit token usage and model complexity.

---

