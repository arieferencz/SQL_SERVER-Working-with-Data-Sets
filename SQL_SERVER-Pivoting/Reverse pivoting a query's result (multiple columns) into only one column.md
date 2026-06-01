# Reverse pivoting a query's result (multiple columns) into only one column

## 🎯 Exercise
Reverse pivot a query result that has multiple columns into a single column — so that the values from each column are stacked vertically into one new column.

---

## 📝 Important note

> This solution works **only for columns of NUMERIC data type**.
>
> Attempting to include columns of `VARCHAR` or `DATE` data types will cause SQL Server to throw a conversion error. This is demonstrated in Queries 3a and 3b below, with a full explanation of why the error occurs.

---

## 💡 Solution 1 — Using nested subqueries

### Approach
We use a **Cartesian Product** (cross join without an explicit `JOIN` keyword) to repeat each employee's row once for every column we want to transpose. We then use `ROW_NUMBER()` to number each repeated row, and a `CASE` statement to select which column value to return based on the row number — effectively stacking the 3 columns into 1.

The 3 columns being reverse pivoted are:
1. `BusinessEntityID`
2. `DepartmentID`
3. `NationalIDNumber`

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    X.RowNumberTranspose
  , CASE X.RowNumberTranspose
        WHEN 1 THEN X.BusinessEntityID
        WHEN 2 THEN X.DepartmentID
        WHEN 3 THEN X.NationalIDNumber
    END AS NewPivotedOneColumn
FROM (
    SELECT
        RemovingDuplicates.BusinessEntityID
      , RemovingDuplicates.DepartmentID
      , RemovingDuplicates.NationalIDNumber
      , ROW_NUMBER() OVER (PARTITION BY RemovingDuplicates.BusinessEntityID
                           ORDER BY RemovingDuplicates.BusinessEntityID) AS RowNumberTranspose
    FROM (
        SELECT
            OriginalTables.BusinessEntityID
          , OriginalTables.DepartmentID
          , OriginalTables.NationalIDNumber
        FROM (
            SELECT
                ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                                   ORDER BY Employee.BusinessEntityID ASC,
                                            EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
              , Employee.BusinessEntityID
              , EmployeeDepartmentHistory.DepartmentID
              , Employee.NationalIDNumber
            FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
            LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
                ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
            LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
                ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
            WHERE Employee.BusinessEntityID <> 1
        ) AS OriginalTables
        WHERE OriginalTables.RowNumberRemovingDuplicates = 1
    ) AS RemovingDuplicates,
    (
        SELECT SalesOrderDetailID AS Position
        FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
        WHERE SalesOrderDetailID < 4
    ) AS Iteration
) AS X
```

---

### Output

```
RowNumberTranspose  NewPivotedOneColumn
1                   2
2                   1
3                   245797967
1                   3
2                   1
3                   509647174
1                   4
2                   2
3                   112457891
...
1                   290
2                   3
3                   134219713
(867 rows affected)
```

---

## 🔍 Step-by-step explanation

### Transposition concept
The goal is to take 3 columns for each employee and stack them into 1 column. For example, for `BusinessEntityID = 2`:

```
Before (1 row × 3 columns):               After (3 rows × 1 column):
BusinessEntityID  DepartmentID  NationalIDNumber      NewPivotedOneColumn
2                 1             245797967     >>>      2            ← BusinessEntityID
                                                       1            ← DepartmentID
                                                       245797967    ← NationalIDNumber
```

---

### Query 1.1 — Remove duplicate employee records
We use `ROW_NUMBER()` partitioned by `BusinessEntityID` and ordered by `StartDate DESC` to keep only the most recent department record per employee, removing duplicates caused by department history changes.

**Output:** 289 unique employee rows with `BusinessEntityID`, `DepartmentID`, and `NationalIDNumber`.

```
BusinessEntityID  DepartmentID  NationalIDNumber
2                 1             245797967
3                 1             509647174
4                 2             112457891
...
289               3             668991357
290               3             134219713
(289 rows affected)
```

---

### Query 1.2 — Create 3 repeated rows per employee using Cartesian Product + `ROW_NUMBER()`
We apply a **Cartesian Product** by joining `RemovingDuplicates` with a small `Iteration` subquery that returns exactly 3 rows (using `SalesOrderDetailID < 4`). This causes each employee's row to repeat 3 times — once per column to be transposed. `ROW_NUMBER()` then numbers each repeated row 1, 2, or 3 within each employee's group.

**Output:** 867 rows (289 employees × 3 repetitions each), with `RowNumberTranspose` values of 1, 2, or 3.

```
BusinessEntityID  DepartmentID  NationalIDNumber  RowNumberTranspose
2                 1             245797967         1
2                 1             245797967         2
2                 1             245797967         3
3                 1             509647174         1
3                 1             509647174         2
3                 1             509647174         3
...
(867 rows affected)
```

---

### Final Query (Query 1) — Select one column value per row using `CASE`
We use a `CASE` statement on `RowNumberTranspose` to select which column value to return for each repeated row:
- Row 1 → return `BusinessEntityID`
- Row 2 → return `DepartmentID`
- Row 3 → return `NationalIDNumber`

This produces a single column `NewPivotedOneColumn` with all values stacked vertically.

---

## 💡 Solution 2 — Using CTEs (alternative approach)

This solution produces the same result as Solution 1 but uses **Common Table Expressions (CTEs)** instead of nested subqueries, making the logic easier to follow step by step.

```sql
USE AdventureWorks2022;
GO

