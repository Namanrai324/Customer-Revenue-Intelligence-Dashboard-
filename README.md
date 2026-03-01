# 🛍️ Customer Revenue Intelligence Dashboard

> **An end-to-end Data Analytics project** — from raw data to business insights using Python, PostgreSQL, and Power BI.

---

## 📌 Project Overview

This project analyzes **3,900+ customer shopping behavior records** to uncover revenue patterns, customer segmentation insights, and discount impact on business performance.

The goal is not just to visualize data — but to answer real business questions that drive decisions: *Who are our most valuable customers? Which categories drive revenue vs. just volume? Does discounting actually help?*

**Tools Used:** Python · PostgreSQL · Power BI · Pandas · Matplotlib · Seaborn · SQLAlchemy

---

## 📊 Dashboard Preview

![Customer Behavior Dashboard](Customer_Revenue_dashboard_screenshot.png)

> **Key Metrics at a glance:**
> - 👥 **3,900+** Total Customers
> - 💰 **$59.76** Average Purchase Amount
> - ⭐ **3.75** Average Review Rating
> - 📦 **4 Product Categories** — Clothing, Accessories, Footwear, Outerwear

---

## 🔑 Key Business Insights

**1. Clothing dominates — but at what cost?**
Clothing is the highest revenue and highest volume category. But without margin data, high volume can mask low profitability. Further investigation recommended.

**2. 73% of customers are non-subscribers**
Only 27% have active subscriptions. Yet subscribed customers show higher average spend. This is a significant revenue growth opportunity — targeted subscription conversion campaigns could materially increase LTV.

**3. Young Adults and Middle-Aged customers drive the most revenue**
These two segments contribute the largest share of total revenue by age group, making them the priority target for retention and upsell strategies.

**4. Discount abuse is a hidden risk**
Several products show 40%+ discount application rates. Customers who use discounts still meet average purchase thresholds — but habitual discounting erodes margin silently.

**5. Repeat buyers (>5 purchases) overwhelmingly don't subscribe**
This is the most actionable finding. Loyal, repeat customers who haven't subscribed represent the highest-probability conversion opportunity with the lowest acquisition cost.

---

## 🗂️ Project Structure

```
customer-revenue-intelligence/
│
├── data/
│   └── customer_shopping_behavior.csv       # Raw dataset
│
├── notebooks/
│   └── EDA_Perform_on_dataset_jupyternotebook.ipynb   # Full EDA + data cleaning + DB load
│
├── sql/
│   └── Customer_revenue_Postgresql_queries.sql        # 10 business queries in PostgreSQL
│
├── dashboard/
│   └── Customer_Revenue_dashboard_screenshot.png      # Power BI dashboard screenshot
│
└── README.md
```

---

## ⚙️ Project Workflow

### Step 1 — Data Loading & Exploration (Python + Pandas)

```python
import pandas as pd
df = pd.read_csv('customer_shopping_behavior.csv')
df.head()
df.info()
df.describe(include='all')
```

Loaded the raw dataset and performed initial profiling to understand shape, data types, and value distributions across all 3,900+ records.

---

### Step 2 — Data Cleaning

**Missing Values:**
```python
# Imputed missing Review Ratings with category-level median
df['Review Rating'] = df.groupby('Category')['Review Rating'].transform(
    lambda x: x.fillna(x.median())
)
```

**Column Standardization:**
```python
df.columns = df.columns.str.lower().str.replace(' ', '_')
df = df.rename(columns={'purchase_amount_(usd)': 'purchase_amount'})
```

**Dropped Redundant Column:**
`promo_code_used` was found to be 100% identical to `discount_applied` — confirmed programmatically and removed to avoid redundancy.

```python
(df['discount_applied'] == df['promo_code_used']).all()  # Returns True
df.drop('promo_code_used', axis=1, inplace=True)
```

---

### Step 3 — Feature Engineering

**Age Segmentation:**
```python
labels = ['Young Adult', 'Adult', 'Middle aged', 'Senior']
df['age_group'] = pd.qcut(df['age'], 4, labels=labels)
```

**Purchase Frequency in Days:**
```python
frequency_mapping = {
    'Weekly': 7, 'Fortnightly': 14, 'Bi-Weekly': 14,
    'Monthly': 30, 'Quarterly': 90, 'Every 3 Months': 90, 'Annually': 365
}
df['purchase_frequency_days'] = df['frequency_of_purchases'].map(frequency_mapping)
```

---

### Step 4 — Load to PostgreSQL

```python
from sqlalchemy import create_engine

engine = create_engine("postgresql+psycopg2://postgres:password@localhost:5432/customer_behavior")
df.to_sql('customer', engine, if_exists='replace', index=False)
```

Clean, transformed data was pushed directly from the notebook into a PostgreSQL database for structured querying.

---

### Step 5 — SQL Business Analysis (PostgreSQL)

10 business-focused queries were written to extract insights. Techniques used include **subqueries, CTEs, Window Functions, CASE statements, and aggregations**.

