# Pivoting employee count by department name

## 🎯 Exercise
Count the number of employees per department and pivot the results so that each department name becomes a column, with its employee count displayed in a single row.

---

## 💡 Solution

### Approach
We use nested subqueries combined with `ROW_NUMBER()`, `CASE` statements, and `SUM()` aggregation to:
1. Remove duplicate employee records caused by department history changes
2. Assign a `1` or `0` to each employee per department using `CASE`
3. Use `SUM()` to total each department column into a single pivoted row

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

### T-SQL code

```sql
SELECT
  SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Document Control'        THEN 1 ELSE 0 END) AS dept_DocControl
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Engineering'             THEN 1 ELSE 0 END) AS dept_Engin
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Executive'               THEN 1 ELSE 0 END) AS dept_Exec
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Facilities and Maintenance' THEN 1 ELSE 0 END) AS dept_FacILMaint
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Finance'                 THEN 1 ELSE 0 END) AS dept_Finance
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Human Resources'         THEN 1 ELSE 0 END) AS dept_HR
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Information Services'    THEN 1 ELSE 0 END) AS dept_IT
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Marketing'               THEN 1 ELSE 0 END) AS dept_Marketing
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Production'              THEN 1 ELSE 0 END) AS dept_Prod
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Production Control'      THEN 1 ELSE 0 END) AS dept_ProdControl
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Purchasing'              THEN 1 ELSE 0 END) AS dept_Purch
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Quality Assurance'       THEN 1 ELSE 0 END) AS dept_QA
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Research and Development' THEN 1 ELSE 0 END) AS dept_R_and_D
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Sales'                   THEN 1 ELSE 0 END) AS dept_Sales
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Shipping and Receiving'  THEN 1 ELSE 0 END) AS dept_ShipReceiv
, SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Tool Design'             THEN 1 ELSE 0 END) AS dept_ToolDesign
FROM (
    SELECT
        OriginalTables.DepartmentName
      , OriginalTables.BusinessEntityID
      , COUNT(*) AS EmployeeCountByDepartment
    FROM (
        SELECT
            ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                               ORDER BY Employee.BusinessEntityID ASC,
                                        EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
          , Employee.BusinessEntityID
          , EmployeeDepartmentHistory.DepartmentID
          , Department.[Name] AS DepartmentName
        FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
        LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
            ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
        LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
            ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
        WHERE Employee.BusinessEntityID <> 1
    ) AS OriginalTables
    WHERE OriginalTables.RowNumberRemovingDuplicates = 1
    GROUP BY OriginalTables.DepartmentName, OriginalTables.BusinessEntityID
) AS PivotDeptNames
```

---

### Output

```
dept_DocControl  dept_Engin  dept_Exec  dept_FacILMaint  dept_Finance  dept_HR  dept_IT  dept_Marketing  dept_Prod  dept_ProdControl  dept_Purch  dept_QA  dept_R_and_D  dept_Sales  dept_ShipReceiv  dept_ToolDesign
5                6           1          7                10            6        10       9               179        6                 12          6        4             18          6                4
(1 row affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Remove duplicate employee records (`RowNumberRemovingDuplicates`)
Some employees have changed departments over time, which creates multiple rows per employee in `EmployeeDepartmentHistory`. We use `ROW_NUMBER()` partitioned by `BusinessEntityID` and ordered by `StartDate DESC` to number each employee's department records, then filter to `RowNumberRemovingDuplicates = 1` to keep only the most recent department per employee.

**Output:** 289 unique employee rows, each with their current department name.

```
DepartmentName            BusinessEntityID
Engineering               2
Engineering               3
Tool Design               4
...
Sales                     289
Sales                     290
(289 rows affected)
```

---

### Query 1.2 — Add `EmployeeCountByDepartment` column
We add `COUNT(*) ... GROUP BY DepartmentName, BusinessEntityID` to prepare the data for aggregation. At this stage every employee still has their own row — each with a count of `1`.

**Output:** 289 rows, each employee with `EmployeeCountByDepartment = 1`.

```
DepartmentName            BusinessEntityID  EmployeeCountByDepartment
Engineering               2                 1
Engineering               3                 1
Tool Design               4                 1
...
Sales                     290               1
(289 rows affected)
```

---

### Query 1.3 — Pivot department names using `CASE` statements
For each row we use a `CASE` statement per department: if the employee belongs to that department, the column returns `1`; otherwise `0`. This converts each row into a wide format with one column per department.

**Output:** 289 rows, each with a `1` in one department column and `0` in all others (truncated).

```
DepartmentName    dept_DocControl  dept_Engin  dept_Exec  ...  dept_ToolDesign
Document Control  1                0           0          ...  0
Document Control  1                0           0          ...  0
Engineering       0                1           0          ...  0
...
Tool Design       0                0           0          ...  1
(289 rows affected)
```

---

### Query 1.4 — Sum per department and remove zeros
We wrap Query 1.3 with `SUM()` per department column and add `GROUP BY DepartmentName`. This collapses the 289 rows into 16 rows (one per department), but each row still contains zeros in the other columns.

**Output:** 16 rows — one per department, zeros still visible in non-matching columns.

```
dept_DocControl  dept_Engin  dept_Exec  ...
5                0           0          ...
0                6           0          ...
0                0           1          ...
...
(16 rows affected)
```

---

### Final Query (Query 1) — Remove zeros by dropping `GROUP BY`
Removing `GROUP BY PivotDeptNames.DepartmentName` causes `SUM()` to aggregate across **all** rows at once, collapsing the entire result into a single row. Each department column now shows its total employee count with no zeros.

**Final output:** 1 row with all 16 department counts.

```
dept_DocControl  dept_Engin  dept_Exec  dept_FacILMaint  dept_Finance  dept_HR  dept_IT  dept_Marketing  dept_Prod  dept_ProdControl  dept_Purch  dept_QA  dept_R_and_D  dept_Sales  dept_ShipReceiv  dept_ToolDesign
5                6           1          7                10            6        10       9               179        6                 12          6        4             18          6                4
(1 row affected)
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
