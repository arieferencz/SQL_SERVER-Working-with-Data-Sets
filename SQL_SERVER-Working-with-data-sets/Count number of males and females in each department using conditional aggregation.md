# Count number of males and females in each department using conditional aggregation

## 🎯 Exercise
Count the number of male and female employees in each department using conditional aggregation — displaying both counts alongside the total employee count per department.

---

## 💡 Solution

### Approach
We join three tables to retrieve each employee's department and gender. We then use `CASE WHEN` statements inside `COUNT()` to conditionally count only male employees in one column and only female employees in another — a technique known as **conditional aggregation**. `GROUP BY` collapses the results into one row per department.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `LEFT JOIN` | Returns all departments — including those with no employees |
| `INNER JOIN` | Connects `EmployeeDepartmentHistory` to `Employee` to retrieve the `Gender` column |
| `CASE WHEN Gender = 'M' THEN 1 END` | Returns `1` for male employees and `NULL` for all others |
| `CASE WHEN Gender = 'F' THEN 1 END` | Returns `1` for female employees and `NULL` for all others |
| `COUNT(CASE WHEN ...)` | Counts only the non-`NULL` values returned by the `CASE` statement — ignoring the other gender |
| `COUNT(BusinessEntityID)` | Counts the total number of employees per department |
| `GROUP BY Department.[Name]` | Groups results by department name |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Department` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Employee` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT
    Departments.[Name]
  , COUNT(CASE WHEN Employees.[Gender] = 'M' THEN 1 END) AS MaleCount
  , COUNT(CASE WHEN Employees.[Gender] = 'F' THEN 1 END) AS FemaleCount
  , COUNT(EmployeesHistorical.[BusinessEntityID])         AS NumberEmployees
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
    ON Employees.[BusinessEntityID] = EmployeesHistorical.[BusinessEntityID]
GROUP BY Departments.[Name]
```

---
### Output
```
Name                        MaleCount  FemaleCount  NumberEmployees
Document Control            4          1            5
Engineering                 4          3            7
Executive                   1          1            2
Facilities and Maintenance  5          2            7
Finance                     5          6            11
Human Resources             4          2            6
Information Services        6          4            10
Marketing                   5          5            10
Production                  134        46           180
Production Control          6          0            6
Purchasing                  9          4            13
Quality Assurance           6          1            7
Research and Development    2          2            4
Sales                       11         7            18
Shipping and Receiving      4          2            6
Tool Design                 3          1            4
(16 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Join the three tables

The `LEFT JOIN` between `Department` and `EmployeeDepartmentHistory` preserves all departments. The `INNER JOIN` to `Employee` retrieves the `Gender` value for each employee. At this stage the result contains one row per department-employee combination with the employee's gender.

**T-SQL code of Query 1.1**
```sql
SELECT
    Departments.[Name] AS DepartmentName
  , Employees.[BusinessEntityID] AS BusinessEntityID
  , Employees.[Gender] AS Gender
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
    ON Employees.[BusinessEntityID] = EmployeesHistorical.[BusinessEntityID]
ORDER BY Departments.[Name]
```

**Output of Query 1.1 (truncated):** One row per department-employee combination.

```
DepartmentName				BusinessEntityID	Gender
Document Control			217					M
Document Control			218					M
Document Control			219					M
Document Control			220					F
Document Control			221					M
Engineering					2					F
Engineering					3					M
Engineering					4					M
Engineering					5					F
Engineering					6					M
Engineering					14					M
Engineering					15					F
Executive					1					M
...
Shipping and Receiving		121					M
Shipping and Receiving		122					F
Shipping and Receiving		123					M
Shipping and Receiving		124					F
Shipping and Receiving		125					M
Shipping and Receiving		126					M
Tool Design					4					M
Tool Design					11					M
Tool Design					12					M
Tool Design					13					F
(296 rows affected)
```

---

### Query 1.2 — Apply conditional aggregation using `CASE WHEN` inside `COUNT()`

For each row we add two `CASE WHEN` expressions:

- `CASE WHEN Gender = 'M' THEN 1 END` → returns `1` for male employees, `NULL` for females
- `CASE WHEN Gender = 'F' THEN 1 END` → returns `1` for female employees, `NULL` for males

Since `COUNT()` ignores `NULL` values, wrapping `COUNT()` around each `CASE` expression counts only the rows where the condition is `TRUE` — effectively counting males and females independently in the same query.

**T-SQL code of Query 1.2: How the conditional aggregation works — example for Engineering:**

```sql
SELECT
    Employees.[BusinessEntityID] AS BusinessEntityID
  , Employees.[Gender] AS Gender
  , CASE WHEN Employees.[Gender] = 'M' THEN 1 END AS "'CASE Gender='M'"
  , CASE WHEN Employees.[Gender] = 'F' THEN 1 END AS "'CASE Gender='F'"
FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
    ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
    ON Employees.[BusinessEntityID] = EmployeesHistorical.[BusinessEntityID]
WHERE Departments.[Name] = 'Engineering'
ORDER BY Departments.[Name]
```

**Output of Query 1.2:**
```
BusinessEntityID	Gender	'CASE Gender='M'	'CASE Gender='F'
2					F		NULL				1
3					M		1					NULL
4					M		1					NULL
5					F		NULL				1
6					M		1					NULL
14					M		1					NULL
15					F		NULL				1
```

---

### Key concept — conditional aggregation

**Conditional aggregation** is the technique of placing a `CASE WHEN` statement inside an aggregate function (`COUNT`, `SUM`, `AVG`, etc.) to aggregate only a subset of rows that meet a specific condition — without needing a separate subquery or `GROUP BY` for each condition.

This is more efficient than running two separate queries and avoids the need for pivoting:

```sql
-- Without conditional aggregation (two separate queries needed):
SELECT COUNT(*) AS MaleCount   FROM Employee WHERE Gender = 'M' ...
SELECT COUNT(*) AS FemaleCount FROM Employee WHERE Gender = 'F' ...

-- With conditional aggregation (single query):
COUNT(CASE WHEN Gender = 'M' THEN 1 END) AS MaleCount
COUNT(CASE WHEN Gender = 'F' THEN 1 END) AS FemaleCount
```

The same pattern can be extended to any number of conditions — for example, counting employees by organisation level, salary band, or hire year — all in a single query.

---

### Key observations from the output

- **Production Control** is the only department with **0 female employees** — all 6 employees are male
- **Finance** is the only department where **females outnumber males** (6 F vs 5 M)
- **Marketing** and **Research and Development** are the only departments with an **equal gender split**
- **Production** has the largest absolute gap — **134 males vs 46 females** — driven by its size of 180 employees

---

### Related exercises

This exercise extends the counting logic from:
- [List all departments and their employee counts, including departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/List%20all%20departments%20and%20their%20employee%20counts.md)
- [Count employees in each department having more than 5 employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Count%20employees%20in%20each%20department%20having%20more%20than%205%20employees.md)

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
