# Data

## Source

This project uses open finance data from the **Higher Education Statistics Agency (HESA)** for UK higher education providers.

Official resources:

- Finance portal: https://www.hesa.ac.uk/data-and-analysis/finances
- Finance release tables: https://www.hesa.ac.uk/data-and-analysis/finances/releases
- Finance user guide: https://www.hesa.ac.uk/data-and-analysis/finances/user-guide

HESA publishes its website open data under the **Creative Commons Attribution 4.0 International (CC BY 4.0)** licence. Confirm the licence displayed on the relevant HESA release page when downloading data.

## Tables Used

The manuscript identifies:

| File | HESA content |
|---|---|
| `table-1.csv` | Consolidated income and expenditure |
| `table-7.csv` | Income analysed by source |
| `table-9.csv` | Capital expenditure |
| `table-12.csv` | Staff costs |
| `table-14.csv` | Key Financial Indicators |

The current notebook scans every CSV in the configured directory. It explicitly accesses the five files above; the main feature-engineering pipeline derives most model inputs from Tables 7, 9, 12, and 14.

## Directory

For the supplied Google Colab notebook, place the downloaded CSV files in:

```text
/content/drive/MyDrive/Fin-dataset/
```

Example:

```text
Fin-dataset/
├── table-1.csv
├── table-7.csv
├── table-9.csv
├── table-12.csv
├── table-14.csv
└── ... other HESA finance CSV files ...
```

For local execution, replace the `DATA_DIR` value in the notebook with the corresponding local directory.

## File Formatting

The notebook includes a dynamic header-row detector because HESA CSV exports may contain metadata before the true header. It searches for a header containing fields such as `UKPRN` and `HE provider`.

Supported input encodings attempted by the notebook are:

- UTF-8 with BOM
- UTF-8
- Latin-1
- CP1252

## Data Handling

The notebook:

- standardizes column names;
- parses numeric monetary and ratio values;
- pivots long-format financial categories;
- merges tables on `UKPRN` and `academic_year`;
- removes duplicate provider-year records;
- converts infinite values to missing values; and
- applies median imputation before modeling.

## Redistribution

The raw data is not included in this GitHub package. This keeps the repository small and ensures users obtain the data and its documentation directly from HESA.

If you redistribute HESA-derived data, provide appropriate HESA attribution and comply with the licence terms shown by HESA.
