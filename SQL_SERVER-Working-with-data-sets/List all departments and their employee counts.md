# List all departments and their employee counts, including departments with zero employees

## 🎯 Exercise
List all departments alongside their total employee count — including any departments that have zero employees assigned to them.

---

## 💡 Solution

### Approach
We use a `LEFT JOIN` between the `Department` table and the `EmployeeDepartmentHistory` table to preserve all departments regardless of whether they have employees. We then use `COUNT(BusinessEntityID)` grouped by department name to count the number of employees per department. Because `COUNT()` ignores `NULL` values, departments with no employees automatically return `0` rather than `NULL`.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `LEFT JOIN` | Returns all departments — including those with no matching rows in `EmployeeDepartmentHistory` |
| `COUNT(BusinessEntityID)` | Counts the number of non-`NULL` employee IDs per department — returns `0` for departments with no employees |
| `GROUP BY Department.[Name]` | Groups the results by department name so `COUNT()` produces one total per department |

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
  , COUNT(Employees.[BusinessEntityID]) AS EmployeeCount
FROM [AdventureWorks2022].[HumanResources].[Department] AS Department
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS Employees
    ON Department.DepartmentID = Employees.DepartmentID
GROUP BY Department.[Name]
ORDER BY Department.[Name] ASC
```

---

### Output

```
Name                        EmployeeCount
Document Control            5
Engineering                 7
Executive                   2
Facilities and Maintenance  7
Finance                     11
Human Resources             6
Information Services        10
Marketing                   10
Production                  180
Production Control          6
Purchasing                  13
Quality Assurance           7
Research and Development    4
Sales                       18
Shipping and Receiving      6
Tool Design                 4
(16 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — `LEFT JOIN` all departments to employee history

We join `Department` to `EmployeeDepartmentHistory` on `DepartmentID`. The `LEFT JOIN` ensures that all 16 departments are returned — even if a department has no rows in `EmployeeDepartmentHistory`. In that case, `BusinessEntityID` would be `NULL` for that department's row.

**Output (truncated):** One row per department-employee combination.

```
Department.Name   Department.DepartmentID  Employees.BusinessEntityID
Engineering       1                        2
Engineering       1                        3
Engineering       1                        5
Engineering       1                        6
...
Production        7                        28
Production        7                        29
...
Executive         11                       1
Executive         11                       25
[Hypothetical]    17                       NULL   ← would appear if dept had no employees
(295 rows affected — includes all department-employee combinations)
```

---

### Query 1.2 — Apply `COUNT()` and `GROUP BY`

`GROUP BY Department.[Name]` groups all rows by department name. `COUNT(Employees.[BusinessEntityID])` then counts the number of non-`NULL` values in `BusinessEntityID` for each group.

**Why `COUNT(BusinessEntityID)` and not `COUNT(*)`?**

`COUNT(*)` counts all rows including `NULL`s — so a department with no employees would return `1` (one row with all `NULL`s from the `LEFT JOIN`) instead of `0`. `COUNT(BusinessEntityID)` counts only non-`NULL` values — so a department with no employees correctly returns `0`.

| Expression | Department with no employees |
|---|---|
| `COUNT(*)` | Returns `1` — counts the `NULL` row from the `LEFT JOIN` |
| `COUNT(BusinessEntityID)` | Returns `0` — ignores `NULL` values ✓ |

---

### Interpreting the output

All 16 departments have at least one employee — consistent with the result from the [Finding the departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Finding%20the%20departments%20with%20zero%20employees.md) exercise, which confirmed no departments have zero employees.

Key observations from the output:

- **Production** is by far the largest department with **180 employees** — more than all other departments combined
- **Executive** is the smallest department with only **2 employees**
- **Research and Development** and **Tool Design** each have **4 employees**
- The total across all departments sums to **296** — slightly more than the 290 unique employees, because some employees have records in more than one department due to historical department changes tracked in `EmployeeDepartmentHistory`

---

### Related exercises

This exercise builds on the same `LEFT JOIN` pattern used in:
- [Finding the departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Finding%20the%20departments%20with%20zero%20employees.md) — uses `LEFT JOIN` + `WHERE IS NULL` to find unmatched departments
- [Find customers who have not made any purchase](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Find%20customers%20who%20have%20not%20made%20any%20purchase.md) — applies the same pattern across a different pair of tables

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
