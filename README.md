# UK Online Retail Analysis: Customer Value, Seasonality & Retention

An end-to-end data analysis project built **entirely in Excel** (Power Query, Power Pivot/DAX, PivotTables and formulas) on more than **1 million real e-commerce transactions** from a UK-based online gift-ware retailer.

![Dashboard](images/dashboard.png)

---

## Business questions

1. How does revenue move through the year, and when is the peak season?
2. How big is the impact of cancellations, and is it getting worse?
3. Which products generate most of the revenue?
4. Who are the most valuable customers, and which customers are at risk of leaving?
5. How many new customers come back after their first purchase?

## Key results

| Metric | Value |
|---|---|
| Gross revenue (Dec 2009 – Dec 2011) | £19.40M |
| Orders | 39,514 |
| Average order value | £490.92 |
| Active customers | 5,852 |
| Cancellation rate | 2.43% |
| Revenue from *Champions* (26% of customers) | 71% |
| Products generating 80% of revenue | 22% (1,187 of 5,333) |
| Average month-1 retention of new customers | ~20% |

---

## Dataset

- **Source:** [Online Retail II, UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii) (Chen, D., 2012), licensed under CC BY 4.0
- **Period:** 1 Dec 2009 – 9 Dec 2011
- **Size:** 1,067,371 rows, 8 columns (Invoice, StockCode, Description, Quantity, InvoiceDate, Price, Customer ID, Country)
- **Business:** UK-based non-store online retailer selling all-occasion gift-ware; many customers are wholesalers
- The raw data is not included in this repository. Download it from the UCI link above.

## Tools

