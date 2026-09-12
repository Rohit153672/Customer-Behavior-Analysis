# Customer Segmentation & Revenue Analytics

An end-to-end analysis of retail customer transaction data — from raw CSV to a business-facing dashboard — combining SQL-based analytics, machine learning segmentation, and interactive BI reporting to uncover where revenue comes from and which customers actually drive it.

## Overview

This project analyzes **3,900 customer transactions (~$233,081 in total purchase value)** to answer three questions:
1. Which product categories drive revenue?
2. How concentrated is spend among top customers?
3. Can customers be meaningfully segmented beyond simple purchase-count rules?

## Tech Stack

- **Python** (Pandas) — data cleaning, feature engineering
- **PostgreSQL** — relational data storage and SQL-based analysis
- **SQLAlchemy** — Python-to-PostgreSQL data pipeline
- **Scikit-learn** — K-Means clustering for customer segmentation
- **Power BI** — interactive dashboard for business stakeholders

## Pipeline

1. **Data Cleaning**
   - Filled missing `review_rating` values using category-wise median imputation (robust to outliers, preserves per-category rating patterns)
   - Removed a redundant duplicate column (`promo_code_used`, identical to `discount_applied`)
   - Standardized column names for consistent use across Python and SQL

2. **Feature Engineering**
   - Quartile-based age segmentation (`Young Adult` / `Adult` / `Middle-aged` / `Senior`) via `pd.qcut`
   - Converted categorical purchase frequency (e.g., "Weekly", "Quarterly") into a numeric `purchase_frequency_days` field

3. **Data Storage**
   - Loaded the cleaned dataset into PostgreSQL via SQLAlchemy, simulating a real analytics pipeline rather than a notebook-only workflow

4. **SQL Analysis** (`database_postgresql.sql`)
   - 10 business-question queries covering revenue by gender, discount behavior, product ratings, shipping comparisons, subscription analysis, rule-based loyalty segmentation, and top-products-per-category (via window functions)

5. **Customer Segmentation (K-Means)**
   - Clustered customers on `purchase_amount`, `previous_purchases`, and `purchase_frequency_days` (scaled via `StandardScaler`)
   - Selected k=4 using the elbow method and silhouette score, balancing statistical separation with business interpretability
   - Identified 4 distinct segments (see Key Findings)

6. **Dashboard (Power BI)**
   - KPI cards: customer count, average purchase amount, average review rating
   - Revenue/Sales by Category and by Age Group
   - Subscription status breakdown (donut chart)
   - Interactive slicers for filtering

## Key Findings

- **Category concentration:** Clothing (44.7%) and Accessories (31.8%) together account for **76.6%** of total revenue — a clear signal for where merchandising focus should go.
- **Customer concentration:** The top 20% of customers by spend contribute **31%** of total revenue — a moderate, not extreme, concentration, meaning revenue is fairly broad-based rather than driven by a small VIP tier.
- **Segmentation beyond purchase count:** A single-variable loyalty rule (purchase count only) classified 80% of customers as "Loyal" — too broad to act on. K-Means revealed that two behaviorally distinct groups were hiding inside that bucket:
  - **High-Value Loyalists** — high spend AND high purchase count (34.6% of revenue from 26% of customers)
  - **Frequent Low-Spenders** — nearly identical purchase count to the above, but less than half the spend
  - Two additional segments — **Growing/Occasional Spenders** (newer, already spending well) and **Annual/Rare Buyers** (buy ~once a year) — were also identified.
- **Subscription status is not a strong standalone value signal:** subscribers show no meaningful difference in spend, frequency, or satisfaction vs. non-subscribers — but is perfectly correlated with discount usage (100% of subscribers used a discount), suggesting either a business rule or a data-generation artifact worth confirming with the data owner.

## Setup

```bash
pip install pandas sqlalchemy psycopg2-binary scikit-learn matplotlib
```

Database credentials are loaded from environment variables — create a `.env` file (not committed) with:
```
DB_USERNAME=your_username
DB_PASSWORD=your_password
DB_HOST=localhost
DB_PORT=5432
DB_NAME=customer_behavior
```

## Repository Structure

```
├── customer_shopping_behavior.csv       # Raw dataset
├── customer_shopping_behavior.ipynb     # Cleaning, feature engineering, K-Means clustering
├── database_postgresql.sql              # SQL analysis queries
├── dashboard.pbix                       # Power BI dashboard
└── README.md
```

## Limitations

- Single-snapshot dataset — no purchase-date history, so trend, retention, and churn analysis aren't possible
- Synthetic/self-contained data — findings (e.g., the subscription/discount correlation) should be validated against real business context before acting on them
- k=4 for clustering was chosen for business interpretability, not because it was the statistically optimal split (silhouette score was highest at k=2)
