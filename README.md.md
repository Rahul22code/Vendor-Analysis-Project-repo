# Vendor Performance & Inventory Analytics

## 📊 Project Overview

This project is an end-to-end **Data Analytics and Business Intelligence
project** built to analyze vendor sales, purchasing, profitability,
product performance, and inventory efficiency.

The project starts with raw business data stored in CSV files, loads the
data into **MySQL using Python**, creates a consolidated analytical
table using **SQL**, performs exploratory and business analysis using
**Pandas**, and finally presents the results through an interactive
**Power BI dashboard**.

The main objective is to answer business questions around:

-   Vendor sales and purchasing performance
-   Gross profit and profit margin
-   Brand performance
-   Purchase contribution by vendor
-   Bulk purchasing and unit purchase price
-   Inventory turnover
-   Slow-moving inventory
-   Capital locked in unsold inventory
-   Products and vendors requiring further attention

------------------------------------------------------------------------

## 🔄 End-to-End Project Workflow

``` text
Raw CSV Files
     ↓
Python + Pandas
     ↓
MySQL Database
     ↓
Database Exploration
     ↓
SQL Joins & Aggregations
     ↓
vendor_sales_summary
     ↓
Python / Pandas EDA
     ↓
Business Analysis
     ↓
Power BI Data Model
     ↓
Interactive Dashboard
```

------------------------------------------------------------------------

# 1. Data Ingestion --- Python → MySQL

The first stage of the project loads the raw CSV files into a MySQL
database named `vendor_db`.

### Technologies used

-   Python
-   Pandas
-   SQLAlchemy
-   PyMySQL
-   MySQL

The database connection was created using SQLAlchemy with the
MySQL/PyMySQL connector.

### Source tables

The ingestion workflow includes:

-   `begin_inventory`
-   `end_inventory`
-   `purchase_prices`
-   `purchases`
-   `sales`
-   `vendor_invoice`

The regular tables were loaded using Pandas `to_sql()`.

### Large Sales Table

The `sales` table was handled separately because of its size.

Instead of loading the complete file into memory at once, the file was
processed in **50,000-row chunks**:

``` python
for chunk in pd.read_csv(file_path, chunksize=50000):
    chunk.to_sql(...)
```

The first chunk replaces the existing table and subsequent chunks are
appended.

This makes the ingestion process more suitable for a large dataset.

------------------------------------------------------------------------

# 2. Database Exploration

After ingestion, Python was connected to MySQL and the available tables
were inspected.

The analysis included:

-   Listing database tables
-   Checking record counts
-   Previewing records
-   Investigating individual vendors
-   Examining purchases
-   Examining sales
-   Examining purchase prices
-   Examining vendor invoice information
-   Grouping purchase and sales data by brand

A sample vendor was also investigated across the different tables to
understand how vendor, purchase, sales, pricing, and freight information
was distributed.

------------------------------------------------------------------------

# 3. Creating the Analytical Dataset

The required information for vendor analysis was distributed across
multiple tables.

Therefore, a consolidated analytical table called:

``` text
vendor_sales_summary
```

was created.

The transformation combines information from:

``` text
purchases
      +
purchase_prices
      +
sales
      +
vendor_invoice
```

### Summary Components

Three major summary components were created:

### Freight Summary

Freight cost was aggregated from `vendor_invoice`:

``` text
VendorNumber
FreightCost
```

### Purchase Summary

Purchase information was aggregated using `purchases` and
`purchase_prices`.

The resulting purchase-level information includes:

-   Vendor Number
-   Vendor Name
-   Brand
-   Description
-   Purchase Price
-   Actual Price
-   Volume
-   Total Purchase Quantity
-   Total Purchase Dollars

### Sales Summary

Sales information was aggregated from the `sales` table:

-   Vendor Number
-   Brand
-   Total Sales Quantity
-   Total Sales Dollars
-   Total Sales Price
-   Total Excise Tax

These summaries were then joined to create the final
`vendor_sales_summary` table.

------------------------------------------------------------------------

# 4. Vendor Sales Summary Dataset

The final analytical table contains fields including:

``` text
VendorNumber
VendorName
Brand
Description
PurchasePrice
ActualPrice
Volume
TotalPurchaseQuantity
TotalPurchaseDollars
TotalSalesQuantity
TotalSalesDollars
TotalSalesPrice
TotalExciseTax
FreightCost
```

Additional analytical metrics were then created.

### Gross Profit

``` text
Grossprofit =
TotalSalesDollars - TotalPurchaseDollars
```

### Profit Margin

``` text
ProfitMargin =
Grossprofit / TotalSalesDollars
```

### Stock Turnover

``` text
StockTurnover =
TotalSalesQuantity / TotalPurchaseQuantity
```

### Sales-to-Purchase Ratio

``` text
SalestoPurchaseRatio =
TotalSalesDollars / TotalPurchaseDollars
```

Missing values were handled, numeric data types were corrected where
required, vendor names were cleaned using string stripping, and infinite
values were replaced.

The final dataframe was written back to MySQL as:

``` text
vendor_sales_summary
```

