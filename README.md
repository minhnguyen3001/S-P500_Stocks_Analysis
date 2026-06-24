# U.S. Stock Market Analysis: S&P 500 Exploration, Clustering, and Sector Classification

An end to end R analysis of the S&P 500 Companies dataset, covering exploratory data analysis, data cleaning, unsupervised clustering, and supervised classification of company sector from financial and firm level features.

## Overview

This project works with a snapshot of the S&P 500 index containing financial and organizational data for 502 companies. The analysis moves through six stages: explore the raw data, clean and preprocess it, search for natural groupings with K-Means and PCA, classify companies into their 11 GICS sectors with a decision tree and a random forest, then collapse those sectors into a simpler Growth versus Defensive split and evaluate that binary model in more depth with a confusion matrix, precision, recall, F1, and an ROC curve.

## Dataset

* Source: [S&P 500 Companies dataset on Kaggle](https://www.kaggle.com/datasets/andrewmvd/sp-500-stocks?resource=download)
* 502 companies, 16 raw columns (the `Longbusinesssummary` free text column is dropped, leaving 15 columns used in the analysis)
* Numerical features: Current Price, Market Cap, EBITDA, Revenue Growth, Full-time Employees, Index Weight
* Categorical features: Sector, Industry, Exchange, Country, State, City

## Repository contents

| File | Description |
|---|---|
| `Project.Rmd` | R Markdown source with the full analysis, organized into parts a through g (load, explore, clean, preprocess, cluster, classify, evaluate). |
| `Project.R` | Plain R script version of the same analysis code, for running outside of R Markdown. |
| `Project.pdf` | Knitted report: code, console output, and all plots in one document. |
| `Minh_Nguyen.docx` | Written discussion of the results and what they mean, organized by analysis stage. |
| `sp500_companies.csv` | Raw dataset used by the analysis. |

## Methodology

1. **Load and inspect.** Read the CSV, drop the free text summary column, and check structure and dimensions (502 rows, 15 columns).
2. **Explore.** Summarize the data with `skimr`, plot company counts by sector and by exchange, look at raw and log transformed distributions of the numeric features, and build a correlation matrix.
3. **Clean.** Impute missing values (median imputation for revenue growth and employee count, sector-grouped median for EBITDA), and winsorize the numeric columns at the 1st and 99th percentiles to limit the effect of extreme outliers.
4. **Preprocess.** One hot encode exchange, a non-US country flag, and the top 5 states (with the rest grouped as "Other"), then min-max normalize all numeric features to [0, 1].
5. **Cluster.** Use the elbow method to choose a number of clusters, fit K-Means (k = 8), and visualize the result with PCA, comparing clusters against actual sector labels.
6. **Classify by sector.** Split the data 80/20, then train and 5-fold cross-validate a decision tree (tuned on complexity parameter) and a random forest (tuned on `mtry`) to predict the 11-class Sector label.
7. **Evaluate a simplified target.** Collapse the 11 sectors into a binary Growth versus Defensive label, retrain a random forest, and evaluate it with a confusion matrix, precision, recall, F1 score, and an ROC curve with AUC.

## Key findings

* Technology (82 companies), Industrials (70), and Financial Services (67) are the most represented sectors. Most companies trade on the NYSE (NYQ, 348 companies) or NASDAQ (NMS, 152 companies).
* Market Cap and EBITDA are strongly correlated (r = 0.86), index Weight is essentially a direct function of Market Cap (r = 1.00), and Current Price is largely uncorrelated with every other variable (r < 0.05), so stock price alone says little about company size.
* K-Means clustering (k = 8, chosen from the elbow plot) grouped companies mainly by size rather than by sector. One cluster captured mega cap technology companies clearly, and a few others lined up reasonably well with Financial Services and Energy, but a single large cluster of 180 companies mixed Industrials, Healthcare, and Consumer sector companies together, meaning many sectors are not financially distinguishable using only market cap, EBITDA, and growth rate.
* Predicting the full 11-class Sector label from financial and firm features alone is hard: the tuned decision tree reached 32.99 percent test accuracy and the tuned random forest reached 38.14 percent, with the random forest performing better. Full-time employee count and EBITDA were the most predictive features in the random forest, ahead of revenue growth and current price; geographic features such as State_CA and State_NY contributed comparatively little.
* Collapsing the problem into a binary Growth versus Defensive sector split is more tractable: a random forest reached 65 percent accuracy, with precision 0.6875, recall 0.7458, F1 0.7154, and an AUC of 0.66, indicating a real but moderate discriminative signal beyond random guessing.

## Discussion

The most notable result is how hard it is to recover a company's sector from financial metrics alone. Sector membership seems to be driven more by what a company actually does than by its size or profitability, which is why an 11-class model tops out under 40 percent accuracy while the simpler Growth versus Defensive split does meaningfully better. The cluster versus sector heatmap reinforces this: K-Means naturally separates companies by scale, not by industry, with only the mega cap technology group standing clearly apart in PCA space. In the binary model, the gap between recall (0.75) and precision (0.69) reflects the class imbalance between Growth (296 companies) and Defensive (206 companies) sectors, and the AUC of 0.66 shows that the model carries real signal that a single accuracy number would otherwise hide.

## Requirements

The analysis uses R with the following packages:

```
tidyverse, ggplot2, skimr, corrplot, factoextra,
caret, randomForest, rpart, rpart.plot, pROC
```

## Running the analysis

1. Place `sp500_companies.csv` in the same directory as the script or R Markdown file.
2. Either:
   * Knit `Project.Rmd` to PDF (requires a working `xelatex` installation), or
   * Run `Project.R` directly in R or RStudio.

## Author

Minh Nguyen
