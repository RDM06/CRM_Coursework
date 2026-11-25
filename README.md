# CRM Coursework – S&P 500 Stock Segmentation

6BUIS017W CRM & CM with Business Intelligence, CWK1: Market Analysis and Stock Segmentation Using RFM.

This project analyses historical S&P 500 stock returns, market exposure and trading activity using hierarchical Ward clustering and K-means.

## Contents

| Path | Description |
|---|---|
| `*.ipynb` | Google Colab notebook containing the analysis, code and visualisations |
| `cache/raw/` | Saved constituent list, daily closing and adjusted closing prices, volume, stock splits and dividends (including benchmark prices in the price tables) |
| `cache/metadata/` | Cache manifest and acquisition metadata |
| `outputs/tables/` | Validation logs, weekly returns, feature tables and worked RFM example |
| `outputs/clustering/` | Cluster comparisons, assignments, profiles, sector composition and sensitivity results |
| `outputs/figures/` | Dendrograms, silhouette and inertia plots, scatterplots, PCA plots and sector charts |

## Reproducing the results

1. Open the notebook from this GitHub repository in Google Colab.
2. Copy the repository's `cache/` folder into `/content/crm_outputs/` in Colab. For example, run the following in a separate cell, replacing the placeholder with this repository's HTTPS clone URL:

   ```python
   !git clone <https://github.com/RDM06/CRM_Coursework> /content/crm_repo
   !mkdir -p /content/crm_outputs
   !cp -r /content/crm_repo/cache /content/crm_outputs/
   ```

3. Keep `USE_CACHE = True` in the setup cell and check that the selected cluster counts are `3, 3, 4, 3` (Risk Ward, Risk K-means, RFM Ward, RFM K-means respectively).
4. Select **Runtime → Run all**. Install the packages listed in the setup cell if required.

The notebook uses the saved snapshot (constituents and market data retrieved on 24 November 2026) rather than requesting fresh data. It rebuilds the analysis and exports results under `/content/crm_outputs/outputs/`. The cache manifest records acquisition details and the notebook reports package versions. No Google Drive mount is required.

Setting `USE_CACHE = False` downloads fresh data and refreshes the cache. Results may differ because the constituent list and historical prices can change. Colab storage is temporary; retain the cache and required outputs outside the runtime.

## Task 1: Data preparation and feature engineering

- S&P 500 constituents from Wikipedia; daily Close, adjusted Close, Volume, dividends and stock splits from Yahoo Finance (`yfinance`), 1 January 2021 to 1 January 2025 (end exclusive), with `^GSPC` as the benchmark.
- Validation against 1,005 benchmark trading sessions: 503 requested shares, 488 included and 15 excluded for missing observations. Exclusion reasons are logged; missing prices are not imputed.
- Corporate-action summaries and stock-split diagnostics support assessment of price and volume adjustment assumptions.
- Weekly adjusted closing prices (`W-FRI`) produce 207 weekly returns per included share. Mean weekly return and weekly beta against the benchmark are calculated.
- Adapted RFM at 27 December 2024: a high-volume event occurs when daily volume exceeds twice the mean of the preceding 20 trading sessions. Recency measures calendar days since the latest qualifying event; Frequency counts qualifying events and Monetary sums Close × Volume over the latest 60 trading sessions.

## Task 2: Stock segmentation and sensitivity analysis

- Two separately standardised feature matrices: risk-return (mean weekly return and weekly beta) and RFM (Recency, Frequency and `log1p(Monetary)`).
- Ward clustering (Euclidean distance) and K-means (`random_state=42`, `n_init=20`) evaluated for k = 2 to 8 using silhouette scores, cluster sizes, original-unit profiles and K-means inertia. Dendrograms support hierarchical clustering analysis.
- Selected solutions: Risk Ward k = 3, Risk K-means k = 3, RFM Ward k = 4 and RFM K-means k = 3.
- Original-unit cluster profiles, labelled scatterplots, two-component PCA visualisations for RFM and adjusted Rand index (ARI) comparisons between algorithms. PCA is used only for visualisation.
- RFM sensitivity: Pearson correlations among Recency, Frequency and log-transformed Monetary; Ward clustering repeated using freshly standardised Recency and Frequency on the same shares and at the selected Ward RFM cluster count. Membership agreement is evaluated using ARI, alongside traded-value profiles.
- Final assignments: `outputs/clustering/cluster_assignments.csv`; sensitivity assignments: `outputs/clustering/cluster_assignments_with_sensitivity.csv`.

## Task 3: Business intelligence and critical review

- The accompanying report proposes a data mart for daily share and benchmark observations, derived features, cluster assignments and calculation provenance. The data mart is a conceptual design, not an implemented database.
- Original-unit cluster profiles and sector counts/percentages support comparisons between risk-return and trading-activity segments and the overall sample composition.
- Correlation and Monetary-removal results support discussion of trading-activity monitoring and portfolio analysis.
- Limitations include survivorship and constituent-selection bias, incomplete-history exclusions, return dependencies, corporate-action adjustment assumptions, correlated features and the use of traded value as a liquidity proxy.
- The clusters describe historical characteristics and do not predict future returns or guarantee reduced portfolio risk.
