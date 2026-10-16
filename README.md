# Data Case Study: Quantifying Hospitality Revenue Leakage

This repository contains an end-to-end data science and engineering project that identifies, quantifies, and predicts financial losses from booking cancellations. Using a **PostgreSQL** database backend, **R (tidyverse, ggplot2)** for data processing, and advanced econometric modeling (**ANOVA, Logistic Regression**), this case study isolates the core operational drivers of revenue leaks to provide data-driven strategies for hotel revenue managers.

---

## 🚀 Key Business Impact & Findings

* **Distribution Channels Matter Most:** A multiple linear regression and ANOVA variance analysis proved that **Market Segment explains up to 80% of macro financial variance**—impacting revenue leakage far more than seasonality or property type.
* **The "Online TA" Risk Profile:** Customers booking through Online Travel Agencies (TAs) have roughly **double the odds of canceling** (Odds Ratio: 1.8 – 2.2) compared to direct bookings. 
* **The Lead-Time Threat Multiplier:** Every single day of advance booking increases cancellation odds. Reservations booked **further out than 60 days** fall into hyper-volatile risk windows.
* **High-Cost Seasonality:** Cancellations in peak summer months (July/August) inflict the highest absolute financial damage due to elevated Average Daily Rates (ADR).

---

## 🛠️ Tech Stack & Architecture

* **Database Engine:** PostgreSQL (Server execution of financial metrics & aggregations)
* **Language:** R (v4.x)
* **Libraries:** `DBI`, `RPostgres`, `tidyverse` (dplyr, ggplot2), `scales`, `broom`
* **Methodologies:** Multiple Linear Regression, ANOVA (Sum of Squares Variance Decomposition), Aggregated Logistic Regression (`glm` Binomial Logit Matrix)

---

## 📂 Project Structure & Workflow

### 1. High-Performance ETL & Data Extraction (`PostgreSQL` + `RPostgres`)
Rather than pulling raw transactional logs into memory, the workflow pushes complex financial engineering directly onto the PostgreSQL database. This includes calculating a custom **Revenue Leakage metric** (ADR × Total Nights × Cancellation Indicator):

```sql
SELECT hotel, market_segment, arrival_date_month,
       COUNT(*) AS total_bookings,
       SUM(is_canceled) AS total_cancellations,
       ROUND((SUM(is_canceled)::NUMERIC / COUNT(*))*100, 2) AS cancellation_rate,
       ROUND(AVG(lead_time), 1) AS avg_lead_time_days,
       SUM(adr*(stays_in_week_nights + stays_in_weekend_nights) * is_canceled) AS lost_revenue
FROM hotel_revenue
WHERE market_segment != 'Undefined' AND adr >= 0
GROUP BY hotel, market_segment, arrival_date_month
ORDER BY lost_revenue DESC;
```

### 2. Macro-Level Diagnostic Analytics (Linear Regression & ANOVA)
To isolate which operational variables control financial losses, the project builds a multiple linear regression model evaluated through an ANOVA framework to quantify the percentage of explained variance:

```r
# Fit linear model to quantify financial loss drivers
model <- lm(lost_revenue ~ hotel + market_segment + arrival_date_month, data = revenue_df)

# Run ANOVA to isolate variance decomposition
anova_results <- anova(model)
```

### 3. Micro-Level Behavioral Modeling (Aggregated Logistic Regression)
Because the data structure is aggregated at the segment level, individual consumer churn choices are modeled using a two-column success/failure matrix (`cbind(Cancellations, Non-Cancellations)`) to extract clean **Odds Ratios (OR)**:

```r
churn_model_agg <- glm(
  cbind(total_cancellations, total_non_cancellations) ~ market_segment + avg_lead_time_days + hotel + arrival_date_month, 
  data = sql_data_cleaned, 
  family = binomial(link = "logit")
)
```

---

## 📈 Visualizations Produced
The R Markdown notebook generates clear, presentation-ready visual layouts. 

*To populate these on your GitHub page, export your R plots as images, upload them to an `/images` directory in this repository, and link them below:*

1. **Top 10 Highest Revenue Leakage Combinations**  
   ![Top Leakage Combinations]<img width="1350" height="705" alt="image" src="https://github.com/user-attachments/assets/2705fe1d-cbfc-4290-a2a4-232749722a17" />

2. **Total Revenue Leakage by Market Segment**  
   ![Leakage by Segment](images/segment_leakage_plot.png)
3. **ANOVA Feature Importance Chart**  
   ![ANOVA Variance Chart](images/anova_importance_plot.png)

---

## 💼 Strategic Business Recommendations

Based on the statistical outputs, this project recommends three immediate programmatic updates for revenue management:
1. **Targeted Deposit Restructuring:** Enforce strict, non-refundable deposits or compressed 48-hour cancellation windows specifically for third-party **Online TA** channels during peak summer occupancy.
2. **Automated Re-Verification Workflows:** Deploy automated email sequences or milestone verification deposits for any reservation passing the highly volatile **60+ day lead time** threshold.
3. **Calibrated Overbooking Matrix:** Use the empirical baseline channel cancellation rates to safely overbook inventory allocations by up to **15%** on volatile channels, protecting property yield without increasing walk risks.





