# Finding the manager for each employee (Parent-Child relationship)

## 🎯 Exercise
Find the manager for each employee based on the organisational hierarchy (Parent-Child relationship).

---

## 💡 Solution

### Approach
We assign each employee a numeric `EmployeeCode` and `ManagerCode` based on their department group and organisation level. We then use a **self-join** via CTEs to match each employee's `ManagerCode` to another employee's `EmployeeCode`, returning the manager's ID and job title alongside each employee.

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

### T-SQL code

```sql
WITH 
OriginalTablesLevel1 AS 
(
    SELECT
    ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                       ORDER BY Employee.BusinessEntityID ASC, EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
    , Employee.BusinessEntityID	
    , Department.GroupName
    , GroupNameCode = CASE Department.GroupName
        WHEN 'Executive General and Administration' THEN 1
        WHEN 'Inventory Management'                 THEN 2
        WHEN 'Manufacturing'                        THEN 3
        WHEN 'Quality Assurance'                    THEN 4
        WHEN 'Research and Development'             THEN 5
        WHEN 'Sales and Marketing'                  THEN 6
        ELSE 'Error'
        END 
    , EmployeeDepartmentHistory.DepartmentID
    , Department.[Name] AS DeparmentName
    , Employee.JobTitle
    , Employee.OrganizationLevel
    , EmployeeDepartmentHistory.StartDate
    , EmployeeDepartmentHistory.EndDate
    FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
    LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
        ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
        ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID	
    WHERE Employee.BusinessEntityID <> 234
),
RemovingDuplicatesLevel2 AS
(
    SELECT 
        OriginalTablesLevel1.BusinessEntityID
        , OriginalTablesLevel1.GroupNameCode
        , OriginalTablesLevel1.GroupName
        , OriginalTablesLevel1.DepartmentID
        , OriginalTablesLevel1.DeparmentName
        , OriginalTablesLevel1.JobTitle
        , OriginalTablesLevel1.OrganizationLevel
        , EmployeeCode = OriginalTablesLevel1.GroupNameCode * 10000 + OriginalTablesLevel1.OrganizationLevel
        , ManagerCode  = OriginalTablesLevel1.GroupNameCode * 10000 + OriginalTablesLevel1.OrganizationLevel - 1
    FROM OriginalTablesLevel1
    WHERE OriginalTablesLevel1.RowNumberRemovingDuplicates = 1
),
SelfJoinBwithDuplicates AS
(
    SELECT
        RemovingDuplicatesLevel2.BusinessEntityID
        , RemovingDuplicatesLevel2.JobTitle
        , RemovingDuplicatesLevel2.EmployeeCode
        , ROW_NUMBER() OVER (PARTITION BY RemovingDuplicatesLevel2.EmployeeCode
                             ORDER BY RemovingDuplicatesLevel2.EmployeeCode ASC,
                                      RemovingDuplicatesLevel2.BusinessEntityID ASC) AS RowNumberEmployeeCode
    FROM RemovingDuplicatesLevel2
)
SELECT
    A.BusinessEntityID
    , A.JobTitle
    , A.OrganizationLevel
    , A.EmployeeCode
    , A.ManagerCode
    , B.BusinessEntityID  AS ManagerBusinessEntityID
    , B.JobTitle          AS ManagerJobTitle
FROM RemovingDuplicatesLevel2 AS A
LEFT JOIN (
    SELECT
        SelfJoinBwithDuplicates.BusinessEntityID
        , SelfJoinBwithDuplicates.JobTitle
        , SelfJoinBwithDuplicates.EmployeeCode
    FROM SelfJoinBwithDuplicates
    WHERE RowNumberEmployeeCode = 1
) AS B
ON A.ManagerCode = B.EmployeeCode
ORDER BY A.BusinessEntityID
```

---

### Output (truncated)

```
BusinessEntityID  JobTitle                           OrgLevel  EmployeeCode  ManagerCode  ManagerID  ManagerJobTitle
1                 Chief Executive Officer            NULL      NULL          NULL         NULL       NULL
2                 Vice President of Engineering      1         50001         50000        NULL       NULL
3                 Engineering Manager                2         50002         50001        2          Vice President of Engineering
4                 Senior Tool Designer               3         50003         50002        3          Engineering Manager
5                 Design Engineer                    3         50003         50002        3          Engineering Manager
6                 Design Engineer                    3         50003         50002        3          Engineering Manager
...
289               Sales Representative               3         60003         60002        17         Marketing Assistant
290               Sales Representative               3         60003         60002        17         Marketing Assistant
(289 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1 — `OriginalTablesLevel1`
Joins the three tables to retrieve each employee's department and organisation level. A `ROW_NUMBER()` window function is added to handle employees who have changed departments (which creates duplicate rows). Each group name is also converted into a numeric `GroupNameCode` using a `CASE` statement — this code is used later to calculate manager relationships mathematically.

### Query 2 — `RemovingDuplicatesLevel2`
Filters `RowNumberRemovingDuplicates = 1` to keep only the most recent department record per employee. Two new calculated columns are created:
- `EmployeeCode` = `GroupNameCode × 10000 + OrganizationLevel`
- `ManagerCode` = `GroupNameCode × 10000 + OrganizationLevel - 1`

This means every employee's `ManagerCode` matches their direct manager's `EmployeeCode` within the same group.

### Query 3 — `SelfJoinBwithDuplicates`
Since multiple employees can share the same `EmployeeCode`, a second `ROW_NUMBER()` is used to pick only one representative per code — this becomes the manager record used in the final join.

### Final SELECT
A `LEFT JOIN` matches each employee's `ManagerCode` to a manager's `EmployeeCode`, returning the manager's `BusinessEntityID` and `JobTitle` alongside the employee. `LEFT JOIN` ensures employees at the top of the hierarchy (with no manager) still appear in the results with `NULL` manager values.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
