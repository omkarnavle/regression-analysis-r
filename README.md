# 📊 Regression Analysis Assignment

> **R-based regression analysis covering Indicator (Dummy) Variables and Autocorrelation,**
> **applied on real-world datasets from Kaggle.**

---

## 📌 Project Overview

This repository contains the complete submission for the **Regression Analysis** assignment as part of the **MSc in Statistics and Data Science** programme at **SVKM's Narsee Monjee Institute of Management Studies (NMIMS), Mumbai**.

The project is divided into two self-contained assignments, each demonstrating a different regression concept — from handling qualitative predictors through dummy variables to diagnosing and correcting serial correlation in time-series data.

| Detail | Value |
|---|---|
| Student | Omkar Navle |
| Roll No | A037 |
| SAP ID | 86062500050 |
| Programme | MSc Statistics and Data Science |
| Subject | Regression Analysis |
| Mentor | Mangesh Kutekar |
| Institution | Nilkamal School of Mathematics, Applied Statistics and Analytics, NMIMS Mumbai |
| Software | R (v4.3+) — R Markdown (.Rmd) |

---

## 📂 Repository Structure

```
regression-analysis/
│
├── code/
│   ├── Assignment1_Indicator_Variables.Rmd   ← R Markdown: dummy variable regression
│   └── Assignment2_Autocorrelation.Rmd       ← R Markdown: autocorrelation detection & correction
│
├── report/
│   └── A037_Omkar_Navle_RA_Assignment.pdf    ← Full written assignment report (PDF)
│
├── data/
│   └── dataset_info.md                       ← Dataset descriptions and Kaggle download links
│
├── images/
│   ├── adr_boxplot_by_hotel_type.png         ← Boxplot: ADR by hotel type
│   ├── full_model_output.png                 ← Full OLS model summary
│   ├── refined_model_output.png              ← Refined model summary
│   ├── residuals_over_time.png               ← Residuals vs time plot
│   ├── residual_lag_plot.png                 ← Residual lag scatter plot
│   ├── histogram_of_residuals.png            ← Histogram of OLS residuals
│   └── ols_vs_gls_comparison.png             ← OLS vs GLS coefficient comparison
│
├── .gitignore                                ← Excludes CSV data files and R output
└── README.md                                 ← This file
```

---

## 📘 Assignment 1 — Multiple Linear Regression with Indicator (Dummy) Variables

**Dataset:** Hotel Booking Demand | **Source:** [Kaggle](https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand)

### Objective

To investigate whether **hotel type** (City Hotel vs Resort Hotel) has a statistically significant effect on the **Average Daily Rate (ADR)** after controlling for booking lead time and length of stay — using a dummy variable regression framework.

### Dataset at a Glance

| Attribute | Detail |
|---|---|
| Total Observations | 119,390 booking records |
| Total Variables | 32 columns |
| Dependent Variable | `adr` — Average Daily Rate (USD per night) |
| Qualitative Predictor | `hotel` → converted to `hotel_dummy` (1 = Resort, 0 = City) |
| Numerical Predictors | `lead_time`, `stays_in_week_nights`, `previous_bookings_not_canceled` |
| Time Period | 2015 – 2017 |
| Missing Values | 4 in `children` — imputed with column mean |

---

### Workflow

#### Step 1 — Data Loading and Inspection

Libraries loaded: `readr`. Dataset read with `read_csv()`. Structure inspected with `str()` and `View()`.

#### Step 2 — Handling Missing Values

```r
df$children[is.na(df$children)] <- mean(df$children, na.rm = TRUE)
df$country[is.na(df$country)]   <- "Unknown"
df$agent[is.na(df$agent)]       <- 0
df$company[is.na(df$company)]   <- 0
```

After imputation, all 32 columns report zero missing values.

#### Step 3 — Creating the Dummy Variable

Since `hotel` has **p = 2** categories, exactly **p − 1 = 1** dummy is created. City Hotel is the reference (absorbed into the intercept).

```r
df$hotel_dummy <- ifelse(df$hotel == "Resort Hotel", 1, 0)
```

**Boxplot — justifying the dummy variable:**

![ADR by Hotel Type](images/adr_boxplot_by_hotel_type.png)

The boxplot confirms a visible distributional difference in ADR between the two hotel types, justifying the inclusion of a dummy predictor in the model.

---

#### Step 4 — Full OLS Model

```r
model <- lm(adr ~ lead_time + stays_in_week_nights +
                  previous_bookings_not_canceled + hotel_dummy,
            data = df)
summary(model)
```

**Full Model Output:**

![Full Model Output](images/full_model_output.png)