------------------------------------------------------------------------

# 5. Exploratory Data Analysis

The `vendor_sales_summary` table was loaded into Pandas for detailed
EDA.

The analysis included:

-   Descriptive statistics
-   Numerical distributions
-   Categorical distributions
-   Correlation analysis
-   Outlier/anomaly investigation
-   Data-quality checks

## Important EDA observations

The analysis identified:

-   Negative gross profit values, indicating cases where sales revenue
    was below purchase cost.
-   Zero sales quantity and sales dollars for some products, indicating
    products that had been purchased but not sold.
-   Very large variation in purchase and actual prices.
-   Large variation in freight costs.
-   Stock turnover ranging from zero to very high values.

For the focused business analysis, records were filtered using:

``` text
GrossProfit > 0
ProfitMargin > 0
TotalSalesQuantity > 0
```

------------------------------------------------------------------------

# 6. Correlation Analysis

A correlation matrix was created to examine relationships between
numerical variables.

Some observations documented in the analysis were:

-   `PurchasePrice` showed weak correlations with `TotalSalesDollars`
    and `Grossprofit`.
-   `TotalPurchaseQuantity` and `TotalSalesQuantity` showed a very
    strong positive correlation in the analyzed dataset.
-   `ProfitMargin` showed a negative correlation with `TotalSalesPrice`.
-   `StockTurnover` showed weak negative correlations with `Grossprofit`
    and `ProfitMargin`.

These correlations were used as exploratory indicators rather than as
standalone explanations of business causality.

------------------------------------------------------------------------

# 7. Brand Performance Analysis

One analysis focused on identifying brands/descriptions with:

``` text
Low Sales + High Profit Margin
```

The analysis calculated:

-   Total sales by description
-   Average profit margin by description

Thresholds were created using:

-   **15th percentile of sales** → low-sales threshold
-   **85th percentile of profit margin** → high-margin threshold

Brands meeting both conditions were identified as potential candidates
for **promotional or pricing adjustments**.

A scatter plot was created to visualize these target brands against the
broader brand population.

------------------------------------------------------------------------

# 8. Top Vendors and Brands by Sales

The project identified the top 10 vendors and top 10 brands based on
total sales dollars.

The analysis used:

``` python
groupby()
nlargest(10)
```

This provided a direct view of the vendors and brands generating the
largest sales values.

------------------------------------------------------------------------

# 9. Vendor Purchase Contribution

Vendor-level purchase performance was analyzed using:

-   Total Purchase Dollars
-   Total Sales Dollars
-   Gross Profit

Each vendor's contribution to total purchasing was calculated as:

``` text
Purchase Contribution % =
Vendor Total Purchase Dollars
/
Total Purchase Dollars
```

The vendors were sorted by purchase contribution and cumulative
contribution was calculated for the top vendors.

This helps understand how purchasing is distributed across vendors and
how concentrated the purchasing base is.

------------------------------------------------------------------------

# 10. Bulk Purchasing Analysis

A business question investigated whether purchasing in larger quantities
reduces unit purchase price.

First, unit purchase price was calculated:

``` text
UnitPurchasePrice =
TotalPurchaseDollars / TotalPurchaseQuantity
```

Purchase quantities were then divided into three groups using
`pd.qcut()`:

``` text
Small
Medium
Large
```

The average unit purchase price was compared across these groups.

### Finding from the analysis

The notebook analysis documented that:

-   Large orders had the lowest average unit purchase price.
-   The large-order group had an average unit price of approximately
    **\$10.78 per unit**.
-   The analysis observed an approximately **72% reduction in unit
    cost** between small and large order groups.

This analysis was used to investigate the relationship between purchase
volume and unit cost.

------------------------------------------------------------------------

# 11. Inventory Turnover Analysis

Stock turnover was used to identify products with slower movement.

Products with:

``` text
StockTurnover < 1
```

were investigated as potential slow-moving inventory.

Vendor-level averages were then calculated to identify vendors
associated with lower inventory turnover.

This analysis supports investigation of excess stock and inventory
movement.

------------------------------------------------------------------------

# 12. Unsold Inventory & Capital Locked in Inventory

A key inventory metric was created:

``` text
UnsoldInventoryValue =
(TotalPurchaseQuantity - TotalSalesQuantity)
× PurchasePrice
```

This estimates the purchase-value of inventory that has not yet been
sold.

The metric was aggregated by vendor to identify vendors with the largest
amount of capital tied up in unsold inventory.

The top vendors were then sorted by `UnsoldInventoryValue`.

------------------------------------------------------------------------

# 13. Power BI Dashboard

The final stage of the project uses **Microsoft Power BI** to create an
interactive dashboard.

The dashboard contains two analytical pages:

``` text
Page 1 → VENDOR PERFORMANCE
Page 2 → PRODUCT & INVENTORY ANALYSIS
```

------------------------------------------------------------------------

## Page 1 --- Vendor Performance

### KPI Cards

The completed dashboard displays:

-   **Total Sales:** 451.62M
-   **Total Purchase:** 321.90M
-   **Total Gross Profit:** 129.72M
-   **Profit Margin:** 28.72%
-   **Total Vendors:** 126

