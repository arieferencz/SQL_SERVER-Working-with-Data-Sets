# Count employees in each department having more than 5 employees

## 🎯 Exercise
List all departments that have more than 5 employees, alongside their total employee count.

---

## 💡 Solution

### Approach
We use a `LEFT JOIN` between the `Department` and `EmployeeDepartmentHistory` tables to count employees per department. We use `GROUP BY` to aggregate by department name and `COUNT()` to total the employees. The `HAVING` clause then filters the grouped results — keeping only departments where the employee count exceeds 5.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `LEFT JOIN` | Returns all departments — including those with no employees |
| `COUNT(BusinessEntityID)` | Counts the number of non-`NULL` employee IDs per department |
| `GROUP BY Department.[Name]` | Groups results by department name so `COUNT()` produces one total per department |
| `HAVING COUNT(...) > 5` | Filters the grouped results — keeps only departments with more than 5 employees |

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
    Departments.[Name]
  , COUNT(EmployeesHistorical.[BusinessEntityID]) AS NumberEmployees
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
GROUP BY Departments.[Name]
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) > 5
```

---

### Output

```
Name                        NumberEmployees
Engineering                 7
Sales                       18
Marketing                   10
Purchasing                  13
Production                  180
Production Control          6
Human Resources             6
Finance                     11
Information Services        10
Quality Assurance           7
Facilities and Maintenance  7
Shipping and Receiving      6
(12 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Count all employees per department (without `HAVING`)

First, we apply `LEFT JOIN`, `COUNT()`, and `GROUP BY` to get the employee count for all 16 departments — the same result as the [List all departments and their employee counts](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/List%20all%20departments%20and%20their%20employee%20counts.md) exercise.

**Output (all 16 departments):**

```
Name                        NumberEmployees
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

### Query 1.2 — Add `HAVING` to filter departments with more than 5 employees

Adding `HAVING COUNT(BusinessEntityID) > 5` filters the grouped result — keeping only departments where the count exceeds 5. This removes 4 departments from the result:

| Department removed | Employee count | Reason |
|---|---|---|
| Document Control | 5 | Not greater than 5 — exactly 5 |
| Executive | 2 | Less than 5 |
| Research and Development | 4 | Less than 5 |
| Tool Design | 4 | Less than 5 |

**Final output:** 12 departments — all with more than 5 employees.

---

### Key concept — `HAVING` vs `WHERE`

`WHERE` and `HAVING` both filter rows, but they operate at different stages of query execution:

| Clause | When it runs | What it filters | Can use aggregate functions? |
|---|---|---|---|
| `WHERE` | Before `GROUP BY` | Individual rows from the raw data | No |
| `HAVING` | After `GROUP BY` | Grouped and aggregated results | Yes |

In this query, we need to filter **after** the employee counts have been calculated per department — so `HAVING` is the correct clause. Using `WHERE COUNT(...) > 5` would cause a syntax error because `WHERE` cannot reference aggregate functions.

**Example showing why `WHERE` cannot be used here:**

```sql
-- INCORRECT — causes an error:
WHERE COUNT(EmployeesHistorical.[BusinessEntityID]) > 5

-- CORRECT — filters after aggregation:
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) > 5
```

---

### Changing the threshold

To retrieve departments with a different employee count threshold, simply change the value in the `HAVING` clause:

```sql
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) > 10   -- more than 10 employees
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) >= 5   -- 5 or more employees
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) = 6    -- exactly 6 employees
HAVING COUNT(EmployeesHistorical.[BusinessEntityID]) BETWEEN 5 AND 10  -- between 5 and 10
```

---

### Related exercises

This exercise builds directly on:
- [List all departments and their employee counts, including departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/List%20all%20departments%20and%20their%20employee%20counts%2C%20including%20departments%20with%20zero%20employees.md) — the same query without the `HAVING` filter
- [Finding the departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Finding%20the%20departments%20with%20zero%20employees.md) — uses `LEFT JOIN` + `WHERE IS NULL` for the opposite scenario

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
