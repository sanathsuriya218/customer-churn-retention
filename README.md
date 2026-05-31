# customer-churn-retention

![Python](https://img.shields.io/badge/Python-3.13+-green)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-orange)
![License](https://img.shields.io/badge/License-MIT-yellow)

**End-to-End Customer Churn Prediction & Cohort Retention Analysis**

*From Raw Transactions to AI-Powered Business Recommendations*

[Overview](#overview) • [Pipeline](#pipeline) • [EDA](#exploratory-data-analysis) • [Model](#churn-prediction-model) • [Results](#results) • [Stack](#stack)

---

## Overview

This project builds a complete data science pipeline on 755K+ UK e-commerce
transactions — from raw data cleaning through cohort analysis, churn prediction,
customer segmentation, and AI-generated business recommendations.

### The Business Problem

E-commerce businesses lose most customers after a single purchase. Without knowing
which customers are at risk and why, retention efforts are untargeted and wasteful.

### The Solution

A self-contained analytics system that:

- Identifies when customers are likely to churn before they do
- Segments customers into actionable risk tiers
- Explains which behaviors drive churn
- Generates plain-English recommendations using a local LLM

---

## Pipeline

```
Raw CSV (755K rows)
        ↓
Data Cleaning
12-step pipeline: duplicates, cancellations, outliers, nulls, junk codes, type fixing
        ↓
Exploratory Data Analysis
Revenue distribution, country analysis, time trends, customer behavior patterns
        ↓
Cohort Retention Analysis
Multi-layer CTE SQL query in PostgreSQL — 25 monthly cohorts, 2 years of data
        ↓
Feature Engineering
RFM features + behavioral metrics — one row per customer, churn labeling
        ↓
Churn Prediction Model
Random Forest classifier — ROC-AUC: 0.7723, leakage-free
        ↓
Customer Segmentation
Quantile-based risk segmentation — High / Medium / Low churn risk tiers
        ↓
AI Recommendation Engine
Local Llama3 generates business recommendations from model output
```

## Exploratory Data Analysis

Key findings from EDA:

- Revenue is heavily right-skewed — majority of transactions fall under £100
- The UK accounts for the dominant share of revenue; top international markets
  are Netherlands, EIRE, Germany and France
- Orders peak mid-week (Tuesday to Thursday) and drop sharply on weekends
- Peak ordering hours are 10am to 3pm — consistent with B2B purchasing behavior
- Top products are low-cost decorative and gift items bought in bulk

---

## Cohort Retention Analysis

Built using multi-layer CTEs in PostgreSQL:

```sql
WITH cohorts AS (
    SELECT customer_id,
           DATE_TRUNC('month', MIN(invoice_date)) AS cohort_month
    FROM retail_orders
    GROUP BY customer_id
),
cohort_periods AS (
    SELECT customer_id, cohort_month,
           EXTRACT(YEAR FROM AGE(order_month, cohort_month)) * 12 +
           EXTRACT(MONTH FROM AGE(order_month, cohort_month)) AS period_number
    FROM orders_with_cohort
)
```

Key findings:

- Average month-1 retention across all cohorts: 28%
- 70% of customers do not return after their first purchase
- Dec 2009 cohort retained 49.7% at month 12 — seasonal repurchase spike
- Long-term retention floor stabilizes at 20-25% after month 3

---

## Churn Prediction Model

### Feature Engineering

| Feature | Description |
|---|---|
| total_orders | Number of unique invoices |
| total_items | Total quantity purchased |
| total_revenue | Cumulative spend |
| unique_months | Months active |
| unique_products | Product variety |
| avg_order_value | Revenue per order |
| orders_per_month | Purchase frequency rate |

Churn definition: no purchase in the final 90 days of the dataset.
Class balance: 50.7% churned / 49.3% retained.

### Model Results

| Metric | Value |
|---|---|
| Algorithm | Random Forest (200 trees) |
| ROC-AUC | 0.7723 |
| Churner recall | 74% |
| Data leakage | None — recency features excluded |

### Top Churn Drivers

1. total_items — customers who buy more items churn less
2. unique_months — broader engagement predicts loyalty
3. total_revenue — higher spenders are more retained

---

## Results

### Customer Risk Segments

| Segment | Customers | Avg Orders | Avg Revenue |
|---|---|---|---|
| High risk | 1,975 | 1.4 | £345 |
| Medium risk | 1,916 | 3.6 | £1,085 |
| Low risk | 1,916 | 13.7 | £5,850 |

Low-risk customers spend 17x more than high-risk customers on average.

### AI-Generated Recommendation (Llama3)

The model output is fed into a local Llama3 instance which generates a
structured business recommendation covering segment profiles, prioritized
actions, and expected retention improvement per tier.

---

## Stack

| Layer | Technology |
|---|---|
| Database | PostgreSQL 16 |
| Data Processing | Python, Pandas, NumPy |
| Machine Learning | Scikit-learn (Random Forest) |
| Visualization | Matplotlib, Seaborn |
| AI Recommendations | Ollama, Llama3 (local) |
| Notebook | Jupyter via VS Code |

---

## View Full Notebook

GitHub's renderer struggles with large notebooks. View the complete notebook with all outputs and charts rendered here:

[![nbviewer](https://img.shields.io/badge/jupyter-nbviewer-orange?logo=jupyter)](https://nbviewer.org/github/sanathsuriya218/customer-churn-retention/blob/main/customer_churn_analysis.ipynb)

or directly at:
https://nbviewer.org/github/sanathsuriya218/customer-churn-retention/blob/main/customer_churn_analysis.ipynb

---

## Files

| File | Description |
|---|---|
| `customer_churn_analysis.ipynb` | Full pipeline notebook |
| `cohort_retention_heatmap.png` | Monthly retention heatmap |
| `confusion_matrix.png` | Model evaluation matrix |
| `feature_importance.png` | Churn driver chart |
| `churn_distribution.png` | Probability distribution |
| `business_recommendation.txt` | LLM-generated report |

---

## License

MIT License
