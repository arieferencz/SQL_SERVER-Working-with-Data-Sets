# Reverse pivoting employee count by department (columns into rows)

## 🎯 Exercise
Reverse the pivoting of the employee count by department — converting a wide result (1 row × 16 columns) back into a tall result (16 rows × 2 columns: department name and employee count).

---

## 💡 Solution

### Approach
We first create a **VIEW** that stores the employee count per department as 16 columns in a single row. We then use a **Cartesian Product** to pair each department name from the `Department` table with the view's single row — generating 16 repeated rows, one per department. Finally, a `CASE` statement selects the correct employee count column for each department name, collapsing the 16 columns into 1.

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

## 🔧 Step 1 — Create the VIEW

We first create a VIEW named `VIEW_CountNumberEmployeesByDepartment` that stores the employee count per department as 16 columns in a single row. This is the data we will reverse pivot.

### T-SQL code — Create the VIEW

```sql
USE AdventureWorks2022;
GO

IF OBJECT_ID(N'VIEW_CountNumberEmployeesByDepartment', N'V') IS NOT NULL
    DROP VIEW VIEW_CountNumberEmployeesByDepartment
GO

CREATE VIEW VIEW_CountNumberEmployeesByDepartment
AS
SELECT
    SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Document Control'            THEN 1 ELSE 0 END) AS dept_DocControl
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Engineering'                 THEN 1 ELSE 0 END) AS dept_Engin
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Executive'                   THEN 1 ELSE 0 END) AS dept_Exec
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Facilities and Maintenance'  THEN 1 ELSE 0 END) AS dept_FacILMaint
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Finance'                     THEN 1 ELSE 0 END) AS dept_Finance
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Human Resources'             THEN 1 ELSE 0 END) AS dept_HR
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Information Services'        THEN 1 ELSE 0 END) AS dept_IT
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Marketing'                   THEN 1 ELSE 0 END) AS dept_Marketing
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Production'                  THEN 1 ELSE 0 END) AS dept_Prod
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Production Control'          THEN 1 ELSE 0 END) AS dept_ProdControl
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Purchasing'                  THEN 1 ELSE 0 END) AS dept_Purch
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Quality Assurance'           THEN 1 ELSE 0 END) AS dept_QA
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Research and Development'    THEN 1 ELSE 0 END) AS dept_R_and_D
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Sales'                       THEN 1 ELSE 0 END) AS dept_Sales
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Shipping and Receiving'      THEN 1 ELSE 0 END) AS dept_ShipReceiv
  , SUM(CASE WHEN PivotDeptNames.DepartmentName = 'Tool Design'                 THEN 1 ELSE 0 END) AS dept_ToolDesign
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

**Output:**
```
Commands completed successfully.
```

### Verify the VIEW contents

```sql
USE AdventureWorks2022;
GO

SELECT *
FROM VIEW_CountNumberEmployeesByDepartment
```

**Output:** 1 row × 16 columns — one employee count per department.

```
dept_DocControl  dept_Engin  dept_Exec  dept_FacILMaint  dept_Finance  dept_HR  dept_IT  dept_Marketing  dept_Prod  dept_ProdControl  dept_Purch  dept_QA  dept_R_and_D  dept_Sales  dept_ShipReceiv  dept_ToolDesign
5                6           1          7                10            6        10       9               179        6                 12          6        4             18          6                4
(1 row affected)
```

---

## 🔧 Step 2 — Reverse pivot using Cartesian Product + `CASE`

### T-SQL code — Final query

```sql
USE AdventureWorks2022;
GO

SELECT
    Department.[Name]
  , CASE Department.[Name]
        WHEN 'Document Control'            THEN VIEW_CountNumberEmployeesByDepartment.dept_DocControl
        WHEN 'Engineering'                 THEN VIEW_CountNumberEmployeesByDepartment.dept_Engin
        WHEN 'Executive'                   THEN VIEW_CountNumberEmployeesByDepartment.dept_Exec
        WHEN 'Facilities and Maintenance'  THEN VIEW_CountNumberEmployeesByDepartment.dept_FacILMaint
        WHEN 'Finance'                     THEN VIEW_CountNumberEmployeesByDepartment.dept_Finance
        WHEN 'Human Resources'             THEN VIEW_CountNumberEmployeesByDepartment.dept_HR
        WHEN 'Information Services'        THEN VIEW_CountNumberEmployeesByDepartment.dept_IT
        WHEN 'Marketing'                   THEN VIEW_CountNumberEmployeesByDepartment.dept_Marketing
        WHEN 'Production'                  THEN VIEW_CountNumberEmployeesByDepartment.dept_Prod
        WHEN 'Production Control'          THEN VIEW_CountNumberEmployeesByDepartment.dept_ProdControl
        WHEN 'Purchasing'                  THEN VIEW_CountNumberEmployeesByDepartment.dept_Purch
        WHEN 'Quality Assurance'           THEN VIEW_CountNumberEmployeesByDepartment.dept_QA
        WHEN 'Research and Development'    THEN VIEW_CountNumberEmployeesByDepartment.dept_R_and_D
        WHEN 'Sales'                       THEN VIEW_CountNumberEmployeesByDepartment.dept_Sales
        WHEN 'Shipping and Receiving'      THEN VIEW_CountNumberEmployeesByDepartment.dept_ShipReceiv
        WHEN 'Tool Design'                 THEN VIEW_CountNumberEmployeesByDepartment.dept_ToolDesign
    END AS CountEmployeeByDept
