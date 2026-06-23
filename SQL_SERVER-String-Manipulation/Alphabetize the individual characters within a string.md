# Alphabetize the individual characters within a string

## 🎯 Exercise
For each unique first name in the database, return a new version of that name where all its individual characters are sorted in alphabetical order.

For example: `ABRAHAM` → `AAABHMR`

---

## 💡 Solution 1 — Using nested subqueries

### Approach
We use a **Cartesian Product** combined with `SUBSTRING()` to split each first name into individual character rows. We then apply a `WHERE` clause to remove empty positions beyond each name's length, and finally use `STRING_AGG()` with `WITHIN GROUP (ORDER BY)` to reassemble the characters back into a single alphabetically sorted string.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `REPLACE()` | Removes spaces and hyphens from first names |
| `UPPER()` | Converts all characters to uppercase |
| `SELECT DISTINCT` | Returns only unique first names |
| `ROW_NUMBER() OVER (ORDER BY)` | Generates sequential position numbers for iteration |
| `SUBSTRING()` | Extracts one character at a time by position |
| `LEN()` | Returns the length of each name to limit iteration |
| `STRING_AGG() WITHIN GROUP (ORDER BY)` | Concatenates characters back in alphabetical order |

### Table used

| Schema | Table |
|---|---|
| `Person` | `Person` |

---

### T-SQL code — Full solution

```sql
SELECT
    X.FirstNames AS OriginalFirstName
  , STRING_AGG(X.CharactersFirstNameNoSpaces, '')
      WITHIN GROUP (ORDER BY X.CharactersFirstNameNoSpaces) AS NewFirstName
FROM (
    SELECT
        UniqueFirstName.DistinctFirstNameNoSpaces AS FirstNames
      , SUBSTRING(UniqueFirstName.DistinctFirstNameNoSpaces, Iteration.Position, 1) AS CharactersFirstNameNoSpaces
    FROM (
        SELECT DISTINCT
            REPLACE(REPLACE(UPPER(FirstName), ' ', ''), '-', '') AS DistinctFirstNameNoSpaces
        FROM [AdventureWorks2022].[Person].[Person] AS UniqueFirstNameNoSpaces
    ) AS UniqueFirstName,
    (
        SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(REPLACE(UPPER(FirstName), ' ', ''), '-', ''))) AS Position
        FROM [AdventureWorks2022].[Person].[Person]
    ) AS Iteration
    WHERE Iteration.Position <= LEN(REPLACE(UniqueFirstName.DistinctFirstNameNoSpaces, ' ', ''))
) AS X
GROUP BY X.FirstNames
ORDER BY X.FirstNames
```

---

### Output (truncated)

```
OriginalFirstName  NewFirstName
A.                 .A
A.SCOTT            .ACOSTT
AARON              AANOR
ABBY               ABBY
ABE                ABE
ABHIJIT            ABHIIJT
...
FRANÇOIS           AÇFINORS
...
DEENA              ADEEN
DEEPAK             ADEEKP
...
ZACHARY            AACHRYZ
ZAINAL             AAILNZ
ZHENG              EGHNZ
ZOE                EOZ
(1018 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Get unique first names with no spaces or hyphens
We use `UPPER()` to capitalise all characters, then `REPLACE()` twice to remove spaces and hyphens. `SELECT DISTINCT` removes duplicate names, leaving only 1 row per unique first name.

**T-SQL code of Query 1.1**
```sql
SELECT REPLACE(REPLACE(UPPER(FirstName), ' ',''),'-','') AS FirstNameNoSpaces            -- UniqueFirstNameNoSpaces1
FROM [AdventureWorks2022].[Person].[Person] AS UniqueFirstNameNoSpaces                   -- UniqueFirstNameNoSpaces1
```

**Output of Query 1.1:** 1,018 unique first names (truncated).
```
FirstNameNoSpaces
A.
A.SCOTT
AARON
ABBY
...
YVONNE
ZACHARY
ZOE
(1018 rows affected)
```

---

### Query 1.2 — Generate all character positions using Cartesian Product
We create a Cartesian Product between the 1,018 unique first names and an `Iteration` subquery that generates 19,972 sequential position numbers (one per row in the `Person` table). For each combination, `SUBSTRING()` extracts the character at that position.

The Cartesian Product creates: **1,018 × 19,972 = 20,331,496 rows**.

At this stage the result includes many empty characters (positions beyond the name's length), which appear as blank values.

**T-SQL code of Query 1.2**
```sql
SELECT UniqueFirstName.FirstNameNoSpaces AS UniqueFirstName                                       -- UniqueFirstName2
, SUBSTRING(UniqueFirstName.FirstNameNoSpaces, Iteration.Position, 1) AS FirstCharacterUniqueFirstName
FROM (
	SELECT DISTINCT REPLACE(REPLACE(UPPER(FirstName), ' ',''),'-','') AS FirstNameNoSpaces        -- UniqueFirstNameNoSpaces1
	FROM [AdventureWorks2022].[Person].[Person] AS UniqueFirstNameNoSpaces                        -- UniqueFirstNameNoSpaces1
	) AS UniqueFirstName,	
	(
	SELECT ROW_NUMBER() OVER(ORDER BY LEN(REPLACE(REPLACE(UPPER(FirstName), ' ',''),'-',''))) AS Position        -- Iteration1
	FROM [AdventureWorks2022].[Person].[Person]                                                                  -- Iteration1
	) AS Iteration                                                                                -- UniqueFirstName2
