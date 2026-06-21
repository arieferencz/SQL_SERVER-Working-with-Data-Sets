# Finding the manager for each employee (Parent-Child relationship) — using the .GetAncestor() method

## 🎯 Exercise
Find the manager for each employee based on the organisational hierarchy (Parent-Child relationship) — using the built-in `.GetAncestor()` method on the `hierarchyid` data type.

---

## 📝 Note

> This exercise solves the same problem as [Finding the manager for each employee (Parent-Child relationship)](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Hierarchical-Parent-Child-relationship/Finding%20the%20manager%20for%20each%20employee%20(Parent-Child%20relationship).md) — but uses the `.GetAncestor()` method instead of the `CEILING() + ROW_NUMBER()` approach used in that exercise.
>
> `.GetAncestor(n)` is a built-in method on the `hierarchyid` data type that navigates up a tree structure and returns the `hierarchyid` of a node's ancestor at `n` levels above the current node:
> - `.GetAncestor(0)` → returns the node itself (`OwnNode`)
> - `.GetAncestor(1)` → returns the direct parent node (`ManagerNode`)
>
> For further information visit: [GetAncestor (Database Engine) — Microsoft documentation](https://learn.microsoft.com/en-us/sql/t-sql/data-types/getancestor-database-engine?view=sql-server-ver17)

---

## 💡 Solution

### Approach
We build the solution using **3 CTEs** combined with a `UNION ALL`:

- **CTE 1 (`Management`)** — Retrieves the CEO and all employees at the first level directly under the CEO, using a `WHERE` clause that filters on `.GetAncestor(1) = 0x` (the root node). Manager name and title are left as `NULL` placeholders at this stage.
- **CTE 2 (`Management2`)** — Populates the `ManagerName` and `ManagerTitle` columns for the upper management rows. Since the CEO's `OwnNode` is not available in this database, we hardcode `'Ken J Sánchez'` as the manager name for direct reports whose `ManagerNode = 0x`.
- **CTE 3 (`Employees`)** — Retrieves all remaining employees (excluding upper management and the CEO) by using an `INNER JOIN` on `.GetAncestor(1)` — matching each employee's parent node to their manager's own node.
- **`UNION ALL`** — Combines `Management2` and `Employees` into a single result set covering all 290 employees.

### T-SQL methods and functions used

| Method / Function | Purpose |
|---|---|
| `.GetAncestor(0)` | Returns the employee's own `hierarchyid` node (`OwnNode`) |
| `.GetAncestor(1)` | Returns the direct parent node — the employee's manager (`ManagerNode`) |
| `CONCAT(FirstName, ' ', MiddleName, ' ', LastName)` | Builds the full name string for employees and managers |
| `CAST(NULL AS NVARCHAR(150))` | Creates a typed `NULL` placeholder for `ManagerName` |
| `CAST(NULL AS NVARCHAR(50))` | Creates a typed `NULL` placeholder for `ManagerTitle` |
| `CASE WHEN ManagerNode = 0x THEN 'Ken J Sánchez' ELSE NULL END` | Populates the CEO's name as manager for direct reports |
| `RIGHT JOIN` | Ensures all employees are returned even if no matching `Person` record exists |
| `INNER JOIN ... ON .GetAncestor(1) = ManagerNode` | Matches each employee to their direct manager by node |
| `UNION ALL` | Combines upper management and all other employees into one result set |
| `WHERE .GetAncestor(1) = 0x OR OrganizationNode IS NULL` | Filters to the CEO and direct reports only |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Person` |
| `HumanResources` | `Employee` |

---

### T-SQL code — Full solution

```sql
USE AdventureWorks2022;
GO

WITH Management AS                                              -- CTE 1: CEO + direct reports
(
    SELECT
        EmployeePerson.[BusinessEntityID] AS BusinessEntityID
      , CONCAT(EmployeePerson.[FirstName], ' ',
               EmployeePerson.[MiddleName], ' ',
               EmployeePerson.[LastName]) AS EmployeeName
      , EmployeeTitle.[JobTitle]          AS EmployeeTitle
      , EmployeeTitle.[OrganizationNode].GetAncestor(0) AS OwnNode
      , EmployeeTitle.[OrganizationNode].GetAncestor(1) AS ManagerNode
      , CAST(NULL AS NVARCHAR(150))                             AS ManagerName
      , CAST(NULL AS NVARCHAR(50))                              AS ManagerTitle
    FROM [AdventureWorks2022].[Person].[Person] AS EmployeePerson
    RIGHT JOIN [AdventureWorks2022].[HumanResources].[Employee] AS EmployeeTitle
        ON EmployeePerson.[BusinessEntityID] = EmployeeTitle.[BusinessEntityID]
    WHERE EmployeeTitle.[OrganizationNode].GetAncestor(1) = 0x
       OR EmployeeTitle.[OrganizationNode] IS NULL
),
Management2 AS                                                  -- CTE 2: Populate ManagerName and ManagerTitle
(
    SELECT
        Management.BusinessEntityID
      , Management.EmployeeName
      , Management.EmployeeTitle
      , CASE WHEN OwnNode IS NOT NULL
             THEN OwnNode
             ELSE CAST(NULL AS NVARCHAR(150))
        END                                   AS OwnNode
      , CASE WHEN ManagerNode IS NOT NULL
             THEN ManagerNode
             ELSE CAST(NULL AS NVARCHAR(150))
        END                                   AS ManagerNode
      , CASE WHEN ManagerNode = 0x
             THEN 'Ken J Sánchez'
             ELSE CAST(NULL AS NVARCHAR(150))
        END                                   AS ManagerName
      , CASE WHEN ManagerNode IS NULL
             THEN 'N/A'
             ELSE ''
        END                                   AS ManagerTitle
    FROM Management
),
Employees AS                                                    -- CTE 3: All remaining employees
(
    SELECT
        EmployeePerson.[BusinessEntityID] AS BusinessEntityID
      , CONCAT(EmployeePerson.[FirstName], ' ',
               EmployeePerson.[MiddleName], ' ',
               EmployeePerson.[LastName]) AS EmployeeName
      , EmployeeTitle.[JobTitle]          AS EmployeeTitle
      , EmployeeTitle.[OrganizationNode].GetAncestor(0) AS OwnNode
      , ManagerTitle.[OrganizationNode].GetAncestor(1)  AS ManagerNode
      , CONCAT(ManagerPerson.[FirstName], ' ',
               ManagerPerson.[MiddleName], ' ',
               ManagerPerson.[LastName]) AS ManagerName
      , ManagerTitle.[JobTitle]          AS ManagerTitle
    FROM [AdventureWorks2022].[Person].[Person] AS EmployeePerson
    RIGHT JOIN [AdventureWorks2022].[HumanResources].[Employee] AS EmployeeTitle
        ON EmployeePerson.[BusinessEntityID] = EmployeeTitle.[BusinessEntityID]
    INNER JOIN [AdventureWorks2022].[HumanResources].[Employee] AS ManagerTitle
        ON EmployeeTitle.[OrganizationNode].GetAncestor(1) = ManagerTitle.[OrganizationNode]
    LEFT JOIN [AdventureWorks2022].[Person].[Person] AS ManagerPerson
        ON ManagerTitle.[BusinessEntityID] = ManagerPerson.[BusinessEntityID]
)
SELECT BusinessEntityID, EmployeeName, EmployeeTitle, OwnNode, ManagerNode, ManagerName, ManagerTitle
FROM Management2
UNION ALL
SELECT BusinessEntityID, EmployeeName, EmployeeTitle, OwnNode, ManagerNode, ManagerName, ManagerTitle
FROM Employees;
```

---

### Output (truncated)

```
BusinessEntityID  EmployeeName                    EmployeeTitle                        OwnNode    ManagerNode    ManagerName                  ManagerTitle
1                 Ken J Sánchez                   Chief Executive Officer              NULL       NULL           NULL                         N/A
2                 Terri Lee Duffy                 Vice President of Engineering        0x58       0x             Ken J Sánchez
16                David M Bradley                 Marketing Manager                    0x68       0x             Ken J Sánchez
25                James R Hamilton                Vice President of Production         0x78       0x             Ken J Sánchez
234               Laura F Norman                  Chief Financial Officer              0x84       0x             Ken J Sánchez
263               Jean E Trenary                  Information Services Manager         0x8C       0x             Ken J Sánchez
273               Brian S Welcker                 Vice President of Sales              0x94       0x             Ken J Sánchez
3                 Roberto  Tamburello             Engineering Manager                  0x5AC0     0x             Terri Lee Duffy              Vice President of Engineering
4                 Rob  Walters                    Senior Tool Designer                 0x5AD6     0x58           Roberto Tamburello           Engineering Manager
5                 Gail A Erickson                 Design Engineer                      0x5ADA     0x58           Roberto Tamburello           Engineering Manager
6                 Jossef H Goldberg               Design Engineer                      0x5ADE     0x58           Roberto Tamburello           Engineering Manager
7                 Dylan A Miller                  Research and Development Manager     0x5AE1     0x58           Roberto Tamburello           Engineering Manager
8                 Diane L Margheim                Research and Development Engineer    0x5AE158   0x5AC0         Dylan A Miller               Research and Development Manager
9                 Gigi N Matthew                  Research and Development Engineer    0x5AE168   0x5AC0         Dylan A Miller               Research and Development Manager
10                Michael  Raheem                 Research and Development Manager     0x5AE178   0x5AC0         Dylan A Miller               Research and Development Manager
...
235               Paula M Barreto de Mattos       Human Resources Manager              0x8560     0x             Laura F Norman               Chief Financial Officer
236               Grant N Culbertson              HR Administrative Assistant          0x856B     0x84           Paula M Barreto de Mattos    Human Resources Manager
...
274               Stephen Y Jiang                 North American Sales Manager         0x9560     0x             Brian S Welcker              Vice President of Sales
275               Michael G Blythe                Sales Representative                 0x956B     0x94           Stephen Y Jiang              North American Sales Manager
...
285               Syed E Abbas                    Pacific Sales Manager                0x95A0     0x             Brian S Welcker              Vice President of Sales
286               Lynn N Tsoflias                 Sales Representative                 0x95AB     0x94           Syed E Abbas                 Pacific Sales Manager
287               Amy E Alberts                   European Sales Manager               0x95E0     0x             Brian S Welcker              Vice President of Sales
288               Rachel B Valdez                 Sales Representative                 0x95EB     0x94           Amy E Alberts                European Sales Manager
289               Jae B Pak                       Sales Representative                 0x95ED     0x94           Amy E Alberts                European Sales Manager
290               Ranjit R Varkey Chudukatil      Sales Representative                 0x95EF     0x94           Amy E Alberts                European Sales Manager
(290 rows affected)
```

---

## 🔍 Step-by-step explanation

### CTE 1 — `Management` (CEO + direct reports)

We join `Person` and `Employee` using a `RIGHT JOIN` to ensure all employees are returned even if no matching `Person` record exists. The `WHERE` clause filters to two groups:

- `OrganizationNode IS NULL` → the **CEO** (Ken J Sánchez) — his node is not stored in this database
- `.GetAncestor(1) = 0x` → employees whose direct parent is the **root node** (`0x`) — meaning they report directly to the CEO

`.GetAncestor(0)` returns the employee's own node (`OwnNode`). `.GetAncestor(1)` returns their parent node (`ManagerNode`). `ManagerName` and `ManagerTitle` are set as typed `NULL` placeholders — they will be populated in CTE 2.

**Output of CTE 1:** 7 rows — the CEO plus 6 direct reports.

```
BusinessEntityID  EmployeeName      EmployeeTitle                    OwnNode  ManagerNode  ManagerName  ManagerTitle
1                 Ken J Sánchez     Chief Executive Officer            NULL     NULL         NULL         NULL
2                 Terri Lee Duffy   Vice President of Engineering      0x58     0x           NULL         NULL
16                David M Bradley   Marketing Manager                  0x68     0x           NULL         NULL
25                James R Hamilton  Vice President of Production        0x78     0x           NULL         NULL
234               Laura F Norman    Chief Financial Officer             0x84     0x           NULL         NULL
263               Jean E Trenary    Information Services Manager        0x8C     0x           NULL         NULL
273               Brian S Welcker   Vice President of Sales             0x94     0x           NULL         NULL
(7 rows affected)
```

---

### CTE 2 — `Management2` (populate ManagerName and ManagerTitle)

We use `CASE` statements to fill in the `ManagerName` and `ManagerTitle` columns:

- For the **CEO** (`ManagerNode IS NULL`) → `ManagerTitle = 'N/A'` (no manager)
- For **direct reports** (`ManagerNode = 0x`) → `ManagerName = 'Ken J Sánchez'` (hardcoded because the CEO's `OwnNode` is unavailable in this database)
- All other values pass through unchanged

**Why is the CEO's `OwnNode` unavailable?**
The `OrganizationNode` column for `BusinessEntityID = 1` (Ken J Sánchez) is `NULL` in the AdventureWorks2022 database — his node was not populated in the sample data. This is why `.GetAncestor()` cannot be used to retrieve his name dynamically, and it must be hardcoded instead.

**Output of CTE 2:** Same 7 rows with `ManagerName` and `ManagerTitle` now populated.

```
BusinessEntityID  EmployeeName      ManagerName      ManagerTitle
1                 Ken J Sánchez     NULL             N/A
2                 Terri Lee Duffy   Ken J Sánchez
16                David M Bradley   Ken J Sánchez
25                James R Hamilton  Ken J Sánchez
234               Laura F Norman    Ken J Sánchez
263               Jean E Trenary    Ken J Sánchez
273               Brian S Welcker   Ken J Sánchez
(7 rows affected)
```

---

### CTE 3 — `Employees` (all remaining employees)

We use an `INNER JOIN` on `.GetAncestor(1)` to match each employee's parent node to their manager's own node:

```sql
INNER JOIN [HumanResources].[Employee] AS ManagerTitle
    ON EmployeeTitle.[OrganizationNode].GetAncestor(1) = ManagerTitle.[OrganizationNode]
```

This `INNER JOIN` automatically **excludes** the CEO (whose `OrganizationNode` is `NULL`) and the direct reports (whose parent is `0x` — the root, which has no matching row). It returns only employees who have a manager with a valid, matchable node.

A second `LEFT JOIN` to `Person` then retrieves the manager's full name.

**Output of CTE 3:** 283 rows — all employees excluding the CEO and his 6 direct reports.

---

### Final `UNION ALL` — combine both result sets

`UNION ALL` appends the 283 rows from `Employees` to the 7 rows from `Management2` — producing all 290 employees in a single result set, ordered from the top of the hierarchy downward.

> **Why `UNION ALL` instead of `UNION`?** `UNION` removes duplicate rows — which would be incorrect here since two employees could theoretically have identical column values. `UNION ALL` keeps all rows regardless, which is the correct behaviour for combining two non-overlapping employee sets.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
