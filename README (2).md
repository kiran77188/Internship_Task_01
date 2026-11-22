# Online Retail Sales Analysis: Data Cleaning, EDA and Power BI Dashboard

End-to-end analytics case study on 541,909 e-commerce transactions: raw data → cleaning (Python) → exploratory analysis (Python) → interactive dashboard (Power BI, DAX) → business insights.

![Dashboard](images/dashboard.png)

## 1. Summary

| | |
|---|---|
| **Business** | UK-based online retailer selling gift and homeware products (many customers are wholesalers, per the dataset source) |
| **Period** | 17 Nov 2026 – 22 Nov 2026 |
| **Raw data** | 541,909 rows × 8 columns |
| **Cleaned data** | 536,641 rows × 9 columns |
| **Gross sales** | £10.63M (£9.74M after cancellations) |
| **Customers** | 4,339 identified customers, 22,061 sales invoices |
| **Tools** | Python (pandas, matplotlib, seaborn), Jupyter, Power BI, DAX |

**Headline findings:** sales are strongly seasonal (Sep–Nov = 34.9% of all revenue), the UK is 84.6% of sales, and cancellations remove £894K (8.4%) of gross sales, with two bulk orders alone making up 27.5% of that.

## 2. Problem Statement

The business has one year of transaction data but no clear view of how it performs. The goal is to turn the raw invoice data into a clean dataset and a dashboard that answers:

1. How much revenue, quantity and how many customers does the business have, and how does revenue change over time?
2. Which months and days of the week are strongest?
3. Which products and countries drive sales?
4. How big a problem are cancellations and returns?
5. Where are the data quality risks that could mislead reporting?

## 3. Dataset

