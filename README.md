# SentinelOne EDR Alert Analysis & False Positive Reduction using Machine Learning

## Overview

Security Operations Centers (SOCs) rely heavily on Endpoint Detection and Response (EDR) platforms to identify malicious activity across endpoints. While modern EDR systems such as SentinelOne provide deep visibility and powerful behavioral detections, they also generate a large volume of alerts — many of which are ultimately classified as **false positives** by human analysts.

This project focuses on **reducing false positives** using a **practical, security-aware machine learning pipeline**. Instead of attempting to replace detection logic, the model learns from historical analyst decisions (True Positive vs False Positive) and acts as a **decision-support layer** to prioritize alerts more effectively.

The implementation is intentionally grounded in real SOC constraints: evolving log schemas, noisy labels, explainability requirements, and operational relevance.

---

## Problem Statement

Given SentinelOne EDR alert logs that have already been reviewed and labeled by analysts as:

* **True Positive (TP):** confirmed malicious activity
* **False Positive (FP):** benign activity dismissed after investigation

The objective is to:

1. Parse and preprocess raw EDR logs
2. Convert semi-structured data into a structured, ML-ready format
3. Learn behavioral patterns that distinguish TP from FP alerts
4. Evaluate performance using security-relevant metrics

The ultimate goal is **false-positive reduction**, not attack discovery.

---

## Dataset Description

* **Source:** SentinelOne Endpoint Detection and Response (EDR)
* **Format:** Syslog-style text with embedded JSON payloads
* **Files:**

  * `edr_tp.log` — alerts confirmed as malicious
  * `edr_fp.log` — alerts dismissed as benign

Both datasets share the same structure and originate from the same detection pipeline. The difference lies purely in **analyst judgment**, making the data suitable for supervised learning.

---

## Design Philosophy

### Schema-on-Read

Security telemetry evolves continuously. New fields appear, nested structures change, and optional attributes may be present or absent depending on detection type. Enforcing a rigid schema at ingestion time would make the pipeline fragile.

This project uses a **schema-on-read** approach:

* Logs are ingested in raw form
* JSON payloads are parsed without strict schema enforcement
* Structure is applied later during feature extraction

This mirrors real-world SOC data pipelines and ensures robustness to schema drift.

---

## Parsing & Preprocessing

Each log line is processed by:

* Separating syslog metadata from the JSON payload using a delimiter
* Parsing the JSON payload into a Python dictionary
* Safely skipping malformed records to preserve pipeline continuity
* Attaching TP/FP labels as supervised ground truth

The result is a collection of heterogeneous but structured records suitable for analysis.

---

## Feature Engineering

Feature selection is guided by security domain knowledge rather than purely statistical criteria. The pipeline focuses on:

* Detection category and analytic type
* Severity and confidence indicators
* Process and environment context

Analyst-facing fields and downstream verdicts are intentionally excluded to prevent information leakage.

---

## Machine Learning Approach

### Why Supervised Learning

Unlike anomaly detection approaches, this dataset contains explicit analyst labels. Using supervised learning allows the model to directly learn the difference between alerts that analysts confirm and those they dismiss — aligning perfectly with the objective of false-positive reduction.

### Model Choice

A **Random Forest classifier** is used as a strong, interpretable baseline:

* Handles non-linear feature interactions
* Robust to noisy labels
* Minimal hyperparameter tuning
* Provides feature importance for explainability

The model is implemented using a **scikit-learn Pipeline**, ensuring consistent preprocessing during both training and inference.

---

## Evaluation Strategy

Traditional metrics such as accuracy can be misleading in security contexts due to class imbalance and asymmetric error costs.

This project emphasizes:

* **Precision–Recall analysis** to visualize tradeoffs
* **Threshold tuning** to balance detection coverage and analyst workload
* **Confusion matrix analysis** to understand operational impact

The evaluation focuses on practical usefulness rather than raw metric maximization.

---

## Key Learnings

* Rare behavior is not necessarily malicious
* Analyst labels encode valuable institutional knowledge
* Robust data handling matters more than complex models
* Precision and false-positive rate matter more than accuracy in SOC environments

---

## Limitations & Future Work

This project represents an offline, static analysis of historical data. Future improvements could include:

* Concept drift detection
* Continuous retraining with analyst feedback
* Online or semi-supervised learning
* Explainability techniques such as SHAP

---

## Repository Structure

```
.
├── edr_tp.log              # True Positive SentinelOne alerts
├── edr_fp.log              # False Positive SentinelOne alerts
├── structured_logs.json    # Parsed schema-on-read dataset
├── notebook.ipynb          # End-to-end ML pipeline and analysis
└── README.md               # Project documentation
```

---

## Disclaimer

This project is for educational and research purposes only. The dataset is assumed to be sanitized and does not contain sensitive or production credentials.

---

## Author

Built with a focus on practical security ML, SOC realities, and explainable decision-support systems.

Feel free to explore the notebook and experiment with the pipeline.