### Visualizations

#### Top Vendors by Sales and Purchase

A comparative bar chart showing sales and purchase values for the
leading vendors.

#### Top Vendors by Gross Profit

A bar chart showing vendors with the highest gross profit.

#### Low Performing Vendors

A visual highlighting vendors with comparatively low performance based
on the dashboard's selected metric.

#### Vendor Sales vs Profit Margin

A scatter chart comparing vendor sales against profit margin.

#### Total Purchase by Vendor

A donut chart showing purchase contribution across major vendors.

### Filters

The page contains interactive filters for:

-   Vendor Name
-   Brand
-   Product

------------------------------------------------------------------------

# 14. Page 2 --- Product & Inventory Analysis

The second dashboard page focuses on product and inventory performance.

### KPI Cards

The completed dashboard displays:

-   **Total Products:** 9,645
-   **Unsold Inventory Value:** 15.60M
-   **Unsold Quantity:** 677.92K
-   **Average Stock Turnover:** 1.71
-   **Slow-Moving Products:** 5,687

### Visualizations

#### Top Products by Gross Profit

A bar chart showing products with the highest gross profit.

#### Top 10 Vendors by Unsold Inventory

A bar chart highlighting vendors associated with the largest unsold
inventory values.

#### Stock Turnover, Unsold Inventory Value and Total Sales

A scatter-style visualization used to compare inventory turnover with
unsold inventory value and sales across selected brands/descriptions.

### Filters

Interactive slicers are available for:

-   Vendor Name
-   Brand
-   Product

------------------------------------------------------------------------

# 15. Power BI Data Model & DAX

The Power BI stage includes additional modeling and calculated metrics
to support the final dashboard.

The project uses DAX/business calculations for metrics such as:

-   Total Sales
-   Total Purchase
-   Gross Profit
-   Profit Margin
-   Unsold Inventory Value
-   Unsold Quantity
-   Stock Turnover
-   Slow-Moving Products
-   Brand/Target classification

A `BrandPerformance` analytical table was also used for brand-level
analysis and target-brand classification.

------------------------------------------------------------------------

# 16. Key Business Questions

This project addresses several practical business questions:

### Vendor Performance

-   Which vendors generate the highest sales?
-   Which vendors generate the highest gross profit?
-   How are purchases distributed among vendors?
-   Which vendors have comparatively low performance?
-   What is the relationship between vendor sales and profit margin?

### Product & Brand Performance

-   Which products generate the highest gross profit?
-   Which brands have low sales but high profit margins?
-   Which brands/products require further promotional or pricing
    analysis?

### Purchasing

-   Does purchasing in bulk reduce unit purchase price?
-   What purchase-volume group has the lowest average unit cost?
-   How concentrated is purchasing among the major vendors?

### Inventory

-   Which products have low stock turnover?
-   Which vendors have slow-moving inventory?
-   How much capital is tied up in unsold inventory?
-   Which vendors contribute most to unsold inventory?

------------------------------------------------------------------------

# 17. Technology Stack

  Area                      Technology
  ------------------------- ---------------------
  Programming               Python
  Data Processing           Pandas, NumPy
  Visualization / EDA       Matplotlib, Seaborn
  Statistics                SciPy
  Database                  MySQL
  Database Connectivity     SQLAlchemy, PyMySQL
  Data Transformation       SQL, Pandas
  BI & Dashboard            Microsoft Power BI
  BI Calculations           DAX
  Development Environment   Jupyter Notebook

------------------------------------------------------------------------


------------------------------------------------------------------------

# 18. Project Highlights

-   Built an end-to-end analytics workflow from **raw CSV data to an
    interactive BI dashboard**.
-   Loaded large datasets into **MySQL using Python and chunk-based
    ingestion**.
-   Used SQL joins and aggregations to create a consolidated
    `vendor_sales_summary` dataset.
-   Performed EDA and data-quality analysis using Pandas, NumPy,
    Matplotlib and Seaborn.
-   Analyzed vendor sales, purchasing, profitability and inventory
    behavior.
-   Investigated bulk purchasing and its relationship with unit purchase
    price.
-   Identified low-turnover inventory and capital tied up in unsold
    stock.
-   Built an interactive two-page Power BI dashboard with KPIs, charts
    and slicers.
-   Used DAX and Power BI data modeling to support business-level
    reporting.

------------------------------------------------------------------------

# 19. Skills Demonstrated

This project demonstrates practical skills in:

**Python \| Pandas \| NumPy \| SQL \| MySQL \| SQLAlchemy \| PyMySQL \|
EDA \| Data Cleaning \| Data Transformation \| Statistical Analysis \|
Power BI \| Power Query \| DAX \| Data Visualization \| Business
Analysis**


------------------------------------------------------------------------

## 👨‍💻 Project Type

**End-to-End Data Analytics & Business Intelligence Project**

**Focus Areas:** Vendor Performance \| Sales Analysis \| Purchasing \|
Profitability \| Inventory Analytics \| Business Intelligence
