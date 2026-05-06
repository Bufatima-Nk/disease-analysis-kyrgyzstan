# Kyrgyzstan Disease Trends Analysis (2011–2022)

> Time series analysis and clustering of **18 disease categories** across Kyrgyzstan using 12 years of official health ministry data (206 observations). Applied K-Means clustering with elbow and silhouette selection, Poisson distribution modeling, year-over-year t-tests, and built an **interactive Dash dashboard** for exploring disease clusters dynamically.

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-KMeans%20%7C%20GMM-orange?logo=scikit-learn)
![Dash](https://img.shields.io/badge/Dash-Interactive%20Dashboard-green?logo=plotly)
![Domain](https://img.shields.io/badge/Domain-Public%20Health%20%7C%20Kyrgyzstan-red)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

---

## Why This Project Matters

Kyrgyzstan's public health data is rarely analyzed computationally. This project applies data science techniques directly to official national health statistics — making patterns in disease burden visible that are difficult to detect from raw tables. The findings have practical implications for healthcare resource allocation and epidemic preparedness in Central Asia.

---

## Results

### Clustering (K-Means, K=2 optimal by silhouette)

| Metric | K=2 | K=3 |
|--------|:---:|:---:|
| Silhouette Score | **0.881** | 0.047 |
| Inertia | 237.8 | 90.9 |

**K=2 is strongly optimal** (silhouette = 0.881 — excellent separation). K=3 was used for richer business interpretation; both available in the dashboard.

### Cluster Profiles (K=3)

| Cluster | Mean Year | Mean Cases/Year | Profile |
|---------|:---------:|:---------------:|---------|
| 0 | 2019.5 | 53,472 | Low-volume, recent years (2017–2022) |
| 1 | 2013.4 | 60,332 | Low-volume, earlier years (2011–2016) |
| **2** | 2016.2 | **517,570** | **High-volume diseases — Respiratory, Blood Circulation, Injuries** |

**Key finding:** Cluster 2 captures the high-burden diseases — predominantly Respiratory Diseases (peak: 687,141 cases in 2022), Blood Circulation Diseases, and Injuries/Poisonings. These three categories account for the majority of total case volume across all years.

### Top 5 Diseases by Total Cases (2011–2022)

| Rank | Disease | Total Cases |
|------|---------|:-----------:|
| 1 | Respiratory Diseases | ~6.9M |
| 2 | Blood Circulation Diseases | ~5.8M |
| 3 | Injuries/Poisonings | ~4.7M |
| 4 | Digestive Diseases | ~3.2M |
| 5 | Urogenital Diseases | ~2.1M |

### Year-Over-Year Statistical Testing (Welch's t-test)

All consecutive year pairs (2011→2012 through 2021→2022) showed **no statistically significant difference** in disease case distributions (all p-values > 0.05, range: 0.53–0.999). This confirms disease burden in Kyrgyzstan is **structurally stable year-to-year** — gradual trends rather than sudden shifts, with one notable exception: 2020 (COVID-19 impact visible in reduced reported cases, mean = 63,428 vs. 88,998 in 2019).

---

## Dataset

- **Source:** Kyrgyzstan Ministry of Health official statistics
- **Coverage:** 2011–2022 (12 years)
- **Diseases tracked:** 18 disease categories
- **Format:** Wide format (years as columns) → reshaped to long format via `pd.melt()`
- **Records after cleaning:** 206 observations (disease × year pairs)
- **Challenges:** Non-breaking spaces (`\xa0`) in numeric values, mixed NaN patterns from category splitting, duplicate "Blood Circulation" entries requiring deduplication

### Disease Categories

Infectious Diseases · Neoplasms · Endocrine/Digestive/Immune Disorders · Blood Circulation Diseases · Mental/Behavioral Disorders · Nervous System/Sense Organs · Eye Diseases · Ear Diseases · Respiratory Diseases · Digestive Diseases · Urogenital Diseases · Reproductive System · Skin Infections · Musculoskeletal Disorders · Congenital Anomalies · Ill-Defined Conditions · Perinatal Conditions · Injuries/Poisonings

---

## Approach

### 1. Data Preparation
The raw dataset arrived in wide format with years as columns — a common structure for government health statistics. Key cleaning steps:
- `pd.melt()` to reshape wide → long format
- Disease name standardization (mapping 20+ variants to 18 clean categories)
- Remove `'\xa0'` (non-breaking space) characters from numeric strings
- Drop rows with NaN (from split category entries in source data)
- Exclude aggregate "Diagnosed First Time" row to avoid double-counting

### 2. Statistical Analysis
- **Descriptive statistics:** Mean (86,189), median (56,364), std (122,053) — high right skew confirms a few high-burden diseases dominate
- **Yearly trends:** Mean cases ranged from 63,428 (2020, COVID impact) to 96,942 (2022)
- **Poisson modeling:** Fit Poisson distribution to disease case counts as a probabilistic model for rare event clustering
- **Normality testing:** Shapiro-Wilk test on synthetic reference data (p=0.207 → normal)
- **Year-over-year t-tests:** Welch's t-test for all 11 consecutive year pairs — no significant shifts detected

### 3. Clustering Pipeline
```python
model = make_pipeline(
    StandardScaler(),
    KMeans(n_clusters=k, random_state=42, n_init=10)
)
```
- Features: `Year` + `Number of cases each year`
- K range tested: 2–12
- Selection: Elbow method + silhouette score → K=2 optimal (0.881)
- K=3 used for interpretability (three temporal-volume groups)

### 4. Interactive Dashboard (Dash)
- Dropdown to select K (2–6) dynamically
- **Scatter plot:** Year vs. cases, color-coded by cluster — updates in real time
- **Bar chart:** Disease counts by cluster and year
- Enables non-technical stakeholders to explore segmentation without running code

---

## Key Findings

**1. Respiratory diseases are Kyrgyzstan's largest health burden** — 687,141 cases in 2022 alone, and growing. This reflects both genuine prevalence and improved reporting infrastructure.

**2. COVID-19 is clearly visible in 2020 data.** Mean cases dropped to 63,428 in 2020 vs. 88,998 in 2019 — a 28.7% reduction. This likely reflects underreporting of routine conditions during the pandemic rather than actual health improvement.

**3. Disease burden is structurally stable** — no year-over-year pair shows a statistically significant shift (all t-test p-values > 0.05). Public health planning in Kyrgyzstan can rely on historical averages for baseline resource allocation.

**4. High-burden and low-burden diseases form completely distinct clusters** (silhouette = 0.881 at K=2). Three diseases (Respiratory, Blood Circulation, Injuries) dominate volume; the remaining 15 categories form a separate low-volume cluster.

**5. Conditional probabilities between disease categories are all 0.** This is expected — each observation is a disease-level aggregate, not a patient-level record. True co-occurrence analysis would require patient-level data.

---

## Project Structure

```
disease-analysis-kyrgyzstan/
│
├── Final_Project_MF.ipynb          # Full pipeline: cleaning → EDA → stats → clustering → dashboard
├── diseases.csv                    # Official Kyrgyzstan health ministry dataset (2011–2022)
├── mffds_disease_analysis.pptx     # Project presentation slides
├── requirements.txt                # Dependencies
└── README.md
```

---

## How to Run

```bash
git clone https://github.com/Bufatima-Nk/disease-analysis-kyrgyzstan
cd disease-analysis-kyrgyzstan
pip install -r requirements.txt
jupyter notebook Final_Project_MF.ipynb
```

To run the interactive Dash dashboard, execute the last cell in the notebook, then open `http://127.0.0.1:8050` in your browser.

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Clustering | scikit-learn (KMeans, silhouette_score, StandardScaler, Pipeline) |
| Statistical Testing | scipy (Shapiro-Wilk, Welch's t-test, Poisson distribution) |
| Probabilistic Modeling | scipy.stats (poisson, norm) |
| Interactive Dashboard | Dash, Plotly Express, plotly.graph_objs |
| Data | pandas (melt, groupby, dropna), NumPy |
| Visualization | Matplotlib, Seaborn, Plotly Express |

---

## Limitations & Future Work

- **Aggregate data only:** The dataset contains population-level counts, not patient records. This limits analysis to macro trends — no individual risk factors, demographics, or co-morbidity analysis possible.
- **Silhouette score at K=3 is low (0.047):** The 3-cluster interpretation is narratively useful but statistically weak. K=2 is the correct mathematical answer; K=3 should be treated as an exploratory view only.
- **Predictive modeling:** Extend from descriptive clustering to forecasting — ARIMA or Prophet for individual disease trajectories, or a regression model predicting next-year case counts from historical trends.
- **Regional disaggregation:** National-level aggregates hide regional variation. Oblast-level data would enable spatial analysis and more targeted public health recommendations.
- **External factors:** Correlate case counts with economic indicators, healthcare spending, population data, and climate variables to understand drivers of disease burden.

---

## Author

**Bufatima N.K.**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-bufatima--n--k-blue?logo=linkedin)](https://linkedin.com/in/bufatima-n-k)
[![GitHub](https://img.shields.io/badge/GitHub-Bufatima--Nk-black?logo=github)](https://github.com/Bufatima-Nk)
