# Video Game Sales Pipeline - Project Report

## Overview

This project builds an end-to-end data pipeline on a video game sales dataset sourced from Kaggle (originally scraped from VGChartz). The goal is to clean and explore the data, then train a model that can predict global sales for a game based on its attributes. The pipeline covers everything from raw CSV ingestion through to a SQLite database, exploratory analysis, and a trained Random Forest model with exported predictions.

---

## Data Source

The raw dataset (`vgsales.csv`) contains 16,598 rows and 11 columns, covering games that sold more than 100,000 copies. Each row represents a single game/platform combination, with columns for rank, name, platform, year, genre, publisher, and regional sales figures (NA, EU, JP, Other, Global).

The file is treated as immutable throughout the pipeline all transformations are applied to copies, not the original.

---

## Cleaning Pipeline (`vgsales_pipeline.ipynb`)

The cleaning step handles the messiest parts of the raw data before anything else touches it.

**Missing values** were handled on a column-by-column basis:
- `Year` had 271 missing values and was dropped entirely for those rows, year is essential for time-series analysis and can't be meaningfully imputed.
- `Publisher` had 58 missing values, filled with `"Unknown"` so those game records could still be used.

**Type conversions** were applied to ensure consistent data types `Year` was converted to nullable `Int64`, and all sales columns were coerced to `float64`.

**Duplicates** were removed based on a composite key of `(Name, Platform, Year)`, which caught 1 duplicate row.

**String normalisation** was applied across text columns: whitespace stripped, Platform uppercased, Genre title-cased.

**Outlier detection** used a 3×IQR fence on `Global_Sales`. This flagged 1,004 games as statistical outliers, but they were all retained these are legitimate mega-hits (e.g., Wii Sports at 82.74M units) and removing them would distort the analysis.

The cleaned output is `vgsales_cleaned.csv` (16,326 rows) and `vgsales.db`, a SQLite database with a `games` table and a `data_dictionary` table that documents every field, its type, and any transformations applied.

---

## Exploratory Data Analysis (`EDA_anaylsis.ipynb`)

With clean data in hand, the EDA notebook looks at distributions, trends, and relationships.

**Sales distribution** is heavily right-skewed most games sell modestly, with a long tail of blockbusters. Action, Sports, and Misc are the most common genres. DS, PS2, and PS3 are the dominant platforms by game count.

**Regional correlations** are strong: NA sales correlate with Global_Sales at 0.94, EU at 0.90, JP at 0.61. This matters a lot for modelling (more on that below).

**Time trends** show a peak around 2008 (678.90M units globally), with a decline in the years after likely reflecting both market shifts and incomplete data for recent years in the VGChartz scrape.

---

## Modelling

Three models were built, each improving on the last.

**Baseline Linear Regression** used the four regional sales columns as features. It achieved an R² of 0.99999 this was suspiciously perfect, and for good reason: `Global_Sales` is literally the sum of regional sales. This is data leakage, not a real result.

**Initial Random Forest** (100 trees) used the same features and achieved R²=0.8242, RMSE=0.8664. Better in terms of realism, but still used the leaky features.

**Improved Random Forest** (the final model) removed all regional sales columns and instead used Platform, Genre, Publisher, Year, and Rank all encoded appropriately. Hyperparameters were tuned to 200 trees, max_depth=20, min_samples_split=5, min_samples_leaf=2. This gave R²=0.8285, RMSE=0.8558, which is a solid result given it's predicting from game metadata alone.

---

## Output

The final model's predictions on the test set are exported to `model_predictions.csv` (3,266 rows), containing the game name, platform, genre, year, actual global sales, and the model's predicted value. This file is intended for further visualisation in Tableau.

---

## Tableau Dashboard

The predictions and cleaned data were visualised in a Tableau dashboard. Below are all five views.

### Full Dashboard Overview

![Dashboard Overview](TableauImages/Screenshot%202026-04-03%20191250.png)

The combined dashboard gives a side-by-side view of all panels, useful for spotting cross-cutting patterns at a glance.

---

### Top 10 Publishers by Global Sales