| Term | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| (Intercept) | 104.388 | 0.273 | 381.77 | < 2e-16 *** |
| lead_time | −0.045 | 0.001 | −32.94 | < 2e-16 *** |
| stays_in_week_nights | 2.853 | 0.079 | 36.07 | < 2e-16 *** |
| previous_bookings_not_canceled | −2.477 | 0.096 | −25.67 | < 2e-16 *** |
| hotel_dummy | −13.790 | 0.316 | −43.65 | < 2e-16 *** |

**Adjusted R² = 0.0308 | Residual SE = 49.75 | F-statistic = 949.9 (p < 2.2e-16)**

Every predictor is statistically significant at the 1% level. The F-statistic confirms joint significance. R² of 0.031 reflects typical variability in cross-sectional booking data.

---

#### Step 5 — Refined Model

`previous_bookings_not_canceled` is removed — statistically significant but practically minor.

```r
model_refined <- lm(adr ~ lead_time + stays_in_week_nights + hotel_dummy,
                    data = df)
summary(model_refined)
```

**Refined Model Output:**

![Refined Model Output](images/refined_model_output.png)

| Term | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| (Intercept) | 103.639 | 0.273 | 380.17 | < 2e-16 *** |
| lead_time | −0.043 | 0.001 | −31.25 | < 2e-16 *** |
| stays_in_week_nights | 2.931 | 0.079 | 36.98 | < 2e-16 *** |
| hotel_dummy | −13.859 | 0.317 | −43.75 | < 2e-16 *** |

**Adjusted R² = 0.0255 | Residual SE = 49.89 | F-statistic = 1041 (p < 2.2e-16)**

---

### Interpretation of Coefficients

The refined model produces **two parallel regression lines** — one per hotel type — sharing a common slope but differing in their intercepts:

**City Hotel (hotel_dummy = 0):**
```
ADR = 103.64 − 0.043 × lead_time + 2.931 × stays_in_week_nights
```

**Resort Hotel (hotel_dummy = 1):**
```
ADR = (103.64 − 13.86) − 0.043 × lead_time + 2.931 × stays_in_week_nights
    =  89.78 − 0.043 × lead_time + 2.931 × stays_in_week_nights
```

| Variable | Coefficient | Interpretation |
|---|---|---|
| (Intercept) | 103.639 | Baseline ADR for City Hotel with zero lead time and no weeknight stays |
| lead_time | −0.043 | Each extra day of advance booking reduces ADR by ~$0.04 (early-bird discounting) |
| stays_in_week_nights | 2.931 | Each additional weeknight stay adds ~$2.93 to the nightly rate |
| hotel_dummy (Resort) | −13.859 | Resort Hotels average $13.86 **less** per night than City Hotels, holding other factors constant |

> **Insight:** The negative dummy coefficient is counterintuitive but makes sense once lead time and stay length are controlled. City hotels serve business travellers who book last-minute for short stays — a premium segment captured by the positive `stays_in_week_nights` effect and the negative `lead_time` effect.

---

## 📗 Assignment 2 — Autocorrelation in Time-Series Regression

