# Removing repeating values from a column

## 🎯 Exercise
Display a list of employees grouped by department — but show each department name only once (on its first occurrence) rather than repeating it on every row.

**Before (department name repeats on every row):**
```
DepartmentName  FirstName  LastName
Engineering     Gail       Erickson
Engineering     Jossef     Goldberg
Engineering     Michael    Sullivan
Tool Design     Ovidiu     Cracium
Tool Design     Rob        Walters
```

**After (department name shown only once per group):**
```
NewDepartmentName  FirstName  LastName
Engineering        Gail       Erickson
                   Jossef     Goldberg
                   Michael    Sullivan
Tool Design        Ovidiu     Cracium
                   Rob        Walters
```

---

## 💡 Solution

### Approach
We use `LAG()` to retrieve the previous row's department name and compare it with the current row's department name using a `CASE` statement. If they are equal, we return an empty string `''`; if they differ, we return the department name. This creates a new column `NewDepartmentName` that shows each department name only on its first occurrence.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `LAG(DepartmentName) OVER (ORDER BY DepartmentName)` | Retrieves the department name from the previous row in alphabetical order |
| `CASE WHEN LAG(...) = DepartmentName THEN '' ELSE DepartmentName END` | Returns an empty string if the department name is the same as the previous row; otherwise returns the name |
| `ROW_NUMBER() OVER (PARTITION BY BusinessEntityID ORDER BY StartDate DESC)` | Removes duplicate employee records caused by department history changes |
| `ISNULL(value, '')` | Replaces `NULL` name values with an empty string |

### Tables used

| Schema | Table |
|---|---|
| `HumanResources` | `Employee` |
| `HumanResources` | `EmployeeDepartmentHistory` |
| `HumanResources` | `Department` |
| `Person` | `Person` |

---

### T-SQL code — Final query

```sql
SELECT
    UniqueDepartmentName.NewDepartmentName
  , UniqueDepartmentName.FirstName
  , UniqueDepartmentName.MiddleName
  , UniqueDepartmentName.LastName
FROM (
    SELECT
        RemovingDepartmentNames.DepartmentID
      , LAG(RemovingDepartmentNames.DepartmentName)
            OVER (ORDER BY RemovingDepartmentNames.DepartmentName) AS LAGDepartmentNumber
      , RemovingDepartmentNames.DepartmentName
      , CASE
            WHEN LAG(RemovingDepartmentNames.DepartmentName)
                OVER (ORDER BY RemovingDepartmentNames.DepartmentName)
                = RemovingDepartmentNames.DepartmentName
            THEN ''
            ELSE RemovingDepartmentNames.DepartmentName
        END AS NewDepartmentName
      , RemovingDepartmentNames.FirstName
      , RemovingDepartmentNames.MiddleName
      , RemovingDepartmentNames.LastName
    FROM (
        SELECT
            OriginalTables.BusinessEntityID
          , OriginalTables.DepartmentID
          , OriginalTables.DepartmentName
          , OriginalTables.FirstName
          , OriginalTables.MiddleName
          , OriginalTables.LastName
        FROM (
            SELECT
                ROW_NUMBER() OVER (PARTITION BY Employee.BusinessEntityID
                                   ORDER BY Employee.BusinessEntityID ASC,
                                            EmployeeDepartmentHistory.StartDate DESC) AS RowNumberRemovingDuplicates
              , Employee.BusinessEntityID
              , EmployeeDepartmentHistory.DepartmentID
              , Department.[Name]              AS DepartmentName
              , ISNULL(Person.FirstName, '')   AS FirstName
              , ISNULL(Person.MiddleName, '')  AS MiddleName
              , ISNULL(Person.LastName, '')    AS LastName
            FROM [AdventureWorks2022].[HumanResources].[Employee] AS Employee
            LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
                ON Employee.BusinessEntityID = EmployeeDepartmentHistory.BusinessEntityID
            LEFT JOIN [AdventureWorks2022].[HumanResources].[Department] AS Department
                ON EmployeeDepartmentHistory.DepartmentID = Department.DepartmentID
            LEFT JOIN [AdventureWorks2022].[Person].[Person] AS Person
                ON Employee.BusinessEntityID = Person.BusinessEntityID
            WHERE Employee.BusinessEntityID <> 1
        ) AS OriginalTables
        WHERE OriginalTables.RowNumberRemovingDuplicates = 1
    ) AS RemovingDepartmentNames
) AS UniqueDepartmentName
ORDER BY UniqueDepartmentName.DepartmentID, UniqueDepartmentName.FirstName
```

---

### Output (truncated)

```
NewDepartmentName       FirstName   MiddleName  LastName
Engineering             Gail        A           Erickson
                        Jossef      H           Goldberg
                        Michael     I           Sullivan
...
                        Terri       Lee         Duffy
                        Janice      M           Galvin      ← last in Engineering
Tool Design             Ovidiu      V           Cracium
                        Rob                     Walters
                        Thierry     B           D'Hers
                        Amy         E           Alberts     ← last in Tool Design
Sales                   Brian       S           Welcker
                        David       R           Campbell
                        Garrett     R           Vargas
...
                        Syed        E           Abbas
                        Tete        A           Mensa-Annan
                        Tsvi        Michael     Reiter      ← last in Sales
...
Shipping and Receiving  Pilar       G           Ackerman
                        Susan       W           Eaton
                        Vamsi       N           Kuppa       ← last in Shipping and Receiving
Executive               Laura       F           Norman      ← only 1 employee in Executive
(289 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Remove duplicate employee records
Some employees have changed departments, creating multiple rows per employee in `EmployeeDepartmentHistory`. We use `ROW_NUMBER()` partitioned by `BusinessEntityID` ordered by `StartDate DESC` to keep only the most recent department per employee. `ISNULL()` replaces any `NULL` name values with empty strings.

**Output:** 289 rows — one per employee with their current department name and full name.

```
BusinessEntityID  DepartmentID  DepartmentName            FirstName  MiddleName  LastName
2                 1             Engineering               Terri      Lee         Duffy
3                 1             Engineering               Roberto                Tamburello
4                 2             Tool Design               Rob                    Walters
5                 1             Engineering               Gail       A           Erickson
6                 1             Engineering               Jossef     H           Goldberg
7                 6             Research and Development  Dylan      A           Miller
...
(289 rows affected)
```

---

### Query 1.2 — Show department name only on its first occurrence
We add `LAG(DepartmentName) OVER (ORDER BY DepartmentName)` to retrieve the previous row's department name in alphabetical order. The `CASE` statement then compares it to the current row:

- If `LAG = DepartmentName` → the department name is **repeating** → return `''`
- If `LAG ≠ DepartmentName` (or `NULL` for the first row) → the department name is **new** → return the name

**How `LAG()` drives the logic — example for Engineering and Executive:**

```
LAGDepartmentNumber       DepartmentName   NewDepartmentName   FirstName
NULL                      Document Control Document Control    Zainal        ← first row, no LAG
Document Control          Document Control (empty)             Tengiz        ← same as LAG
Document Control          Document Control (empty)             Sean          ← same as LAG
Document Control          Engineering      Engineering         Gail          ← LAG differs → show name
Engineering               Engineering      (empty)             Roberto       ← same as LAG
Engineering               Executive        Executive           Laura         ← LAG differs → show name
Executive                 Facilities...    Facilities...       Gary          ← LAG differs → show name
...
```

### Final step — clean up and sort
We remove the intermediate `LAGDepartmentNumber` and `DepartmentName` columns from the output and sort by `DepartmentID` then `FirstName` to produce the final grouped presentation.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
