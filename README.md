# COSC301 Project — Video Game Sales Data Pipeline

## Dataset

**Name:** Video Game Sales  
**Source:** https://www.kaggle.com/datasets/gregorut/videogamesales  
**License:** CC0 Public Domain — no restrictions on use  
**File:** `vgsales.csv`

This dataset contains sales records for video games that sold more than 100,000 copies, scraped from VGChartz. It covers game title, platform, release year, genre, publisher, and regional sales figures (NA, EU, JP, Other) plus a global total in millions of units.

---

## Reproducing the Dataset Download

### Option 1 — Manual (no account needed)
1. Visit https://www.kaggle.com/datasets/gregorut/videogamesales
2. Click **Download** (requires a free Kaggle account)
3. Unzip the downloaded file
4. Place `vgsales.csv` in the root of this repository

### Option 2 — Kaggle CLI (recommended)
```bash
# 1. Install the Kaggle CLI
pip install kaggle

# 2. Generate your API token
#    Go to https://www.kaggle.com/settings → API → "Create New Token"
#    This downloads kaggle.json

# 3. Place the token file
mkdir -p ~/.kaggle
mv ~/Downloads/kaggle.json ~/.kaggle/
chmod 600 ~/.kaggle/kaggle.json

# 4. Download and unzip the dataset
kaggle datasets download -d gregorut/videogamesales
unzip videogamesales.zip

# 5. Move the CSV into the repo root
mv vgsales.csv /path/to/COSC301PROJECT/
```

---

## Running the Pipeline

### Requirements

Install all dependencies with:

```bash
pip install pandas scikit-learn matplotlib seaborn
```


### Steps

1. Clone this repository
   ```bash
   git clone <repo-url>
   cd COSC301Project
   ```

2. Place `vgsales.csv` in the repo root (see dataset download steps above)

3. Install dependencies
   ```bash
   pip install pandas scikit-learn matplotlib seaborn
   ```

4. Open `run_pipeline.ipynb` in VS Code or Jupyter and run all cells. Figures render inline.

### Outputs generated

| File / Folder | Description |
|---|---|
| `vgsales.csv` | Original raw data — never modified |
| `vgsales_cleaned.csv` | Cleaned dataset (missing values handled, duplicates removed, types fixed) |
| `vgsales.db` | SQLite database with two tables: `games` and `data_dictionary` |
| `model_predictions.csv` | Actual vs predicted Global Sales for the test set (3,266 rows) |

---

## Pipeline Overview

| Stage | Description |
|---|---|
| 1. Load | Read raw CSV, inspect shape and dtypes |
| 2. Clean / ETL | Handle missing values, fix types, remove duplicates, normalise strings, flag outliers |
| 3. Storage | Save cleaned CSV + load into SQLite DB with data dictionary |
| 4. EDA | Generate six exploratory figures rendered inline in the notebook |
| 5. Baseline Model | Linear Regression using regional sales (note: near-perfect R² due to data leakage) |
| 6. Improved Model | Tuned Random Forest using platform, genre, publisher, and year (no leakage) |
| 7. Export | Save model predictions with metadata to `model_predictions.csv` |