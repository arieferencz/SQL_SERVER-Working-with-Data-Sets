# Creating evenly sized groups of vendors

## 🎯 Exercise
Divide all vendors into groups of exactly 7 vendors per group, ranked by their historical purchasing amount from highest to lowest. The last group may have fewer than 7 vendors if the total number of vendors is not perfectly divisible by 7.

---

## 💡 Solution

### Approach
We use `ROW_NUMBER()` to assign a sequential rank to each vendor ordered by their total purchasing amount descending. We then divide each row number by `7.0` to get a decimal position within the group structure, and apply `CEILING()` to round up to the nearest whole number — producing the group number.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (ORDER BY ...)` | Assigns a sequential rank to each vendor ordered by total purchase amount descending |
| `ROW_NUMBER() / 7.0` | Divides the rank by 7 to get a decimal position — values 0.01–1.00 belong to group 1, 1.01–2.00 to group 2, etc. |
| `CEILING(value)` | Rounds up to the nearest integer — converts the decimal position into a group number |
| `SUM()` | Calculates the total purchasing amount per vendor |
| `FORMAT(value, format)` | Formats amounts with comma separators |

### Tables used

| Schema | Table |
|---|---|
| `Purchasing` | `PurchaseOrderHeader` |
| `Purchasing` | `Vendor` |

---

### T-SQL code

```sql
SELECT
    PurchaseOrderHeader.VendorID
  , Vendor.[Name]                                                                AS VendorName
  , FORMAT(SUM(PurchaseOrderHeader.TotalDue), '#,##.#')                         AS TotalPurchasesAmount
  , ROW_NUMBER() OVER (ORDER BY SUM(PurchaseOrderHeader.TotalDue) DESC) / 7.0   AS GroupsRowNumber
  , CEILING(ROW_NUMBER() OVER (ORDER BY SUM(PurchaseOrderHeader.TotalDue) DESC)
      / 7.0)                                                                    AS Groups_Plus
FROM [AdventureWorks2022].[Purchasing].[PurchaseOrderHeader] AS PurchaseOrderHeader
JOIN [AdventureWorks2022].[Purchasing].[Vendor] AS Vendor
    ON PurchaseOrderHeader.VendorID = Vendor.BusinessEntityID
GROUP BY PurchaseOrderHeader.VendorID, Vendor.[Name]
```

---

### Output (truncated)

```
VendorID  VendorName                         TotalPurchasesAmount  GroupsRowNumber  Groups_Plus
1576      Superior Bicycles                  5,034,266.70          0.142857         1
1684      Professional Athletic Consultants  3,379,946.30          0.285714         1
1696      Chicago City Saddles               3,347,165.20          0.428571         1
1680      Jackson Authority                  2,821,333.50          0.571428         1
1578      Vision Cycles, Inc.                2,777,684.90          0.714285         1
1632      Sport Fan Co.                      2,675,889.20          0.857142         1
1678      Proseware, Inc.                    2,593,901.30          1                1   ← Group 1: 7 vendors
1658      Crowley Sport                      2,472,770.10          1.142857         2
1506      Greenwood Athletic Company         2,472,770.10          1.285714         2
...
1538      Vista Road Bikes                   2,090,857.50          2                2   ← Group 2: 7 vendors
1652      Victory Bikes                      2,052,173.60          2.142857         3
...
1526      International Bicycles             1,589,173             3                3   ← Group 3: 7 vendors
...
1510      International                      8,061.10              11               11  ← Group 11: 7 vendors
1648      Wide World Importers               8,025.60              11.142857        12
1612      Midwest Sport, Inc.                7,328.70              11.285714        12
1688      Wood Fitness                       6,947.60              11.428571        12
1618      Metro Sport Equipment              6,324.50              11.571428        12
1566      Burnett Road Warriors              5,780                 11.714285        12
1592      Lindell                            5,412.60              11.857142        12
1520      G & K Bicycle Corp.               5,036.10              12               12  ← Group 12: 7 vendors
1548      Consumer Cycles                    3,378.20              12.142857        13
1662      Northern Bike Travel               2,048.40              12.285714        13  ← Group 13: 2 vendors
(86 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the grouping formula works

There are 86 vendors in total. `ROW_NUMBER()` assigns each vendor a rank from 1 to 86 ordered by total purchase amount descending.

Dividing by `7.0` converts each rank into a decimal:

| Row number | ÷ 7.0 | `CEILING()` result | Group |
|---|---|---|---|
| 1 | 0.142857 | 1 | Group 1 |
| 2 | 0.285714 | 1 | Group 1 |
| 7 | 1.000000 | 1 | Group 1 — last in group |
| 8 | 1.142857 | 2 | Group 2 |
| 14 | 2.000000 | 2 | Group 2 — last in group |
| 84 | 12.000000 | 12 | Group 12 — last in group |
| 85 | 12.142857 | 13 | Group 13 |
| 86 | 12.285714 | 13 | Group 13 — last in group (2 vendors only) |

`CEILING()` always rounds up to the next integer — so any decimal value between `N.000001` and `(N+1).000000` maps to group `N+1`. A whole number like `7.000000` maps to group 7 (the last vendor in that group).

### Why the last group has only 2 vendors
86 ÷ 7 = 12 groups of 7 (84 vendors) + 1 partial group of 2 (vendors 85 and 86). This is expected behaviour — when the total count is not perfectly divisible by the group size, the last group will always contain the remainder.

### Why divide by `7.0` instead of `7`
Dividing by `7` (integer) in T-SQL performs integer division and discards the decimal — `8 / 7 = 1` instead of `1.142857`. Dividing by `7.0` (decimal) forces a decimal result, which is essential for `CEILING()` to produce the correct group numbers.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
