# XAI for Financial Sustainability Assessment and Prediction in Higher Education Institutions

This repository contains the Python implementation associated with the manuscript **“XAI for Financial Sustainability Assessment and Prediction in Higher Education Institutions”** by **Budoor Allehyani** and **Salam Al-E’mari**.

The project develops an AI-enabled analytical pipeline for assessing and forecasting the financial sustainability of higher education institutions (HEIs) using publicly available UK Higher Education Statistics Agency (HESA) finance data.

## Overview

The implementation combines:

1. HESA finance data ingestion and cleaning
2. Provider-year panel construction
3. Financial Key Performance Indicator (KPI) engineering
4. Financial Sustainability Index (FSI) construction
5. Next-year FSI prediction using Random Forest regression
6. Next-year financial risk classification
7. Institutional profiling using K-Means clustering
8. Financial anomaly detection using Isolation Forest
9. Explainable AI using Random Forest feature importance and permutation importance
10. Export of processed datasets, trained models, tables, and figures

The analytical unit is a **provider-year observation**, identified primarily by the UK Provider Reference Number (**UKPRN**) and academic year.

## Repository Contents

```text
.
├── hesa_finance_model_Final.ipynb   # Main analysis notebook
├── README.md                        # Project documentation
├── requirements.txt                 # Python dependencies
├── DATA.md                          # Dataset acquisition and organization
├── REPRODUCIBILITY_NOTES.md         # Important implementation/manuscript alignment notes
├── CITATION.cff                     # Citation metadata
└── .gitignore                       # Files excluded from Git tracking
```

Raw HESA data and generated outputs are intentionally not included in this repository.

## Data Source

The study uses open financial data published by the **Higher Education Statistics Agency (HESA)**:

- HESA Finance portal: https://www.hesa.ac.uk/data-and-analysis/finances
- Finance release tables: https://www.hesa.ac.uk/data-and-analysis/finances/releases
- Finance data user guide: https://www.hesa.ac.uk/data-and-analysis/finances/user-guide

HESA publishes its open data under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** licence. Users should provide appropriate attribution to HESA when redistributing or publishing derived data.

The manuscript identifies the following principal finance tables:

| HESA table | Purpose |
|---|---|
| Table 1 | Institutional income and expenditure |
| Table 7 | Income sources analysis |
| Table 9 | Capital expenditure |
| Table 12 | Staff costs analysis |
| Table 14 | Key Financial Indicators (KFIs) |

The supplied notebook scans all CSV files in the configured data directory. Its main feature-engineering path uses Tables 7, 9, 12, and 14, while Table 1 is also inspected during the workflow.

See [DATA.md](DATA.md) for download and folder-layout instructions.

## Methodology

### 1. Data preparation

The notebook:

- recursively discovers HESA CSV files;
- detects non-standard CSV header rows dynamically;
- handles common encodings (`utf-8-sig`, `utf-8`, `latin1`, and `cp1252`);
- standardizes provider, academic-year, region, country, monetary, and ratio fields;
- converts monetary and ratio values to numeric form;
- reshapes long-format finance categories into provider-year features;
- merges finance tables using `UKPRN` and `academic_year`;
- removes duplicate provider-year observations;
- replaces infinite values with missing values; and
- uses median imputation before modeling.

### 2. KPI engineering

The notebook constructs financial indicators including:

- tuition dependency ratio;
- funding-body dependency ratio;
- research income ratio;
- income diversification score;
- income growth rate;
- research income growth rate; and
- capital investment ratio.

The code also attempts to construct KFI-based indicators for financial resilience and operational efficiency from HESA Table 14.

Income diversification is based on the Herfindahl-Hirschman Index (HHI):

```text
Income Diversification Score = 1 - HHI
```

where HHI is computed from the shares of the major institutional income sources.

### 3. Financial Sustainability Index

Available KPIs are normalized using Min-Max scaling. Indicators for which a lower value represents stronger sustainability are directionally reversed:

```text
Adjusted KPI = 1 - Normalized KPI
```

The Financial Sustainability Index is then calculated as the equal-weight arithmetic mean of the available adjusted KPI values and scaled to 0–100:

```text
FSI = mean(adjusted normalized KPIs) × 100
```

Higher FSI values represent stronger **relative** sustainability within the analyzed provider-year sample. The FSI is an analytical index rather than a regulatory rating.

### 4. Next-year FSI prediction

The prediction target is:

```text
FSI(t + 1)
```

for each provider.

The notebook evaluates two Random Forest regression configurations:

- **Model A:** current-year KPIs + current FSI
- **Model B:** current-year KPIs only

Main Random Forest settings:

```python
RandomForestRegressor(
    n_estimators=500,
    max_depth=10,
    random_state=42,
    n_jobs=-1
)
```

A 75:25 train/test split is used.

Evaluation metrics:

- Mean Absolute Error (MAE)
- Root Mean Squared Error (RMSE)
- Coefficient of determination (R²)

### 5. Financial risk classification

FSI values are converted into relative risk groups using empirical quantiles:

- lower third → **High Risk**
- middle third → **Medium Risk**
- upper third → **Low Risk**

A Random Forest classifier predicts the next-year risk category.

```python
RandomForestClassifier(
    n_estimators=500,
    max_depth=10,
    random_state=42,
    n_jobs=-1,
    class_weight="balanced"
)
```

