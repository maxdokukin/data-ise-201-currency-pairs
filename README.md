# Euro Currency Pair Analysis — PCA and seasonality in 23 years of EUR exchange rates

ISE 201 — Math Foundations for Decision and Data Sciences (Prof. S. Gupta), San José State University · Fall 2025 · Solo (Maxim Dokukin) · Status: Completed

## Overview

The project studies how 40 currencies moved against the euro from 1999-01-04 to 2022-01-10, using the European Central Bank's daily reference rates. It asks two questions: can most exchange-rate variance be explained by a few latent factors, and does the USD/EUR pair show a statistically significant calendar effect in December? The daily rates are reshaped to long format, converted to log returns, screened for calendar gaps and |z| > 4 outliers, and ranked by volatility. Two PCA configurations — a 40-currency intersection that collapses to a 115-day window in 2000 and a 31-currency "survivor" panel covering 2000–2022 — are compared, and December returns are tested against the rest of the year with Welch's t-test and a Mann-Whitney U test. Everything runs in one Colab notebook with Plotly charts.

## Highlights

- One global factor (PC1) explains **56.54%** of standardized return variance in the 115 × 40 intersection window (Jul–Dec 2000) but **34.79%** over the 5,496 × 31 survivor panel (2000–2022), where PC2 rises from 4.56% to **12.02%** — `ISE_201_Final.ipynb` cells 25, 27.
- December USD/EUR monthly log returns average **+1.33%** versus **−0.14%** in the other months (≈147 bp gap; 23 vs 254 months); Welch t-test **p = 0.0425**, Mann-Whitney U **p = 0.0438** — cell 31.
- Most volatile currencies against the euro: Icelandic króna (daily log-return std 0.0185), Turkish lira (0.0131), Brazilian real (0.0115) — cell 15.
- Most |z| > 4 return outliers: Indonesian rupiah and Lithuanian litas (55 each), Croatian kuna and Russian rouble (48 each) — cell 13.

## How it works

```
Kaggle (kagglehub) CSV, wide: date × 40 currencies
  → normalise column names → melt to long (date, ccy, rate) → coerce numeric, drop missing
  → TARGET-calendar gap count per currency
  → log returns r_t = ln(P_t / P_{t-1}) per currency, drop 40 non-finite values
  → EDA: rate plots in 7 magnitude bins · |z| > 4 outliers · volatility ranking · return time series + histograms
  → PCA A (all 40 currencies, complete rows only) vs PCA B (currencies with > 90% coverage)
  → USD/EUR monthly sums → December vs rest: Welch t-test + Mann-Whitney U
```

- **Data preparation** — pandas reshaping, a custom TARGET business-day calendar (python-dateutil Easter rules) for gap counting, per-currency log returns.
- **EDA** — Plotly line charts grouped by rate magnitude so currencies quoted below 1 and in the thousands per euro stay readable; z-scores computed within each currency; descriptive statistics sorted by volatility.
- **PCA** — `StandardScaler` + scikit-learn `PCA(n_components=10)` on a date × currency return matrix; scree plot and PC1 factor-intensity time series for each approach.
- **Hypothesis testing** — SciPy `ttest_ind(equal_var=False)` and `mannwhitneyu` at α = 0.05 on month-end sums of daily log returns; bar chart of monthly means ± SEM and a December-vs-rest box plot.

## Results

| Metric | Value | Baseline / note |
|---|---|---|
| PC1 explained variance, Approach A (115 × 40, 2000-07-20 → 2000-12-29) | 56.54% | PC2 4.56%, PC3 3.93%; top-3 65.03% |
| PC1 explained variance, Approach B (5,496 × 31, 2000-07-20 → 2022-01-10) | 34.79% | PC2 12.02%, PC3 4.39%; top-3 51.20% |
| Mean monthly USD/EUR log return, December (n = 23) | +1.33% | −0.14% for the other months (n = 254) |
| Welch t-test, December vs rest | p = 0.0425 | significant at α = 0.05 |
| Mann-Whitney U, December vs rest | p = 0.0438 | significant at α = 0.05 |
| Highest daily log-return std | ISK 0.0185 | TRY 0.0131, BRL 0.0115, ZAR 0.0097, MXN 0.0090 |

The contrast between the two PCA windows is the main finding: a short window dominated by one factor versus a long panel where a second factor carries 12% of the variance. The December result is significant at the 5% level but rests on 23 Decembers, and only one month was tested.

## Getting started

The notebooks run on [Google Colab](https://colab.research.google.com/) (or Kaggle Notebooks). The data is not stored in this repo; the final notebook downloads it with `kagglehub`.

```bash
pip install pandas numpy scipy scikit-learn plotly python-dateutil kagglehub
jupyter notebook ISE_201_Final.ipynb   # or upload to Colab and Run all
```

Run order and roles:

1. `ISE201_Project_Proposal_Maxim_Dokukin_Revised.ipynb` — revised proposal (Nov 6, 2025): dataset background, research questions, data-quality plan. Markdown only.
2. `ISE201_EDA_Maxim_Dokukin.ipynb` — first EDA pass (Nov 6, 2025): long-format reshape, gap counts, z-score outliers, peg and legacy-currency handling, magnitude-binned charts. Expects the CSV at `/content/euro-daily-hist_1999_2022.csv` (download it from Kaggle and upload it to Colab). Saved without outputs.
3. `ISE_201_Final.ipynb` — final analysis (Dec 5, 2025): data, EDA, PCA, hypothesis testing, conclusion. Downloads the data itself. Saved with outputs (17 Plotly figures, ≈10 MB); if charts render blank in a local viewer, open it in Colab.

Dataset: [Daily Exchange Rates per Euro 1999–2022](https://www.kaggle.com/datasets/lsind18/euro-exchange-daily-rates-19992020/versions/59) (Kaggle, Daria Chemkaeva; source: ECB Statistical Data Warehouse), file `euro-daily-hist_1999_2022.csv`.

## Documents

- [Final notebook — report with outputs](ISE_201_Final.ipynb)
- [EDA notebook](ISE201_EDA_Maxim_Dokukin.ipynb)
- [Revised proposal](ISE201_Project_Proposal_Maxim_Dokukin_Revised.ipynb)
- References cited in the final notebook: Girardin & Salimi Namin (2019), *The January effect in the foreign exchange market*, Economic Modelling 81; Bassett et al. (2022), *Window dressing of regulatory metrics*, ECB Working Paper 2771; Hull (2018), *Options, Futures, and Other Derivatives*.
