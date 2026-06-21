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

### T-SQL code — Full solution

```sql
USE AdventureWorks2022;
GO

WITH 
OriginalTablesLevel1 AS                                              -- CTE 1: OriginalTablesLevel1
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
RemovingDuplicatesLevel2 AS                                              -- CTE 2: RemovingDuplicatesLevel2 
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
ORDER BY A.BusinessEntityID;
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

## 📝 Note

> This solution does not rely on the `.GetAncestor()` method.
>
> `.GetAncestor()` is a built-in method used with the `hierarchyid` data type to navigate up a tree structure and return the `hierarchyid` of a node's ancestor at a specified level.
>
> For further information visit: [GetAncestor (Database Engine) — Microsoft documentation](https://learn.microsoft.com/en-us/sql/t-sql/data-types/getancestor-database-engine?view=sql-server-ver17)

---
<br>

## 🔍 Step-by-step explanation

### Query 1 — `OriginalTablesLevel1`
Joins the three tables to retrieve each employee's department and organisation level. A `ROW_NUMBER()` window function is added to handle employees who have changed departments (which creates duplicate rows). Each group name is also converted into a numeric `GroupNameCode` using a `CASE` statement — this code is used later to calculate manager relationships mathematically.

**T-SQL code of CTE 1**

```
SELECT											        -- OriginalTablesLevel1
ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID	ORDER BY Employee.BusinessEntityID ASC, EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
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
WHERE Employee.BusinessEntityID	<> 234					-- OriginalTablesLevel1
```

---

**Output of CTE 1 (truncated)**

```
RowNumberRemovingDuplicates		BusinessEntityID	GroupName								GroupNameCode	DepartmentID	DeparmentName				JobTitle							OrganizationLevel	StartDate	EndDate
1								1					Executive General and Administration	1				16				Executive					Chief Executive Officer				NULL				2009-01-14	NULL
1								2					Research and Development				5				1				Engineering					Vice President of Engineering		1					2008-01-31	NULL
1								3					Research and Development				5				1				Engineering					Engineering Manager					2					2007-11-11	NULL
1								4					Research and Development				5				2				Tool Design					Senior Tool Designer				3					2010-05-31	NULL
2								4					Research and Development				5				1				Engineering					Senior Tool Designer				3					2007-12-05	2010-05-30
1								5					Research and Development				5				1				Engineering					Design Engineer						3					2008-01-06	NULL
1								6					Research and Development				5				1				Engineering					Design Engineer						3					2008-01-24	NULL
1								7					Research and Development				5				6				Research and Development	Research and Development Manager	3					2009-02-08	NULL
1								8					Research and Development				5				6				Research and Development	Research and Development Engineer	4					2008-12-29	NULL
1								9					Research and Development				5				6				Research and Development	Research and Development Engineer	4					2009-01-16	NULL
1								10					Research and Development				5				6				Research and Development	Research and Development Manager	4					2009-05-03	NULL
1								11					Research and Development				5				2				Tool Design					Senior Tool Designer				3					2010-12-05	NULL
1								12					Research and Development				5				2				Tool Design					Tool Designer						4					2007-12-11	NULL
...
1								288					Sales and Marketing						6				3				Sales						Sales Representative				3					2013-05-30	NULL
1								289					Sales and Marketing						6				3				Sales						Sales Representative				3					2012-05-30	NULL
1								290					Sales and Marketing						6				3				Sales						Sales Representative				3					2012-05-30	NULL
(294 rows affected)
```

---

### Query 2 — `RemovingDuplicatesLevel2`
Filters `RowNumberRemovingDuplicates = 1` to keep only the most recent department record per employee. Two new calculated columns are created:
- `EmployeeCode` = `GroupNameCode × 10000 + OrganizationLevel`
- `ManagerCode` = `GroupNameCode × 10000 + OrganizationLevel - 1`

This means every employee's `ManagerCode` matches their direct manager's `EmployeeCode` within the same group.

**T-SQL code of CTE 2**

```
WITH OriginalTablesLevel1 AS
(
SELECT														-- OriginalTablesLevel1
ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID	ORDER BY Employee.BusinessEntityID ASC, EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
, Employee.BusinessEntityID	
, Department.GroupName
, GroupNameCode = CASE Department.GroupName
	WHEN 'Executive General and Administration' 	THEN 1
	WHEN 'Inventory Management' 					THEN 2
	WHEN 'Manufacturing' 							THEN 3
	WHEN 'Quality Assurance' 						THEN 4
	WHEN 'Research and Development' 				THEN 5
	WHEN 'Sales and Marketing' 						THEN 6
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
WHERE Employee.BusinessEntityID	<> 234						-- OriginalTablesLevel1
)
SELECT 
	OriginalTablesLevel1.BusinessEntityID							-- RemovingDuplicatesLevel2
	, OriginalTablesLevel1.GroupNameCode
	, OriginalTablesLevel1.GroupName
	, OriginalTablesLevel1.DepartmentID
	, OriginalTablesLevel1.DeparmentName
	, OriginalTablesLevel1.JobTitle
	, OriginalTablesLevel1.OrganizationLevel
	, EmployeeCode = OriginalTablesLevel1.GroupNameCode*10000 + OriginalTablesLevel1.OrganizationLevel
	, ManagerCode = OriginalTablesLevel1.GroupNameCode*10000 + OriginalTablesLevel1.OrganizationLevel - 1
