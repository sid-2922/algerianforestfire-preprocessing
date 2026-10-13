# Algerian Forest Fires: Data Preprocessing

A Python/Jupyter project that cleans and validates the **Algerian Forest Fires dataset** so it is ready for exploratory analysis and fire / no-fire classification.

The raw CSV contains two regional tables stacked in one file, with a title row, a repeated header row, blank rows, padded column names, text-typed numbers and inconsistent class labels. The notebook turns it into a clean, fully numeric dataset of **243 daily observations**.

---

## Table of Contents

- [Dataset](#dataset)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Getting Started](#getting-started)
- [Preprocessing Pipeline](#preprocessing-pipeline)
- [Results](#results)
- [Output Columns](#output-columns)
- [Known Issues](#known-issues)
- [Next Steps](#next-steps)

---

## Dataset

- **File:** `Dataset/Algerian_forest_fires_dataset.csv`
- **Period:** June to September 2012
- **Regions:** two regions of Algeria (Bejaia in the public UCI version, and Sidi-Bel Abbes), 122 daily records each
- **Source:** the Algerian Forest Fires dataset, available from the UCI Machine Learning Repository

| Group | Variables |
|---|---|
| Date | `day`, `month`, `year` |
| Weather | `Temperature` (°C), `RH` (%), `Ws` (km/h), `Rain` (mm) |
| Fire Weather Index system | `FFMC`, `DMC`, `DC`, `ISI`, `BUI`, `FWI` |
| Target | `Classes` (`fire` / `not fire`) |
| Added in preprocessing | `region` (0 or 1) |

## Project Structure

```
.
├── Dataset/
│   └── Algerian_forest_fires_dataset.csv   # raw input
├── preprocessing.ipynb                     # cleaning and inspection notebook
├── clean_dataset.csv                       # output, created by the notebook
├── Forest_Fires_Preprocessing_Report.docx  # project report
└── README.md
```

## Requirements

- Python 3.12 (the notebook was run on 3.12.7)
- pandas
- NumPy
- Matplotlib
- Seaborn
- Jupyter (Notebook or JupyterLab) or VS Code with the Jupyter extension

## Getting Started

1. **Create and activate a virtual environment**

   ```bash
   python -m venv .venv
   source .venv/bin/activate        # Windows: .venv\Scripts\activate
   ```

2. **Install dependencies**

   ```bash
   pip install pandas numpy matplotlib seaborn jupyter
   ```

3. **Check the data path.** The notebook reads `Dataset/Algerian_forest_fires_dataset.csv` relative to the notebook's folder, so keep the folder layout shown above.

4. **Run the notebook**

   ```bash
   jupyter notebook preprocessing.ipynb
   ```

   Run all cells from top to bottom. The last executed cell writes `clean_dataset.csv` next to the notebook.

## Preprocessing Pipeline

| Step | What happens | Why |
|---|---|---|
| 1. Load and inspect | `read_csv`, then look at rows 120 to 130 and `describe(include='all')` | Reveals the blank row, title row and repeated header in the middle of the data |
| 2. Label regions | `region = 0` for the first block, `1` for the second | Keeps the region information before the title row is removed |
| 3. Clean column names | `df.columns.str.strip()` | Names such as `' RH'`, `'Rain '` and `'Classes  '` had stray spaces |
| 4. Check duplicates | `duplicated()` / `drop_duplicates()` | No duplicates were found |
| 5. Convert to numeric | `pd.to_numeric(errors='coerce')` on the 13 date and measurement columns | All columns were read as text because of the embedded header row |
| 6. Remove bad rows | `dropna()`, then `reset_index` | Drops 3 non-data rows and 1 corrupt record (row 168 had unparseable `DC` and `FWI`) |
| 7. Encode the target | Strip whitespace, map `not fire` to 0 and `fire` to 1 | `Classes` had 8 padded spellings of 2 labels |
| 8. Validate | Min/max checks, box plots, histograms | Confirms plausible ranges and shows skew and outliers |
| 9. Export | `to_csv('clean_dataset.csv', index=False)` | Analysis-ready file |

Outliers were inspected but **not removed**: all values are within valid ranges and the extreme values represent real high-risk fire-weather conditions.

## Results

| Aspect | Raw file | Cleaned dataset |
|---|---|---|
| Rows | 247 | 243 |
| Columns | 14 | 15 (adds `region`) |
| Missing values | Present | 0 |
| Numeric measurement columns | 0 of 13 | 13 of 13 |
| Distinct `Classes` values | 9 | 2 (0 and 1) |
| Records per region | Not labelled | 122 (region 0) and 121 (region 1) |

Observations from the plots:

- `year` is constant (2012) and carries no information.
- `Rain` is zero-inflated; `DMC`, `DC`, `ISI`, `BUI` and `FWI` are right-skewed; `FFMC` is left-skewed.
- The target is mildly imbalanced, with roughly 56% `fire` and 44% `not fire`.

## Output Columns

`clean_dataset.csv` has 243 rows and these columns:

`day`, `month`, `year`, `Temperature`, `RH`, `Ws`, `Rain`, `FFMC`, `DMC`, `DC`, `ISI`, `BUI`, `FWI`, `Classes`, `region`

All feature columns are `float64`, `Classes` is `int64` (0 = not fire, 1 = fire), and there are no missing values.

## Known Issues

- **Date columns are still exported.** `df.drop(columns=['day','month','year'])` in the notebook is not assigned back, so `clean_dataset.csv` still contains them. Use `df = df.drop(columns=[...])` if you want them removed.
- **Region split is hard-coded** to row 122. This works for the provided file but would break if the file layout changes.
- **Row 168 is dropped** even though only two of its fields were unreadable. It could be repaired manually from the raw file.
- **Duplicate check runs before cleaning**, so it would miss rows that differ only by whitespace. Re-run it on the final data.

## Next Steps

- Drop `year` and decide how to use `month` (or convert dates to day-of-year).
- Plot a correlation heatmap. `FWI` is derived from `ISI` and `BUI`, so these columns are strongly related.
- Consider a `log1p` transform on skewed features for linear or distance-based models.
- Scale features with `StandardScaler`, fitted on the training split only.
- Use a stratified train/test split and cross-validation, since the dataset is small.
- Wrap the cleaning steps into a reusable function or scikit-learn pipeline.

## Report

A full write-up of the methodology, figures and findings is in `Forest_Fires_Preprocessing_Report.docx`.
