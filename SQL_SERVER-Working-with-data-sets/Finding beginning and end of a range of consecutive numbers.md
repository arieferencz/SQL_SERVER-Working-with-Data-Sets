# Finding beginning and end of a range of consecutive numbers

## 🎯 Exercise
Group employees together based on whether they share the same start date in their current department. Each unique date forms a new group — employees hired on the same date belong to the same group, and each new date marks the start of the next group.

---

## 📝 Note

> Since some employees have changed departments over time, this exercise uses column `StartDate` from table `EmployeeDepartmentHistory` — understood here as the date the employee started in their **current** department, not their original hire date.

---

## 💡 Solution

### Approach
We build the solution in 4 steps:
1. Remove duplicate employee records caused by department history changes
2. Use `LAG()` and `DATEDIFF()` to calculate the number of days between each consecutive start date
3. Use a `CASE` statement to flag rows where the date changes (`1` = new group, `0` = same group)
4. Use a running `SUM()` window function to convert those flags into a cumulative group number

### T-SQL functions and case expressions used

| Function | Purpose |
|---|---|
| `ROW_NUMBER() OVER (PARTITION BY ... ORDER BY ...)` | Removes duplicate employee records by numbering them per employee |
| `ROW_NUMBER() OVER (ORDER BY StartDate)` | Assigns a sequential row number to all employees ordered by start date |
| `LAG(StartDate) OVER (ORDER BY StartDate)` | Returns the previous row's start date for comparison |
| `DATEDIFF(DAY, previous, current)` | Calculates the number of days between consecutive start dates |
| `CASE WHEN DatediffStartDate = 0 THEN 0 ELSE 1 END` | Flags `0` for same-date rows and `1` for new-date rows |
| `SUM(StartDateGroup) OVER (ORDER BY RowNumber)` | Converts the `0/1` flags into a cumulative group number using a running total |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |

---

### T-SQL code — Final query

```sql
SELECT
    StartDateGroupNumber.RowNumber
  , StartDateGroupNumber.StartDate
  , StartDateGroupNumber.BusinessEntityID
  , StartDateGroupNumber.JobTitle
  , SUM(StartDateGroupNumber.StartDateGroup)
      OVER (ORDER BY StartDateGroupNumber.RowNumber) AS NewGroup
FROM (
    SELECT
        DatediffHireDates.RowNumber
      , DatediffHireDates.PreviousStartDate
      , DatediffHireDates.StartDate
      , DatediffHireDates.DatediffStartDate
      , CASE
            WHEN DatediffHireDates.DatediffStartDate = 0 THEN 0
            ELSE 1
        END AS StartDateGroup
      , DatediffHireDates.BusinessEntityID
      , DatediffHireDates.JobTitle
    FROM (
        SELECT
            ROW_NUMBER() OVER (ORDER BY OriginalTables.StartDate) AS RowNumber
          , LAG(OriginalTables.StartDate) OVER (ORDER BY OriginalTables.StartDate) AS PreviousStartDate
          , OriginalTables.StartDate
          , DATEDIFF(DAY,
              LAG(OriginalTables.StartDate) OVER (ORDER BY OriginalTables.StartDate),
              OriginalTables.StartDate) AS DatediffStartDate
          , OriginalTables.BusinessEntityID
          , OriginalTables.JobTitle
          , OriginalTables.DepartmentID
          , OriginalTables.DeparmentName
        FROM (
            SELECT
                ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                                   ORDER BY Employee.BusinessEntityID ASC,
                                            EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
              , Employee.BusinessEntityID
              , Employee.JobTitle
              , EmployeeDepartmentHistory.DepartmentID
              , Department.[Name] AS DeparmentName
              , EmployeeDepartmentHistory.StartDate
              , EmployeeDepartmentHistory.EndDate
            FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
            LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
                ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
            LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
                ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
            WHERE Employee.BusinessEntityID <> 1
        ) AS OriginalTables
        WHERE OriginalTables.RowNumberRemovingDuplicates = 1
    ) AS DatediffHireDates
) AS StartDateGroupNumber
```

---

### Output (truncated)

