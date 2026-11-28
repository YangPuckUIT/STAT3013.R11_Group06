# Randomness, Pity Mechanisms, and Pricing in Gacha Games

STAT3013 group project

**Project title:** The Effects of Randomness, Pity Mechanisms, and Pricing on Player Satisfaction and Spending in Gacha Games: Implications for Game Revenue

This repository holds the analysis code, data and outputs for the group project. The report answers six research questions in six stages, using 510 cleaned survey responses and monthly revenue data for Genshin Impact and Honkai: Star Rail.

## The six stages

| Stage | Question | Methods |
|---|---|---|
| 1. Descriptive | Who are the players, how much do they spend, is spending concentrated? | Descriptive statistics, Cronbach's alpha, Gini, Lorenz curve |
| 2. Inferential | Are luck, pity and price linked to satisfaction, and satisfaction to continued play? | Spearman, chi-square, Holm correction |
| 3. Regression | Which factors are associated with higher spending? | Ordered logit |
| 4. Satisfaction | Do players who spend more also feel more satisfied? | % satisfied with Wilson 95% CI |
| 5. ML forecasting | Do new-character banners improve the revenue forecast? | ARIMA, ARIMAX, LightGBM |
| 6. Classification | Which factors help identify players who are playing less or have quit? | Logistic regression, random forest |

We write "is associated with", not "causes": the survey is a one-time snapshot.

## Data

- **Survey:** Google Forms, anonymous, adults (18+) who played a gacha game in the past 12 months. Recruitment was through online communities, so the sample is **not random**. The target population is adult Vietnamese gacha players active in online communities.
- **Cleaning (761 → 510 rows):** removed 8 non-players, 104 under-18s, 72 who failed the attention check (Q31), 60 with contradictory answers and 7 who gave the same answer to every question.
- **File:** `Dataset/gacha_clean.csv` (510 rows, 52 columns). Do not edit it by hand or re-save it from Excel.

Variables used in the core Stage 2 tests:

| Variable | From | Meaning |
|---|---|---|
| LUCK | Q13–15 | How lucky players feel (1–5 average) |
| PITY2 | Q17–18 | How fair the pity system feels (replaces PITY, alpha 0.54) |
| PRICE | Q19–21 | How fair prices feel |
| SAT | Q22–24 | Satisfaction |
| RET1 | Q28 | Plans to keep playing (replaces RET, alpha 0.48) |
| PROMO | Q33 | Spends more during promotion events (Yes/No) |
| SPEND_MONTH4 | Q5 | 0 = nothing, 1 = <100k, 2 = 100–300k, 3 = 300k+ VND per month |

The top three monthly spending bands (26, 10 and 15 respondents) are merged into one group of 51 so that the four spending groups are large enough and consistent across Stages 2 to 4.

## Stage 1 results

**Sample (n = 510).** 80.2% of respondents are aged 18–22. The most played games are Zenless Zone Zero (42.9%), Genshin Impact (29.6%) and Honkai: Star Rail (15.9%). 59.4% have played for more than two years and 74.9% still play as usual. 69.8% have ever spent money on their game and 57.3% spend every month.

