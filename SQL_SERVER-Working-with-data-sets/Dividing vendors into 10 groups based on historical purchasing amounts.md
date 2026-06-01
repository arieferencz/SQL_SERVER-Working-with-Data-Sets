# Dividing vendors into 10 groups based on historical purchasing amounts

## 🎯 Exercise
Divide all vendors into exactly 10 groups based on their historical purchasing amount from highest to lowest — distributing vendors as evenly as possible across the groups.

---

## 📝 Note

> This exercise is related to [Creating evenly sized groups of vendors](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Creating%20evenly%20size%20groups%20of%20vendors.md), which uses `CEILING()` and `ROW_NUMBER()` to create fixed-size groups. This exercise uses `NTILE()` instead, which automatically handles uneven distributions.

---

## 💡 Solution

### Approach
We use `NTILE(10)` as a window function over the vendors ordered by their total purchasing amount descending. `NTILE()` automatically distributes the 86 vendors as evenly as possible across 10 groups — assigning any remainder vendors to the first groups.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `NTILE(n) OVER (ORDER BY ...)` | Divides the result set into `n` groups as evenly as possible, assigning a group number to each row |
| `SUM()` | Calculates the total purchasing amount per vendor |
| `FORMAT(value, format)` | Formats amounts with comma separators |

### Tables used

| Schema | Table |
|---|---|
| `Purchasing` | `PurchaseOrderHeader` |
| `Purchasing` | `Vendor` |

---

### T-SQL code — Query 1: Assign group numbers using NTILE

```sql
SELECT
    PurchaseOrderHeader.VendorID
  , Vendor.[Name]                                                              AS VendorName
  , FORMAT(SUM(PurchaseOrderHeader.TotalDue), '#,##.#')                       AS TotalPurchasesAmount
  , NTILE(10) OVER (ORDER BY SUM(PurchaseOrderHeader.TotalDue) DESC)          AS GroupNumber
FROM [AdventureWorks2022].[Purchasing].[PurchaseOrderHeader] AS PurchaseOrderHeader
JOIN [AdventureWorks2022].[Purchasing].[Vendor] AS Vendor
    ON PurchaseOrderHeader.VendorID = Vendor.BusinessEntityID
GROUP BY PurchaseOrderHeader.VendorID, Vendor.[Name]
```

### Output (truncated)

```
VendorID  VendorName                         TotalPurchasesAmount  GroupNumber
1576      Superior Bicycles                  5,034,266.70          1
1684      Professional Athletic Consultants  3,379,946.30          1
1696      Chicago City Saddles               3,347,165.20          1
1680      Jackson Authority                  2,821,333.50          1
1578      Vision Cycles, Inc.                2,777,684.90          1
1632      Sport Fan Co.                      2,675,889.20          1
1678      Proseware, Inc.                    2,593,901.30          1
1658      Crowley Sport                      2,472,770.10          1
1506      Greenwood Athletic Company         2,472,770.10          1   ← Group 1: 9 vendors
1586      Mitchell Sports                    2,424,284.40          2
1570      First Rate Bicycles                2,304,231.60          2
...
1508      Compete Enterprises, Inc           1,731,662.70          2   ← Group 2: 9 vendors
1542      Hill's Bicycle Service             1,597,309.20          3
...
1682      Premier Sport, Inc.                1,191,083.60          3   ← Group 3: 9 vendors
...
1562      Norstan Bike Hut                   31,824.20             6   ← Group 6: 9 vendors
1536      Cruger Bike Company                31,499.80             7
...
1666      Leaf River Terrain                 27,177.80             7   ← Group 7: 8 vendors
...
1534      Ready Rentals                      23,635.10             8   ← Group 8: 8 vendors
...
1648      Wide World Importers               8,025.60              9   ← Group 9: 8 vendors
1612      Midwest Sport, Inc.                7,328.70              10
...
1662      Northern Bike Travel               2,048.40              10  ← Group 10: 8 vendors
(86 rows affected)
```

---

### T-SQL code — Query 1.1: Count vendors per group

```sql
SELECT
    NtileGroups.GroupNumber
  , COUNT(*) AS CountVendorsPerGroupNumber
FROM (
    SELECT
        PurchaseOrderHeader.VendorID
      , Vendor.[Name]                                                          AS VendorName
      , FORMAT(SUM(PurchaseOrderHeader.TotalDue), '#,##.#')                   AS TotalPurchasesAmount
      , NTILE(10) OVER (ORDER BY SUM(PurchaseOrderHeader.TotalDue) DESC)      AS GroupNumber
    FROM [AdventureWorks2022].[Purchasing].[PurchaseOrderHeader] AS PurchaseOrderHeader
    JOIN [AdventureWorks2022].[Purchasing].[Vendor] AS Vendor
        ON PurchaseOrderHeader.VendorID = Vendor.BusinessEntityID
    GROUP BY PurchaseOrderHeader.VendorID, Vendor.[Name]
) AS NtileGroups
GROUP BY NtileGroups.GroupNumber
```

### Output

```
GroupNumber  CountVendorsPerGroupNumber
1            9
2            9
3            9
4            9
5            9
6            9
7            8
8            8
9            8
10           8
(10 rows affected)
```

---

## 🔍 Step-by-step explanation

### How `NTILE()` distributes vendors
With 86 vendors and 10 groups: **86 ÷ 10 = 8 remainder 6**. This means 6 groups will have 9 vendors and 4 groups will have 8 vendors. `NTILE()` always assigns the extra vendors to the **first** groups — so groups 1–6 get 9 vendors and groups 7–10 get 8 vendors.

The general `NTILE()` distribution rule is:

| Situation | Groups with `⌊N/G⌋ + 1` vendors | Groups with `⌊N/G⌋` vendors |
|---|---|---|
| N = 86, G = 10 | First 6 groups → 9 vendors | Last 4 groups → 8 vendors |
| N = 86, G = 15 | First 11 groups → 6 vendors | Last 4 groups → 5 vendors |
| N = 86, G = 20 | First 6 groups → 5 vendors | Last 14 groups → 4 vendors |

### Comparison with other group sizes

**NTILE(15) — 15 groups:**

```
GroupNumber  CountVendorsPerGroupNumber
1–11         6
12–15        5
```

**NTILE(20) — 20 groups:**

```
GroupNumber  CountVendorsPerGroupNumber
1–6          5
7–20         4
```

### `NTILE()` vs `CEILING() + ROW_NUMBER()`

| Approach | How groups are sized | Last group behaviour |
|---|---|---|
| `NTILE(n)` | Distributes remainder to the **first** groups — first groups are slightly larger | Last groups are slightly smaller |
| `CEILING(ROW_NUMBER() / n)` | Creates fixed-size groups — all groups have exactly `n` vendors | Last group contains the remainder and may be smaller |

Use `NTILE()` when you want the database to handle distribution automatically and evenly. Use `CEILING() + ROW_NUMBER()` when you need a fixed number of vendors per group.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