```
RowNumber  StartDate   BusinessEntityID  JobTitle                        NewGroup
1          2006-06-30  28                Production Technician - WC60    1
2          2007-01-26  17                Marketing Assistant             2
3          2007-11-11  3                 Engineering Manager             3
4          2007-12-11  12                Tool Designer                   4
5          2007-12-26  40                Production Supervisor - WC60    5
6          2008-01-06  48                Production Technician - WC10    6
7          2008-01-06  5                 Design Engineer                 6   ← same date as row 6
8          2008-01-07  49                Production Technician - WC10    7
9          2008-01-24  6                 Design Engineer                 8
10         2008-01-31  2                 Vice President of Engineering   9
...
271        2011-05-31  275               Sales Representative            158
272        2011-05-31  276               Sales Representative            158
273        2011-05-31  277               Sales Representative            158
274        2011-05-31  278               Sales Representative            158
275        2011-05-31  279               Sales Representative            158
276        2011-05-31  280               Sales Representative            158
277        2011-05-31  281               Sales Representative            158
278        2011-05-31  282               Sales Representative            158
279        2011-05-31  283               Sales Representative            158  ← 9 employees in group 158
280        2011-09-01  224               Scheduling Assistant            159
...
287        2013-05-30  286               Sales Representative            165
288        2013-05-30  288               Sales Representative            165
289        2013-11-14  234               Chief Financial Officer         166
(289 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Remove duplicate employee records
Some employees have multiple rows in `EmployeeDepartmentHistory` due to department changes. We use `ROW_NUMBER()` partitioned by `BusinessEntityID` ordered by `StartDate DESC` to keep only the most recent record per employee. The outer `WHERE RowNumberRemovingDuplicates = 1` filters to 289 unique employees ordered by `StartDate`.

**Output:** 289 rows — one per employee with their most recent start date.

```
RowNumber  BusinessEntityID  JobTitle                      DepartmentID  DepartmentName  StartDate
1          28                Production Technician - WC60  7             Production      2006-06-30
2          17                Marketing Assistant            4             Marketing       2007-01-26
3          3                 Engineering Manager            1             Engineering     2007-11-11
...
289        234               Chief Financial Officer        16            Executive       2013-11-14
(289 rows affected)
```

---

### Query 1.2 — Calculate days between consecutive start dates
We add `LAG(StartDate)` to retrieve the previous row's start date, and `DATEDIFF(DAY, previous, current)` to calculate the number of days between them. A value of `0` means two employees share the same start date. The first row has `NULL` since there is no previous row.

**Output:** 289 rows — each with its previous start date and day difference (truncated).

```
RowNumber  PreviousStartDate  StartDate   DatediffStartDate  BusinessEntityID  JobTitle
1          NULL               2006-06-30  NULL               28                Production Technician - WC60
2          2006-06-30         2007-01-26  210                17                Marketing Assistant
3          2007-01-26         2007-11-11  289                3                 Engineering Manager
...
6          2007-12-26         2008-01-06  11                 48                Production Technician - WC10
7          2008-01-06         2008-01-06  0                  5                 Design Engineer
8          2008-01-06         2008-01-07  1                  49                Production Technician - WC10
...
271        2011-02-15         2011-05-31  105                275               Sales Representative
272        2011-05-31         2011-05-31  0                  276               Sales Representative
273        2011-05-31         2011-05-31  0                  277               Sales Representative
...
(289 rows affected)
```

---

### Query 1.3 — Flag new groups using `CASE`
We add a `CASE` statement on `DatediffStartDate`:
- `0` → same date as the previous row → same group → flag = `0`
- Any other value (or `NULL`) → new date → new group → flag = `1`

**Output:** 289 rows — each with a `StartDateGroup` flag of `0` or `1`.

```
RowNumber  PreviousStartDate  StartDate   DatediffStartDate  StartDateGroup  BusinessEntityID  JobTitle
1          NULL               2006-06-30  NULL               1               28                Production Technician - WC60
2          2006-06-30         2007-01-26  210                1               17                Marketing Assistant
...
6          2007-12-26         2008-01-06  11                 1               48                Production Technician - WC10
7          2008-01-06         2008-01-06  0                  0               5                 Design Engineer
8          2008-01-06         2008-01-07  1                  1               49                Production Technician - WC10
...
271        2011-02-15         2011-05-31  105                1               275               Sales Representative
272        2011-05-31         2011-05-31  0                  0               276               Sales Representative
273        2011-05-31         2011-05-31  0                  0               277               Sales Representative
...
(289 rows affected)
```

---

### Final Query (Query 1.4) — Convert flags to group numbers using running `SUM()`
`SUM(StartDateGroup) OVER (ORDER BY RowNumber)` computes a running total of the `0/1` flags. Each time a `1` appears, the running total increases by 1 — incrementing the group number. Each time a `0` appears, the total stays the same — keeping the same group number for employees sharing the same date.

**How the running sum builds group numbers:**

```
RowNumber  StartDateGroup  Running SUM  → NewGroup
1          1               1            → 1
2          1               2            → 2
3          1               3            → 3
...
6          1               6            → 6
7          0               6            → 6  ← same group as row 6
8          1               7            → 7
...
271        1               158          → 158
272        0               158          → 158  ← same group
273        0               158          → 158  ← same group
...
```

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
