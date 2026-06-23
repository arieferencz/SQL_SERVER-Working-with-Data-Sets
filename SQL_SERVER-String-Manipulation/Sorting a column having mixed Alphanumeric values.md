# Sorting a column having mixed AlphaNumeric values

## 🎯 Exercise
Given a column that contains mixed alphanumeric values (full names combined with phone numbers), sort the results either by the **numeric portion** (phone number digits) or by the **character portion** (full name) — independently of each other.

**Example of the mixed alphanumeric column:**
```
Ken J Sánchez 697-555-0142
Terri Lee Duffy 819-555-0175
Roberto Tamburello 212-555-0187
```

---

## 💡 Solution 1 — Using SUBSTRING and PATINDEX

### Approach
We build the mixed alphanumeric column `AlphaNumericText` by concatenating each person's full name with their phone number. We then use `PATINDEX()` to locate the first digit in the string and `SUBSTRING()` to extract the numeric or character portions for use in `ORDER BY`.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `PATINDEX('%[0-9]%', string)` | Finds the position of the first digit in the string |
| `SUBSTRING(string, start, length)` | Extracts the numeric or character portion for sorting |
| `COALESCE(value, replacement)` | Replaces `NULL` middle names with an empty string |
| `REPLACE()` | Removes extra spaces from full names and phone numbers |
| `LTRIM()` | Removes leading spaces from phone numbers |
| `LEN()` | Returns string length — used to define sort boundaries |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Person` |
| `Person` | `PersonPhone` |

---

### T-SQL code — Full solution: Build the mixed alphanumeric column and sorting columns

```sql
SELECT
    X.FirstName
  , X.MiddleName
  , X.LastName
  , X.FullName
  , X.PhoneNumber
  , X.AlphaNumericText
  , PATINDEX('%[0-9]%', AlphaNumericText) AS NumStartPosition
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText), 3) AS SortFirst3Numbers
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText) + 4, 3) AS SortSecond3Numbers
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText) + 8, 4) AS SortLast4Numbers
  , SUBSTRING(X.FullName, 1, LEN(AlphaNumericText)) AS SortCharPortion
FROM (
    SELECT
        E.FirstName
      , CASE E.MiddleName WHEN NULL THEN '' ELSE E.MiddleName END AS MiddleName
      , E.LastName
      , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' + E.LastName,
          ' ', '<>'), '><', ''), '<>', ' ') AS FullName
      , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' + E.LastName,
          ' ', '<>'), '><', ''), '<>', ' ') + ' ' +
        REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1,
          LEN(PN.PhoneNumber))), ' ', '-')  AS AlphaNumericText
      , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1,
          LEN(PN.PhoneNumber))), ' ', '-')  AS PhoneNumber
    FROM [AdventureWorks2022].[Person].[Person] AS E
    INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
        ON E.BusinessEntityID = PN.BusinessEntityID
) AS X
```

**Output (truncated):**

```
FirstName    MiddleName    LastName        FullName                PhoneNumber     AlphaNumericText                    NumStartPos  SortFirst3  SortSecond3  SortLast4  SortCharPortion
Ken          J             Sánchez         Ken J Sánchez           697-555-0142    Ken J Sánchez 697-555-0142          15           697         555          142        Ken J Sánchez
Terri        Lee           Duffy           Terri Lee Duffy         819-555-0175    Terri Lee Duffy 819-555-0175        17           819         555          175        Terri Lee Duffy
Roberto      NULL          Tamburello      Roberto Tamburello      212-555-0187    Roberto Tamburello 212-555-0187     20           212         555          187        Roberto Tamburello
Rob          NULL          Walters         Rob Walters             612-555-0100    Rob Walters 612-555-0100            13           612         555          100        Rob Walters
Gail         A             Erickson        Gail A Erickson         849-555-0139    Gail A Erickson 849-555-0139        17           849         555          139        Gail A Erickson
Jossef       H             Goldberg        Jossef H Goldberg       122-555-0189    Jossef H Goldberg 122-555-0189      19           122         555          189        Jossef H Goldberg
Dylan        A             Miller          Dylan A Miller          181-555-0156    Dylan A Miller 181-555-0156         16           181         555          156        Dylan A Miller
...
(19972 rows affected)
```

---

## 🔍 How the AlphaNumericText column is built

The phone numbers in the database are stored in the format `(area-code) number`, e.g. `(697) 555-0142`. We use `PATINDEX('%)%', PhoneNumber)` to find the position of the closing parenthesis `)`, then `SUBSTRING()` to extract everything after it. `LTRIM()` removes the leading space and `REPLACE()` replaces the remaining space with a hyphen, giving us `697-555-0142`.

The full name is built by concatenating `FirstName + MiddleName + LastName`. The triple `REPLACE()` pattern (`' '→'<>'→''→' '`) is used to collapse any double spaces that result from `NULL` middle names into a single space.

---

### Query 1.1 — Sort by numeric portion (first 3 digits of phone number)

**T-SQL code of Query 1.1**
```sql
SELECT X.AlphaNumericText 
FROM (
    SELECT  E.FirstName
	    , CASE E.MiddleName 
	    WHEN NULL THEN ''
	    ELSE E.MiddleName
	    END AS MiddleName 
	    , E.LastName AS LastName
	    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') AS FullName
	    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') + ' ' + REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS AlphaNumericText
	    , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS PhoneNumber
	FROM [AdventureWorks2022].[Person].[Person] AS E
	INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
		ON E.BusinessEntityID = PN.BusinessEntityID
	) AS X
