# Dataset Information

## Assignment 1 — Hotel Booking Demand

| Attribute | Detail |
|---|---|
| **Name** | Hotel Booking Demand |
| **Source** | Kaggle |
| **Link** | https://www.kaggle.com/datasets/jessemostipak/hotel-booking-demand |
| **Rows** | 119,390 booking records |
| **Columns** | 32 |
| **Period** | 2015 – 2017 |
| **Hotel Types** | City Hotel and Resort Hotel |
| **Dependent Variable** | `adr` — Average Daily Rate (USD per night) |
| **Key Predictors** | `lead_time`, `stays_in_week_nights`, `hotel` (dummy), `previous_bookings_not_canceled` |
| **Missing Values** | 4 in `children` (imputed with column mean) |

### How to Download

1. Visit the Kaggle link above (Kaggle account required)
2. Click **Download** → save as `hotel_bookings.csv`
3. Place the file in the root of this repository
4. Update the file path in `code/Assignment1_Indicator_Variables.Rmd` if needed

---

## Assignment 2 — Walmart Sales Dataset

| Attribute | Detail |
|---|---|
| **Name** | Walmart Sales Dataset |
| **Source** | Kaggle |
| **Link** | https://www.kaggle.com/datasets/yasserh/walmart-dataset |
| **Rows** | 6,435 weekly store-level records |
| **Columns** | 8 |
| **Period** | February 2010 – October 2012 |
| **Stores** | 45 Walmart stores across the United States |
| **Dependent Variable** | `Weekly_Sales` — total weekly revenue per store (USD) |
| **Key Predictors** | `Temperature`, `Fuel_Price`, `CPI`, `Unemployment`, `Holiday_Flag` |
| **Missing Values** | None |

### How to Download

1. Visit the Kaggle link above (Kaggle account required)
2. Click **Download** → save as `Walmart.csv`
3. Place the file in the root of this repository
4. Update the file path in `code/Assignment2_Autocorrelation.Rmd` if needed
