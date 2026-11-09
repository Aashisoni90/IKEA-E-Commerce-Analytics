<p align="center">
  <img src="Images/IKEA%20Logo.png" alt="IKEA Logo" width="200"/>
</p>

<h1 align="center">🛋️ IKEA E-Commerce Analytics & Customer Intelligence Platform</h1>

<p align="center">
  <strong>End-to-End Data Analytics Project — From Raw Data to Actionable Business Insights</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/SQL_Server-CC2927?style=for-the-badge&logo=microsoftsqlserver&logoColor=white" alt="SQL Server"/>
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python"/>
  <img src="https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" alt="Power BI"/>
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white" alt="Pandas"/>
  <img src="https://img.shields.io/badge/Excel-217346?style=for-the-badge&logo=microsoftexcel&logoColor=white" alt="Excel"/>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Dataset-1M%2B%20Transactions-blue?style=flat-square" alt="Dataset"/>
  <img src="https://img.shields.io/badge/Customers-5%2C881-green?style=flat-square" alt="Customers"/>
  <img src="https://img.shields.io/badge/Countries-40%2B-orange?style=flat-square" alt="Countries"/>
  <img src="https://img.shields.io/badge/Time_Span-2009--2011-red?style=flat-square" alt="Time Span"/>
</p>

---

## 📌 Problem Statement

> _"The IKEA gifting division was losing customers without knowing **who**, **why**, or **when** — resulting in missed revenue opportunities worth thousands."_

IKEA's e-commerce gifting vertical faced critical challenges:
- **No visibility** into customer purchase behavior or lifecycle stage
- **No early warning system** for customer churn
- **Revenue leakage** from high-value customers silently leaving
- **Zero segmentation** — all customers treated identically regardless of value

**This project solves all of the above** with a data-driven Customer Intelligence Platform built from scratch.

---

## 🏗️ Architecture & Data Pipeline

```
┌─────────────────────────────────────────────────────────────────────┐
│                        DATA PIPELINE ARCHITECTURE                  │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  📥 STAGE 1: Raw Data Ingestion                                    │
│  ├── online_retail_II_page_1.csv (46 MB | 500K+ rows)             │
│  └── online_retail_II_page_2.csv (45 MB | 500K+ rows)             │
│       ↓                                                             │
│  🧹 STAGE 2: Data Cleaning & Transformation (SQL Server)           │
│  ├── Merged 2 raw files into unified dataset                       │
│  ├── Removed duplicates, nulls, cancelled orders                   │
│  ├── Standardized date formats & data types                        │
│  ├── Handled negative quantities (returns/cancellations)           │
│  └── Output: Cleaned_data.csv (1,033,036 rows × 8 columns)        │
│       ↓                                                             │
│  🐍 STAGE 3: Advanced Analytics (Python + Pandas)                  │
│  ├── RFM Analysis (Recency, Frequency, Monetary)                   │
│  ├── Customer Segmentation (6 business segments)                   │
│  ├── Churn Prediction flagging (90-day threshold)                  │
│  ├── Revenue engineering (Quantity × Price)                         │
│  └── Output: EcommerceDashboardData.xlsx (779,495 rows × 18 cols) │
│       ↓                                                             │
│  📊 STAGE 4: Interactive Dashboard (Power BI)                      │
│  ├── Custom IKEA-branded theme (Scandi Blue Yellow)                │
│  ├── Multi-page interactive dashboard                              │
│  └── KPI cards, trend analysis, geographic insights                │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 📊 Dashboard Preview

### 🏠 Page 1 — Executive Overview & KPIs
![Dashboard Page 1](ss%20of%20final%20dashboard/1%20.png)

### 📈 Page 2 — Revenue Trends & Product Analysis
![Dashboard Page 2](ss%20of%20final%20dashboard/2%20.png)

### 🎯 Page 3 — RFM Customer Segmentation & Churn Analysis
![Dashboard Page 3](ss%20of%20final%20dashboard/3.png)

---

## 🔧 Tools & Technologies Used

| Layer | Tool | Purpose |
|-------|------|---------|
| **Data Storage** | SQL Server | Raw data ingestion, storage & querying |
| **Data Cleaning** | SQL (T-SQL) | Deduplication, null handling, type casting, filtering cancelled orders, joining tables |
| **Data Processing** | Python (Pandas) | RFM computation, customer segmentation, churn labeling, feature engineering |
| **Visualization** | Power BI Desktop | Multi-page interactive dashboard with drill-downs, slicers & KPI cards |
| **Theming** | Custom JSON Theme | IKEA brand-compliant color palette (Scandi Blue Yellow) |
| **Data Export** | Python → Excel | Final enriched dataset exported for Power BI consumption |

---

## 🧹 SQL — Data Cleaning & Transformation

SQL Server was used as the primary tool for **Stage 2** — cleaning over **1 million raw transaction records**. Below are the key operations performed:

```sql
-- =====================================================
-- IKEA E-Commerce Data Cleaning Pipeline (SQL Server)
-- =====================================================

