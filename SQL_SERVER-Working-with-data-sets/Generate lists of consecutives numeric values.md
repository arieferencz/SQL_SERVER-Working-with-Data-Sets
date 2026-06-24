# Generate lists of consecutive numeric values

## 🎯 Exercise
Generate lists of consecutive numeric values (1, 2, 3, 4, ...) using three different approaches: the `UNION` operator, a Cartesian Product, and Common Table Expressions (CTEs).

---

## 💡 Solution 1 — Using the `UNION` operator

### Approach
We use `UNION` to combine a single row containing `0` (or `1`) with all existing ID values from a table — effectively prepending a starting value to an already-consecutive sequence. Since `UNION` removes duplicates, the result is a clean ordered list.

### Tables used

| Schema | Table |
|---|---|
| `Person` | `BusinessEntity` |
| `Sales` | `SalesOrderDetail` |

---

### Query 1 — List starting from zero

```sql
-- Using BusinessEntity (20,777 rows)
SELECT 0 AS Position
FROM [AdventureWorks2022].[Person].[BusinessEntity]
UNION
SELECT BusinessEntityID AS Position
FROM [AdventureWorks2022].[Person].[BusinessEntity]
ORDER BY Position;

-- Using SalesOrderDetail (121,317 rows)
SELECT 0 AS Position
FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
UNION
SELECT SalesOrderDetailID AS Position
FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
ORDER BY Position;
```

**Output 1 — BusinessEntity (truncated):**

```
Position
0
1
2
3
4
...
20775
20776
20777
(20778 rows affected)
```

**Output 2 — SalesOrderDetail (truncated):**

```
Position
0
1
2
3
4
...
121315
121316
121317
(121318 rows affected)
```

> **Note:** `UNION` removes duplicate records. Since `BusinessEntityID` already starts at 1 and the prepended value is `0`, there are no duplicates — hence the total row count is `N + 1` (one extra row for the `0`).

---

### Query 1.1 — List starting from one

```sql
-- Using BusinessEntity
SELECT 1 AS Position
FROM [AdventureWorks2022].[Person].[BusinessEntity]
UNION
SELECT BusinessEntityID AS Position
FROM [AdventureWorks2022].[Person].[BusinessEntity]
ORDER BY Position;

-- Using SalesOrderDetail
SELECT 1 AS Position
FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
UNION
SELECT SalesOrderDetailID AS Position
FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
ORDER BY Position;
```

**Output 1 — BusinessEntity (truncated):**

```
Position
1
2
3
4
...
20775
20776
20777
(20777 rows affected)
```

**Output 2 — SalesOrderDetail (truncated):**

```
Position
1
2
3
4
...
121315
121316
121317
(121317 rows affected)
```

> **Note:** Because `BusinessEntityID` already starts at `1`, the prepended `1` is a duplicate and `UNION` removes it. The total row count is therefore `N` — one fewer than Query 1.

---

## 💡 Solution 2 — Using a Cartesian Product

### Approach
We create a Cartesian Product between a small `InitialList` subquery (containing 1 or more rows) and the full `BusinessEntity` table. Multiplying `InitialList.Position × Iteration.Position` generates a list of consecutive multiples. The size of the result is controlled by how many rows are in `InitialList`.

---

### Query 2 — Cartesian Product of dimension 1 × 20,777

```sql
SELECT
    InitialList.Position
  , InitialList.Position * Iteration.Position AS CartesianProductDim1X20777
FROM (
    SELECT BusinessEntityID AS Position
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
    WHERE BusinessEntityID < 2                                     -- 1 row: Position = 1
) AS InitialList,
(
    SELECT BusinessEntityID AS Position
    FROM [AdventureWorks2022].[Person].[BusinessEntity]            -- 20,777 rows
) AS Iteration
ORDER BY CartesianProductDim1X20777
```

**Output (truncated):**

