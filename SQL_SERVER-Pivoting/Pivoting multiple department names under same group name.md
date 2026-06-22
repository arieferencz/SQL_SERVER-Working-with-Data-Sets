# Pivoting multiple department names under same group name

## 🎯 Exercise
Pivot the department names under their respective group name by turning rows into columns — so that each group name becomes a column header, with its department names listed vertically underneath.

---

## 💡 Solution

### Approach
We use `ROW_NUMBER()` to assign a sequential number to each department within its group. We then use `CASE` statements with `MAX()` to pivot the department names into columns grouped by their row number. Finally, `COALESCE()` replaces any remaining `NULL` values with empty strings for a clean output.

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
    PivotingDeptNameGroup.RowNmber
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Executive General and Administration' THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS ExecutiveGeneralandAdmin
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Inventory Management'                 THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS InventoryManagement
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Manufacturing'                        THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS Manufacturing
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Quality Assurance'                    THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS QualityAssurance
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Research and Development'             THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS ResearchandDevelopment
  , COALESCE(MAX(CASE WHEN PivotingDeptNameGroup.GroupName = 'Sales and Marketing'                  THEN PivotingDeptNameGroup.DepartmentName ELSE NULL END), '') AS SalesandMarketing
FROM (
    SELECT
        Department.GroupName
      , Department.[Name] AS DepartmentName
      , ROW_NUMBER() OVER (PARTITION BY Department.GroupName ORDER BY Department.[Name]) AS RowNmber
    FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
    LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
        ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
    GROUP BY Department.GroupName, Department.[Name]
) AS PivotingDeptNameGroup
GROUP BY PivotingDeptNameGroup.RowNmber
```

---

### Output

```
RowNumber  ExecutiveGeneralandAdmin    InventoryManagement     Manufacturing       QualityAssurance   ResearchandDevelopment       SalesandMarketing
1          Executive                   Purchasing              Production          Document Control   Engineering                  Marketing
2          Facilities and Maintenance  Shipping and Receiving  Production Control  Quality Assurance  Research and Development     Sales
3          Finance                                                                                    Tool Design
4          Human Resources
5          Information Services
(5 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Assign a row number to each department within its group (`RowNumber`)
We join the three tables and use `ROW_NUMBER()` partitioned by `GroupName` and ordered alphabetically by department name. This assigns each department a sequential number (1, 2, 3...) within its group. The `GROUP BY` removes duplicate rows caused by multiple employees belonging to the same department.

**T-SQL code of Query 1.1**
```sql
SELECT Department.GroupName                                            -- RowNumberDeptGroupNameLevel1
, Department.[Name] AS DepartmentName
, ROW_NUMBER() OVER(PARTITION BY Department.GroupName ORDER BY Department.[Name]) AS RowNumber
FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
	ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
	ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID	
GROUP BY Department.GroupName, Department.[Name]                        -- RowNumberDeptGroupNameLevel1
```

**Output of Query 1.1:** 16 rows — one per unique department, each with its group name and row number.

```
GroupName                             DepartmentName              RowNumber
Executive General and Administration  Executive                   1
Executive General and Administration  Facilities and Maintenance  2
Executive General and Administration  Finance                     3
Executive General and Administration  Human Resources             4
Executive General and Administration  Information Services        5
Inventory Management                  Purchasing                  1
Inventory Management                  Shipping and Receiving      2
Manufacturing                         Production                  1
Manufacturing                         Production Control          2
Quality Assurance                     Document Control            1
Quality Assurance                     Quality Assurance           2
Research and Development              Engineering                 1
Research and Development              Research and Development    2
Research and Development              Tool Design                 3
Sales and Marketing                   Marketing                   1
Sales and Marketing                   Sales                       2
(16 rows affected)
```

---

### Query 1.2 — Pivot department names using `CASE` statements
For each row we use a `CASE` statement per group: if the row belongs to that group, the column returns the department name; otherwise `NULL`. This spreads each department into its group's column, but the result still has `NULL` values in all non-matching columns and duplicate row numbers across groups.

**T-SQL code of Query 1.2**
```sql

```

**Output:** 16 rows, each with a department name in one column and `NULL` in all others.

```
RowNumber  ExecutiveGeneralandAdmin    InventoryManagement  Manufacturing  QualityAssurance  ResearchandDevelopment  SalesandMarketing
1          Executive                   NULL                 NULL           NULL              NULL                    NULL
2          Facilities and Maintenance  NULL                 NULL           NULL              NULL                    NULL
3          Finance                     NULL                 NULL           NULL              NULL                    NULL
4          Human Resources             NULL                 NULL           NULL              NULL                    NULL
5          Information Services        NULL                 NULL           NULL              NULL                    NULL
1          NULL                        Purchasing           NULL           NULL              NULL                    NULL
2          NULL                        Shipping and Rec...  NULL           NULL              NULL                    NULL
...
(16 rows affected)
```

---

### Query 1.3 — Group by `RowNumber` and collapse using `MAX()`
We add `GROUP BY RowNumber` and wrap each `CASE` statement with `MAX()`. Since each row number within a group contains only one non-`NULL` value, `MAX()` returns that value while ignoring all `NULL`s. This collapses the 16 rows into 5 rows (one per row number), aligning all departments into their correct columns.

**Output:** 5 rows — departments aligned under their group columns, but `NULL` still visible where a group has fewer departments than others.

```
RowNumber  ExecutiveGeneralandAdmin    InventoryManagement     Manufacturing       QualityAssurance   ResearchandDevelopment    SalesandMarketing
1          Executive                   Purchasing              Production          Document Control   Engineering               Marketing
2          Facilities and Maintenance  Shipping and Receiving  Production Control  Quality Assurance  Research and Development  Sales
3          Finance                     NULL                    NULL                NULL               Tool Design               NULL
4          Human Resources             NULL                    NULL                NULL               NULL                      NULL
5          Information Services        NULL                    NULL                NULL               NULL                      NULL
(5 rows affected)
```

---

### Final Query (Query 1) — Replace `NULL` with empty strings using `COALESCE()`
We wrap each `MAX(CASE...)` expression with `COALESCE(..., '')`. This replaces any remaining `NULL` values with an empty string, producing a clean final output with no `NULL` values visible.

**Final output:** 5 rows, clean with no NULLs.

```
RowNumber  ExecutiveGeneralandAdmin    InventoryManagement     Manufacturing       QualityAssurance   ResearchandDevelopment       SalesandMarketing
1          Executive                   Purchasing              Production          Document Control   Engineering                  Marketing
2          Facilities and Maintenance  Shipping and Receiving  Production Control  Quality Assurance  Research and Development     Sales
3          Finance                                                                                    Tool Design
4          Human Resources
5          Information Services
(5 rows affected)
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
