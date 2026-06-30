# Customer Cohort Analysis — E-commerce Sales Dashboard

> A full end-to-end data analysis project built in **Power BI**, covering customer cohort segmentation, lifetime value, retention behaviour, revenue trends, and order performance across an e-commerce dataset of **286,392 transactions** spanning October 2020 to September 2021.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Dataset Description](#dataset-description)
- [Tools Used](#tools-used)
- [Data Preparation & Cleaning](#data-preparation--cleaning)
- [Dashboard Walkthrough](#dashboard-walkthrough)
  - [Page 1 — Revenue & Customer Overview](#page-1--revenue--customer-overview)
  - [Page 2 — Order Performance & Payment Analysis](#page-2--order-performance--payment-analysis)
- [DAX Measures](#dax-measures)
- [Cohort Analysis](#cohort-analysis)
- [Key Findings & Insights](#key-findings--insights)
- [Recommendations](#recommendations)
- [Project Structure](#project-structure)
- [How to Use This Report](#how-to-use-this-report)

---

## Project Overview

This project analyses a real-world e-commerce transactional dataset to answer critical business questions around **customer loyalty**, **revenue growth**, **purchase behaviour**, and **order fulfilment quality**.

The analysis covers the complete data lifecycle — from raw CSV ingestion and cleaning in Python, to DAX measure creation and data modelling in Power BI, to interactive two-page visual storytelling. The report is fully interactive, with slicers for **Region**, **Payment Method**, **Year**, and **Gender** across both pages.

---

## Dataset Description

| Attribute | Detail |
|---|---|
| Source | E-commerce transactional records |
| Total rows | 286,392 order line items |
| Unique orders | 201,713 |
| Unique customers | 64,248 |
| Product categories | 15 |
| Date range | October 2020 – September 2021 |

**Columns in raw data:**

`order_id` · `cust_id` · `order_date` · `Customer Since` · `category` · `status` · `qty_ordered` · `price` · `discount_amount` · `payment_method` · `Region` · `Gender` · `age`

**Derived fields created during preparation:**

| Field | Formula |
|---|---|
| `total_line` | `qty_ordered × price − discount_amount` |
| `acquired_year` | Year extracted from `Customer Since` |
| `acquired_year_bin` | 5-year cohort bucket (e.g. 2000–2004) |
| `year_month` | Month-level period for trend analysis |

---

## Tools Used

| Tool | Purpose |
|---|---|
| **Python (pandas, numpy)** | Data cleaning, transformation, cohort binning, EDA |
| **Power BI Desktop** | Data modelling, DAX measures, interactive dashboard |
| **DAX** | KPI calculations, retention metrics, time intelligence |
| **GitHub** | Version control and project documentation |

---

## Data Preparation & Cleaning

All cleaning and transformation was done in Python before loading into Power BI.

### 1. Load and inspect
```python
import pandas as pd

df = pd.read_csv('sales.csv', low_memory=False)
print(df.shape)        # (286392, 13)
print(df.isnull().sum())  # No missing values — dataset is fully clean
```

### 2. Date standardisation
```python
df['order_date'] = pd.to_datetime(df['order_date'], dayfirst=True, errors='coerce')
df['Customer Since'] = pd.to_datetime(df['Customer Since'], errors='coerce')
df['acquired_year'] = df['Customer Since'].dt.year
df['year_month'] = df['order_date'].dt.to_period('M')
```

### 3. Revenue line calculation
```python
df['total_line'] = df['qty_ordered'] * df['price'] - df['discount_amount']
```

### 4. Cohort binning (5-year acquisition periods)
```python
df['acquired_year_bin'] = pd.cut(
    df['acquired_year'],
    bins=[1999, 2004, 2009, 2014, 2019, 2024],
    labels=['2000–2004', '2005–2009', '2010–2014', '2015–2019', '2020–2024']
)
```

### 5. Order-level aggregation
```python
orders = df.groupby('order_id').agg(
    total_sales=('total_line', 'sum'),
    total_qty=('qty_ordered', 'sum'),
    order_date=('order_date', 'first'),
    cust_id=('cust_id', 'first'),
    status=('status', 'first'),
    category=('category', 'first'),
    payment_method=('payment_method', 'first'),
    region=('Region', 'first')
).reset_index()
```

### 6. Customer-level aggregation
```python
cust = df.groupby('cust_id').agg(
    acquired_year=('acquired_year', 'first'),
    acquired_year_bin=('acquired_year_bin', 'first'),
    region=('Region', 'first'),
    gender=('Gender', 'first')
).reset_index()

cust_orders = orders.groupby('cust_id').agg(
    num_orders=('order_id', 'nunique'),
    total_revenue=('total_sales', 'sum'),
    avg_order_value=('total_sales', 'mean'),
    first_order=('order_date', 'min'),
    last_order=('order_date', 'max')
).reset_index()

cust = cust.merge(cust_orders, on='cust_id', how='left')
```

---

## Dashboard Walkthrough

### Page 1 — Revenue & Customer Overview

<img width="1311" height="735" alt="page 1" src="https://github.com/user-attachments/assets/9ac29300-4ed4-4955-aedc-8242e4fa3754" />

This page provides a top-level view of business performance, revenue distribution, and customer loyalty patterns.

#### KPI Cards

| Metric | Value |
|---|---|
| **Total Revenue** | $477.5M |
| **Total Orders** | 201,713 unique order IDs |
| **Avg Order Value** | $2,367 per customer avg |
| **Avg Purchase Frequency** | 3.14 orders per customer |

#### Monthly Revenue Trend (Clustered Column Chart)
- Fields: `order_date [Month]` × `Total Revenue`
- Revenue peaks sharply in **December** — the tallest bar by a wide margin, indicating a major year-end campaign or festive sale
- Strong secondary peaks visible in **April** and **June**
- **February** is consistently the weakest month

#### Revenue by Category — Top 6 (Clustered Bar Chart)
- Fields: `category` × `Total Revenue`

| Category | Revenue |
|---|---|
| Mobiles & Tablets | **$270.06M** |
| Appliances | $68.62M |
| Entertainment | $61.68M |
| Others | $21.7M |
| Computing | $20.42M |
| Women's Fashion | $11.17M |

> Mobiles & Tablets alone accounts for **56.5% of total revenue** — a significant category concentration.

#### Customer Retention (Donut Chart)
- Fields: `Total Customers` vs `Repeat Customers`
- **67.65%** are **one-time buyers**
- **32.35%** are **repeat buyers**
- Just under 1 in 3 customers returns — highlighting a strong re-engagement opportunity

#### Revenue by Region (Table)
- Fields: `Region` × `Total Revenue`

| Region | Total Revenue |
|---|---|
| **South** | $182,121,672.06 |
| Midwest | $128,505,736.43 |
| West | $84,779,086.71 |
| Northeast | $82,073,624.72 |

#### Slicers
Region · Payment Method · Year · Gender

---

### Page 2 — Order Performance & Payment Analysis

<img width="1313" height="739" alt="image" src="https://github.com/user-attachments/assets/4d270755-503e-4a84-8f47-e8ef9423295e" />

This page examines how orders are paid for, fulfilled, and how revenue has grown year-over-year.

#### Top Payment Methods by Order Volume (Clustered Bar Chart)
- Fields: `payment_method` × `Total Orders`

| Payment Method | Orders |
|---|---|
| **Cash on Delivery** | **61K** |
| Easypay | 51K |
| easypay_voucher | 26K |
| Payaxis | 24K |
| bankalfalah | 17K |
| Easypay_MA | 8K |
| jazzvoucher | 5K |
| jazzwallet | 5K |
| customercredit | 2K |
| apg | 1K |

> Cash on Delivery dominates at 61K orders — **20% more than Easypay**, the second most common method.

#### Order Status Breakdown (Funnel Chart)
- Fields: `status` × `Total Orders` (shown as % of total)

| Status | Share |
|---|---|
| **canceled** | **42.64%** |
| complete | 32.15% |
| received | 13.84% |
| order_refunded | 10.27% |
| refund | 0.67% |
| Cash on Delivery | 0.43% |

> ⚠️ More than **42% of all orders are canceled** — a major operational and revenue risk. Combined with the 10.27% refund rate, over **half of all orders never result in a retained sale**.

#### Revenue by Region — Year-over-Year (Table)
- Fields: `Year` × `Total Orders` × `Total Revenue`

| Year | Total Orders | Total Revenue |
|---|---|---|
| **2020** | 75,280 | $162,746,635.41 |
| **2021** | 126,433 | $314,733,484.51 |

> Revenue grew by **+93.4%** year-over-year and order volume grew by **+67.9%** — indicating both more customers and higher average spending in 2021.

#### Slicers
Region · Payment Method · Year · Gender

---

## DAX Measures

All KPIs and metrics in the dashboard are calculated using the following DAX measures:

### Revenue

```dax
Total Revenue =
SUMX(
    Sales,
    Sales[qty_ordered] * Sales[price] - Sales[discount_amount]
)

Avg Order Value =
DIVIDE(
    [Total Revenue],
    [Total Orders],
    0
)
```

### Orders & Customers

```dax
Total Orders =
DISTINCTCOUNT( Sales[order_id] )

Total Customers =
DISTINCTCOUNT( Sales[cust_id] )

Avg Purchase Frequency =
AVERAGEX(
    VALUES( Sales[cust_id] ),
    CALCULATE( DISTINCTCOUNT( Sales[order_id] ) )
)
```

### Retention

```dax
Repeat Customers =
COUNTROWS(
    FILTER(
        ADDCOLUMNS(
            VALUES( Sales[cust_id] ),
            "OrderCount", CALCULATE( DISTINCTCOUNT( Sales[order_id] ) )
        ),
        [OrderCount] > 1
    )
)

Retention Rate % =
DIVIDE( [Repeat Customers], [Total Customers], 0 ) * 100
```

### Customer Lifetime Value

```dax
Avg CLV =
AVERAGEX(
    VALUES( Sales[cust_id] ),
    CALCULATE( [Total Revenue] )
)

Cohort Avg CLV =
AVERAGEX(
    VALUES( Sales[cust_id] ),
    CALCULATE(
        SUMX( Sales, Sales[qty_ordered] * Sales[price] - Sales[discount_amount] )
    )
)
```

### Time Intelligence

```dax
Revenue PY =
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR( 'Date'[Date] )
)

YoY Growth % =
DIVIDE(
    [Total Revenue] - [Revenue PY],
    [Revenue PY],
    0
) * 100

MoM Growth % =
VAR _currentRev = [Total Revenue]
VAR _prevRev =
    CALCULATE(
        [Total Revenue],
        DATEADD( 'Date'[Date], -1, MONTH )
    )
RETURN
    DIVIDE( _currentRev - _prevRev, _prevRev, 0 ) * 100
```

### Order Status

```dax
Cancellation Rate % =
VAR _canceled =
    CALCULATE(
        DISTINCTCOUNT( Sales[order_id] ),
        Sales[status] = "canceled"
    )
RETURN
    DIVIDE( _canceled, [Total Orders], 0 ) * 100

Completion Rate % =
VAR _complete =
    CALCULATE(
        DISTINCTCOUNT( Sales[order_id] ),
        Sales[status] = "complete"
    )
RETURN
    DIVIDE( _complete, [Total Orders], 0 ) * 100
```

---

## Cohort Analysis

Customers were segmented by acquisition year into 5-year bins to understand how lifetime value varies across different customer generations.

| Cohort | Customers | Avg CLV | Avg AOV | Avg Purchase Freq |
|---|---|---|---|---|
| 2000–2004 | 4,726 | **$8,217** | $1,665 | 3.33× |
| 2015–2019 | 9,653 | $7,361 | $1,732 | 3.16× |
| 2010–2014 | 10,371 | $7,165 | $1,681 | 3.08× |
| 2005–2009 | 6,920 | $6,804 | $1,722 | 3.17× |

**What this tells us:**
- The oldest cohort (2000–2004) generates **21% more CLV** than the youngest measured cohort (2005–2009) despite having the fewest customers
- Purchase frequency is nearly identical across all cohorts (~3.1–3.3×) — meaning **loyalty drives spend, not just visit rate**
- Newer cohorts (2015–2019) are tracking close to the oldest cohort in CLV, suggesting the customer base quality is improving again after a mid-period dip

---

## Key Findings & Insights

### 📈 Revenue Growth
- Total revenue: **$477.5M** over 12 months
- Year-over-year growth: **+93.4%** ($162.7M → $314.7M)
- Order volume growth: **+67.9%** (75,280 → 126,433 orders)
- Revenue grew faster than order volume — meaning **average spend per order also increased** in 2021

### 📅 Seasonality
- **December** is by far the strongest month — the column chart shows a spike that dwarfs all other months, suggesting a major sale event or festive promotion
- **April** and **June** are secondary peaks — potential mid-year campaign windows
- **February** is the weakest month — typical post-holiday consumer slowdown

### 🏷️ Category Concentration Risk
- **Mobiles & Tablets: $270.06M** — 56.5% of all revenue from one category
- Top 3 categories (Mobiles, Appliances, Entertainment) account for **~84% of total revenue**
- Heavy concentration in one category creates significant revenue risk if supply, pricing, or competition shifts

### 👤 Customer Retention
- **67.65%** of customers are one-time buyers — the majority never return
- **32.35%** are repeat buyers — nearly 1 in 3 customers comes back
- The repeat buyer minority drives a disproportionate share of revenue through higher CLV

### ⚠️ Order Fulfilment Crisis
- **42.64%** of all orders are **canceled** — the single largest order outcome
- Only **32.15%** of orders are marked complete
- **10.27%** are refunded — bringing total non-retained orders to over **53%**
- Cash on Delivery (61K orders, the #1 payment method) is the most likely driver — these orders carry no upfront payment commitment, making cancellation easy

### 🌍 Regional Performance
- **South** leads with $182.1M — **38% of total revenue**
- **Midwest** is second at $128.5M
- **West** and **Northeast** contribute roughly equally (~$83M each)
- The South-Midwest axis drives **65% of all revenue**

### 💳 Payment Behaviour
- **Cash on Delivery (61K)** and **Easypay (51K)** dominate
- COD accounts for roughly **30% of all orders** — the highest of any single method
- The correlation between COD dominance and the 42.64% cancellation rate is a key area requiring business attention

---

## Recommendations

| # | Recommendation | Addresses |
|---|---|---|
| 1 | **Incentivise prepaid orders** — offer small discounts or loyalty points for non-COD payments to reduce cancellation risk | 42.64% cancellation rate |
| 2 | **Launch a 30-day win-back campaign** for one-time buyers — targeted emails or push notifications post-first-purchase to convert them to repeat buyers | 67.65% one-time buyer rate |
| 3 | **Diversify promotional investment** across Appliances and Entertainment — reduce over-reliance on Mobiles & Tablets | Category concentration (56.5%) |
| 4 | **Double down on December and Q2 peaks** — plan inventory, staffing, and campaigns well in advance of the seasonal spikes | Seasonal revenue pattern |
| 5 | **Study the oldest cohort's behaviour** — the 2000–2004 cohort's CLV of $8,217 reveals what a loyal customer looks like. Use it as an acquisition and retention benchmark | Cohort CLV gap |
| 6 | **Investigate COD-to-cancellation pipeline** — A/B test a "confirm before shipping" SMS/email flow for COD orders to reduce last-mile cancellations | Operational efficiency |

---

## Project Structure

```
Customer-Cohort-Analysis/
│
├── Customer_Cohort_Analysis.pbix     # Power BI report (2 pages)
├── sales.csv                         # Raw dataset (286,392 rows)
├── README.md                         # This document
├── screenshots/
│   ├── page1.png                     # Dashboard Page 1 screenshot
│   └── page2.png                     # Dashboard Page 2 screenshot
└── notebooks/
    └── data_preparation.ipynb        # Python data cleaning & EDA
```

---

## How to Use This Report

1. Clone or download this repository
2. Open https://app.powerbi.com/view?r=eyJrIjoiMzUzYjgyYTQtNzdlNS00OTRkLWIzNzMtZGRmYTVlNzhmYzNkIiwidCI6ImU2M2U3ZmQyLTk4ZjktNDNhMS1iNmU4LTg5ZDE1M2Q1OTg5MCJ9 in **Power BI Desktop** (free from Microsoft)
3. If prompted to refresh data, ensure `sales.csv` is in the same directory
4. Use the slicers on each page to filter by **Region**, **Payment Method**, **Year**, and **Gender**
5. Hover over any visual for detailed tooltips
6. Switch between pages using the tabs at the bottom of Power BI Desktop

---