FROM OriginalTablesLevel1
WHERE OriginalTablesLevel1.RowNumberRemovingDuplicates = 1			-- RemovingDuplicatesLevel2
```
---

**Output of CTE 2 (truncated)**

```
BusinessEntityID	GroupNameCode	GroupName								DepartmentID	DeparmentName				JobTitle								OrganizationLevel	EmployeeCode	ManagerCode
1					1				Executive General and Administration	16				Executive					Chief Executive Officer					NULL				NULL			NULL
2					5				Research and Development				1				Engineering					Vice President of Engineering			1					50001			50000
3					5				Research and Development				1				Engineering					Engineering Manager						2					50002			50001
4					5				Research and Development				2				Tool Design					Senior Tool Designer					3					50003			50002
5					5				Research and Development				1				Engineering					Design Engineer							3					50003			50002
6					5				Research and Development				1				Engineering					Design Engineer							3					50003			50002
7					5				Research and Development				6				Research and Development	Research and Development Manager		3					50003			50002
8					5				Research and Development				6				Research and Development	Research and Development Engineer		4					50004			50003
9					5				Research and Development				6				Research and Development	Research and Development Engineer		4					50004			50003
10					5				Research and Development				6				Research and Development	Research and Development Manager		4					50004			50003
11					5				Research and Development				2				Tool Design					Senior Tool Designer					3					50003			50002
12					5				Research and Development				2				Tool Design					Tool Designer							4					50004			50003
13					5				Research and Development				2				Tool Design					Tool Designer							4					50004			50003
14					5				Research and Development				1				Engineering					Senior Design Engineer					3					50003			50002
15					5				Research and Development				1				Engineering					Design Engineer							3					50003			50002
16					6				Sales and Marketing						4				Marketing					Marketing Manager						1					60001			60000
17					6				Sales and Marketing						4				Marketing					Marketing Assistant						2					60002			60001
18					6				Sales and Marketing						4				Marketing					Marketing Specialist					2					60002			60001
...
273					6				Sales and Marketing						3				Sales						Vice President of Sales					1					60001			60000
274					6				Sales and Marketing						3				Sales						North American Sales Manager			2					60002			60001
275					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
276					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
...
285					6				Sales and Marketing						3				Sales						Pacific Sales Manager					2					60002			60001
286					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
287					6				Sales and Marketing						3				Sales						European Sales Manager					2					60002			60001
288					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
289					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
290					6				Sales and Marketing						3				Sales						Sales Representative					3					60003			60002
(289 rows affected)
```

---

### Query 3 — `SelfJoinBwithDuplicates`
Since multiple employees can share the same `EmployeeCode`, a second `ROW_NUMBER()` is used to pick only one representative per code — this becomes the manager record used in the final join.

**T-SQL code of CTE 3**

```
WITH OriginalTablesLevel1 AS
(
	SELECT													-- OriginalTablesLevel1
	ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID	ORDER BY Employee.BusinessEntityID ASC, EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
	, Employee.BusinessEntityID	
	, Department.GroupName
	, GroupNameCode = CASE Department.GroupName
		WHEN 'Executive General and Administration' THEN 1
		WHEN 'Inventory Management' THEN 2
		WHEN 'Manufacturing' THEN 3
		WHEN 'Quality Assurance' THEN 4
		WHEN 'Research and Development' THEN 5
		WHEN 'Sales and Marketing' THEN 6
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
	WHERE Employee.BusinessEntityID	<> 234					-- OriginalTablesLevel1
),
RemovingDuplicatesLevel2 AS
(
	SELECT 
		OriginalTablesLevel1.BusinessEntityID						-- RemovingDuplicatesLevel2
		, OriginalTablesLevel1.GroupNameCode
		, OriginalTablesLevel1.GroupName
		, OriginalTablesLevel1.DepartmentID
		, OriginalTablesLevel1.DeparmentName
		, OriginalTablesLevel1.JobTitle
		, OriginalTablesLevel1.OrganizationLevel
		, EmployeeCode = OriginalTablesLevel1.GroupNameCode*10000 + OriginalTablesLevel1.OrganizationLevel
		, ManagerCode = OriginalTablesLevel1.GroupNameCode*10000 + OriginalTablesLevel1.OrganizationLevel - 1
	FROM OriginalTablesLevel1
	WHERE OriginalTablesLevel1.RowNumberRemovingDuplicates = 1		-- RemovingDuplicatesLevel2
		--AND OriginalTablesLevel1.GroupNameCode <> 3
)
	SELECT											-- ScalarSubqueryReturnManagerLevel3
		RemovingDuplicatesLevel2.BusinessEntityID
		, RemovingDuplicatesLevel2.JobTitle
		, RemovingDuplicatesLevel2.EmployeeCode
		, ROW_NUMBER() OVER (PARTITION BY RemovingDuplicatesLevel2.EmployeeCode ORDER BY RemovingDuplicatesLevel2.EmployeeCode ASC, RemovingDuplicatesLevel2.BusinessEntityID ASC) AS RowNumberEmployeeCode
	FROM RemovingDuplicatesLevel2					-- ScalarSubqueryReturnManagerLevel3
