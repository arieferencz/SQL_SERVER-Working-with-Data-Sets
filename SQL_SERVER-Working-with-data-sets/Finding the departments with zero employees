# Finding the departments with zero employees

## 🎯 Exercise
Retrieve the list of departments that have zero employees assigned to them.

---

## 💡 Solution

### Approach
We use a `LEFT JOIN` between the `Department` table and the `EmployeeDepartmentHistory` table. A `LEFT JOIN` returns all rows from the left table (`Department`) regardless of whether a matching row exists in the right table (`EmployeeDepartmentHistory`). Where no match exists, the columns from the right table are filled with `NULL`. The `WHERE` clause then filters to keep only rows where `BusinessEntityID IS NULL` — identifying departments with no employees assigned.

### T-SQL functions and clauses used

| Clause | Purpose |
|---|---|
| `LEFT JOIN` | Returns all departments — including those with no matching rows in `EmployeeDepartmentHistory` |
| `WHERE BusinessEntityID IS NULL` | Keeps only departments where no employee record was found — i.e. departments with zero employees |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Department` |
| `HumanResources` | `EmployeeDepartmentHistory` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    Department.[Name]
  , Department.[DepartmentID]
FROM [AdventureWorks2022].[HumanResources].[Department] AS Department
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
    ON Department.DepartmentID = EmployeeDepartmentHistory.DepartmentID
WHERE EmployeeDepartmentHistory.[BusinessEntityID] IS NULL
```

---

### Output

```
Name  DepartmentID
(0 rows affected)
```

---

## 🔍 Step-by-step explanation

### How `LEFT JOIN` works in this context

A `LEFT JOIN` between `Department` and `EmployeeDepartmentHistory` produces one row per department-employee combination. For departments that have at least one employee, all matching rows are returned with full data in both columns. For departments that have **no** employees, one row is returned with `NULL` in all columns from `EmployeeDepartmentHistory` — including `BusinessEntityID`.

**Illustration of how the join works:**

```
Department.Name          Department.DepartmentID  EmployeeDepartmentHistory.BusinessEntityID
Engineering              1                        2
Engineering              1                        3
Engineering              1                        5
...
[Hypothetical empty dept] 17                      NULL   ← no employee match
```

The `WHERE BusinessEntityID IS NULL` clause keeps only rows from the second category — departments where no employee record exists in `EmployeeDepartmentHistory`.

---

### Interpreting the output

The query returns **0 rows** — meaning every department in the `Department` table has at least one employee assigned to it in `EmployeeDepartmentHistory`. There are no departments with zero employees in the AdventureWorks2022 database.

This is a meaningful result in itself — it confirms that the database has no unassigned or empty departments. If a new department were added to the `Department` table without any employees being assigned to it, that department would appear in this query's output.

---

### Why `LEFT JOIN` and not `INNER JOIN`?

An `INNER JOIN` only returns rows where a match exists in **both** tables. If a department had no employees, it would be silently excluded from the result — making it impossible to detect empty departments. `LEFT JOIN` is essential here because it preserves all departments regardless of whether they have employees, allowing the `WHERE IS NULL` filter to identify the ones with none.

| Join type | Behaviour | Use when |
|---|---|---|
| `INNER JOIN` | Returns only departments that have at least 1 employee | Finding departments with employees |
| `LEFT JOIN` + `WHERE IS NULL` | Returns only departments that have no employees | Finding departments with zero employees |

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