WITH
OriginalTables AS
(
    SELECT
        ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                           ORDER BY Employee.BusinessEntityID ASC,
                                    EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
      , Employee.BusinessEntityID
      , EmployeeDepartmentHistory.DepartmentID
      , Employee.NationalIDNumber
    FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
    LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
        ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
    WHERE Employee.BusinessEntityID <> 1
),
RemovingDuplicates AS
(
    SELECT
        OriginalTables.BusinessEntityID
      , OriginalTables.DepartmentID
      , OriginalTables.NationalIDNumber
    FROM OriginalTables
    WHERE OriginalTables.RowNumberRemovingDuplicates = 1
),
Iteration AS
(
    SELECT SalesOrderDetailID AS Position
    FROM [AdventureWorks2022].[Sales].[SalesOrderDetail]
    WHERE SalesOrderDetailID < 4
),
RepeatedRows AS
(
    SELECT
        RemovingDuplicates.BusinessEntityID
      , RemovingDuplicates.DepartmentID
      , RemovingDuplicates.NationalIDNumber
      , ROW_NUMBER() OVER (PARTITION BY RemovingDuplicates.BusinessEntityID
                           ORDER BY RemovingDuplicates.BusinessEntityID) AS RowNumberTranspose
    FROM RemovingDuplicates, Iteration
)
SELECT
    RepeatedRows.RowNumberTranspose
  , CASE RepeatedRows.RowNumberTranspose
        WHEN 1 THEN RepeatedRows.BusinessEntityID
        WHEN 2 THEN RepeatedRows.DepartmentID
        WHEN 3 THEN RepeatedRows.NationalIDNumber
    END AS NewPivotedOneColumn
FROM RepeatedRows
```

**Output:** Identical to Solution 1 — 867 rows affected.

---

## ⚠️ Data type limitation — Queries 3a and 3b

### Query 3a — Attempt with `VARCHAR` column (`DepartmentName`) — FAILS

Adding a `VARCHAR` column to the `CASE` statement causes SQL Server to throw a conversion error:

```sql
-- Adding DepartmentName (VARCHAR) as column 3, shifting NationalIDNumber to column 4
CASE X.RowNumberTranspose
    WHEN 1 THEN X.BusinessEntityID
    WHEN 2 THEN X.DepartmentID
    WHEN 3 THEN X.DepartmentName      -- VARCHAR column
    WHEN 4 THEN X.NationalIDNumber
END AS NewPivotedOneColumn
```

**Error:**
```
Msg 245, Level 16, State 1, Line 3
Conversion failed when converting the nvarchar value 'Engineering' to data type int.
```

**Why this fails:** All expressions in a `CASE` statement must return the same data type. `RowNumberTranspose` is an `INTEGER`, which has higher data type precedence than `NVARCHAR`. SQL Server therefore tries to implicitly convert `'Engineering'` from `NVARCHAR` to `INTEGER` — which fails.

---

### Query 3b — Attempt with `DATE` column (`HireDate`) — FAILS

Adding a `DATE` column causes a different but related error:

```sql
CASE X.RowNumberTranspose
    WHEN 1 THEN X.BusinessEntityID
    WHEN 2 THEN X.DepartmentID
    WHEN 3 THEN X.NationalIDNumber
    WHEN 4 THEN X.HireDate            -- DATE column
END AS NewPivotedOneColumn
```

**Error:**
```
Msg 206, Level 16, State 2, Line 2
Operand type clash: int is incompatible with date
Msg 206, Level 16, State 2, Line 2
Operand type clash: smallint is incompatible with date
```

**Why this fails:** SQL Server cannot perform arithmetic operations between `INTEGER` and `DATE` values — the data types are fundamentally incompatible and implicit conversion is not possible.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
