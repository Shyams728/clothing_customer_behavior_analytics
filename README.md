# Clothing Customer Behavior Analytics (Power BI)

Power BI report and data model that analyzes customer behavior and sales trends for a retail clothing business. This project surfaces actionable insights such as seasonality, customer segmentation (RFM), retention & churn, product affinity, promotional uplift, and lifetime value — all designed to help merchandising, marketing, and executive teams make data-driven decisions.

---

## Table of contents

- [Project overview](#project-overview)  
- [Highlights & key questions answered](#highlights--key-questions-answered)  
- [Data sources & ingestion](#data-sources--ingestion)  
- [Data model & transformations](#data-model--transformations)  
- [Core measures & DAX examples](#core-measures--dax-examples)  
- [Reports & visuals](#reports--visuals)  
- [How to open and use the Power BI file (PBIX)](#how-to-open-and-use-the-power-bi-file-pbix)  
- [Publishing, refresh, and security](#publishing-refresh-and-security)  
- [Performance & best practices](#performance--best-practices)  
- [Future improvements & roadmap](#future-improvements--roadmap)  
- [Contributing & contact](#contributing--contact)  
- [License](#license)

---

## Project overview

This Power BI project is built to analyze historical customer transactions and product data to answer business-critical questions:

- Which products, categories, and collections drive the most revenue and margin?
- How do cohort retention and repeat purchase rates change over time?
- What are the high-value customer segments (RFM / CLV)?
- How effective were promotions and discounts in driving incremental sales?
- What seasonality and trend signals should merchandising plan for?

The report is organized with an executive dashboard, a customer insights section, merchandising & SKU analysis, promotions analysis, and an experiments/cohort view.

---

## Highlights & key questions answered

- Executive KPIs: Total Sales, Orders, Average Order Value (AOV), YoY / MoM growth, Gross Margin.
- Customer metrics: New vs Returning customers, Repeat Purchase Rate, Customer Lifetime Value (CLV).
- Segmentation: RFM segments (Champions, Loyal, At Risk, etc.), Average revenue per segment.
- Retention & Cohorts: Visual cohort retention heatmap and retention curves.
- Promotions & Discounts: Uplift analysis, cannibalization, promo ROI.
- Product performance: Top SKUs, sell-through rate, inventory turnover proxy.
- Funnel & conversion: Site visits -> add to cart -> purchase (if web analytics are present).

---

## Data sources & ingestion

Typical data sources used in this project (adjust to your environment):

- Transactional sales data (CSV / SQL / Azure / BigQuery)  
  - Orders table: OrderID, CustomerID, OrderDate, OrderTotal, Discount, Channel, PromoCode  
  - OrderItems table: OrderItemID, OrderID, SKU, Category, Quantity, Price, Cost
- Customer master (customer profile): CustomerID, JoinDate, EmailHash, Location (country/region), AcquisitionChannel
- Product master: SKU, ProductName, Category, Subcategory, Season, LaunchDate
- Optional: Web analytics (sessions, conversions), returns data, inventory snapshots

Ingest into Power BI Desktop using the appropriate connector (DirectQuery for large live sources, Import for snapshots). Apply staging queries in Power Query to clean, standardize, and reduce payload.

---

## Data model & transformations

Recommended model shape:

- Star schema with fact table(s):
  - FactSales (Orders + OrderItems joined or an OrderItems-level fact)
- Dimension tables:
  - DimCustomer, DimProduct, DimDate, DimPromo, DimChannel

Transformations applied in Power Query:
- Date table generation (full calendar with fiscal attributes)
- Standardize SKUs and categories (trim, upper/lower, mapping tables)
- Currency & price normalization (if multi-currency)
- Flag returns and cancellations and handle negative adjustments
- De-duplicate orders and fix missing CustomerIDs (mark as "Guest")

Modeling tips:
- Use surrogate keys (integers) for joins if possible.
- Mark Date table as the official date table in the model.
- Avoid bi-directional cross-filtering broadly; prefer single-direction relationships and explicit USERELATIONSHIP when needed.

---

## Core measures & DAX examples

Below are example DAX measures you can add and adapt to your model.

Total Sales
```DAX
Total Sales = SUM( FactSales[SalesAmount] )
```

Total Orders
```DAX
Total Orders = DISTINCTCOUNT( FactSales[OrderID] )
```

Average Order Value (AOV)
```DAX
AOV = DIVIDE( [Total Sales], [Total Orders] )
```

Year-over-Year Growth (Sales YoY)
```DAX
Sales YoY % = 
VAR Prev = CALCULATE( [Total Sales], SAMEPERIODLASTYEAR( 'DimDate'[Date] ) )
RETURN IF( NOT(ISBLANK(Prev)), DIVIDE( [Total Sales] - Prev, Prev ), BLANK() )
```

Rolling 3-Month Average Sales
```DAX
3M Rolling Sales = 
CALCULATE(
  [Total Sales], 
  DATESINPERIOD( 'DimDate'[Date], MAX('DimDate'[Date]), -3, MONTH )
)
```

Repeat Purchase Rate
```DAX
Repeat Customers = 
CALCULATE( DISTINCTCOUNT( FactSales[CustomerID] ), FILTER( VALUES(FactSales[CustomerID]), CALCULATE( DISTINCTCOUNT( FactSales[OrderID] ) ) > 1 ) )

Repeat Purchase Rate = DIVIDE( [Repeat Customers], DISTINCTCOUNT( FactSales[CustomerID] ) )
```

Customer Lifetime Value (basic avg)
```DAX
Avg CLV = DIVIDE( [Total Sales], DISTINCTCOUNT( FactSales[CustomerID] ) )
```

Cohort Retention (example approach: cohort by first purchase month)
- Build calculated columns for CustomerFirstOrderMonth and OrderMonth, then compute retention as count of customers in cohort who purchased in subsequent months divided by cohort size. Visualize as a retention matrix.

---

## Reports & visuals

Suggested report pages and visuals:

1. Executive Summary
   - KPI cards (Sales, Orders, AOV, YoY)
   - Trend line (Sales by Day/Week/Month)
   - Top categories & top SKUs
   - Map or region performance

2. Customer Insights
   - RFM segmentation heatmap
   - CLV distribution (boxplot/histogram)
   - New vs Returning customers (area chart)
   - Cohort retention heatmap and retention curve

3. Merchandising & Product
   - SKU-level sales & margin table
   - Sell-through / inventory velocity proxy
   - Price band performance

4. Promotions & Uplift
   - Promo vs non-promo cohort, uplift funnel
   - Promo ROI scatter (discount % vs incremental sales)

5. Funnel & Conversion (if web data available)
   - Session -> Add to Cart -> Checkout -> Purchase funnel
   - Conversion rate by channel

6. Ad-hoc Analysis / Drillthrough
   - Drillthrough pages for Customer profile and Order details
   - Bookmarks for common insights (e.g., Peak Season Analysis)

Design suggestions:
- Use consistent color palette aligned with brand.
- Use tooltips and drillthrough to avoid overcluttering visuals.
- Provide slicers for Time, Region, Channel, Category, and Segment.
- Use small multiples for category comparisons.

---

## How to open and use the Power BI file (PBIX)

1. Download the .pbix file from the repository (if present) or build from source using the Power Query files / CSV exports in /data.
2. Open in Power BI Desktop (latest version recommended).
3. Under Home > Transform Data, review the Power Query steps and update data source credentials.
4. Refresh the model to pull the latest data.
5. Publish to Power BI Service (app.powerbi.com) to share dashboards and enable scheduled refresh.

If you're using DirectQuery:
- Ensure your data source allows gateway access for the Power BI Service (on-premises data gateway).
- Tune queries and limit visuals to avoid timeouts.

---

## Publishing, refresh, and security

- Publish to a workspace in Power BI Service. Create an app for broader distribution.
- Configure scheduled refresh (daily/hourly as needed) and ensure credentials/gateway are configured.
- Row-level security (RLS): implement RLS roles if different users should see limited regions/customers.
- Content packaging: create an app and version releases for business stakeholders.

---

## Performance & best practices

- Import mode for smaller datasets; DirectQuery for large-scale real-time needs. Hybrid (composite models) as needed.
- Use aggregations for large fact tables to improve query speed.
- Reduce visuals on a page; each visual queries the model.
- Avoid card stacks with many measures — precompute where possible.
- Use incremental refresh for large historical data.

---

## Future improvements & roadmap

- Enrich customer profiles with marketing touchpoints and web behavior.
- Add propensity models (purchasing likelihood) using Power BI AI visuals or Azure ML.
- Implement automated anomaly detection and data alerts in Power BI Service.
- Build paginated reports for printable order summaries and executive PDFs.
- Add a live KPI scoreboard with alerts for out-of-range metrics.

---

## Contributing & contact

Contributions are welcome. To contribute:
- Fork the repo, make changes (Power Query, DAX, PBIX), and open a PR with a description of changes.
- Prefer small, focused PRs (e.g., a new measure, a query improvement, or a report page).
- For questions, open an issue in this repository or contact the maintainer.

Maintainer: Shyams728 (GitHub)

---

## License

Specify your license here (e.g., MIT). Update LICENSE file in the repo.

---

Appendix: Example DAX snippets, recommended bookmarks for stakeholders, and link placeholders for the PBIX file, screenshots, and sample data will be added to this README as they become available.
