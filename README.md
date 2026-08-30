# U.S. Stock Market Analysis: S&P 500 Exploration, Clustering, and Sector Classification

![R](https://img.shields.io/badge/R-language-276DC3?logo=r&logoColor=white)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](LICENSE)
![Status](https://img.shields.io/badge/status-complete-brightgreen)

An end-to-end R analysis of the S&P 500 Companies dataset: exploratory data analysis, data cleaning, unsupervised clustering, and supervised classification of company sector from financial and firm-level features.

---

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Repository Contents](#repository-contents)
- [Methodology](#methodology)
- [Results at a Glance](#results-at-a-glance)
- [Key Findings](#key-findings)
- [Discussion](#discussion)
- [Requirements](#requirements)
- [Running the Analysis](#running-the-analysis)
- [Author](#author)
- [License](#license)

## Overview

This project works with a snapshot of the S&P 500 index covering financial and organizational data for 502 companies. The analysis moves through six stages: explore the raw data, clean and preprocess it, search for natural groupings with K-Means and PCA, classify companies into their 11 GICS (Global Industry Classification Standard) sectors with a decision tree and a random forest, then collapse those sectors into a simpler Growth vs. Defensive split and evaluate that binary model in depth with a confusion matrix, precision, recall, F1, and an ROC curve.

## Dataset

- **Source:** [S&P 500 Companies dataset on Kaggle](https://www.kaggle.com/datasets/andrewmvd/sp-500-stocks?resource=download)
- **Size:** 502 companies, 16 raw columns — the free-text `Longbusinesssummary` column is dropped, leaving 15 columns used in the analysis
- **Numerical features:** Current Price, Market Cap, EBITDA, Revenue Growth, Full-Time Employees, Index Weight
- **Categorical features:** Sector, Industry, Exchange, Country, State, City

## Repository Contents

| File | Description |
|---|---|
| `Project.Rmd` | R Markdown source with the full analysis, organized into parts a–g (load, explore, clean, preprocess, cluster, classify, evaluate) |
| `Project.R` | Plain R script version of the same analysis, for running outside R Markdown |
| `Project.pdf` | Knitted report — code, console output, and all plots in one document |
| `Minh_Nguyen.docx` | Written discussion of the results and what they mean, organized by analysis stage |
| `sp500_companies.csv` | Raw dataset used by the analysis |
| `LICENSE` | GPL-3.0 license text |

## Methodology

1. **Load and inspect** — read the CSV, drop the free-text summary column, and check structure and dimensions (502 rows × 15 columns).
2. **Explore** — summarize the data with `skimr`, plot company counts by sector and exchange, examine raw and log-transformed distributions of the numeric features, and build a correlation matrix.
3. **Clean** — impute missing values (median imputation for revenue growth and employee count, sector-grouped median for EBITDA) and winsorize the numeric columns at the 1st and 99th percentiles to limit the effect of extreme outliers.
4. **Preprocess** — one-hot encode exchange, a non-US country flag, and the top 5 states (grouping the rest as "Other"), then min-max normalize all numeric features to [0, 1].
5. **Cluster** — use the elbow method to choose a cluster count, fit K-Means (k = 8), and visualize the result with PCA, comparing clusters against actual sector labels.
6. **Classify by sector** — split the data 80/20 and train and 5-fold cross-validate a decision tree (tuned on complexity parameter) and a random forest (tuned on `mtry`) to predict the 11-class Sector label.
7. **Evaluate a simplified target** — collapse the 11 sectors into a binary Growth vs. Defensive label, retrain a random forest, and evaluate it with a confusion matrix, precision, recall, F1 score, and an ROC curve with AUC.

## Results at a Glance

| Task | Model | Accuracy | Other metrics |
|---|---|---|---|
| 11-class sector classification | Decision tree (tuned on cp) | 32.99% | — |
| 11-class sector classification | Random forest (tuned on mtry) | 38.14% | Top features: full-time employees, EBITDA |
| Binary Growth vs. Defensive | Random forest | 65% | Precision 0.6875 · Recall 0.7458 · F1 0.7154 · AUC 0.66 |

## Key Findings

- **Sector composition:** Technology (82 companies), Industrials (70), and Financial Services (67) are the most represented sectors. Most companies trade on the NYSE (NYQ, 348 companies) or NASDAQ (NMS, 152 companies).
- **Correlations:** Market Cap and EBITDA are strongly correlated (r = 0.86); Index Weight is essentially a direct function of Market Cap (r = 1.00); Current Price is largely uncorrelated with every other variable (r < 0.05) — stock price alone says little about company size.
- **Clustering:** K-Means (k = 8, chosen from the elbow plot) grouped companies mainly by size rather than by sector. One cluster clearly captured mega-cap technology companies, and a few others lined up reasonably well with Financial Services and Energy — but a single large cluster of 180 companies mixed Industrials, Healthcare, and Consumer companies together, showing that many sectors aren't financially distinguishable using only market cap, EBITDA, and growth rate.
- **11-class sector classification is hard:** the tuned decision tree reached 32.99% test accuracy and the tuned random forest reached 38.14%. Full-time employee count and EBITDA were the most predictive features in the random forest, ahead of revenue growth and current price; geographic features like `State_CA` and `State_NY` contributed comparatively little.
- **Binary classification is more tractable:** collapsing the problem into Growth vs. Defensive, a random forest reached 65% accuracy (precision 0.6875, recall 0.7458, F1 0.7154, AUC 0.66) — a real but moderate discriminative signal beyond random guessing.

## Discussion

The standout result is how hard it is to recover a company's sector from financial metrics alone. Sector membership appears to be driven more by what a company actually does than by its size or profitability — which is why the 11-class model tops out under 40% accuracy while the simpler Growth vs. Defensive split does meaningfully better. The cluster-vs-sector comparison reinforces this: K-Means naturally separates companies by scale rather than by industry, with only the mega-cap technology group standing clearly apart in PCA space. In the binary model, the gap between recall (0.75) and precision (0.69) reflects the class imbalance between Growth (296 companies) and Defensive (206 companies), and the AUC of 0.66 shows the model carries real signal that a single accuracy number would otherwise hide.

## Requirements

The analysis uses R with the following packages:

```r
install.packages(c(
  "tidyverse", "ggplot2", "skimr", "corrplot", "factoextra",
  "caret", "randomForest", "rpart", "rpart.plot", "pROC"
))
```

## Running the Analysis

1. Place `sp500_companies.csv` in the same directory as the script or R Markdown file.
2. Then either:

```r
# Option 1 — knit to PDF (requires a working `xelatex` installation)
rmarkdown::render("Project.Rmd")

# Option 2 — run the plain R script directly
source("Project.R")
```

## Author

**Minh Nguyen** — [@minhnguyen3001](https://github.com/minhnguyen3001)

## License

This project is licensed under **GPL-3.0** — see [LICENSE](LICENSE) for details.

---

Maintained by [@minhnguyen3001](https://github.com/minhnguyen3001)
