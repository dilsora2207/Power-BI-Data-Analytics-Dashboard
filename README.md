# Basic Measures Report (Power BI)

A sample Power BI report demonstrating core sales measures — Total Revenue, YTD Revenue, Rolling 3-Month Average, Unique Customers, and Average Sale Value — built on a small Sales/Products/Customers dataset.

## Data Model

| Table | Description |
|---|---|
| `Sales.csv` | Transaction-level sales log (TransactionID, ProductID, CustomerID, SaleDate, Quantity, UnitPrice) |
| `Products.csv` | Product catalog (ProdID, ProductName, Category, CostPrice) |
| `Customers.csv` | Customer directory (CustID, CustomerName, City, Country) |
| `Calendar` | Dedicated date table (created in-model via DAX), marked as the official Date Table |

**Relationships:**
- `Products[ProdID]` → `Sales[ProductID]`
- `Customers[CustID]` → `Sales[CustomerID]`
- `Calendar[Date]` → `Sales[SaleDate]` (1-to-many, single direction)

## Why a Calendar Table?

Power BI's time-intelligence **quick measures** (YTD, rolling averages, etc.) only work when grouped or filtered by a properly marked **Date Table** — a transaction table like `Sales` can't be marked as one because its dates have gaps (it's a log of events, not a continuous calendar).

The `Calendar` table is generated with:

```dax
Calendar = CALENDAR(DATE(2024, 1, 1), DATE(2024, 12, 31))
```

with helper columns:

```dax
Year      = YEAR(Calendar[Date])
Month     = FORMAT(Calendar[Date], "MMMM")
MonthNum  = MONTH(Calendar[Date])
Quarter   = "Q" & FORMAT(Calendar[Date], "Q")
QuarterNo = QUARTER(Calendar[Date])
Day       = Calendar[Date]
```

It's marked as the model's **Date Table** (Model view → Table tools → Mark as Date Table → `Date` column), and all time-intelligence measures and chart axes reference `Calendar[Date]` rather than `Sales[SaleDate]`.

## Key Measures

| Measure | Definition |
|---|---|
| `Total Revenue` | `SUMX(Sales, Sales[Quantity] * Sales[UnitPrice])` |
| `Revenue YTD (Quick)` | `TOTALYTD([Total Revenue], 'Calendar'[Date])` |
| `Revenue Rolling Avg 3M (Quick)` | 3-month centered/trailing average of `[Total Revenue]`, calculated against `Calendar[Date]` |
| `Average Sale Value` | `[Total Revenue] / [Number of Transactions]` |
| `Unique Customers Served` | `DISTINCTCOUNT(Sales[CustomerID])` |
| `Number of Transactions` | `COUNTROWS(Sales)` |

## Known Data Issues (Sample Data)

The sample `Sales.csv` intentionally/accidentally contains a couple of dirty rows, useful for testing error handling:
- Row with `ProductID = "XYZ"` (non-numeric) — causes a data type conversion error if `ProductID` is typed as Whole Number.
- Row with a blank `CustomerID`.

These should be cleaned or filtered in Power Query before production use.

## Report Visuals

- **SaleDate slicer** — filter all visuals by transaction date
- **KPI cards** — Total Revenue, Unique Customers Served, Average Sale Value, Number of Transactions
- **Total Revenue by Category** — bar chart
- **Revenue YTD (Quick) by Date** — line chart (X-axis: `Calendar[Date]`, set to Continuous for correct chronological ordering)
- **Country summary table** — Total Revenue and Number of Transactions by Country

## Setup Notes

1. Open `basicMeasuresReport.pbix` in Power BI Desktop.
2. Verify the `Calendar` table and its relationship to `Sales[SaleDate]` are present (Model view).
3. If measures show red squiggly errors after any model changes, re-point any `'Sales'[SaleDate].[...]` references to the equivalent `'Calendar'[...]` field.
4. Refresh data if source CSVs have changed.
