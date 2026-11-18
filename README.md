# Sales Analytics Dashboard | Power BI, DAX, Power Query

An interactive 2-page Power BI dashboard built on the **Global Superstore** sales dataset (Kaggle). The project covers the full BI pipeline: cleaning data in Power Query, designing a star-schema model, writing DAX measures, and building a dashboard that shows sales, profit, top products and regional trends.

## Dashboard preview

**Page 1: Executive Overview**

![Executive Overview](sandbox:/mnt/data/Screenshot%202026-11-19%20033511.png)

**Page 2: Regional & Product Analysis**

![Regional Analysis](sandbox:/mnt/data/Screenshot%202026-11-19%20033246.png)

## Key numbers (all years, all markets)

| Metric | Value |
|---|---|
| Total Sales | $12.64M |
| Total Profit | $1.47M |
| Profit Margin | 11.61% |
| Total Orders | ~25K |

## Tools used

- **Power BI Desktop**: data model, visuals, slicers
- **Power Query (M)**: ETL and data cleaning
- **DAX**: measures and date table
- **Dataset**: [Global Superstore Dataset on Kaggle](https://www.kaggle.com/) (about 51K rows, 27 columns)

## 1. ETL (Power Query)

Steps applied in the Power Query Editor:

1. Loaded the CSV and promoted the first row to headers.
2. Removed columns not needed for analysis (Row ID, record-count column, duplicate market/year helper columns).
3. Trimmed and cleaned all text columns to remove hidden spaces.
4. Fixed data types (Order Date and Ship Date to Date, Sales/Profit/Discount to Decimal, Quantity to Whole Number, IDs and names to Text).
5. Replaced nulls in text columns with `Unknown`; removed rows with null Sales or Profit (filling with 0 would distort margins).
6. Removed fully duplicate rows (not by Order ID, since one order can have several line items).
7. Created a `GeoKey` column (Country | State | City) because city names repeat across regions.

**Cleaning results:** [fill in: rows before = ___, duplicates removed = ___, null rows handled = ___, rows after = ___]

## 2. Data model (star schema)

One fact table surrounded by dimension tables, all one-to-many with single-direction filters.

| Table | Type | Key columns |
|---|---|---|
| `FactSales` | Fact | Order ID, Order Date, Product ID, GeoKey, Segment, Quantity, Discount, Sales, Profit |
| `DimProduct` | Dimension | Product ID, Product Name, Category, Sub-Category |
| `DimGeography` | Dimension | GeoKey, City, State, Country, Region, Market |
| `DimDate` | Dimension | Date, Year, Quarter, Month, Month No |

`DimDate` is marked as the date table, and `Month` is sorted by `Month No`.

## 3. DAX

```DAX
DimDate =
ADDCOLUMNS(
    CALENDAR(
        DATE(YEAR(MIN(FactSales[Order Date])), 1, 1),
        DATE(YEAR(MAX(FactSales[Order Date])), 12, 31)
    ),
    "Year", YEAR([Date]),
    "Quarter", "Q" & QUARTER([Date]),
    "Month No", MONTH([Date]),
    "Month", FORMAT([Date], "MMM"),
    "Year-Month", FORMAT([Date], "YYYY-MM")
)

Total Sales = SUM(FactSales[Sales])

Total Profit = SUM(FactSales[Profit])

Profit Margin % = DIVIDE([Total Profit], [Total Sales])

Total Orders = DISTINCTCOUNT(FactSales[Order ID])

Total Quantity = SUM(FactSales[Quantity])

Avg Order Value = DIVIDE([Total Sales], [Total Orders])

Sales LY = CALCULATE([Total Sales], SAMEPERIODLASTYEAR(DimDate[Date]))

YoY Growth % = DIVIDE([Total Sales] - [Sales LY], [Sales LY])
```

## 4. Dashboard pages

**Page 1: Executive Overview**
- KPI cards: Total Sales, Total Profit, Profit Margin %, YoY Growth %
- Line chart: monthly sales trend by year
- Bar chart: top products by sales
- Donut chart: sales by category
- Slicers: Year, Market, Category

**Page 2: Regional & Product Analysis**
- Clustered column: sales vs profit by market
- Bar chart: top 10 countries by sales
- Matrix heatmap: profit margin % by region and category
- Scatter chart: sales vs profit margin by sub-category
- Slicers: Year (synced across pages), Segment, Market

## Insights

- **Canada** has the highest profit margin (about 26.6%), while **EMEA** has the lowest (about 5.3%).
- **US Furniture** earns only about 2.5% margin, far below other US categories, which makes it a clear problem area.
- The scatter chart shows at least one sub-category selling well but running at a loss. [fill in: sub-category name = ___]

## How to open this project

1. Download `Sales_Dashboard.pbix` from this repository.
2. Open it in [Power BI Desktop](https://powerbi.microsoft.com/desktop/) (free, Windows).
3. If Power BI asks for the data source, point it to the CSV in the `data/` folder (Home > Transform data > Data source settings).

## Repository structure

```
sales-analytics-dashboard/
├── README.md
├── Sales_Dashboard.pbix
├── data/
│   └── Global_Superstore.csv
└── screenshots/
    ├── page1_executive_overview.png
    ├── page2_regional_analysis.png
    ├── model_view.png
    └── demo.gif
```

## What I learned

- Why a star schema keeps DAX simple and performance fast.
- How filter context works, and how `CALCULATE` with `SAMEPERIODLASTYEAR` needs a proper marked date table.
- Why nulls should be handled per column instead of blanket-filled with 0.
- How to design slicers and visuals that cross-filter each other.

## Author

[Your name] | [LinkedIn link] | [Email]
