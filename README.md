# 🛍️ Customer Shopping Behavior Analysis

> **End-to-End Data Analytics Project** | Python • SQL • Power BI

A comprehensive retail analytics project analyzing **3,900 customer transactions** to uncover spending patterns, customer segments, product preferences, and subscription behavior — translating raw data into actionable business strategy.

---

## 📌 Problem Statement

A leading retail company is experiencing shifts in customer purchasing patterns across demographics, product categories, and sales channels — but lacks clarity on what is driving these changes.

Without understanding factors like discounts, seasonal trends, reviews, and payment preferences, the company cannot effectively engage customers or retain loyalty. This insight gap risks **missed revenue opportunities** and declining customer satisfaction.

**Overarching Question:** *"How can the company leverage consumer shopping data to identify trends, improve customer engagement, and optimize marketing and product strategies?"*

---

## 🎯 Project Objectives

| # | Objective | Description |
|---|-----------|-------------|
| 1 | **Analyze Spending Patterns** | Examine 3,900 transactions across product categories and demographics |
| 2 | **Identify Purchase Drivers** | Determine how discounts, reviews, seasons, and payment methods influence buying |
| 3 | **Segment Customers** | Group customers by behavior and preferences for targeted marketing |
| 4 | **Evaluate Sales Channels** | Compare online vs. offline performance to optimize resource allocation |
| 5 | **Subscription & Loyalty** | Analyze subscription patterns to improve retention strategies |
| 6 | **Optimize Strategy** | Translate insights into actionable marketing and product recommendations |

---

## 🗃️ Dataset Overview

- **Rows:** 3,900 customer records
- **Columns:** 18 features
- **Missing Values:** 37 missing Review Ratings (handled via imputation)

### Key Features
- **Customer Demographics:** Age, Gender, Location, Subscription Status
- **Purchase Details:** Item Purchased, Category, Purchase Amount (USD), Season, Size, Color
- **Shopping Behavior:** Discount Applied, Promo Code Used, Previous Purchases, Frequency of Purchases, Review Rating, Shipping Type

---

## 🔧 Analytical Workflow

```
Phase 1: Preparation (Python)
    ├── Impute missing ratings
    ├── Standardise columns
    └── Demographic binning
           ↓
Phase 2: Investigation (SQL)
    ├── Revenue analysis
    ├── Loyalty segmentation
    └── Product performance
           ↓
Phase 3: Visualisation (Power BI)
    ├── 3,900 customer records
    ├── Interactive dashboard
    └── Unified view
           ↓
Phase 4: Strategy
    ├── Subscription growth
    ├── Inventory optimisation
    └── Inventory management
```

---

## 🐍 Phase 1: Data Engineering (Python)

### Data Ingestion
```python
import pandas as pd
df = pd.read_csv('customer_shopping_behavior.csv')
```

### Missing Value Imputation
```python
# Category-based median imputation to preserve statistical validity
df['Review Rating'] = df.groupby('Category')['Review Rating'] \
                        .transform(lambda x: x.fillna(x.median()))
```

### Standardization
```python
df.columns = df.columns.str.lower()
df.columns = df.columns.str.replace(' ', '_')
df = df.rename(columns={'purchase_amount_(usd)': 'purchase_amount'})
```

### Feature Engineering
```python
# Demographic Segmentation
labels = ['Young Adult', 'Adult', 'Middle-aged', 'Senior']
df['age_group'] = pd.qcut(df['Age'], q=4, labels=labels)

# Behavioral Quantification
frequency_mapping = {
    'Fortnightly': 14, 'Weekly': 7, 'Monthly': 30,
    'Quarterly': 90, 'Bi-Weekly': 14, 'Annually': 365, 'Every 3 Months': 90
}
df['purchase_frequency_days'] = df['Frequency of Purchases'].map(frequency_mapping)
```

---

## 🗄️ Phase 2: SQL Investigation

### Demographics & Revenue
```sql
-- Revenue by Gender
SELECT Gender, SUM(`Purchase Amount (USD)`) AS Total_Revenue
FROM customer
GROUP BY Gender;

-- Revenue by Age Group
SELECT CASE
    WHEN Age < 18 THEN 'Under 18'
    WHEN Age BETWEEN 18 AND 25 THEN '18-25'
    WHEN Age BETWEEN 26 AND 35 THEN '26-35'
    WHEN Age BETWEEN 36 AND 50 THEN '36-50'
    WHEN Age BETWEEN 51 AND 65 THEN '51-65'
    ELSE '65+' END AS age_group,
    SUM(`Purchase Amount (USD)`) AS total_revenue
FROM customer
GROUP BY age_group
ORDER BY total_revenue DESC;
```

### Spending Behavior & Discounts
```sql
-- Value Shoppers: price-sensitive but high-value customers
SELECT `Customer ID`, `Purchase Amount (USD)`
FROM customer
WHERE `Discount Applied` = 'Yes'
  AND `Purchase Amount (USD)` >= (SELECT AVG(`Purchase Amount (USD)`) FROM customer);

-- Discount Dependency by Product
SELECT `Item Purchased`,
    COUNT(*) AS total_purchases,
    SUM(CASE WHEN `Discount Applied` = 'Yes' THEN 1 ELSE 0 END) AS discounted_purchases,
    ROUND(SUM(CASE WHEN `Discount Applied` = 'Yes' THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS discount_pct
FROM customer
GROUP BY `Item Purchased`
ORDER BY discount_pct DESC
LIMIT 5;
```