ORDER BY SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%',X.AlphaNumericText), 3)
```

**Output of Query 1.1 (truncated):**

```
Elijah Alexander 100-555-0155
Louis Zhao 100-555-0146
Eduardo Collins 100-555-0124
...
Paula A Rubio 200-555-0116
Adam Henderson 200-555-0144
...
Jessica Martinez 901-555-0112
Jodi C Pal 901-555-0178
...
Marcus A Powell 999-555-0152
Jordyn Wood 999-555-0183
Logan H Wilson 999-555-0111
(19972 rows affected)
```

> **Note:** You can also sort using `SortSecond3Numbers` or `SortLast4Numbers` from Query 1 for finer ordering within groups sharing the same first 3 digits.

---

### Query 1.2 — Sort by character portion (full name)

**T-SQL code of Query 1.2**
```sql
SELECT X.AlphaNumericText 
FROM (
    SELECT  E.FirstName
	    , CASE E.MiddleName 
	    WHEN NULL THEN ''
	    ELSE E.MiddleName
	    END AS MiddleName 
	    , E.LastName AS LastName
	    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') AS FullName
	    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') + ' ' + REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS AlphaNumericText
	    , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS PhoneNumber
	FROM [AdventureWorks2022].[Person].[Person] AS E
	INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
		ON E.BusinessEntityID = PN.BusinessEntityID
	) AS X
ORDER BY SUBSTRING(X.FullName, 1, LEN(AlphaNumericText))
```


**Output of Query 1.2 (truncated):**
```
A. Francesca Leonetti 645-555-0193
A. Scott Wright 675-555-0100
A. Scott Wright 992-555-0194
Aaron A Allen 648-555-0141
...
Bailey A Bailey 873-555-0132
...
Zoe Rogers 194-555-0199
Zoe S Sanchez 137-555-0150
Zoe W Watson 166-555-0180
(19972 rows affected)
```

---
<br>

## 💡 Solution 2 — Using a VIEW (alternative to Solution 1)

We create a VIEW named `VIEW_MixedAlphaNumeric_FullnamePhonenumbers` that stores the `AlphaNumericText` column. Sorting queries then reference the VIEW instead of repeating the subquery each time.

**T-SQL code of Solution 2**
```sql
IF OBJECT_ID(N'VIEW_MixedAlphaNumeric_FullnamePhonenumbers', N'V') IS NOT NULL
    DROP VIEW VIEW_MixedAlphaNumeric_FullnamePhonenumbers
GO