```
---

**Output of CTE 3 (truncated)**

```
BusinessEntityID	JobTitle							EmployeeCode	RowNumberEmployeeCode
1					Chief Executive Officer				NULL			1
2					Vice President of Engineering		50001			1
3					Engineering Manager					50002			1
4					Senior Tool Designer				50003			1
5					Design Engineer						50003			2
6					Design Engineer						50003			3
7					Research and Development Manager	50003			4
8					Research and Development Engineer	50004			1
9					Research and Development Engineer	50004			2
10					Research and Development Manager	50004			3
11					Senior Tool Designer				50003			5
12					Tool Designer						50004			4
13					Tool Designer						50004			5
14					Senior Design Engineer				50003			6
15					Design Engineer						50003			7
16					Marketing Manager					60001			1
17					Marketing Assistant					60002			1
18					Marketing Specialist				60002			2
...			
273					Vice President of Sales				60001			2
274					North American Sales Manager		60002			9
275					Sales Representative				60003			1
276					Sales Representative				60003			2
...			
285					Pacific Sales Manager				60002			10
286					Sales Representative				60003			11
287					European Sales Manager				60002			11
288					Sales Representative				60003			12
289					Sales Representative				60003			13
290					Sales Representative				60003			14
(289 rows affected)
```

---


### Final SELECT
A `LEFT JOIN` matches each employee's `ManagerCode` to a manager's `EmployeeCode`, returning the manager's `BusinessEntityID` and `JobTitle` alongside the employee. `LEFT JOIN` ensures employees at the top of the hierarchy (with no manager) still appear in the results with `NULL` manager values.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
