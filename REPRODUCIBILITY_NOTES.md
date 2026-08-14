# Reproducibility and Manuscript-Implementation Alignment Notes

This file records differences observed between the supplied manuscript and the executed `hesa_finance_model_Final.ipynb`. These points should be resolved before using the repository as the definitive reproducibility package for the paper.

## 1. FSI normalization scope

### Manuscript

The manuscript states that Min-Max normalization parameters are estimated **exclusively from the training period/data** and then applied unchanged to validation/test observations. This design is intended to prevent information leakage.

### Supplied notebook

The current notebook constructs the FSI before the predictive train/test split:

```python
scaler = MinMaxScaler()
norm = scaler.fit_transform(x)[:, 0]
```

This fits each KPI scaler on the complete provider-year panel used for FSI construction.

### Recommended correction

For strict agreement with the manuscript, define the temporal/train split first, estimate each KPI's scaling parameters on the training subset only, and apply those stored parameters to validation/test/future observations.

If the study intentionally defines the FSI as a descriptive index over the full historical sample, revise the manuscript wording instead. The final paper and public code should describe the same procedure.

## 2. Table-14 KFI availability in the final notebook run

The manuscript defines a multidimensional FSI containing indicators from revenue sustainability, financial resilience, operational efficiency, and strategic investment capacity.

However, in the supplied notebook's executed output, the final KFI reconstruction reports zero non-null values for the following variables:

```text
kfi_surplus_deficit_ratio
kfi_staff_cost_ratio
kfi_premises_cost_ratio
kfi_unrestricted_reserves_ratio
kfi_external_borrowing_ratio
kfi_net_assets_days
kfi_current_ratio
kfi_operating_cashflow_ratio
kfi_net_liquidity_days
kfi_general_funds_ratio
kfi_debt_service_ratio
```

Consequently, the final `available_kpis` list in the executed notebook contains only:

```text
research_income_ratio
research_income_growth_rate
income_growth_rate
income_diversification_score
capital_investment_ratio
tuition_dependency_ratio
funding_body_dependency_ratio
```

### Recommended correction

Inspect the Table-14 pivot/merge logic and column suffix handling. Table 14 is merged more than once in the notebook, which creates suffixed column names and can complicate later reconstruction. The final public notebook should merge Table 14 once, validate non-null coverage immediately after the merge, and assert the intended KPI list before FSI construction.

A useful validation is:

```python
for col in intended_kpis:
    assert col in panel.columns
    print(col, panel[col].notna().sum())
```

## 3. FSI descriptive statistics differ between notebook stages

An earlier notebook output reports:

```text
count = 2207
mean  = 25.241
std   = 6.169
min   = 16.824
max   = 61.686
```

These values match the descriptive statistics reported in the manuscript.

After the later “FIX KFI COLUMNS + REBUILD CORE KPIs SAFELY” and FSI rebuild, the executed notebook reports:

```text
count = 2207
mean  = 27.102
std   = 6.780
min   = 14.594
max   = 48.844
```

The public reproducibility notebook should have one authoritative FSI construction path so that all descriptive tables, figures, predictive targets, clustering, anomaly analysis, and XAI outputs are generated from the same FSI definition.

## 4. HESA tables

The manuscript lists Tables 1, 7, 9, 12, and 14 as data sources.

The notebook discovers 16 HESA CSV files and explicitly inspects those five tables. The principal feature-engineering path uses:

- Table 7 for income composition;
- Table 9 for capital expenditure;
- Table 12 for staff cost information; and
- Table 14 for KFI ratios.

Table 1 is inspected but is not directly merged into the main final feature path in the supplied notebook.

The README therefore distinguishes between manuscript-listed tables and tables used directly by the current implementation.

## 5. Google Colab path dependency

The notebook uses:

```text
/content/drive/MyDrive/Fin-dataset
/content/drive/MyDrive/HESA_outputs
```

and imports:

```python
from google.colab import drive
```

This is appropriate for Colab but not portable to a standard local Python environment.

For a publication-quality repository, consider moving these paths to a configuration cell:

```python
DATA_DIR = "data/raw"
OUTPUT_DIR = "outputs"
```

and making Google Drive mounting optional.

## 6. Exact software versions

The notebook metadata identifies a Python 3 kernel but does not store the exact Python, NumPy, pandas, scikit-learn, matplotlib, and joblib versions used in the original run.

The included `requirements.txt` therefore specifies compatible version ranges rather than claiming an exact environment.

For stronger reproducibility, run the finalized notebook once in the archival environment and save:

```bash
python --version
pip freeze > requirements-lock.txt
```

Then retain both:

- `requirements.txt` for readable dependency constraints; and
- `requirements-lock.txt` for the exact archival environment.

## 7. Recommended final checks before GitHub release

Before making the repository public:

1. Consolidate the notebook so every analytical stage is executed once.
2. Fix and validate the Table-14 KFI merge.
3. Decide whether FSI normalization is training-only or full-sample and make paper/code consistent.
4. Re-run the notebook from a clean runtime.
5. Restart the kernel and run all cells in order without manual intervention.
6. Confirm all paper tables and figures are reproduced by the final run.
7. Export exact package versions.
8. Add a software licence chosen by the authors.
9. Replace the manuscript placeholder DOI after publication.
