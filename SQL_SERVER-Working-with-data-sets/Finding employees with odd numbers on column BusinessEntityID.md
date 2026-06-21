# Finding employees with odd numbers on column BusinessEntityID

## 🎯 Exercise
Retrieve the list of employees whose `BusinessEntityID` is an odd number.

---

## 💡 Solution 1 — Using `ROW_NUMBER()` and the modulus operator `%`

### Approach
We use `ROW_NUMBER()` to assign a sequential row number to each employee ordered by `BusinessEntityID`. We then apply the modulus operator (`%`) in the `WHERE` clause to keep only rows where the row number is odd — i.e. where `RowNumber % 2 = 1`.

### T-SQL functions and operators used

| Function / Operator | Purpose |
|---|---|
| `ROW_NUMBER() OVER (ORDER BY BusinessEntityID)` | Assigns a sequential number to each employee ordered by their ID |
| `%` (modulus operator) | Returns the remainder of a division — `RowNumber % 2 = 1` is true for all odd numbers |

### Table used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
SELECT X.BusinessEntityID
FROM (
    SELECT
        ROW_NUMBER() OVER (ORDER BY BusinessEntityID) AS RowNumber
      , BusinessEntityID
      , JobTitle
    FROM [AdventureWorks2022].[HumanResources].[Employee]
) AS X
WHERE X.RowNumber % 2 = 1
```

---

### Output (truncated)

```
BusinessEntityID
1
3
5
7
9
11
...
279
281
283
285
287
289
(145 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the modulus operator works
The modulus operator `%` returns the **remainder** of an integer division:
- `1 % 2 = 1` → odd
- `2 % 2 = 0` → even
- `3 % 2 = 1` → odd
- `4 % 2 = 0` → even

So `WHERE RowNumber % 2 = 1` keeps only rows with an odd row number.

### Why `ROW_NUMBER()` is used instead of filtering directly on `BusinessEntityID`
The `BusinessEntityID` values in the `Employee` table are not perfectly consecutive — some IDs are missing (not all persons are employees). Using `ROW_NUMBER()` ensures we are filtering on a clean 1, 2, 3... sequence rather than the actual ID values, which may skip numbers.

For example, if `BusinessEntityID` values are `1, 2, 3, 5, 7...`, filtering `BusinessEntityID % 2 = 1` would return `1, 3, 5, 7` — but filtering `ROW_NUMBER() % 2 = 1` returns every other employee in order regardless of the actual ID value.

---

## 💡 Solution 2 — Using `ROW_NUMBER()`, modulus `%` and a `CASE` statement

This solution produces the same result as Solution 1 but adds an intermediate `CASE` statement that explicitly labels each row as `'ODD'` or `'EVEN'` before filtering. This makes the logic more readable and easier to extend — for example, if you later want to retrieve both odd and even rows in separate columns.

```sql
SELECT Y.BusinessEntityID
FROM (
    SELECT
        X.BusinessEntityID
      , CASE
            WHEN X.RowNumber % 2 = 0 THEN 'EVEN'
            WHEN X.RowNumber % 2 = 1 THEN 'ODD'
        END AS Even_Odd
    FROM (
        SELECT
            ROW_NUMBER() OVER (ORDER BY BusinessEntityID) AS RowNumber
          , BusinessEntityID
          , JobTitle
        FROM [AdventureWorks2022].[HumanResources].[Employee]
    ) AS X
) AS Y
WHERE Y.Even_Odd = 'ODD'
```

**Output:** Identical to Solution 1 — 145 rows affected.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