```

**Output of Query 1.2:** 20,331,496 rows (truncated).
```
Row #		UniqueFirstName        FirstCharacterUniqueFirstName
1			A.			           A
2			A.			           .        
3			A.			           (emt
4			A.                     			
5			A.                     			
.................................................................. TRUNCATED RESULTS .....
19971		A.	
19972		A.	
19973		A.SCOTT			A
19974		A.SCOTT			.
19975		A.SCOTT			S
19976		A.SCOTT			C
19977		A.SCOTT			O
19978		A.SCOTT			T
19979		A.SCOTT			T
19980		A.SCOTT	
19981		A.SCOTT	
.................................................................. TRUNCATED RESULTS .....
39944		A.SCOTT			
39945		AARON			A
39946		AARON			A
39947		AARON			R
39948		AARON			O
39949		AARON			N
39950		AARON	
.................................................................. TRUNCATED RESULTS .....
(20331496 rows affected)






UniqueFirstName  FirstCharacterUniqueFirstName
A.               A
A.               .
A.               (empty)
A.               (empty)
...
A.SCOTT          A
A.SCOTT          .
A.SCOTT          S
A.SCOTT          C
A.SCOTT          O
A.SCOTT          T
A.SCOTT          T
A.SCOTT          (empty)
...
(20331496 rows affected)
```

---

### Query 1.2.1 — Filter out empty positions using `WHERE`
We add a `WHERE` clause that keeps only rows where `Iteration.Position` is less than or equal to the length of the name. This removes all empty character positions.

**Output:** 5,880 rows — only the valid characters for all 1,018 names.

```
FirstName  CharactersFirstNameNoSpaces
A.         A
A.         .
A.SCOTT    A
A.SCOTT    .
A.SCOTT    S
A.SCOTT    C
A.SCOTT    O
A.SCOTT    T
A.SCOTT    T
AARON      A
AARON      A
AARON      R
AARON      O
AARON      N
...
ZOE        Z
ZOE        O
ZOE        E
(5880 rows affected)
```

---

### Final Query (Query 1.3) — Reassemble characters alphabetically using `STRING_AGG()`
We group all character rows by their original first name and use `STRING_AGG()` with `WITHIN GROUP (ORDER BY CharactersFirstNameNoSpaces)` to concatenate the characters back into a single string — sorted alphabetically. The separator is set to `''` (empty string) so no separator appears between characters.

**Final output:** 1,018 rows — each original first name alongside its alphabetised version.

```
OriginalFirstName  NewFirstName
ABRAHAM            AAABHMR
ADINA              AADIN
ADRIENNE           ADEEINNR
AIDAN              AADIN
AIMEE              AEEIM
ABE                ABE
(1018 rows affected)
```

---

## 💡 Solution 2 — Using CTEs (alternative approach)

This solution produces the same result as Solution 1 but uses **Common Table Expressions (CTEs)** instead of nested subqueries, making each step easier to read and follow individually.

```sql
WITH
UniqueFirstName AS
(
    SELECT DISTINCT
        REPLACE(REPLACE(UPPER(FirstName), ' ', ''), '-', '') AS DistinctFirstNameNoSpaces
    FROM [AdventureWorks2022].[Person].[Person] AS UniqueFirstNameNoSpaces
),
Iteration AS
(
    SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(REPLACE(UPPER(FirstName), ' ', ''), '-', ''))) AS Position
    FROM [AdventureWorks2022].[Person].[Person]
),
IteratedUniqueFirstName AS
(
    SELECT
        UniqueFirstName.DistinctFirstNameNoSpaces AS FirstNames
      , SUBSTRING(UniqueFirstName.DistinctFirstNameNoSpaces, Iteration.Position, 1) AS CharactersFirstNameNoSpaces
    FROM UniqueFirstName, Iteration
    WHERE Iteration.Position <= LEN(REPLACE(UniqueFirstName.DistinctFirstNameNoSpaces, ' ', ''))
)
SELECT
    FirstNames
  , STRING_AGG(CharactersFirstNameNoSpaces, '')
      WITHIN GROUP (ORDER BY CharactersFirstNameNoSpaces) AS NewFirstName
FROM IteratedUniqueFirstName
GROUP BY FirstNames
ORDER BY FirstNames
```

**Output:** Identical to Solution 1 — 1,018 rows affected.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