**Scale reliability (Cronbach's alpha, threshold 0.70).**

| Scale | Items | Alpha | Decision |
|---|---|---|---|
| LUCK | Q13–15 | 0.82 | Used |
| PRICE | Q19–21 | 0.70 | Used |
| SAT | Q22–24 | 0.82 | Used |
| PITY | Q16–18 | 0.54 | Replaced by PITY2 (Q17–18, alpha 0.61, inter-item r = 0.44) |
| RET | Q28–30 | 0.48 | Replaced by RET1 (Q28, single item) |

**Spending concentration.** Lifetime spending is recorded in bands, so each respondent gets the band midpoint. The top band has no upper limit, so the Gini coefficient is computed under three assumptions:

| Top band | Gini (all) | Gini (spenders only) | Top 10% share |
|---|---|---|---|
| 20M VND | 0.75 | 0.64 | 56.9% |
| 25M VND | 0.77 | 0.66 | 60.5% |
| 40M VND | 0.80 | 0.72 | 68.4% |

Spending is highly concentrated under every assumption: the top 10% of spenders account for roughly 57–68% of all spending. Because band midpoints ignore differences within a band, the true Gini may be higher.

All tables and figures are written to `outputs/stage1/`.

## Stage 2 results

**Core tests (5), Holm-corrected within the family.** All five remain significant.

| Test | Effect | 95% CI | Adjusted p |
|---|---|---|---|
| LUCK ~ SAT (Spearman) | rho = 0.308 | [0.219, 0.393] | < 0.001 |
| PITY2 ~ SAT (Spearman) | rho = 0.443 | [0.366, 0.515] | < 0.001 |
| PRICE ~ SAT (Spearman) | rho = 0.454 | [0.382, 0.524] | < 0.001 |
| SAT ~ RET1 (Spearman) | rho = 0.504 | [0.434, 0.570] | < 0.001 |
| PROMO x SPEND_MONTH4 (chi-square) | chi2(3) = 144.44, V = 0.532 | – | < 0.001 |

Spearman confidence intervals are bootstrap percentile intervals (5,000 resamples, fixed seed). Sensitivity checks: replacing SAT with SAT2 (Q22–23 only) gives the same pattern, and the promotion association remains among the 292 respondents with positive monthly spending (V = 0.405).

**Supplementary tests (12), Holm-corrected within their own family (exploratory).** Four remain significant: objective versus perceived luck (rho = 0.198), getting the featured character versus satisfaction (rank-biserial r = 0.28), and game differences in satisfaction and price fairness (epsilon-squared = 0.056 and 0.041; Zenless Zone Zero players rank higher on both). We found no evidence of a trend of satisfaction across monthly spending groups (H5), of an association between being near soft pity and spending (H6), or between top-up trigger and spending.

All tables and figures are written to `outputs/stage2/`.

## Stage 5 results

**Data.** Monthly GLOBAL mobile revenue (Sensor Tower estimates via revenue.ennead.cc, USD, excluding China): Genshin Impact 2022-12 to 2026-09 (46 months) and Honkai: Star Rail 2023-04 to 2026-09 (42 months). Banner variables (`BANNER`, `HOT_CHAR`, `PROMO_EVENT`, 0/1 per month) were coded by hand from public banner histories. Sources and coding rules: `Dataset/stage5_sources.md`.

**Models.** ARIMA on log revenue, one model per game (baseline): ARIMA(1,1,2) for Genshin and ARIMA(0,1,1) for HSR, Ljung–Box p > 0.05 for both. ARIMAX uses the same orders with `BANNER`, `HOT_CHAR` and `PROMO_EVENT` as exogenous regressors (Ljung–Box p > 0.05 for both). LightGBM pooled over both games with revenue lags 1–3 and game, run without and with the banner variables (parameters fixed in advance, seed 42).

| Approach | Input variables | Number |
|---|---|---|
| ARIMA | Past revenue of the same game only | 1 series |
| ARIMAX | Past revenue + BANNER, HOT_CHAR, PROMO_EVENT | 1 series + 3 |
| LightGBM without banner | LAG1, LAG2, LAG3, GAME | 4 |
| LightGBM with banner | LAG1, LAG2, LAG3, GAME, BANNER, HOT_CHAR, PROMO_EVENT | 7 |

`variables_timeline.png` plots revenue with the months in which each banner variable equals 1; `variables_effect.png` shows the ARIMAX coefficients (with p-values) and the LightGBM gain share of every variable.

**Evaluation.** Rolling-origin one-month-ahead forecasts over the last 12 months (2025-10 to 2026-09).

| Game | Approach | RMSE (million USD) | MAPE (%) |
|---|---|---|---|
| Genshin | ARIMA | 6.96 | 28.3 |
| Genshin | ARIMAX (ARIMA with banner) | 6.16 | 27.3 |
| Genshin | LightGBM without banner | 7.13 | 32.8 |
| Genshin | LightGBM with banner | 6.43 | 30.3 |
| HSR | ARIMA | 7.60 | 53.7 |
| HSR | ARIMAX (ARIMA with banner) | 5.97 | 35.5 |
| HSR | LightGBM without banner | 8.19 | 58.4 |
| HSR | LightGBM with banner | 8.24 | 54.9 |

Adding the banner variables to ARIMA lowers RMSE by 11.4% for Genshin and 21.4% for HSR, and ARIMAX is the most accurate approach for both games. In LightGBM, banner variables lower Genshin RMSE by 9.9% but bring almost no change for HSR, where a new 5-star character debuts in 40 of 42 months. The individual ARIMAX banner coefficients are not statistically significant, so the gain comes from the variables together rather than from one clearly identified effect. MAPE is high for HSR because revenue fell to USD 5 million in 2026-01. With 12 evaluation months per game, these differences are tendencies, not firm conclusions.

All tables and figures are written to `outputs/stage5/`.

## Repository structure

```
.
├── Data_Analysis/
│   ├── Stage1_Descriptive.ipynb   # Stage 1 notebook
│   ├── Stage2_Inferential.ipynb   # Stage 2 notebook (single notebook)
│   ├── Stage3_Regression.ipynb    # Stage 3 notebook
│   └── Stage5_Forecasting.ipynb   # Stage 5 notebook
├── Dataset/
│   ├── gacha_clean.csv            # cleaned survey data (510 rows)
│   ├── revenue_monthly.csv        # Stage 5: monthly GLOBAL revenue (input)
│   ├── banner_calendar.csv        # Stage 5: banner variables (input)
│   ├── stage5_data.csv            # Stage 5: merged dataset (built by the notebook)
│   └── stage5_sources.md          # Stage 5: data sources and coding rules
├── outputs/
│   ├── stage1/                    # sample_profile, scale_scores, lorenz_curve (PNG) + 4 tables (CSV)
│   └── stage2/
│       ├── figures/               # spearman_forest, promotion_by_spending_group, sat_by_game (PNG)
│       └── tables/                # stage2_results, stage2_dunn_posthoc,
│   │                              # stage2_sensitivity_sat2, stage2_sensitivity_h5_six_levels (CSV)
│   └── stage5/
│       ├── figures/               # arima_forecast, lightgbm_forecast, forecast_comparison,
│       │                          # variables_timeline, variables_effect (PNG)
│       └── tables/                # stage5_data_summary, stage5_arima_*, stage5_arimax_*, stage5_lgbm_*,
│                                  # stage5_metrics, stage5_banner_effect, stage5_forecasts_all (CSV)
├── requirements.txt
└── README.md
```

## How to run

Requires Python 3.13.

```powershell
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
```

Open `Data_Analysis/Stage2_Inferential.ipynb` in VS Code, select the `.venv` kernel and choose **Restart and Run All**. The notebook reads `Dataset/gacha_clean.csv` and writes its tables and figures to `outputs/stage2/`.

Run `Data_Analysis/Stage1_Descriptive.ipynb` the same way; it writes its tables and figures to `outputs/stage1/`.

Run `Data_Analysis/Stage5_Forecasting.ipynb` the same way; it reads `Dataset/revenue_monthly.csv` and `Dataset/banner_calendar.csv`, builds `Dataset/stage5_data.csv` and writes its tables and figures to `outputs/stage5/`. LightGBM uses a fixed seed, so reruns give identical results.

## Limitations

- Non-random sample recruited online; about 80% of respondents are aged 18–22, and most play Zenless Zone Zero or Genshin Impact.
- Cross-sectional survey: results describe associations, not causes.
- Spending is self-reported in bands; the top bands are merged.
- Intention to keep playing (RET1) is concentrated at the top of the scale, which may lower its correlation with satisfaction.
- Some comparison groups are small (38 respondents who did not get the featured character; 26 Wuthering Waves players).

## Data source and use

Survey collected by the group for the STAT3013 course project. Responses are anonymous. This repository is for academic use.