![Top 10 Publishers by Global Sales](TableauImages/Screenshot%202026-04-03%20191552.png)

A stacked bar chart broken down by year. Nintendo and Electronic Arts are the clear leaders in total global sales volume, with Nintendo's output concentrated in particular hardware generations and EA maintaining a more consistent multi-year presence.

---

### Genre Popularity Heatmap

![Genre Popularity Heatmap](TableauImages/Screenshot%202026-04-03%20191457.png)

Plots genre vs. platform across every platform in the dataset, coloured by total global sales (dark blue = low, orange/red = high). PS2 stands out as the highest-intensity platform across nearly every genre. Role-Playing games show notably strong performance on PS and PS2, while Action and Sports are consistently warm across most modern platforms.

---

### Actual vs. Predicted Sales

![Actual vs Predicted Sales](TableauImages/Screenshot%202026-04-03%20191417.png)

A scatter plot of the Random Forest model's test set predictions against true global sales values. The diagonal line represents a perfect prediction. Most games cluster tightly along it, confirming the model generalises well for typical titles. The single outlier far to the right is a genuine mega-hit the model underestimates this expected behaviour given how rare those events are.

---

### Regional Sales by Genre

![Regional Sales by Genre](TableauImages/Screenshot%202026-04-03%20191404.png)

Stacked bars showing how each genre's total sales split across EU, Global, JP, NA, and Other markets. Action dominates overall, but the regional breakdown reveals Japan's strong preference for Role-Playing games relative to other markets. Sports has an outsized EU contribution compared to most genres.

---

### Platform Trends

![Platform Trends](TableauImages/Screenshot%202026-04-03%20191437.png)

Line chart of global sales over time per platform. Each platform follows a clear rise-and-fall arc tied to its hardware lifecycle. PS peaks early (~2000), PS2 dominates mid-decade, and DS and Wii both spike sharply around 2008-2009 before declining. PS3 and PS4 show a longer, flatter curve reflecting a more sustained release cycle.

---

Together these panels give a useful at-a-glance view of what drives sales across genres, platforms, publishers, and regions.

---

## Can We Predict Global Sales?

**Short answer: yes, reasonably well.** The final Random Forest model explains ~83% of variance in global sales (R²=0.8285) using only game metadata no regional sales figures. For most games, that's a useful prediction. The model struggles at the extremes, particularly for rare mega-hits where platform timing and cultural factors are hard to encode, but for a typical release it's a solid estimate.

**Applied scenario: Nintendo Racing game, Sweden, 2027**

Sweden falls under the "Other" regional market in this dataset. To project global sales for a hypothetical Nintendo Racing title in 2027, the model would take:
- **Platform:** A current Nintendo platform (e.g., the Switch 2), historically Nintendo platforms show strong global attach rates
- **Genre:** Racing usually mid-tier in terms of global sales volume, typically outperforms in NA and EU, weaker in Japan
- **Publisher:** Nintendo is consistently one of the top global publishers, adds a meaningful uplift in the model's feature importance
- **Year:** 2027 is outside the training data range (which cuts off around 2016), so the model would be extrapolating; treat the output as a rough directional estimate, not a precise forecast

Based on historical patterns in the data, a Nintendo Racing title from a comparable era (e.g., Mario Kart 8) achieved ~35M global sales which was well above what the model would naively predict for an "average" Racing game (~0.5-1M). The model would likely estimate somewhere in the 1-3M range for a new Nintendo Racing title, reflecting the genre average, with Nintendo's publisher uplift pushing it toward the higher end. The true ceiling depends heavily on IP strength and platform install base, which aren't captured in the dataset.

The key takeaway is that **publisher and platform are the strongest predictors** and genre shapes expectations, but a Nintendo title on a popular platform will consistently outperform the genre baseline.

---

## Summary

The pipeline goes: raw CSV → cleaning → SQLite storage → EDA → modelling → predictions export. The biggest design decisions were retaining outliers (they're real data), dropping Year NaN rows (imputation would be misleading), and stripping regional sales from the final model to avoid leakage. The result is a reproducible pipeline with an honest predictive model that explains ~83% of variance in global game sales.