- **Source:** [UCI Machine Learning Repository, Online Retail dataset](https://archive.ics.uci.edu/dataset/352/online+retail) (included as `data/raw/Online Retail.xlsx`)
- **Grain:** one row per product line on an invoice
- **Countries:** 38 country values (after standardisation)

| Column | Description |
|---|---|
| `invoiceno` | Invoice number. Prefix `C` = cancellation, prefix `A` = accounting adjustment |
| `stockcode` | Product code (also holds non-product codes such as `POST`, `DOT`, `M`) |
| `description` | Product name |
| `quantity` | Units per line (negative on cancellations) |
| `invoicedate` | Invoice date and time |
| `unitprice` | Price per unit in GBP |
| `customerid` | Customer ID (missing for ~25% of rows) |
| `country` | Customer country |
| `transaction_type` | **Added in cleaning:** `Normal / Other`, `Cancellation`, `Accounting Adjustment` |

## 4. Data Cleaning

Notebook: [`notebooks/data_cleaning.ipynb`](notebooks/data_cleaning.ipynb). Output: `data/cleaned/cleaned_retails.csv`.

| Step | Finding | Action |
|---|---|---|
| Column names | Mixed case, spaces | Stripped, lowercased, removed spaces |
| Missing values | `customerid` 135,080 (24.93%), `description` 1,454 (0.27%) | **Kept** blank customer IDs (guest checkouts), excluded them from customer counts only |
| Duplicates | 5,268 exact duplicate rows | Removed → 536,641 rows |
| Negative quantity | 10,587 rows (1.97%): 9,251 on `C` invoices, 1,336 on other invoices | Classified, not deleted |
| Transaction type | Sales, cancellations and adjustments mixed in one column set | Created `transaction_type`: 527,387 Normal / 9,251 Cancellation / 3 Accounting Adjustment |
| Negative price | 2 rows, both "Adjust bad debt" | Labelled Accounting Adjustment, excluded from KPIs |
| Country names | `EIRE`, `USA`, `RSA` | Standardised to `Ireland`, `United States`, `South Africa` |

Additional checks in [`notebooks/EDA.IPYNB`](notebooks/EDA.IPYNB):

- Missing descriptions filled from other rows with the same `stockcode`; one product code (22139) fixed manually.
- 2,510 zero-price rows identified.
- Unit price outliers checked with the IQR rule: 39,450 rows (7.35%) flagged and **kept**, since they are genuine bulk and wholesale lines rather than errors.
- Derived `Year`, `month`, `day_name` and `revenue = quantity × unitprice`.

## 5. Analysis

Notebook: [`notebooks/EDA.IPYNB`](notebooks/EDA.IPYNB). Questions explored: country distribution, transaction types, price distribution and outliers, monthly and weekday revenue, top products by quantity and revenue, customer revenue and frequency, cancellations by product and country, and correlation of quantity, price and revenue.

### Monthly revenue
![Monthly revenue](c:\Users\Admin\Downloads\Screenshot 2026-11-22 141558.png)

### Revenue by day of week
![Revenue by day](c:\Users\Admin\Downloads\Screenshot 2026-11-22 141525.png)

### Top countries
![Top countries](c:\Users\Admin\Downloads\Screenshot 2026-11-22 141558.png)

## 6. Dashboard

File: [`retail.dashboard.pbix`](retail.dashboard.pbix). One page (1600 × 1000) built in Power BI Desktop on `cleaned_retails.csv`.

| Visual | Purpose |
|---|---|
| 4 KPI cards | Total Sales, Total Quantity, Cancelled Orders, Total Customers |
| Line chart | Monthly Revenue Analysis |
| Column chart | Revenue by day of week (Top 6 days) |
| Bar chart | Top 5 best-selling products (by quantity) |
| Donut chart | Top 5 best-selling countries (by sales) |
| 3 slicers | Year, Month, Day |

**DAX measures** (all sales measures filter on `transaction_type = "Normal / Other"`):

```DAX
Total Sales =
CALCULATE(
    SUMX(cleaned_retails, cleaned_retails[quantity] * cleaned_retails[unitprice]),
    cleaned_retails[transaction_type] = "Normal / Other"
)

Total Quantity =
CALCULATE(SUM(cleaned_retails[quantity]), cleaned_retails[transaction_type] = "Normal / Other")

Total Customers =
CALCULATE(DISTINCTCOUNTNOBLANK(cleaned_retails[customerid]), cleaned_retails[transaction_type] = "Normal / Other")

Total Orders =
CALCULATE(DISTINCTCOUNT(cleaned_retails[invoiceno]), cleaned_retails[transaction_type] = "Normal / Other")

Cancelled Orders =
CALCULATE(COUNT(cleaned_retails[invoiceno]), cleaned_retails[transaction_type] = "Cancellation")
```

**KPI values (all data, no slicers):**

| KPI | Value |
|---|---|
| Total Sales | £10,631,049 |
| Total Quantity | 5,438,059 units |
| Total Customers | 4,339 |
| Total Orders | 22,061 invoices (average £481.89 per invoice) |
| Cancelled Orders | 9,251 cancelled lines (across 3,836 invoices) |

## 7. Key Business Insights

1. **Strong seasonality.** Sep–Nov 2011 brought in £3.71M, 34.9% of all revenue in the period. November 2011 is the peak at £1.50M, 1.8× the average full month (£0.83M). *Action: build stock and staffing from September.*
2. **Heavy UK dependence.** The UK generates £8.99M (84.6%). The next four markets, Netherlands, Ireland, Germany and France, are each only 2.0–2.7%. *Action: the UK is a concentration risk, and the export markets are the growth opportunity.*
3. **Tuesday and Thursday lead.** Thursday (£2.20M) and Tuesday (£2.18M) together make up 41.2% of revenue. Sunday is the weakest day at 7.6%, and there are no Saturday transactions at all. This pattern is consistent with business (wholesale) buyers, though the data cannot confirm it.
4. **Cancellations are costly and concentrated.** Cancelled lines total £894K (8.4% of gross sales), and 14.8% of all invoices are cancellations. Two bulk orders, 80,995 units of *Paper Craft, Little Birdie* (£168K) and 74,494 units of *Medium Ceramic Top Storage Jar* (£77K), account for 27.5% of the cancelled value. *Action: confirmation or deposit rules for very large orders.*
5. **Real best sellers differ from the raw ranking.** After netting cancellations, the top products by quantity are Popcorn Holder (56,427), WW2 Gliders (53,751), Jumbo Bag Red Retrospot (47,260), White Hanging Heart T-Light Holder (39,103) and Assorted Colour Bird Ornament (36,122). By net revenue: Regency Cakestand 3 Tier (£164K), White Hanging Heart T-Light Holder (£100K), Party Bunting (£98K).



6. **Loyal customer base.** Of 4,339 identified customers, 65.6% ordered at least twice (average 4.3 invoices each), and the top 10 customers contribute 17.3% of identified-customer sales. However, 16.4% of sales carry no customer ID, so customer analysis covers only part of the business. *Action: capture the customer ID at checkout.*

## 8. Limitations and Next Steps

Known issues, listed so the numbers above are read correctly:

- **Gross vs net.** The dashboard's Total Sales (£10.63M) excludes cancellations. Net of cancellations it is £9.74M. A net-sales measure is the next improvement.
- **Top 5 products chart.** It ranks by gross quantity on sales invoices only, so the two cancelled bulk orders appear as #1 and #2. Insight 5 and the chart above show the net view; the dashboard chart should be switched to a net quantity measure.
- **Duplicate product names.** 1,320 of 3,958 stock codes have more than one description (for example *Popcorn Holder* and *Small Popcorn Holder*), and the dashboard groups by description. Grouping by `stockcode` would be more accurate.
- **Non-product codes.** Postage, manual entries and fees total £384K (3.6% of gross sales). *Dotcom Postage* is the single largest "product" by revenue and should be excluded from product rankings.
- **Partial December.** Data ends on 9 Dec 2011. The dashboard's monthly line uses month name only, so December combines Dec 2010 with the partial Dec 2011. Use the Year slicer to separate them.
- **Cancelled Orders card** counts cancelled *lines* (9,251), not invoices (3,836).
- **Small EDA vs dashboard differences.** The EDA notebook reports £10.64M and 19,960 invoices (filter: quantity > 0 and price > 0). The dashboard reports £10.63M and 22,061 invoices because it also includes 1,336 negative-quantity non-cancellation lines and zero-price lines.
- **Scope.** There is no cost data, so this is revenue, not profit. One year of data also means seasonality comes from a single cycle.

Next steps: net sales measure, product-level view by stock code, customer segmentation (RFM), and a returns-rate view by product.

## 9. How to Reproduce

```bash
pip install -r requirements.txt
```

1. Run `notebooks/data_cleaning.ipynb` → writes `data/cleaned/cleaned_retails.csv`
2. Run `notebooks/EDA.IPYNB` for the analysis
3. Open `retail.dashboard.pbix` in Power BI Desktop. The data source path points to a local folder, so update it via **Transform data → Data source settings → Change Source**

## 10. Repository Structure

```
Internship_Task_01/
├── data/
│   ├── raw/Online Retail.xlsx
│   └── cleaned/cleaned_retails.csv
├── notebooks/
│   ├── data_cleaning.ipynb
│   └── EDA.IPYNB
├── images/                    # charts and dashboard screenshot used in this README
├── reports/data_quality_report.xlsx
├── retail.dashboard.pbix
├── requirements.txt
└── README.md
```

## Author

**Kiran** · Data Analytics (BBS final-year student, Tribhuvan University)
GitHub: [kiran77188](https://github.com/kiran77188) · LinkedIn: [www.linkedin.com/in/kirankc1077]