**Sample Queries:**

**Customer Segmentation using CTE:**
```sql
WITH customer_type AS (
    SELECT customer_id, previous_purchases,
        CASE 
            WHEN previous_purchases = 1 THEN 'New'
            WHEN previous_purchases BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal'
        END AS customer_segment
    FROM customer
)
SELECT customer_segment, COUNT(*) AS "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```

**Top 3 Products Per Category using Window Function:**
```sql
WITH item_counts AS (
    SELECT category, item_purchased,
           COUNT(customer_id) AS total_orders,
           ROW_NUMBER() OVER (PARTITION BY category ORDER BY COUNT(customer_id) DESC) AS item_rank
    FROM customer
    GROUP BY category, item_purchased
)
SELECT item_rank, category, item_purchased, total_orders
FROM item_counts
WHERE item_rank <= 3;
```

**Discount Abuse Detection:**
```sql
SELECT item_purchased,
    ROUND(100.0 * SUM(CASE WHEN discount_applied='Yes' THEN 1 ELSE 0 END) / COUNT(*), 2) AS discount_rate
FROM customer
GROUP BY item_purchased
ORDER BY discount_rate DESC
LIMIT 5;
```

➡️ See all 10 queries: [`Customer_revenue_Postgresql_queries.sql`](sql/Customer_revenue_Postgresql_queries.sql)

---

### Step 6 — Power BI Dashboard

Connected PostgreSQL database directly to Power BI. Built an interactive dashboard with slicers for **Subscription Status, Gender, Category, and Shipping Type**.

**Dashboard Pages:**
- **Executive Summary** — KPI cards: Total Customers, Avg Purchase Amount, Avg Review Rating
- **Revenue Analysis** — Revenue and Sales by Category and Age Group
- **Customer Segmentation** — Subscription breakdown, discount behavior, repeat buyer analysis

---

## 📋 Business Recommendations

Based on the full analysis, here are three actionable recommendations:

**1. Launch a loyalty-to-subscription conversion campaign**
Loyal customers (>10 previous purchases) who haven't subscribed represent ~X% of the base. A targeted offer — e.g., first month free or exclusive discount — has the highest probability of conversion at the lowest cost.

**2. Audit discount strategy for high-discount-rate products**
Products with 40%+ discount rates need margin review. If these customers meet average spend thresholds even with discounts, the discount may be unnecessary and can be reduced without losing the sale.

**3. Double down on Young Adult retention**
This segment drives the highest revenue. Retention investment here — personalized recommendations, loyalty rewards — delivers disproportionate ROI compared to acquiring new customers from lower-value segments.

---

## 🛠️ How to Run This Project

**Prerequisites:**
- Python 3.8+
- PostgreSQL installed and running
- Power BI Desktop (for dashboard)

**Setup:**

```bash
# Clone the repository
git clone https://github.com/yourusername/customer-revenue-intelligence.git
cd customer-revenue-intelligence

# Install Python dependencies
pip install pandas numpy matplotlib seaborn sqlalchemy psycopg2-binary jupyter

# Create PostgreSQL database
# Open pgAdmin → Create database named: customer_behavior

# Run the notebook
jupyter notebook notebooks/EDA_Perform_on_dataset_jupyternotebook.ipynb
```

After running the notebook, the cleaned data will be loaded into PostgreSQL automatically. Then open the SQL file in pgAdmin to run business queries.

---

## 📦 Dataset

**Source:** [Customer Shopping Behavior Dataset — Kaggle](https://www.kaggle.com/)

| Column | Description |
|---|---|
| customer_id | Unique customer identifier |
| age / age_group | Customer age and derived segment |
| gender | Male / Female |
| item_purchased | Product name |
| category | Clothing, Accessories, Footwear, Outerwear |
| purchase_amount | Transaction value in USD |
| review_rating | Product rating (1–5) |
| subscription_status | Yes / No |
| shipping_type | Standard, Express, etc. |
| discount_applied | Whether discount was used |
| previous_purchases | Count of past transactions |
| purchase_frequency_days | Engineered: frequency in days |

---

## 🧰 Tech Stack

| Tool | Purpose |
|---|---|
| Python + Pandas | Data loading, cleaning, feature engineering |
| Matplotlib + Seaborn | Exploratory data visualization |
| PostgreSQL | Structured business querying |
| SQLAlchemy | Python-to-PostgreSQL pipeline |
| Power BI | Interactive dashboard |

---

## 👤 Author

**Naman Kumar Rai**
B.Tech Data Science & AI — IIIT Naya Raipur

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue)](https://linkedin.com/in/yourprofile)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black)](https://github.com/yourusername)

---

## ⚠️ Important Note

Before running the notebook, replace the PostgreSQL password in the connection string with your own credentials. Never commit real passwords to GitHub — use environment variables in production.

```python
# Use this pattern instead
import os
password = os.environ.get("DB_PASSWORD")
```