The implementation reports accuracy, precision, recall, F1-score, and a confusion matrix.

### 6. Institutional clustering

K-Means clustering is applied to normalized sustainability features.

Candidate values:

```text
k = 2, 3, ..., 8
```

The final value of `k` is selected using the highest silhouette score. PCA is used only for two-dimensional visualization and does not determine the clusters.

### 7. Anomaly detection

Isolation Forest is used to identify atypical provider-year observations:

```python
IsolationForest(
    n_estimators=300,
    contamination=0.05,
    random_state=42
)
```

An anomaly should not automatically be interpreted as financial distress. It may also represent unusual growth, investment, research income, or revenue structure.

### 8. Explainable AI

Two complementary explainability methods are included:

- Random Forest built-in feature importance
- Permutation importance using test-set R²

Explainability is evaluated both with and without current-year FSI as an input feature.

## Results Reproduced by the Supplied Notebook

The executed notebook contains **2,207 provider-year observations** after provider-year de-duplication.

| Task | Result |
|---|---:|
| RF regression with current FSI | MAE = 1.413, RMSE = 2.479, R² = 0.871 |
| RF regression without current FSI | MAE = 1.358, RMSE = 2.352, R² = 0.884 |
| Next-year risk classification | Accuracy = 0.853 |
| Best K-Means solution | k = 3 |
| Best silhouette score | 0.492 |
| Isolation Forest anomalies | 111 of 2,207 observations (~5.03%) |

For the model that excludes current FSI, the strongest features in the supplied run include tuition dependency, research income, income diversification, funding-body dependency, and capital investment.

## Installation

### Option A — Google Colab

The notebook was written and executed in Google Colab and contains Google Drive paths. Upload the notebook to Colab, mount Google Drive, place the HESA CSV files in:

```text
/content/drive/MyDrive/Fin-dataset/
```

and run the notebook from top to bottom.

Generated outputs are written to:

```text
/content/drive/MyDrive/HESA_outputs/
```

### Option B — Local Python/Jupyter

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it.

**Windows**

```bash
.venv\Scripts\activate
```

**macOS/Linux**

```bash
source .venv/bin/activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

Start Jupyter:

```bash
jupyter lab
```

Before running locally, change the notebook paths:

```python
DATA_DIR = "/content/drive/MyDrive/Fin-dataset"
```

and

```python
output_dir = "/content/drive/MyDrive/HESA_outputs"
```

to local directories. The final Google Drive export cell also imports `google.colab`; skip or adapt that cell outside Colab.

## Expected Data Layout

A minimal layout compatible with the notebook is:

```text
Fin-dataset/
├── table-1.csv
├── table-7.csv
├── table-9.csv
├── table-12.csv
└── table-14.csv
```

The original executed notebook discovered 16 HESA CSV files. Additional HESA finance tables may remain in the directory; the loader discovers them automatically.

## Generated Outputs

The notebook can export:

### Data and summaries

- `hesa_financial_sustainability_final_panel.csv`
- `hesa_fsi_risk_cluster_anomaly_summary.csv`
- `prediction_model_dataset.csv`
- `risk_classification_dataset.csv`
- `prediction_actual_vs_predicted.csv`
- `risk_actual_vs_predicted.csv`
- `cluster_summary.csv`
- `anomaly_cases.csv`
- XAI importance CSV files
- `model_performance_summary.csv`

### Trained models

- `random_forest_fsi_prediction_model.pkl`
- `random_forest_fsi_prediction_model_without_fsi.pkl`
- `random_forest_risk_classification_model.pkl`
- `kmeans_clustering_model.pkl`
- `isolation_forest_anomaly_model.pkl`

### Figures

The notebook creates figures for:

- FSI distribution and temporal trend;
- KPI correlation;
- actual vs. predicted FSI;
- regression residuals;
- risk distribution and confusion matrix;
- silhouette scores and PCA cluster visualization;
- cluster-level FSI;
- anomaly scores and anomaly status; and
- XAI feature/permutation importance.

## Reproducibility Note

The manuscript and supplied notebook are closely related, but the current notebook contains implementation details that should be aligned with the final manuscript before archival or publication. In particular, the supplied execution reports that the reconstructed Table-14 KFI columns contain no non-null values in the final KPI rebuild, and the notebook currently computes Min-Max normalization before the predictive train/test split.

These points are documented in [REPRODUCIBILITY_NOTES.md](REPRODUCIBILITY_NOTES.md).

## Citation

If you use this implementation, please cite the associated manuscript:

```bibtex
@article{allehyani2026xai,
  title  = {XAI for Financial Sustainability Assessment and Prediction in Higher Education Institutions},
  author = {Allehyani, Budoor and Al-E'mari, Salam},
  year   = {2026},
  note   = {Manuscript}
}
```

The manuscript currently contains a placeholder DOI. Replace the citation above with the official bibliographic record and DOI after publication.

## Data Attribution

When using the HESA open data, provide attribution to the Higher Education Statistics Agency (HESA) and comply with the applicable CC BY 4.0 licence terms.

## License

No software licence is included in this repository package because the authors should explicitly choose the intended reuse terms. Common options for academic research code include MIT and BSD-3-Clause.

Do not reuse the HESA data under the code licence; HESA data remains subject to its own open-data licence and attribution requirements.