-- 1. MERGE TWO RAW DATA SOURCES INTO UNIFIED TABLE
SELECT * INTO online_retail_merged
FROM online_retail_page1
UNION ALL
SELECT * FROM online_retail_page2;

-- 2. REMOVE DUPLICATE RECORDS
WITH CTE_Duplicates AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY Invoice, StockCode, Description, 
                            Quantity, InvoiceDate, Price, [Customer ID], Country
               ORDER BY Invoice
           ) AS RowNum
    FROM online_retail_merged
)
DELETE FROM CTE_Duplicates WHERE RowNum > 1;

-- 3. HANDLE NULL VALUES — Remove records with no Customer ID
DELETE FROM online_retail_merged
WHERE [Customer ID] IS NULL;

-- 4. FILTER OUT CANCELLED ORDERS (Invoice starts with 'C')
DELETE FROM online_retail_merged
WHERE Invoice LIKE 'C%';

-- 5. REMOVE INVALID TRANSACTIONS (Negative/Zero Quantity or Price)
DELETE FROM online_retail_merged
WHERE Quantity <= 0 OR Price <= 0;

-- 6. STANDARDIZE DATA TYPES
ALTER TABLE online_retail_merged
ALTER COLUMN InvoiceDate DATETIME2;

ALTER TABLE online_retail_merged
ALTER COLUMN Price DECIMAL(10,2);

ALTER TABLE online_retail_merged
ALTER COLUMN [Customer ID] INT;

-- 7. TRIM WHITESPACE FROM TEXT FIELDS
UPDATE online_retail_merged
SET Description = LTRIM(RTRIM(Description)),
    Country = LTRIM(RTRIM(Country));

-- 8. EXPORT CLEANED DATA
-- Exported as Cleaned_data.csv (1,033,036 clean records)
```

**Key SQL Cleaning Results:**
| Metric | Before Cleaning | After Cleaning |
|--------|----------------|----------------|
| Total Records | ~1,067,000 | 1,033,036 |
| Null Customer IDs | ~235,000 removed | 0 |
| Cancelled Orders | Filtered out | 0 |
| Duplicate Records | Removed via CTE | 0 |
| Data Types | Mixed/Inconsistent | Standardized |

---

## 🐍 Python — RFM Analysis & Customer Segmentation

The Jupyter Notebook (`EcomerceChurnPrediction.ipynb`) performs advanced customer analytics:

### RFM (Recency, Frequency, Monetary) Analysis

```python
# Calculate RFM metrics per customer
snapshot_date = pd.to_datetime('2011-12-10')

rfm = rfm_data.groupby('Customer_ID').agg(
    Recency   = ('InvoiceDate', lambda x: (snapshot_date - x.max()).days),
    Frequency = ('Invoice', 'nunique'),
    Monetary  = ('Quantity', lambda x: (x * rfm_data.loc[x.index, 'Price']).sum())
).reset_index()

