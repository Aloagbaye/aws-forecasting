# AWS Forecasting

End-to-end time series forecasting pipeline built for the **AWS ML Associate** certification study, using a retail sales dataset (500 SKUs, 5 categories, 730 days). Covers data processing, local baseline modelling, SageMaker XGBoost, and DeepAR with a cloud-native SageMaker Pipeline.

---

## Project structure

```
aws-forecasting/
├── data/
│   ├── raw/
│   │   └── m5_sales_raw.csv          # 365k rows, 500 SKUs × 730 days
│   └── processed/
│       ├── train_processed.csv       # tabular features, time-based train split
│       └── valid_processed.csv       # last 28 days holdout
├── notebooks/
│   ├── data_processing.ipynb         # feature engineering (lags, rolling means, calendar)
│   └── sagemaker.ipynb               # SageMaker session setup + XGBoost baseline
├── src/
│   ├── preprocess.py                 # SageMaker ProcessingStep script
│   ├── evaluate.py                   # SageMaker evaluation script (MAE / RMSE / WAPE)
│   └── generate_config.py            # SageMaker Clarify config generator
├── docs/
│   └── timeseries-pipeline-plan.md   # full pipeline plan with budget estimates
├── titanic/                          # earlier domain 1 practice notebooks
├── certification study/              # AWS ML Specialty reference PDFs
└── mind map/                         # study mind maps by exam domain
```

---

## Dataset

| Attribute | Value |
|-----------|-------|
| Source | Walmart M5 retail sales data|
| SKUs | 500 |
| Categories | Books, Clothing, Electronics, Home & Garden, Sports |
| Subcategories | 18 |
| Date range | 2023-12-18 → 2025-12-16 (730 days) |
| Target | `units_sold` |

---

## Pipeline overview

```
data/raw/ → data_processing.ipynb → data/processed/
                                          ↓
                              Phase 1: Local XGBoost baseline
                                          ↓
                              Phase 3: SageMaker XGBoost training job
                                          ↓
                              Phase 4: SageMaker GluonTS DeepAR
                              (cloud-native pipeline: ProcessingStep →
                               TrainingStep → ConditionStep → Model Registry)
                                          ↓
                              Phase 5: Hierarchical forecasting
                              (category → subcategory → SKU)
```

See [`docs/timeseries-pipeline-plan.md`](docs/timeseries-pipeline-plan.md) for the full step-by-step tutorial, cost estimates, and cost-saving recommendations.

---

## Processed features

Features engineered in `notebooks/data_processing.ipynb`:

| Feature | Description |
|---------|-------------|
| `lag_7`, `lag_14`, `lag_28` | Lagged units sold (7 / 14 / 28 days) |
| `roll_mean_7`, `roll_mean_14`, `roll_mean_28` | Rolling mean units sold |
| `day_of_week`, `week_of_year`, `month` | Calendar features |
| `category_code` | Encoded product category |
| `units_sold` | Target variable |

Train/validation split is **time-based**: last 28 days held out as validation.

---

## Setup

### Prerequisites

- Python 3.10+
- Anaconda (recommended)
- AWS account with an IAM role that has `AmazonSageMakerFullAccess` + S3 access

### Install dependencies

```bash
pip install pandas numpy scikit-learn xgboost boto3 "sagemaker>=2.232,<3.0" gluonts torch matplotlib
```

On Windows, also install to fix SSL certificate issues with boto3:

```bash
pip install python-certifi-win32
```

### AWS credentials

```bash
aws configure
# enter: Access Key ID, Secret Access Key, region (e.g. us-east-1), output format (json)
```

Verify the connection:

```bash
aws sts get-caller-identity
```

---

## Quick start

1. **Process the data** — run all cells in `notebooks/data_processing.ipynb`
2. **Local baseline** — run Phase 1 cells in `notebooks/sagemaker.ipynb`
---

## Estimated AWS cost

| Scenario | Cost |
|----------|------|
| Minimal (local + 1 SageMaker job) | ~$0.50–$1.00 |
| Full tutorial with DeepAR + tuning | ~$2–$7 |
| With SageMaker free tier (first 2 months) | ~$0 |

> Use Spot instances and Batch Transform (not real-time endpoints) to stay within the low-cost range.

---

## Exam domains covered

| Domain | Topics practised |
|--------|-----------------|
| Domain 1 — Data Engineering | S3 data ingestion, feature store, processing jobs |
| Domain 2 — EDA & Feature Engineering | Time series feature engineering, lag features, rolling statistics |
| Domain 3 — Modelling | XGBoost regression, DeepAR probabilistic forecasting, hierarchical reconciliation |
| Domain 4 — Deployment & MLOps | SageMaker Pipelines, Model Registry, ConditionStep, Batch Transform, Model Monitor |
