# Retail Sales Analysis

Power BI portfolio project analysing retail sales data by product SKU, customer demographics and store location.

The data was cleaned and modelled in **Google BigQuery** (SQL) and visualised in a multi-page, interactive **Power BI** report with drill-through, custom tooltips and conditional formatting.

![Sales Overview](images/sales_overview.png)

## Business questions

- How are revenue, profit and margin trending over time?
- Which product categories, products and stores drive the most revenue and profit?
- Who are the customers (gender, age group, tenure), and how much do they spend?
- Do discounts increase the amount customers buy?
- How do customers pay?

## Dataset

A retail capstone dataset with four tables:

| Table | Rows | Description |
|---|---|---|
| Transactions | 5,000 | One row per sale: date, customer, product, store, quantity, discount, payment method |
| Customers | 200 | Gender, birth date, city, join date |
| Products | 50 | Name, category, sub-category, unit price, cost price (3 categories) |
| Stores | 5 | Store name, city, region (4 regions) |

Sales cover **10 Sep 2023 to 9 Sep 2025**.

## Tools and skills

- **Google BigQuery:** SQL data cleaning, validation and modelling (`CREATE TABLE AS SELECT`, window functions, `SAFE` and `CAST` conversions, `INFORMATION_SCHEMA` checks)
- **Power BI Desktop:** data modelling, DAX measures, Power Query, report design, tooltips pages, drill-through, conditional formatting, synced slicers
- **Data modelling:** star schema, date dimension, one-to-many relationships

## Approach

### 1. Load
Each source sheet was saved as a CSV and loaded into BigQuery as a `raw_` table. Schemas and row counts were checked against the source.

### 2. Clean (`raw_` to `clean_`)
The raw tables were left untouched and cleaned copies were built, one task at a time, with a verification query after each step:

- Standardised and validated dates (types, nulls, ranges, gaps)
- Removed null, blank, placeholder and duplicate IDs, and trimmed and standardised ID text
- Converted prices to `NUMERIC` and quantities to integers, and checked for zero, negative and out-of-range values
- Validated foreign keys (no orphan customers, products or stores) and confirmed joins did not change row counts or totals
- Added calculated fields: `Age`, `Tenure` and `Profit`

### 3. Model (star schema)

```
                dim_date
                   |
dim_customers --- fact_sales --- dim_products
                   |
               dim_stores
```

| Table | Grain | Notes |
|---|---|---|
| `fact_sales` | One row per transaction | Gross and net revenue, cost, gross and net profit, discount band |
| `dim_customers` | One row per customer | Age, age range, tenure, join cohort |
| `dim_products` | One row per product | Category, sub-category, prices |
| `dim_stores` | One row per store | Region, city, store label |
| `dim_date` | One row per day | Year, quarter, month, week and day attributes, with sort columns |

All relationships are one-to-many, single direction, from each dimension to the fact table.

### 4. Report (Power BI)

| Page | Content |
|---|---|
| Sales Overview | Revenue and profit cards, monthly revenue trend, revenue by category, revenue by store, discount impact |
| Customer Insights | Customer mix by gender and age group, revenue by age group and gender, customer tenure |
| Product and Store Performance | Top 10 and bottom 10 products, profitability by category, store comparison, payment method breakdown |
| Drill-through | Store, then Product, then Transaction detail |
| Tooltip | A custom tooltip page showing revenue, profit, margin, units, order value, discount and growth for the hovered item |

Interactivity: slicers for date, store, category and gender (synced across pages), drill-through, report-page tooltips, and margin colour rules (green, amber, red).

## Key findings

- **Growth:** revenue reached **15.48M** with a **32.3%** gross margin. Year 2 revenue was **7.1% higher** than year 1, and all five stores grew.
- **Categories:** Electronics and Fashion produce **88%** of revenue. Groceries is the smallest category but has the highest margin (35.8%) and the fastest growth (+23.1%).
- **Discounts:** discounts cost about **1.17M (7.6% of gross revenue)**, yet units per transaction are about 3.0 in every discount band, so they did not lift volume. Margin after discounts is 26.8%.
- **Products:** revenue follows unit price more than sales volume, so the top 10 products are mostly the highest-priced items.
- **Stores and customers:** stores perform evenly (19% to 20% of revenue each), and customers aged 35 to 64 provide 58% of revenue.

A one-page write-up is in [`docs/Executive_Summary.docx`](docs/Executive_Summary.docx).

## Key design decisions

- **Gross vs net revenue:** headline revenue and profit are **gross** (unit price x quantity, before discounts). Net columns are kept in the fact table for discount analysis.
- **Age and tenure** are calculated as full years elapsed (not calendar-year differences), as at the date the table was built.
- **Date table** covers whole calendar years and is marked as the date table, so time intelligence works.
- **Discount bands** have an explicit sort order, so they display as No discount, 1-5%, 6-10%, 11-15%.
- **Margins** are always calculated as total profit divided by total revenue, never as an average of row-level margins.

## Data notes and limitations

- The first and last months (Sep 2023 and Sep 2025) are partial, so month-on-month and year-on-year figures at the edges are overstated or understated.
- Each customer has a different city in the source data, so city-level customer analysis is not meaningful. Region and store location are used instead.
- Some transactions predate the customer's recorded join date, which affects tenure slightly.
- Currency is not specified in the source data.
- Age and tenure are a snapshot and will not update automatically.

## Repository contents

```
Retail-Sales-Analysis/
├── README.md
├── sql/                      BigQuery scripts (raw, clean and curated layers)
├── powerbi/
│   └── project_report.pbix   Power BI report
├── docs/
│   └── Executive_Summary.docx
└── images/                   Report screenshots
```

## How to use

1. Open `powerbi/project_report.pbix` in Power BI Desktop. The model is embedded, so it opens without a database connection.
2. To rebuild the data layer, run the scripts in `sql/` in BigQuery, in order (load, clean, curated), then refresh the Power BI connection.

## Author

[Your name] | [LinkedIn or portfolio link]