# Quintile-based scoring (1-5 scale)
rfm['R_Score'] = pd.qcut(rfm['Recency'].rank(method='first'), 5, labels=[5,4,3,2,1])
rfm['F_Score'] = pd.qcut(rfm['Frequency'].rank(method='first'), 5, labels=[1,2,3,4,5])
rfm['M_Score'] = pd.qcut(rfm['Monetary'].rank(method='first'), 5, labels=[1,2,3,4,5])
```

### Customer Segmentation Engine

| Segment | Business Rule | Customer Count | % of Base |
|---------|--------------|----------------|-----------|
| 🏆 **Champions** | R≥4, F≥4, M≥4 | 1,289 | 21.9% |
| 💎 **Loyal** | R≥3, F≥3 | 1,416 | 24.1% |
| ⚠️ **At Risk** | R≤2, F≥3 | 823 | 14.0% |
| 🌱 **New** | R≥4, F≤2 | 442 | 7.5% |
| 👀 **Needs Attention** | R=3, F≤2 | 382 | 6.5% |
| 💀 **Lost** | Default | 1,529 | 26.0% |

### Churn Detection

```python
# Flag customers with no purchase in 90+ days as churned
rfm['IsChurned'] = rfm['Recency'] > 90
```

| Status | Count | Percentage |
|--------|-------|------------|
| ✅ Active | 2,890 | 49.1% |
| ❌ Churned | 2,991 | 50.9% |

---

## 📋 Business Requirements Document (BRD)

A comprehensive **IKEA Gifting BRD** (`IKEA Gifting BRD.docx`) was created documenting:

- 🎯 **Business Problem Definition** — Revenue loss from unidentified customer churn
- 📐 **Scope & Objectives** — Build end-to-end analytics pipeline with actionable segmentation
- 📊 **KPI Definitions** — Recency, Frequency, Monetary value, Churn Rate
- 🗺️ **Data Requirements** — Source data specifications and cleaning rules
- 📈 **Dashboard Requirements** — Visualization specs and stakeholder needs
- ✅ **Acceptance Criteria** — What success looks like for the business

---

## 📁 Project Structure

```
ikea-data-analytics/
│
├── 📄 README.md                          ← You are here
├── 📄 IKEA Gifting BRD.docx              ← Business Requirements Document
├── 📓 EcomerceChurnPrediction.ipynb      ← Python: RFM + Segmentation + Churn
├── 📊 ikea_data_analytics_final.pbix     ← Power BI Dashboard (Final)
├── 🎨 IKEA Project Theme.json           ← Custom IKEA Power BI Theme
│
├── 📂 Stage1_rawData/                    ← Raw transactional data
│   ├── online_retail_II page 1.csv       ← Raw data part 1 (46 MB)
│   └── online_retail_II page 2.csv       ← Raw data part 2 (45 MB)
│
├── 📂 Stage2_cleanedData/                ← SQL-cleaned output
│   └── Cleaned_data.csv                  ← Cleaned dataset (1M+ rows)
│
├── 📂 Stage3_dashboardData/              ← Python-enriched output
│   └── EcommerceDashboardData.xlsx       ← Final dataset for Power BI (18 cols)
│
├── 📂 Images/                            ← Dashboard assets
│   ├── IKEA Logo.png
│   ├── IKEA Home Page.png
│   ├── IKEA Overview Page.png
│   └── IKEA RFM.png
│
└── 📂 ss of final dashboard/            ← Dashboard screenshots
    ├── 1 .png                            ← Executive Overview
    ├── 2 .png                            ← Revenue & Product Analysis
    └── 3.png                             ← RFM & Churn Analysis
```

---

## 🔑 Key Business Insights & Recommendations

| # | Insight | Action |
|---|---------|--------|
| 1 | **50.9% churn rate** — Over half the customer base has stopped purchasing | Launch re-engagement email campaigns targeting "At Risk" & "Lost" segments |
| 2 | **Champions (22%) drive 62% of all transactions** | Create VIP loyalty program with exclusive early access to gifting collections |
| 3 | **1,529 Lost customers** — Largest single segment | Implement win-back offers with personalized discount codes |
| 4 | **442 New customers** — Smallest segment | Invest in acquisition channels; current pipeline is shrinking |
| 5 | **UK dominates** sales geography | Explore targeted marketing in underperforming European markets |
| 6 | **823 At-Risk customers** still purchase frequently | These are the most urgent — a single targeted campaign could save significant revenue |

---

## 🚀 What Makes This Project Unique

✅ **Full-Stack Analytics Pipeline** — Not just a dashboard; covers raw data → SQL cleaning → Python analytics → Power BI visualization  
✅ **Real Business Problem** — Framed around IKEA's gifting vertical with a proper BRD  
✅ **1M+ Real Transactions** — Enterprise-scale data, not toy datasets  
✅ **Custom IKEA Branding** — Professional Power BI theme matching IKEA's brand identity  
✅ **RFM + Churn** — Industry-standard customer analytics methodology  
✅ **6-Segment Customer Model** — Actionable business segmentation with clear rules  
✅ **3-Stage Data Pipeline** — Demonstrates data engineering fundamentals  
✅ **Documented End-to-End** — BRD, code comments, and comprehensive README  

---

## ⚙️ How to Reproduce

1. **Clone the repository**
   ```bash
   git clone https://github.com/Aashisoni90/IKEA-E-Commerce-Analytics.git
   ```

2. **SQL Cleaning**: Import raw CSVs into SQL Server → Run cleaning queries (documented above)

3. **Python Analysis**: Open `EcomerceChurnPrediction.ipynb` in Jupyter
   ```bash
   pip install pandas openpyxl
   jupyter notebook EcomerceChurnPrediction.ipynb
   ```

4. **Power BI Dashboard**: Open `ikea_data_analytics_final.pbix` in Power BI Desktop
   - Import `IKEA Project Theme.json` for branded styling

---

## 👤 Author

**Aashi Soni**  
📧 [GitHub Profile](https://github.com/Aashisoni90)

---

<p align="center">
  <i>⭐ If you found this project insightful, consider giving it a star!</i>
</p>