CREATE VIEW VIEW_MixedAlphaNumeric_FullnamePhonenumbers
AS
SELECT X.AlphaNumericText
FROM (
    SELECT
        E.FirstName
      , CASE E.MiddleName WHEN NULL THEN '' ELSE E.MiddleName END AS MiddleName
      , E.LastName
      , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' + E.LastName,
          ' ', '<>'), '><', ''), '<>', ' ') AS FullName
      , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' + E.LastName,
          ' ', '<>'), '><', ''), '<>', ' ') + ' ' +
        REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1,
          LEN(PN.PhoneNumber))), ' ', '-') AS AlphaNumericText
      , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1,
          LEN(PN.PhoneNumber))), ' ', '-') AS PhoneNumber
    FROM [AdventureWorks2022].[Person].[Person] AS E
    INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
        ON E.BusinessEntityID = PN.BusinessEntityID
) AS X
```

**Output of Solution 2:**
```
Commands completed successfully.
```

---

### Query 2.1 — Sort VIEW by numeric portion

**T-SQL code of Query 2.1**
```sql
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY SUBSTRING(AlphaNumericText, PATINDEX('%[0-9]%', AlphaNumericText), 3)
```

**Output of Query 2.1:** Identical to Query 1.1.

---

### Query 2.2 — Sort VIEW by character portion

**T-SQL code of Query 2.2**
```sql
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY SUBSTRING(AlphaNumericText, 1, LEN(AlphaNumericText))
```

**Output of Query 2.2:** Identical to Query 1.2.

---
<br>

## 💡 Solution 3 — Using TRANSLATE and REPLICATE

### Approach
Instead of using `PATINDEX()` to find the digit start position, we use `TRANSLATE()` to replace all letters with `'z'` and then `REPLACE()` to strip them — leaving only the digits. For character sorting, we do the reverse: replace all digits with `'0'` and strip them.

### T-SQL code — Full solution: Build the sorting columns

**T-SQL code of Solution 3**
```sql
SELECT X.FirstName
, X.MiddleName
, X.LastName
, X.FullName
, X.PhoneNumber
, X.AlphaNumericText 
, LTRIM(REPLACE(REPLACE(REPLACE(TRANSLATE(LOWER(X.AlphaNumericText), 'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)), 'z',''), '-',''),CHAR(39),'')) AS SortNumericOnly
, SUBSTRING(LTRIM(REPLACE(REPLACE(REPLACE(TRANSLATE(LOWER(X.AlphaNumericText), 'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)), 'z',''), '-',''),CHAR(39),'')), 1, 3) AS SortFirst3Numbers
, SUBSTRING(LTRIM(REPLACE(REPLACE(REPLACE(TRANSLATE(LOWER(X.AlphaNumericText), 'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)), 'z',''), '-',''),CHAR(39),'')), 4, 3) AS SortSecond3Numbers
, SUBSTRING(LTRIM(REPLACE(REPLACE(REPLACE(TRANSLATE(LOWER(X.AlphaNumericText), 'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)), 'z',''), '-',''),CHAR(39),'')), 7, 4) AS SortLast4Numbers
, RTRIM(REPLACE(REPLACE(TRANSLATE(X.AlphaNumericText, '0123456789', '0000000000'), '-',''), '0','')) SortCharacterOnly
FROM (
    SELECT  E.FirstName
    , CASE E.MiddleName 
    WHEN NULL THEN ''
    ELSE E.MiddleName
    END AS MiddleName 
    , E.LastName AS LastName
    , PN.PhoneNumber AS PhoneNumberOriginal
    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') AS FullName
    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') + ' ' + REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS AlphaNumericText
    , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS PhoneNumber
    FROM [AdventureWorks2022].[Person].[Person] AS E
    INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
		ON E.BusinessEntityID = PN.BusinessEntityID
    ) AS X
```


**Output  of Solution 3 (truncated):**
```
FirstName	MiddleName	LastName	FullName	PhoneNumber	AlphaNumericText	SortNumericOnly	SortFirst3Numbers	SortSecond3Numbers	SortLast4Numbers	SortCharacterOnly
Ken	J	Sánchez	Ken J Sánchez	697-555-0142	Ken J Sánchez 697-555-0142	6975550142	697	555	0142	Ken J Sánchez
Terri	Lee	Duffy	Terri Lee Duffy	819-555-0175	Terri Lee Duffy 819-555-0175	8195550175	819	555	0175	Terri Lee Duffy
Roberto	NULL	Tamburello	Roberto Tamburello	212-555-0187	Roberto Tamburello 212-555-0187	2125550187	212	555	0187	Roberto Tamburello
Rob	NULL	Walters	Rob Walters	612-555-0100	Rob Walters 612-555-0100	6125550100	612	555	0100	Rob Walters
Gail	A	Erickson	Gail A Erickson	849-555-0139	Gail A Erickson 849-555-0139	8495550139	849	555	0139	Gail A Erickson
Jossef	H	Goldberg	Jossef H Goldberg	122-555-0189	Jossef H Goldberg 122-555-0189	1225550189	122	555	0189	Jossef H Goldberg
Dylan	A	Miller	Dylan A Miller	181-555-0156	Dylan A Miller 181-555-0156	1815550156	181	555	0156	Dylan A Miller
...
(19972 rows affected)
```

---
### How TRANSLATE works for numeric sorting
`TRANSLATE(LOWER(string), 'abcde...z...special chars', REPLICATE('z', 37))` replaces every letter and special character with `'z'`. Then `REPLACE(..., 'z', '')` removes all the `'z'`s, and `REPLACE(..., '-', '')` removes the hyphens — leaving only the raw digits.

---

### How TRANSLATE works for character sorting
`TRANSLATE(string, '0123456789', '0000000000')` replaces every digit with `'0'`. Then `REPLACE(..., '-', '')` removes hyphens and `REPLACE(..., '0', '')` removes all the `'0'`s — leaving only the letters and spaces.

---

### Query 3.1 — Sort by all 10 digits of phone number

```sql
SELECT X.AlphaNumericText 
FROM (
    SELECT  E.FirstName
    , CASE E.MiddleName 
    WHEN NULL THEN ''
    ELSE E.MiddleName
    END AS MiddleName 
    , E.LastName AS LastName
    , PN.PhoneNumber AS PhoneNumberOriginal
    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') AS FullName
    , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' +  E.LastName,' ','<>'),'><',''),'<>',' ') + ' ' + REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS AlphaNumericText
    , REPLACE(LTRIM(SUBSTRING(PN.PhoneNumber, PATINDEX('%)%', PN.PhoneNumber) + 1, LEN(PN.PhoneNumber))), ' ','-') AS PhoneNumber
    FROM [AdventureWorks2022].[Person].[Person] AS E
    INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
	ON E.BusinessEntityID = PN.BusinessEntityID
    ) AS X 
ORDER BY LTRIM(REPLACE(REPLACE(REPLACE(TRANSLATE(LOWER(X.AlphaNumericText), 'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)), 'z',''), '-',''),CHAR(39),''))
```

**Output of Query 3.1 (truncated):**

```
Isabella Roberts 100-555-0115
Eduardo Collins 100-555-0124
Isaiah Sanchez 100-555-0137
...
Jessica Martinez 901-555-0112
...
Kok-Ho T Loh 999-555-0155
Jordyn Wood 999-555-0183
Cassidy Griffin 999-555-0198
(19972 rows affected)
```

> **Note:** We can also sort Query 3.1 using "SortFirst3Numbers", "SortSecond3Numbers" or "SortLast4Numbers" from Query 3.

---

### Query 3.2 — Sort by character portion only
```sql
SELECT X.AlphaNumericText
FROM ( ... ) AS X
ORDER BY RTRIM(REPLACE(REPLACE(
    TRANSLATE(X.AlphaNumericText, '0123456789', '0000000000'),
    '-', ''), '0', ''))
```

**Output of Query 3.2:** Identical to Queries 1.2 and 2.2.

---

## 💡 Solution 4 — Using the VIEW from Solution 2 with TRANSLATE

Uses the same VIEW from Solution 2, but with the `TRANSLATE`-based `ORDER BY` from Solution 3.

```sql
-- Sort by all 10 digits
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY LTRIM(REPLACE(REPLACE(REPLACE(
    TRANSLATE(LOWER(AlphaNumericText),
      'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)),
    'z', ''), '-', ''), CHAR(39), ''))

-- Sort by character portion
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY RTRIM(REPLACE(REPLACE(
    TRANSLATE(AlphaNumericText, '0123456789', '0000000000'),
    '-', ''), '0', ''))
```

**Output:** Both queries return the same results as Queries 3.1 and 3.2 respectively.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