**Dataset:** Walmart Sales Dataset | **Source:** [Kaggle](https://www.kaggle.com/datasets/yasserh/walmart-dataset)

### Objective

To detect and correct **autocorrelation** in OLS residuals from a weekly retail sales regression model, using graphical diagnostics, the Durbin-Watson test, GLS, and the manual Cochrane-Orcutt transformation.

### Dataset at a Glance

| Variable | Type | Description |
|---|---|---|
| `Weekly_Sales` (Y) | Numerical | Total weekly revenue per store — **Dependent variable** |
| `Temperature` | Numerical | Average regional temperature that week (°F) |
| `Fuel_Price` | Numerical | Average fuel price in the region (USD/gallon) |
| `CPI` | Numerical | Consumer Price Index — regional inflation measure |
| `Unemployment` | Numerical | Regional unemployment rate (%) |
| `Holiday_Flag` | Binary | 1 = week includes a public holiday, 0 = otherwise |
| `Store` | Integer | Store identifier (1 to 45) |
| `Date` | Date | Week ending date — used to sort observations chronologically |

**Total Observations:** 6,435 | **Period:** February 2010 – October 2012 | **Stores:** 45

---

### Workflow

#### Step 1 — Data Loading and Preprocessing

```r
data$Date <- as.Date(data$Date, format = "%d-%m-%Y")
data <- data[order(data$Date), ]   # sort chronologically — critical for time-series
data <- na.omit(data)
```

#### Step 2 — OLS Regression Model

```r
model <- lm(Weekly_Sales ~ Temperature + Fuel_Price + CPI + Unemployment,
            data = data)
summary(model)
```

| Term | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| (Intercept) | 1,743,607.6 | 79,553.1 | 21.918 | < 2e-16 *** |
| Temperature | −885.7 | 396.2 | −2.235 | 0.0254 * |
| Fuel_Price | −12,248.4 | 15,751.8 | −0.778 | 0.4368 |
| CPI | −1,585.8 | 195.2 | −8.126 | 5.3e-16 *** |
| Unemployment | −41,215.0 | 3,972.7 | −10.375 | < 2e-16 *** |

**Adjusted R² = 0.0237 | Residual SE = 557,600 | F-statistic = 40.09 (p < 2.2e-16)**

CPI and Unemployment are strong negative predictors. `Fuel_Price` is not significant (p = 0.437). However, because observations are chronologically ordered, standard errors may be understated — autocorrelation must be tested formally.

---

#### Step 3 — Detection of Autocorrelation

**Plot 1 — Residuals Over Time:**

![Residuals Over Time](images/residuals_over_time.png)

The residuals display a cyclical wave pattern — long runs of positive residuals followed by long runs of negative residuals. This is the classic visual signature of **positive autocorrelation**.

**Plot 2 — Residual Lag Plot:**

![Residual Lag Plot](images/residual_lag_plot.png)

A clear positive linear relationship between e(t) and e(t−1) confirms that consecutive residuals are correlated.

**Plot 3 — Histogram of Residuals:**

![Histogram of Residuals](images/histogram_of_residuals.png)

The residuals are roughly symmetric but right-skewed, with a long positive tail driven by high-sales holiday weeks.

---

#### Step 4 — Durbin-Watson Test

```r
dwtest(model)
## DW = 1.9329, p-value = 0.003313
## alternative hypothesis: true autocorrelation is greater than 0
```

| Test Component | Value | Interpretation |
|---|---|---|
| DW Statistic | 1.9329 | Below 2 — indicates positive autocorrelation |
| p-value | 0.003313 | Highly significant — reject H₀ at 1% level |
| Decision | **Reject H₀** | Statistically significant positive autocorrelation confirmed |
| Estimated ρ | 0.03346 | Mild but real first-order autocorrelation |

Although the DW statistic is only slightly below 2 — suggesting **mild** rather than severe autocorrelation — the rejection is statistically unambiguous. Any inference that ignores this will produce unreliable standard errors.

---

#### Step 5 — Correction via GLS and Cochrane-Orcutt

**GLS (AR(1) via nlme):**

```r
gls_model <- gls(Weekly_Sales ~ Temperature + Fuel_Price + CPI + Unemployment,
                 data        = data,
                 correlation = corAR1())
```

Estimated AR(1) parameter **Phi = 0.034** | Residual SE = 557,642.5

**Manual Cochrane-Orcutt Transformation:**

```r
rho <- sum(e[-1] * e[-length(e)]) / sum(e[-length(e)]^2)
# rho = 0.03346

Y_star  <- Y[-1]  - rho * Y[-length(Y)]
X1_star <- X1[-1] - rho * X1[-length(X1)]
# ... (repeated for all predictors)

model_gls_manual <- lm(Y_star ~ X1_star + X2_star + X3_star + X4_star)
```

---

#### Step 6 — OLS vs GLS Comparison

![OLS vs GLS Comparison](images/ols_vs_gls_comparison.png)

| Variable | OLS Coefficient | GLS Coefficient | Key Change |
|---|---|---|---|
| (Intercept) | 1,743,607.6 | 1,755,066.2 | Slight increase in baseline |
| Temperature | −885.7 | −882.0 | Remains negative and significant |
| Fuel_Price | −12,248.4 | −15,601.4 | Magnitude increases; still **insignificant** |
| CPI | −1,585.8 | −1,577.4 | Very stable — robust predictor |
| Unemployment | −41,215.0 | −41,448.0 | Highly stable — strongest predictor |

`Fuel_Price` remains insignificant after GLS correction (p = 0.337). Corrected standard errors are slightly larger than OLS, reflecting the true uncertainty that serial correlation had masked.

#### Step 7 — Refined GLS Model

`Fuel_Price` is dropped as it is consistently insignificant across both OLS and GLS models.

```r
model_gls_manual_refined <- lm(Y_star ~ X1_star + X2_star + X4_star)
```

| Term | Estimate | Std. Error | t value | p-value |
|---|---|---|---|---|
| (Intercept) | 1,297,302.2 | 60,797.4 | 21.338 | < 2e-16 *** |
| X1_star (Temperature) | −1,717.0 | 394.7 | −4.350 | 1.38e-05 *** |
| X2_star (Fuel_Price) | 15,848.0 | 15,847.6 | 1.000 | 0.317 |
| X4_star (Unemployment) | −30,554.9 | 3,757.7 | −8.131 | 5.06e-16 *** |

---

### Summary and Conclusions

| Step | Action | Finding |
|---|---|---|
| 1 | OLS Regression | CPI and Unemployment significant; Fuel_Price not significant |
| 2 | Residual Plots | Time plot shows cyclical clustering; lag plot shows positive linear pattern |
| 3 | Durbin-Watson Test | DW = 1.9329, p = 0.003 — positive autocorrelation confirmed |
| 4 | GLS (nlme) | AR(1) Phi = 0.034; corrected SEs; Fuel_Price remains insignificant |
| 5 | Cochrane-Orcutt | Manual transformation confirms GLS results; rho = 0.033 |
| 6 | Refined Model | Temperature and Unemployment are robust predictors |

> **Key Takeaway:** Autocorrelation — even when mild — distorts standard errors and should never be ignored. The GLS Cochrane-Orcutt correction produces valid inference. CPI and Unemployment emerge as the most robust macroeconomic predictors of weekly retail sales, consistent with the economic theory that inflation and unemployment both suppress consumer purchasing power.

---

## 🛠️ How to Run the Code

### Prerequisites

- **R** (version 4.3 or later) — [Download here](https://cran.r-project.org/)
- **RStudio** (recommended) — [Download here](https://posit.co/downloads/)
- Required R packages (install once):

```r
install.packages(c("readr", "lmtest", "nlme"))
```

### Steps

1. **Clone this repository**

```bash
git clone https://github.com/your-username/regression-analysis.git
cd regression-analysis
```

2. **Download the datasets** from Kaggle (see `data/dataset_info.md` for links)
   - Save `hotel_bookings.csv` to the repository root
   - Save `Walmart.csv` to the repository root

3. **Open the R Markdown files** in RStudio

```
code/Assignment1_Indicator_Variables.Rmd
code/Assignment2_Autocorrelation.Rmd
```

4. **Update the file paths** inside each `.Rmd` file if your CSV is saved elsewhere

5. **Knit the document** — click **Knit → Knit to Word** (or HTML) in RStudio

---

## 📦 R Packages Used

| Package | Purpose |
|---|---|
| `readr` | Fast CSV loading with `read_csv()` |
| `lmtest` | Durbin-Watson test via `dwtest()` |
| `nlme` | GLS estimation with AR(1) correlation via `gls()` |
| Base R | `lm()`, `summary()`, `plot()`, `hist()`, `boxplot()`, `cor()` |

---

## 📈 Key Results Summary

### Assignment 1

| Finding | Value |
|---|---|
| Full model F-statistic | 949.9 (p < 2.2e-16) — jointly significant |
| Refined model F-statistic | 1,041 (p < 2.2e-16) |
| hotel_dummy coefficient | −13.859 — Resort Hotels earn $13.86 less per night than City Hotels |
| lead_time effect | −$0.043 per additional day — early-bird discount confirmed |
| stays_in_week_nights effect | +$2.93 per additional weeknight |

### Assignment 2

| Finding | Value |
|---|---|
| DW Statistic | 1.9329 (p = 0.003) — positive autocorrelation confirmed |
| Estimated ρ | 0.033 — mild first-order serial correlation |
| GLS AR(1) Phi | 0.034 — consistent with DW estimate |
| Fuel_Price | Insignificant in both OLS and GLS (p > 0.33) |
| Strongest predictor | Unemployment (t = −10.375) |

---

## 📄 Report

The complete written assignment report is available in the `report/` folder:

📋 [`A037_Omkar_Navle_RA_Assignment.pdf`](report/A037_Omkar_Navle_RA_Assignment.pdf)

The report covers:
1. Introduction and theory — indicator variables, dummy variable trap, autocorrelation
2. Dataset overviews with variable descriptions
3. Step-by-step R code with console outputs and plots
4. Full model and refined model estimation
5. Interpretation of coefficients with economic reasoning
6. Detection of autocorrelation — graphical and formal tests
7. GLS and Cochrane-Orcutt correction
8. OLS vs GLS comparison table
9. Conclusions and references

---

## 🔗 References

1. IGNOU. *Autocorrelation* (Unit 7). eGyanKosh Study Material.
   https://egyankosh.ac.in/bitstream/123456789/78535/1/Unit-12.pdf

2. IGNOU. *Regression Models with Indicator Variables* (Unit 9). eGyanKosh Study Material.
   https://egyankosh.ac.in/bitstream/123456789/88184/1/Unit-9.pdf

3. Hotel Booking Demand Dataset. Kaggle.
   https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand

4. Walmart Sales Dataset. Kaggle.
   https://www.kaggle.com/datasets/yasserh/walmart-dataset

---

## 📜 Licence

This project is submitted as academic coursework for the MSc Statistics and Data Science programme at NMIMS Mumbai.
The datasets are publicly available on Kaggle under their respective licences.
Code, analysis, and report content are original work by Omkar Navle.

---

*Built with R · NMIMS Mumbai · March 2026*