| Tool | Used for |
|---|---|
| Power Query | Loading 1M+ rows (beyond Excel's 1,048,576-row sheet limit), cleaning, customer and cohort tables |
| Power Pivot & DAX | Data model, Calendar table, 9 measures including time intelligence |
| PivotTables & PivotCharts | Trend, Pareto, cancellation and segment analysis |
| Excel formulas | RFM scoring (`PERCENTRANK.INC`, two-way `XLOOKUP`, structured table references) |
| Conditional formatting, slicers, linked pictures | Cohort heatmap and interactive dashboard |

---

## Process

### 1. Data cleaning (Power Query)

The two yearly sheets were appended, data types were set manually (Invoice and Customer ID as text, so cancellation invoices starting with "C" don't turn into errors), and helper columns were added: `Revenue`, `Is_Cancelled`, `Code_Type` and `OrderDate`.

| Step | Rows remaining | Removed | Reason |
|---|---|---|---|
| Raw data | 1,067,371 | – | |
| Remove non-product codes | 1,061,436 | 5,935 | Postage, discounts, manual adjustments, bank charges and fees are not product sales |
| Remove price ≤ 0 | 1,055,287 | 6,149 | Stock adjustments, damaged goods and bad-debt write-offs |
| Remove exact duplicates | 1,021,252 | 34,035 | Rows identical in every column |
| Remove anomalous orders | **1,021,248** | 4 | Two very large orders (£168,470 and £77,184) that were cancelled almost immediately |
| **Total kept** | **95.7%** | **46,119 (4.3%)** | |

**Kept on purpose:**
- **Cancellations** are kept and flagged with `Is_Cancelled`, so the cancellation rate can be measured.
- **Rows without a Customer ID** (~23% of raw rows) are real sales and count toward revenue, but they are excluded from customer-level analysis (RFM and cohorts).

**Rule validation:** the first non-product rule ("StockCode starts with 5 digits") wrongly flagged real products with codes like `DCGS0058`. The rule was checked against the flagged rows and corrected before any rows were removed.

Details: [docs/power_query_steps.md](docs/power_query_steps.md)

### 2. Data model (Power Pivot)

- `Sales_Clean` fact table (1,021,248 rows) related to a `Calendar` date table
- 9 DAX measures: Gross Revenue, Cancelled Value, Net Revenue, Total Orders, AOV, Cancellation Rate, Active Customers, Revenue PM and Revenue MoM %

Details: [docs/dax_measures.md](docs/dax_measures.md)

### 3. Analysis

- **Monthly trend** of gross revenue and month-over-month growth
- **Cancellation investigation**: traced a 2011 spike in the cancellation rate to two anomalous orders
- **Pareto analysis** of products with a cumulative revenue share

### 4. RFM segmentation

A customer table (one row per customer) was built in Power Query with two Group By steps (invoice level, then customer level). Each customer was scored 1–5 on Recency, Frequency and Monetary value using `PERCENTRANK.INC`, then mapped to 11 segments (Champions, Loyal Customers, At Risk, Can't Lose Them, Lost and others) with a two-way `XLOOKUP` on an R × FM grid.

Details: [docs/rfm_method.md](docs/rfm_method.md)

### 5. Cohort retention

Customers were grouped by the month of their first purchase. For each cohort, the share of customers who purchased again in each following month was calculated with a PivotTable (`Show Values As → % Of`, base item = month 0) and shown as a heatmap.

![Cohort retention heatmap](images/cohort_heatmap.png)

### 6. Dashboard

One-page Excel dashboard with KPI cards, monthly trend, top products, top international markets, RFM segments and the cohort heatmap. Year and Country slicers control the KPI cards and the sales charts. RFM and cohort views always show the full period.

---

## Insights

### 1. Revenue peaks every November
Sales climb from September and peak in **November** in both years (**£1.43M in 2010** and **£1.45M in 2011**), about **2.2× a typical month** (Feb–Aug average ≈ £656K). Since most customers are wholesalers buying gift-ware, they likely stock up ahead of Christmas. Off-season revenue was flat year over year (+0.2%), while September 2011 was up ~18% on September 2010. Growth came almost entirely from the peak season.

### 2. The 2011 "cancellation spike" was caused by two orders
At first the cancellation rate looked like it nearly doubled in 2011 (2.54% → 4.84%). The jump came entirely from **two anomalous orders worth £245K combined** that were placed and cancelled almost immediately. After removing them, 2011 (**2.31%**) is in line with 2010 (**2.54%**). Cancellations are stable at around 2.3–2.5% of gross revenue.

### 3. 22% of products generate 80% of revenue
**1,187 of 5,333 products (22.3%)** account for 80% of revenue, led by *Regency Cakestand 3 Tier* (£330.6K) and *White Hanging Heart T-Light Holder* (£261.0K). Top sellers are mostly affordable home décor and party items that wholesalers buy in bulk.

### 4. 26% of customers drive 71% of revenue
| Segment | % of customers | % of revenue | Avg revenue per customer |
|---|---|---|---|
| Champions | 26.0% | 71.4% | £7,885 |
| Loyal Customers | 9.5% | 11.0% | £3,322 |
| At Risk | 13.7% | 6.1% | £1,292 |
| Can't Lose Them | 1.7% | 4.1% | £6,801 |
| Potential Loyalists | 11.8% | 2.7% | £657 |
| Need Attention | 8.3% | 2.0% | £684 |
| Lost | 14.6% | 1.4% | £274 |
| Hibernating | 10.0% | 1.1% | £305 |
| About to Sleep | 2.1% | 0.1% | £155 |
| Promising | 1.6% | 0.1% | £157 |
| New Customers | 0.7% | 0.0% | £168 |

**901 customers (15%)** in *At Risk* and *Can't Lose Them* once spent **£1.73M (10% of revenue)** but haven't purchased recently. *Can't Lose Them* customers are almost as valuable as Champions (avg £6.8K each).

### 5. Only ~20% of new customers come back the next month
Average month-1 retention across cohorts from Jan 2010 to Nov 2011 is **~20%** (range 9–32%). Returning activity spikes every November. Customers acquired in December have the weakest retention (**Dec 2010: 9%**), suggesting many are one-off Christmas buyers.

## Recommendations

1. **Protect Champions:** a loyalty or VIP program with early access to the seasonal catalog.
2. **Win back At Risk and Can't Lose Them customers** with a targeted campaign **before the September peak**.
3. **Prepare for the peak:** secure stock for the ~1,200 core products and fulfilment capacity by August.
4. **Grow the second order:** a first-30-days reorder offer for new customers, plus a January follow-up for customers acquired in December.
5. **Add a validation check** for unusually large single orders before they are processed.

---

## Limitations

- **Left-censoring:** the data starts in December 2009, so the Dec 2009 cohort includes long-standing customers, not only new ones. It is not used as the retention benchmark.
- **Partial last month:** December 2011 only covers 1–9 December, so it looks lower in the trend and in the last cohort column.
- **Missing Customer IDs:** ~13% of clean revenue (£2.58M) has no Customer ID, so RFM and cohort results cover £16.82M of the £19.40M total.
- **Products are grouped by Description:** some StockCodes have more than one description, so product counts can differ slightly from a StockCode-level count.
- **RFM thresholds are relative** (percentile-based) and describe customers within this dataset only.

## Repository structure

```
UK-Online-Retail-Analysis/
├── README.md
├── Online_Retail_Analysis.xlsx   # Excel workbook (Power Query, data model, analysis, dashboard)
├── images/
│   ├── dashboard.png
│   └── cohort_heatmap.png
└── docs/
    ├── power_query_steps.md
    ├── dax_measures.md
    └── rfm_method.md
```

## How to open the workbook

1. Use Excel for Windows (Microsoft 365 or 2016+). Power Pivot is required for the data model.
2. Download `online_retail_II.xlsx` from the UCI link above.
3. Open `Online_Retail_Analysis.xlsx`, go to **Data → Queries & Connections**, and update the source path of the two yearly queries to your downloaded file if prompted.
4. Click **Data → Refresh All**.

---

## Author

**M Fariel Fahriza Tielung**
[LinkedIn](https://www.linkedin.com/in/m-fariel-fahriza-tielung-8250522a4) · [GitHub](https://github.com/farielfahriza)