### Product Performance
```sql
-- Top Rated Items
SELECT `Item Purchased`, ROUND(AVG(`Review Rating`), 2) AS avg_rating
FROM customer
GROUP BY `Item Purchased`
ORDER BY avg_rating DESC
LIMIT 5;

-- Category Bestsellers (Window Function)
WITH item_counts AS (
    SELECT Category, `Item Purchased`, COUNT(`Customer ID`) AS total_orders,
        ROW_NUMBER() OVER (PARTITION BY Category ORDER BY COUNT(`Customer ID`) DESC) AS item_rank
    FROM customer
    GROUP BY Category, `Item Purchased`
)
SELECT item_rank, Category, `Item Purchased`, total_orders
FROM item_counts
WHERE item_rank <= 1;
```

### Loyalty & Retention
```sql
-- Customer Segmentation
WITH customer_type AS (
    SELECT `Customer ID`, `Previous Purchases`,
        CASE
            WHEN `Previous Purchases` = 1 THEN 'New'
            WHEN `Previous Purchases` BETWEEN 2 AND 10 THEN 'Returning'
            ELSE 'Loyal'
        END AS customer_segment
    FROM customer
)
SELECT customer_segment, count(*) as "Number of Customers"
FROM customer_type
GROUP BY customer_segment;
```

---

## 📊 Key Findings

### Demographics
| Metric | Value |
|--------|-------|
| Male Revenue | $157,890 |
| Female Revenue | $75,191 |
| Top Age Group (Revenue) | 18–25 (26.66%) |
| Avg Purchase Amount | $59.76 |
| Avg Review Rating | 3.75 ⭐ |

### Products
| Category | Top Item | Orders |
|----------|----------|--------|
| Clothing | Blouse / Pants | 171 each |
| Accessories | Jewelry | 171 |
| Footwear | Sandals | 160 |
| Outerwear | Jacket | 163 |

**Top Rated Items:** Gloves (3.86⭐), Sandals (3.84⭐), Boots (3.82⭐)

### Loyalty
| Segment | Count |
|---------|-------|
| Loyal (>10 purchases) | 3,116 |
| Returning (2–10) | 701 |
| New (1 purchase) | 83 |

**Subscription Gap:** 2,518 repeat buyers are NOT subscribed — a major monetization opportunity.

### Logistics
- Express Shipping avg spend: **$60.48**
- Standard Shipping avg spend: **$58.46**
- Express users are less price-sensitive → prime upsell segment

---

## 💡 Strategic Recommendations

### Marketing & Product
1. **Target High-Value Demographics** — Focus ad spend on Young Adults (18–25), the top revenue generators ($62K total contribution)
2. **Product Positioning** — Highlight top-rated items like Gloves and Sandals in campaigns to build brand trust (satisfaction > 3.8 stars)
3. **Discount Optimization** — Hats and Sneakers have ~50% discount penetration; pivot from flat discounts to bundles to protect margins

### Loyalty & Retention
1. **Close the Subscription Gap** — 73% of customers are non-subscribers. Create a 'Subscriber-Only' tier for Express Shipping to leverage higher spend ($60.48)
2. **Gamify Retention** — 701 customers are in the 'Returning' phase. Implement a specific reward for the 11th purchase to migrate them to the 'Loyal' segment (3,116 users)

---

## 🛠️ Tech Stack

| Tool | Purpose |
|------|---------|
| ![Python](https://img.shields.io/badge/Python-3776AB?style=flat&logo=python&logoColor=white) | Data cleaning, imputation, feature engineering |
| ![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat&logo=mysql&logoColor=white) | Analytical queries, segmentation, aggregation |
| ![Power BI](https://img.shields.io/badge/Power_BI-F2C811?style=flat&logo=powerbi&logoColor=black) | Interactive dashboard & visualization |

---

## 📁 Repository Structure

```
customer-shopping-behavior-analysis/
│
├── data/
│   └── customer_shopping_behavior.csv
│
├── notebooks/
│   └── data_cleaning_and_engineering.ipynb
│
├── sql/
│   ├── 01_demographics_revenue.sql
│   ├── 02_spending_discounts.sql
│   ├── 03_product_performance.sql
│   ├── 04_logistics_operations.sql
│   └── 05_loyalty_retention.sql
│
├── dashboard/
│   └── customer_behavior_dashboard.pbix
│
└── README.md
```

---

## 👤 Author

**Soumyadeep Dhar** — Passionate Data Analyst, transforming raw data into actionable insights.

[![LinkedIn](www.linkedin.com/in/soumyadeep-dhar-785724333)
[![GitHub](https://img.shields.io/badge/GitHub-dharsoumyadeep96-181717?style=flat&logo=github)](https://github.com/dharsoumyadeep96)
[![Gmail](https://img.shields.io/badge/Gmail-soumyadeepdhar433@gmail.com-EA4335?style=flat&logo=gmail)](mailto:soumyadeepdhar433@gmail.com)

*Currently doing Virtual Internship at AtliQ Technologies*

---

> ⭐ If you found this project helpful, please consider starring the repository!
