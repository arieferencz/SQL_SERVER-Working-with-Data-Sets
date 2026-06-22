# Pivoting query results for year-to-year comparison of sales

## 🎯 Exercise
Pivot the sales amounts by year and calculate both the absolute difference and the percentage change between each consecutive year (2011–2012, 2012–2013, 2013–2014).

---

## 💡 Solution

### Approach
We use nested subqueries combined with `CASE` statements and `SUM()` to pivot sales amounts into one column per year. We then wrap the pivoted result in an outer query that calculates year-over-year differences and percentage changes using arithmetic and the `FORMAT()` function.

### Table used

| Schema | Table |
|---|---|
| `Sales` | `SalesOrderHeader` |

---

### T-SQL code — Full solution

```sql
USE AdventureWorks2022;
GO

SELECT
    FORMAT(SalesPerYear.Sales2014 - SalesPerYear.Sales2013, '#,#.##;(#,#.##)')         AS SalesAmountDiff_2014_2013
  , FORMAT(SalesPerYear.Sales2013 - SalesPerYear.Sales2012, '#,#.##;(#,#.##)')         AS SalesAmountDiff_2013_2012
  , FORMAT(SalesPerYear.Sales2012 - SalesPerYear.Sales2011, '#,#.##;(#,#.##)')         AS SalesAmountDiff_2012_2011
  , FORMAT(1.0 * SalesPerYear.Sales2014 / SalesPerYear.Sales2013 - 1, '#.##%;(#.##%)') AS SalesPercDiff_2014_2013
  , FORMAT(1.0 * SalesPerYear.Sales2013 / SalesPerYear.Sales2012 - 1, '#.##%;(#.##%)') AS SalesPercDiff_2013_2012
  , FORMAT(1.0 * SalesPerYear.Sales2012 / SalesPerYear.Sales2011 - 1, '#.##%;(#.##%)') AS SalesPercDiff_2012_2011
FROM (
    SELECT
        SUM(CASE WHEN Sales.SalesYear = 2011 THEN Sales.SubTotal END) AS Sales2011
      , SUM(CASE WHEN Sales.SalesYear = 2012 THEN Sales.SubTotal END) AS Sales2012
      , SUM(CASE WHEN Sales.SalesYear = 2013 THEN Sales.SubTotal END) AS Sales2013
      , SUM(CASE WHEN Sales.SalesYear = 2014 THEN Sales.SubTotal END) AS Sales2014
    FROM (
        SELECT
            SalesOrderID
          , DATEPART(YEAR, OrderDate) AS SalesYear
          , SubTotal
        FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader
    ) AS Sales
) AS SalesPerYear
```

---

### Output

```
SalesAmountDiff_2014_2013  SalesAmountDiff_2013_2012  SalesAmountDiff_2012_2011  SalesPercDiff_2014_2013  SalesPercDiff_2013_2012  SalesPercDiff_2012_2011
(23,564,550.24)            10,098,177.73              20,882,629.11              (54.02%)                 30.12%                   165.19%

Warning: Null value is eliminated by an aggregate or other SET operation.
(1 row affected)
```

> **Note:** The warning *"Null value is eliminated by an aggregate or other SET operation"* is expected. It appears because `SUM(CASE WHEN SalesYear = XXXX THEN SubTotal END)` returns `NULL` for years that don't match, and `SUM()` ignores those `NULL` values. This does not affect the correctness of the result.

---

## 🔍 Step-by-step explanation

### Query 1.1 — Retrieve the sales amount for each order (`OriginalTablesLevel1`)
We extract `SalesOrderID`, the order year using `DATEPART(YEAR, OrderDate)`, and `SubTotal` (the amount before taxes and shipping) from `SalesOrderHeader`.

**T-SQL code of Query 1.1**
```sql
SELECT SalesOrderID									                            -- OriginalTablesLevel1
, DATEPART(YEAR, OrderDate) AS SalesYear
, SubTotal
FROM [AdventureWorks2022].[Sales].[SalesOrderHeader] AS SalesOrderHeader		-- OriginalTablesLevel1
```

**Output of Query 1.1 (Truncated):** 31,465 rows — one per sales order, with its year and subtotal amount (truncated).

```
SalesOrderID	SalesYear	SubTotal
43659		    2011        20565.6206
43660		    2011        1294.2529
43661		    2011        32726.4786
43662		    2011        28832.5289
43663		    2011	    419.4589
43664		    2011	    24432.6088
...
75117		    2014	    29.48
75118		    2014	    135.23
75119		    2014	    42.28
75120		    2014	    84.96
75121		    2014	    74.98
75122		    2014	    30.97
75123		    2014	    189.97
(31465 rows affected)
```

---

### Query 1.2 — Pivot the subtotal into one column per year using `CASE`
For each row we use a `CASE` statement per year: if the order belongs to that year, the column returns the `SubTotal`; otherwise `NULL`. This spreads each order's amount into its year column, with `NULL` in all other year columns.

**Output:** 31,465 rows, each with an amount in one year column and `NULL` in the others (truncated).

```
SalesYear  Sales2011   Sales2012    Sales2013   Sales2014
2011       20565.6206  NULL         NULL        NULL
2011       1294.2529   NULL         NULL        NULL
...
2012       NULL        24509.8281   NULL        NULL
2012       NULL        3463.2998    NULL        NULL
...
2013       NULL        NULL         2181.5625   NULL
...
2014       NULL        NULL         NULL        27.28
(31465 rows affected)
```

---

### Query 1.3 — Sum all amounts per year into a single row
We wrap Query 1.2 with `SUM()` per year column. Since each row contains only one non-`NULL` value, `SUM()` totals all amounts per year and collapses the 31,465 rows into a single row with one total per year.

**Output:** 1 row with the total sales amount for each year.

```
Sales2011         Sales2012        Sales2013         Sales2014
12,641,672.2129   33,524,301.326   43,622,479.0537   20,057,928.8113
(1 row affected)
```

---

### Final Query (Query 1) — Calculate year-over-year differences and percentage changes
We wrap Query 1.3 and add 6 calculated columns:

| Column | Formula |
|---|---|
| `SalesAmountDiff_2014_2013` | `Sales2014 − Sales2013` |
| `SalesAmountDiff_2013_2012` | `Sales2013 − Sales2012` |
| `SalesAmountDiff_2012_2011` | `Sales2012 − Sales2011` |
| `SalesPercDiff_2014_2013` | `(Sales2014 / Sales2013) − 1` |
| `SalesPercDiff_2013_2012` | `(Sales2013 / Sales2012) − 1` |
| `SalesPercDiff_2012_2011` | `(Sales2012 / Sales2011) − 1` |

The `FORMAT()` function formats the results as readable numbers and percentages. Negative values are wrapped in parentheses `()` using the format pattern `#,#.##;(#,#.##)` and `#.##%;(#.##%)` — this is a standard accounting convention for displaying losses.

The `1.0 *` multiplier before each percentage division forces the result to be a decimal instead of an integer, ensuring accurate percentage calculations.

**Final output:** 1 row showing all year-over-year comparisons.

```
SalesAmountDiff_2014_2013  SalesAmountDiff_2013_2012  SalesAmountDiff_2012_2011  SalesPercDiff_2014_2013  SalesPercDiff_2013_2012  SalesPercDiff_2012_2011
(23,564,550.24)            10,098,177.73              20,882,629.11              (54.02%)                 30.12%                   165.19%
(1 row affected)
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