```
Position  CartesianProductDim1X20777
1         1
1         2
1         3
1         4
1         5
...
1         20775
1         20776
1         20777
(20777 rows affected)
```

> Since `InitialList` contains only `Position = 1`, multiplying `1 × Iteration.Position` simply returns `Iteration.Position` — producing a list from 1 to 20,777.

---

### Query 2.1 — Cartesian Product of dimension 2 × 20,777

```sql
SELECT
    ROW_NUMBER() OVER (ORDER BY Iteration.Position) AS RowNumber
  , InitialList.Position * Iteration.Position AS CartesianProductDim2X20777
FROM (
    SELECT BusinessEntityID AS Position
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
    WHERE BusinessEntityID < 3                                    -- 2 rows: Position = 1 and 2
) AS InitialList,
(
    SELECT BusinessEntityID AS Position
    FROM [AdventureWorks2022].[Person].[BusinessEntity]            -- 20,777 rows
) AS Iteration
ORDER BY Iteration.Position ASC
```

**Output (truncated):**

```
RowNumber  CartesianProductDim2X20777
1          1
2          2
3          2
4          4
5          3
...
41553      20777
41554      41554
(41554 rows affected)
```

> `InitialList` now contains `Position = 1` and `Position = 2`. The Cartesian Product produces `2 × 20,777 = 41,554` rows. `ROW_NUMBER()` is used to generate unique consecutive numbers from 1 to 41,554.

### Scaling the Cartesian Product

By changing the `WHERE BusinessEntityID < N` clause in `InitialList`, you control the total number of rows:

| `WHERE BusinessEntityID <` | Rows in InitialList | Total rows produced |
|---|---|---|
| `< 2` | 1 | 20,777 |
| `< 3` | 2 | 41,554 |
| `< 11` | 10 | 207,770 |
| `< 20778` | 20,777 | 431,683,729 (max) |

---

## 💡 Solution 3 — Using CTEs

### Approach
We use a CTE with `UNION ALL` to build a consecutive list by starting at `1` and adding `1` to each existing ID value. A `WHERE` clause in the recursive part controls the upper limit of the list.

---

### Query 3 — CTE-based consecutive list

```sql
-- Up to 19,999 using BusinessEntity
WITH ConsecutiveNumbers AS
(
    SELECT 1 AS RowNumber
    UNION ALL
    SELECT BusinessEntityID + 1 AS RowNumber
    FROM [AdventureWorks2022].[Person].[BusinessEntity]
    WHERE BusinessEntityID + 1 < 20000
)
SELECT *
FROM ConsecutiveNumbers
ORDER BY RowNumber;

-- Up to 99,999 using SalesOrderDetail
WITH ConsecutiveNumbers AS
(
    SELECT 1 AS RowNumber
    UNION ALL
    SELECT SalesOrderDetailID + 1 AS RowNumber
    FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
    WHERE SalesOrderDetailID + 1 < 100000
)
SELECT *
FROM ConsecutiveNumbers
ORDER BY RowNumber;
```

**Output 1 — BusinessEntity (truncated):**

```
RowNumber
1
2
3
4
5
...
19997
19998
19999
(19999 rows affected)
```

**Output 2 — SalesOrderDetail (truncated):**

```
RowNumber
1
2
3
4
5
...
99997
99998
99999
(99999 rows affected)
```

---

## 🔍 Comparison of the three approaches

| Approach | How it works | Max rows | Flexibility |
|---|---|---|---|
| `UNION` operator | Prepends a starting value to an existing consecutive ID column | Limited to the number of rows in the source table | Simple — change the source table to change the range |
| Cartesian Product | Multiplies two subqueries to generate a larger sequence | Up to `N × N` rows (e.g. 20,777 × 20,777 = 431M) | Scale by adjusting the `WHERE` clause in `InitialList` |
| CTE with `UNION ALL` | Starts at 1 and increments using existing IDs | Limited to the number of rows in the source table | Control upper limit with `WHERE` clause |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