FROM VIEW_CountNumberEmployeesByDepartment
   , [AdventureWorks2022].[HumanResources].[Department]
```

### Output

```
Name                        CountEmployeeByDept
Document Control            5
Engineering                 6
Executive                   1
Facilities and Maintenance  7
Finance                     10
Human Resources             6
Information Services        10
Marketing                   9
Production                  179
Production Control          6
Purchasing                  12
Quality Assurance           6
Research and Development    4
Sales                       18
Shipping and Receiving      6
Tool Design                 4
(16 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1 — Create the VIEW (`VIEW_CountNumberEmployeesByDepartment`)
This VIEW stores the employee count per department as a single row with 16 columns — one per department. It reuses the same pivoting logic from the earlier exercise [Pivoting employee count by department name](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Pivoting%20employee%20count%20by%20department%20name.md). The VIEW has dimensions: **1 row × 16 columns**.

---

### Query 2 — Apply the Cartesian Product
We perform a Cartesian Product by listing two tables in the `FROM` clause without a `JOIN` condition:
- **Table 1:** `Department` — contains 16 rows, one per department name
- **Table 2:** `VIEW_CountNumberEmployeesByDepartment` — contains 1 row × 16 columns

A Cartesian Product pairs every row from Table 1 with every row from Table 2. Since Table 2 has only 1 row, the result is 16 rows — each department name paired with all 16 employee count columns repeated on every row.

**How the pairing works:**

```
Table 1 (Department.Name)    Table 2 (dept_DocControl)    Paired result
Document Control             5                            (Document Control, 5)
Engineering                  5                            (Engineering, 5)
Executive                    5                            (Executive, 5)
Facilities and Maintenance   5                            (Facilities and Maintenance, 5)
... (16 combinations total)
```

**T-SQL code of Query 2**
```sql
SELECT Department.[Name]
, VIEW_CountNumberEmployeesByDepartment.dept_DocControl
, VIEW_CountNumberEmployeesByDepartment.dept_Engin
, VIEW_CountNumberEmployeesByDepartment.dept_Exec
, VIEW_CountNumberEmployeesByDepartment.dept_FacILMaint
, VIEW_CountNumberEmployeesByDepartment.dept_Finance
, VIEW_CountNumberEmployeesByDepartment.dept_HR
, VIEW_CountNumberEmployeesByDepartment.dept_IT
, VIEW_CountNumberEmployeesByDepartment.dept_Marketing
, VIEW_CountNumberEmployeesByDepartment.dept_Prod
, VIEW_CountNumberEmployeesByDepartment.dept_ProdControl
, VIEW_CountNumberEmployeesByDepartment.dept_Purch
, VIEW_CountNumberEmployeesByDepartment.dept_QA
, VIEW_CountNumberEmployeesByDepartment.dept_R_and_D
, VIEW_CountNumberEmployeesByDepartment.dept_Sales
, VIEW_CountNumberEmployeesByDepartment.dept_ShipReceiv
, VIEW_CountNumberEmployeesByDepartment.dept_ToolDesign
FROM VIEW_CountNumberEmployeesByDepartment
, [AdventureWorks2022].[HumanResources].[Department]
```

**Output of Query 2:** 16 rows, each containing the department name and all 16 employee count columns repeated.
```
Name				        dept_DocControl		dept_Engin	dept_Exec	dept_FacILMaint		...		dept_R_and_D	dept_Sales		dept_ShipReceiv		dept_ToolDesign
Document Control		    5			        6		    1		    7			        ...		4				18				6					4
Engineering			        5			        6		    1		    7					...		4				18				6					4
Executive			        5			        6		    1		    7					...		4				18				6					4
Facilities and Maintenance	5			        6		    1		    7					...		4				18				6					4
Finance				        5			        6		    1		    7					...		4				18				6					4
...
Research and Development	5			        6			1		    7					...		4				18				6					4
Sales				        5			        6		    1		    7					...		4				18				6					4
Shipping and Receiving		5			        6		    1		    7					...		4				18				6					4
Tool Design			        5			        6		    1		    7					...		4				18				6					4
(16 rows affected)
```

---

### Query 2.1 — Select the correct count per department using `CASE`
We add a `CASE` statement that checks the department name on each row and returns only the matching employee count column — all other columns are discarded. This collapses the 16 count columns into a single `CountEmployeeByDept` column with the correct value for each department.

**Final output:** 16 rows × 2 columns — one row per department with its employee count.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
