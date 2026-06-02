# Find the number of employees in each job title

## 🎯 Exercise
Count the number of employees per job title — consolidating the `Production Supervisor` and `Production Technician` job title variants into single categories to reduce the number of distinct job titles from 67 to 55.

---

## 💡 Solution

### Approach
We use two levels of queries — an inner subquery and an outer query:

- **Inner query:** Joins three tables to retrieve each employee's job title and applies a `CASE WHEN` statement to consolidate `Production Supervisor` and `Production Technician` variants (e.g. `Production Supervisor - WC60`) into a single label each. The result is grouped by job title and `BusinessEntityID` to produce one row per employee.
- **Outer query:** Groups the inner result by the consolidated job title and uses `COUNT()` to count the number of employees per title.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `CASE WHEN JobTitle LIKE 'Production Supervisor%' THEN 'Production Supervisor'` | Consolidates all Production Supervisor variants into one label |
| `CASE WHEN JobTitle LIKE 'Production Technician%' THEN 'Production Technician'` | Consolidates all Production Technician variants into one label |
| `LIKE 'text%'` | Matches any job title that starts with the given text — the `%` wildcard matches any characters that follow |
| `LEFT JOIN` | Returns all departments — including those with no employees |
| `INNER JOIN` | Connects `EmployeeDepartmentHistory` to `Employee` to retrieve `JobTitle` |
| `GROUP BY JobTitle, BusinessEntityID` | Inner grouping — produces one row per employee with their consolidated job title |
| `COUNT(BusinessEntityID)` | Outer count — counts the number of employees per consolidated job title |

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
    X.JobTitle
  , COUNT(X.[BusinessEntityID]) AS NumberEmployees
