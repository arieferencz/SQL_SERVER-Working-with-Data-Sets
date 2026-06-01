# Calculating the historic subtotal purchasing amount for all vendors

## 🎯 Exercise
Calculate the total historic purchasing amount per vendor, and include a grand total row that sums all vendors together — displayed as a single result set.

---

## 💡 Solution

### Approach
We join `PurchaseOrderHeader` with `Vendor` to get each vendor's name alongside their purchase totals. We then use `GROUP BY ... WITH ROLLUP` to automatically generate a grand total row at the end. `COALESCE()` replaces the `NULL` value that `ROLLUP` places in the vendor name column for the grand total row with the label `'Total'`.

### T-SQL functions used

| Function / Clause | Purpose |
|---|---|
| `GROUP BY ... WITH ROLLUP` | Groups results by vendor name and automatically adds a grand total row with `NULL` in the grouping column |
| `SUM()` | Calculates the total purchasing amount per vendor |
| `COALESCE(value, replacement)` | Replaces the `NULL` vendor name on the grand total row with `'Total'` |
| `FORMAT(value, format)` | Formats the numeric amount as a readable number with commas and decimal places |

### Tables used

| Schema | Table |
|---|---|
| `Purchasing` | `PurchaseOrderHeader` |
| `Purchasing` | `Vendor` |

---

### T-SQL code

```sql
SELECT
    COALESCE(Vendor.[Name], 'Total') AS VendorName
  , FORMAT(SUM(PurchaseOrderHeader.TotalDue), '#,##.#') AS TotalPurchasesAmount
FROM [AdventureWorks2022].[Purchasing].[PurchaseOrderHeader] AS PurchaseOrderHeader
JOIN [AdventureWorks2022].[Purchasing].[Vendor] AS Vendor
    ON PurchaseOrderHeader.VendorID = Vendor.BusinessEntityID
GROUP BY Vendor.[Name] WITH ROLLUP
```

---

### Output (truncated)

```
VendorName                      TotalPurchasesAmount
Advanced Bicycles               28,502.10
Allenson Cycles                 498,589.60
American Bicycles and Wheels    9,641
American Bikes                  1,149,489.80
Anderson's Custom Bikes         824,365.20
Aurora Bike Center              30,229.40
Australia Bike Retailer         25,060
Beaumont Bikes                  79,384.80
Bergeron Off-Roads              38,622.10
Bicycle Specialists             1,952,375.30
...
Vista Road Bikes                2,090,857.50
West Junction Cycles            1,410,602.90
WestAmerica Bicycle Co.         25,060
Wide World Importers            8,025.60
Wood Fitness                    6,947.60
Total                           70,479,332.60
(87 rows affected)
```

---

## 🔍 Step-by-step explanation

### How `WITH ROLLUP` works
`GROUP BY Vendor.[Name] WITH ROLLUP` instructs SQL Server to produce two levels of aggregation in one query:

1. **One row per vendor** — the `SUM()` of all purchase amounts for that vendor
2. **One grand total row** — the `SUM()` across all vendors, with `NULL` in the `Vendor.[Name]` column

Without `WITH ROLLUP`, the query would return 86 rows (one per vendor) with no grand total. With `WITH ROLLUP`, it returns 87 rows — 86 vendors plus 1 grand total.

### How `COALESCE()` labels the grand total row
`ROLLUP` places `NULL` in the grouping column (`Vendor.[Name]`) for the grand total row. `COALESCE(Vendor.[Name], 'Total')` checks this value — if it is `NULL`, it returns `'Total'`; otherwise it returns the vendor name as normal.

### How `FORMAT()` formats the amounts
`FORMAT(SUM(...), '#,##.#')` formats the result as a number with comma thousands separators and one decimal place. For example:
- `70479332.6` → `70,479,332.60`
- `9641.0` → `9,641`

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