FROM (
    SELECT
        CASE
            WHEN Employees.[JobTitle] LIKE 'Production Supervisor%' THEN 'Production Supervisor'
            WHEN Employees.[JobTitle] LIKE 'Production Technician%' THEN 'Production Technician'
            ELSE Employees.[JobTitle]
        END                                                          AS JobTitle
      , COUNT(EmployeesHistorical.[BusinessEntityID])               AS NumberEmployees
      , EmployeesHistorical.[BusinessEntityID]
    FROM [AdventureWorks2022].[HumanResources].[Department] AS Departments
    LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeesHistorical
        ON Departments.[DepartmentID] = EmployeesHistorical.[DepartmentID]
    INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS Employees
        ON Employees.[BusinessEntityID] = EmployeesHistorical.[BusinessEntityID]
    GROUP BY Employees.[JobTitle], EmployeesHistorical.[BusinessEntityID]
) AS X
GROUP BY X.JobTitle
```

---

### Output

```
JobTitle                                      NumberEmployees
Accountant                                    2
Accounts Manager                              1
Accounts Payable Specialist                   2
Accounts Receivable Specialist                3
Application Specialist                        4
Assistant to the Chief Financial Officer      1
Benefits Specialist                           1
Buyer                                         9
Chief Executive Officer                       1
Chief Financial Officer                       1
Control Specialist                            2
Database Administrator                        2
Design Engineer                               3
Document Control Assistant                    2
Document Control Manager                      1
Engineering Manager                           1
European Sales Manager                        1
Facilities Administrative Assistant           1
Facilities Manager                            1
Finance Manager                               1
Human Resources Administrative Assistant      2
Human Resources Manager                       1
Information Services Manager                  1
Janitor                                       4
Maintenance Supervisor                        1
Marketing Assistant                           3
Marketing Manager                             1
Marketing Specialist                          5
Master Scheduler                              1
Network Administrator                         2
Network Manager                               1
North American Sales Manager                  1
Pacific Sales Manager                         1
Production Control Manager                    1
Production Supervisor                         21
Production Technician                         157
Purchasing Assistant                          2
Purchasing Manager                            1
Quality Assurance Manager                     1
Quality Assurance Supervisor                  1
Quality Assurance Technician                  4
Recruiter                                     2
Research and Development Engineer             2
Research and Development Manager              2
Sales Representative                          14
Scheduling Assistant                          4
Senior Design Engineer                        1
Senior Tool Designer                          2
Shipping and Receiving Clerk                  2
Shipping and Receiving Supervisor             1
Stocker                                       3
Tool Designer                                 2
Vice President of Engineering                 1
Vice President of Production                  1
Vice President of Sales                       1
(55 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Why job title consolidation is needed

Without consolidation, the `JobTitle` column in the `Employee` table contains **67 distinct values** — because Production Supervisor and Production Technician roles are stored with their work centre suffix (e.g. `Production Supervisor - WC10`, `Production Supervisor - WC20`, `Production Supervisor - WC60`, etc.).

**Example of unconsolidated Production titles:**

```
JobTitle
Production Supervisor - WC10
Production Supervisor - WC20
Production Supervisor - WC30
Production Supervisor - WC40
Production Supervisor - WC45
Production Supervisor - WC50
Production Supervisor - WC60
Production Technician - WC10
Production Technician - WC20
Production Technician - WC30
Production Technician - WC40
Production Technician - WC45
Production Technician - WC50
Production Technician - WC60
```

Counting employees by these 14 variants would fragment the Production workforce into small groups — making the results harder to read and interpret. The `CASE WHEN ... LIKE 'Production Supervisor%'` and `CASE WHEN ... LIKE 'Production Technician%'` statements collapse all variants into two labels, reducing the total from **67 to 55 distinct job titles**.

---

### Query 1.2 — Inner query: one row per employee with consolidated job title

The inner query joins three tables and applies the `CASE WHEN` consolidation. `GROUP BY Employees.[JobTitle], EmployeesHistorical.[BusinessEntityID]` produces one row per employee — using the original (unconsolidated) job title for grouping to avoid prematurely merging variants.

**Output (truncated):** 290 rows — one per employee with their consolidated job title.

```
JobTitle                BusinessEntityID  NumberEmployees
Chief Executive Officer 1                 1
Vice President of Eng.  2                 1
Engineering Manager     3                 1
Senior Tool Designer    4                 1
Design Engineer         5                 1
...
Production Supervisor   26                1   ← 'Production Supervisor - WC60' → consolidated
Production Supervisor   27                1   ← 'Production Supervisor - WC60' → consolidated
Production Technician   28                1   ← 'Production Technician - WC60' → consolidated
Production Technician   29                1   ← 'Production Technician - WC60' → consolidated
...
(290 rows affected)
```

---

### Query 1.3 — Outer query: count employees per consolidated job title

The outer query groups the 290 rows by the consolidated `JobTitle` and uses `COUNT(BusinessEntityID)` to total the employees per title. This collapses all Production Supervisor variants (21 employees across 7 work centres) into one row, and all Production Technician variants (157 employees) into another.

**Key consolidation result:**

| Before consolidation | Count | After consolidation | Count |
|---|---|---|---|
| Production Supervisor - WC10 | 3 | Production Supervisor | 21 |
| Production Supervisor - WC20 | 3 | | |
| Production Supervisor - WC30 | 3 | | |
| Production Supervisor - WC40 | 3 | | |
| Production Supervisor - WC45 | 3 | | |
| Production Supervisor - WC50 | 3 | | |
| Production Supervisor - WC60 | 3 | | |

---

### Key observations from the output

- **Production Technician** is the largest job title group with **157 employees** — more than half the entire workforce
- **Production Supervisor** has **21 employees** across 7 work centres
- **31 out of 55 job titles** have only **1 employee** — reflecting the many specialised management and individual contributor roles in the company
- **Sales Representative** has **14 employees** — the largest non-Production title after consolidation

---

### Related exercises

This exercise extends the conditional aggregation technique from:
- [Count number of males and females in each department using conditional aggregation](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Count%20number%20of%20males%20and%20females%20in%20each%20department%20using%20conditional%20aggregation.md)
- [Count employees in each department having more than 5 employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Count%20employees%20in%20each%20department%20having%20more%20than%205%20employees.md)

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
