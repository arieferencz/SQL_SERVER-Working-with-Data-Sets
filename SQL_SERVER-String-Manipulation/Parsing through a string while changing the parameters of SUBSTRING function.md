# Parsing through a string while changing the parameters of SUBSTRING function

## 🎯 Exercise
Using `SUBSTRING()` with varying parameters, demonstrate how changing the `start` and `length` arguments produces both descending and ascending character extractions from a string — and apply this to sort a mixed alphanumeric column by either its numeric or character portion.

---

## 💡 Solution 1 — Visualising SUBSTRING parameters with ascending and descending extractions

### Approach
We apply `SUBSTRING()` three times on the same string with different parameter combinations to produce:
1. A single character at each position (`FirstNameCharacter`)
2. A progressively shorter suffix starting at each position (`DescCharacters`)
3. A progressively longer suffix ending at the last character (`AscCharacters`)

This illustrates how `SUBSTRING(string, start, length)` behaves when both `start` and `length` change with each iteration.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `SUBSTRING(string, start, length)` | Extracts a substring from a given start position with a given length |
| `LEN()` | Returns the length of the string — used to calculate descending/ascending lengths |
| `REPLACE()` | Removes spaces from first names |
| `UPPER()` | Converts all characters to uppercase |
| `ROW_NUMBER() OVER (ORDER BY)` | Generates sequential position numbers for iteration |

### Table used

| Schema | Table |
|---|---|
| `Person` | `Person` |

---

### T-SQL code

```sql
SELECT DISTINCT
    Person.FirstNameNoSpaces
  , Iteration.Position AS IterationPosition
  , SUBSTRING(Person.FirstNameNoSpaces, Iteration.Position, 1)
      AS FirstNameCharacter
  , SUBSTRING(Person.FirstNameNoSpaces, Iteration.Position,
      LEN(Person.FirstNameNoSpaces) - Iteration.Position + 1)
      AS DescCharacters
  , SUBSTRING(Person.FirstNameNoSpaces,
      LEN(Person.FirstNameNoSpaces) - Iteration.Position + 1, Iteration.Position)
      AS AscCharacters
FROM (
    SELECT REPLACE(UPPER(FirstName), ' ', '') AS FirstNameNoSpaces
    FROM [AdventureWorks2022].[Person].[Person]
    WHERE REPLACE(FirstName, ' ', '') = 'JanainaBarreiroGambaro'
) AS Person,
(
    SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(FirstName, ' ', ''))) AS Position
    FROM [AdventureWorks2022].[Person].[Person]
) AS Iteration
WHERE Iteration.Position <= LEN(REPLACE(Person.FirstNameNoSpaces, ' ', ''))
ORDER BY Iteration.Position
```

---

### Output

```
FirstNameNoSpaces       IterPos  FirstNameCharacter  DescCharacters          AscCharacters
JANAINABARREIROGAMBARO  1        J                   JANAINABARREIROGAMBARO  O
JANAINABARREIROGAMBARO  2        A                   ANAINABARREIROGAMBARO   RO
JANAINABARREIROGAMBARO  3        N                   NAINABARREIROGAMBARO    ARO
JANAINABARREIROGAMBARO  4        A                   AINABARREIROGAMBARO     BARO
JANAINABARREIROGAMBARO  5        I                   INABARREIROGAMBARO      MBARO
JANAINABARREIROGAMBARO  6        N                   NABARREIROGAMBARO       AMBARO
JANAINABARREIROGAMBARO  7        A                   ABARREIROGAMBARO        GAMBARO
JANAINABARREIROGAMBARO  8        B                   BARREIROGAMBARO         OGAMBARO
JANAINABARREIROGAMBARO  9        A                   ARREIROGAMBARO          ROGAMBARO
JANAINABARREIROGAMBARO  10       R                   RREIROGAMBARO           IROGAMBARO
JANAINABARREIROGAMBARO  11       R                   REIROGAMBARO            EIROGAMBARO
JANAINABARREIROGAMBARO  12       E                   EIROGAMBARO             REIROGAMBARO
JANAINABARREIROGAMBARO  13       I                   IROGAMBARO              RREIROGAMBARO
JANAINABARREIROGAMBARO  14       R                   ROGAMBARO               ARREIROGAMBARO
JANAINABARREIROGAMBARO  15       O                   OGAMBARO                BARREIROGAMBARO
JANAINABARREIROGAMBARO  16       G                   GAMBARO                 ABARREIROGAMBARO
JANAINABARREIROGAMBARO  17       A                   AMBARO                  NABARREIROGAMBARO
JANAINABARREIROGAMBARO  18       M                   MBARO                   INABARREIROGAMBARO
JANAINABARREIROGAMBARO  19       B                   BARO                    AINABARREIROGAMBARO
JANAINABARREIROGAMBARO  20       A                   ARO                     NAINABARREIROGAMBARO
JANAINABARREIROGAMBARO  21       R                   RO                      ANAINABARREIROGAMBARO
JANAINABARREIROGAMBARO  22       O                   O                       JANAINABARREIROGAMBARO
(22 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the three SUBSTRING calls work

For a string of length `N` at position `P`:

| Column | Formula | Effect |
|---|---|---|
| `FirstNameCharacter` | `SUBSTRING(string, P, 1)` | Always 1 character — moves forward one step at a time |
| `DescCharacters` | `SUBSTRING(string, P, N - P + 1)` | Gets shorter by 1 character each row — descending suffix |
| `AscCharacters` | `SUBSTRING(string, N - P + 1, P)` | Gets longer by 1 character each row — ascending suffix from the end |

**Example for position 1 and position 22 (`N = 22`):**
- Position 1: `DescCharacters` = full string (length 22), `AscCharacters` = last 1 character (`O`)
- Position 22: `DescCharacters` = last 1 character (`O`), `AscCharacters` = full string (length 22)

This creates a mirror effect — as `DescCharacters` shrinks, `AscCharacters` grows.

---

## 💡 Solution 2 — Sorting a mixed alphanumeric column using SUBSTRING and PATINDEX

### Approach
We build a mixed alphanumeric column (`AlphaNumericText`) by concatenating each person's full name with their phone number. We then sort this column by either its numeric portion (phone number digits) or its character portion (full name) using `SUBSTRING()` combined with `PATINDEX()` to locate where the digits begin.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `PATINDEX('%[0-9]%', string)` | Finds the position of the first digit in the string |
| `SUBSTRING()` | Extracts the numeric or character portion for sorting |
| `COALESCE()` | Replaces `NULL` middle names with an empty string |
| `REPLACE()` | Cleans up extra spaces in full names and phone numbers |
| `LTRIM()` | Removes leading spaces |
| `LEN()` | Returns string length |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Person` |
| `Person` | `PersonPhone` |

---

### T-SQL code — Build the mixed alphanumeric column

```sql
SELECT
    X.FirstName
  , X.MiddleName
  , X.LastName
  , X.FullName
  , X.PhoneNumber
  , X.AlphaNumericText
  , PATINDEX('%[0-9]%', AlphaNumericText)                                         AS NumStartPosition
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText), 3)     AS SortFirst3Numbers
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText) + 4, 3) AS SortSecond3Numbers
  , SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText) + 8, 4) AS SortLast4Numbers
  , SUBSTRING(X.FullName, 1, LEN(AlphaNumericText))                               AS SortCharPortion
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

**Output (truncated):**

```
FirstName  MiddleName  LastName    FullName           PhoneNumber   AlphaNumericText               NumStartPos  SortFirst3  SortSecond3  SortLast4  SortCharPortion
Ken        J           Sánchez     Ken J Sánchez       697-555-0142  Ken J Sánchez 697-555-0142     15           697         555          142        Ken J Sánchez
Terri      Lee         Duffy       Terri Lee Duffy     819-555-0175  Terri Lee Duffy 819-555-0175   17           819         555          175        Terri Lee Duffy
Roberto    NULL        Tamburello  Roberto Tamburello  212-555-0187  Roberto Tamburello 212-555-0187 20           212         555          187        Roberto Tamburello
...
(19972 rows affected)
```

---

### Query 1.1 — Sort by numeric portion (first 3 digits of phone number)

```sql
SELECT X.AlphaNumericText
FROM ( ... ) AS X
ORDER BY SUBSTRING(X.AlphaNumericText, PATINDEX('%[0-9]%', X.AlphaNumericText), 3)
```

**Output (truncated):**

```
Elijah Alexander 100-555-0155
Louis Zhao 100-555-0146
Eduardo Collins 100-555-0124
...
Paula A Rubio 200-555-0116
...
Jessica Martinez 901-555-0112
...
Marcus A Powell 999-555-0152
Jordyn Wood 999-555-0183
(19972 rows affected)
```

> **Note:** You can also sort using `SortSecond3Numbers` or `SortLast4Numbers` from Query 1 for finer ordering within groups.

---

### Query 1.2 — Sort by character portion (full name)

```sql
SELECT X.AlphaNumericText
FROM ( ... ) AS X
ORDER BY SUBSTRING(X.FullName, 1, LEN(AlphaNumericText))
```

**Output (truncated):**

```
A. Francesca Leonetti 645-555-0193
A. Scott Wright 675-555-0100
Aaron A Allen 648-555-0141
...
Bailey A Bailey 873-555-0132
...
Zoe Rogers 194-555-0199
Zoe S Sanchez 137-555-0150
(19972 rows affected)
```

---

## 💡 Solution 3 — Using TRANSLATE and REPLICATE to isolate numeric or character data

### Approach
Instead of using `PATINDEX()` to find where digits start, we use `TRANSLATE()` to replace all letters with `'z'` and then `REPLACE()` to remove them — leaving only the digits. For character sorting we do the reverse: replace all digits with `'0'` and remove them.

### T-SQL code — Build the sorting columns

```sql
SELECT
    X.AlphaNumericText
  , LTRIM(REPLACE(REPLACE(REPLACE(
      TRANSLATE(LOWER(X.AlphaNumericText),
        'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)),
      'z', ''), '-', ''), CHAR(39), ''))                    AS SortNumericOnly
  , SUBSTRING(LTRIM(REPLACE(REPLACE(REPLACE(
      TRANSLATE(LOWER(X.AlphaNumericText),
        'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)),
      'z', ''), '-', ''), CHAR(39), '')), 1, 3)             AS SortFirst3Numbers
  , RTRIM(REPLACE(REPLACE(
      TRANSLATE(X.AlphaNumericText, '0123456789', '0000000000'),
      '-', ''), '0', ''))                                   AS SortCharacterOnly
FROM ( ... ) AS X
```

**Output (truncated):**

```
AlphaNumericText               SortNumericOnly  SortFirst3  SortCharacterOnly
Ken J Sánchez 697-555-0142     6975550142       697         Ken J Sánchez
Terri Lee Duffy 819-555-0175   8195550175       819         Terri Lee Duffy
Roberto Tamburello 212-555-0187 2125550187      212         Roberto Tamburello
...
```

### Query 3.1 — Sort by all 10 digits of phone number

```sql
ORDER BY LTRIM(REPLACE(REPLACE(REPLACE(
    TRANSLATE(LOWER(X.AlphaNumericText),
      'abcdefghijklmnopqrstuvwxyzáéíóúñ.¡ãçø', REPLICATE('z', 37)),
    'z', ''), '-', ''), CHAR(39), ''))
```

### Query 3.2 — Sort by character portion only

```sql
ORDER BY RTRIM(REPLACE(REPLACE(
    TRANSLATE(X.AlphaNumericText, '0123456789', '0000000000'),
    '-', ''), '0', ''))
```

Both produce identical results to Queries 1.1 and 1.2 respectively.

---

## 💡 Solutions 2 and 4 — Using a VIEW

As an alternative to the inline subquery approach, both Solutions 2 and 4 create a **VIEW** named `VIEW_MixedAlphaNumeric_FullnamePhonenumbers` that stores the `AlphaNumericText` column. The sorting queries then reference the VIEW instead of repeating the subquery each time.

```sql
IF OBJECT_ID(N'VIEW_MixedAlphaNumeric_FullnamePhonenumbers', N'V') IS NOT NULL
    DROP VIEW VIEW_MixedAlphaNumeric_FullnamePhonenumbers
GO

CREATE VIEW VIEW_MixedAlphaNumeric_FullnamePhonenumbers
AS
SELECT X.AlphaNumericText
FROM ( ... ) AS X
```

Then sort using the VIEW:

```sql
-- Sort by numeric portion
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY SUBSTRING(AlphaNumericText, PATINDEX('%[0-9]%', AlphaNumericText), 3)

-- Sort by character portion
SELECT AlphaNumericText
FROM [AdventureWorks2022].[dbo].[VIEW_MixedAlphaNumeric_FullnamePhonenumbers]
ORDER BY SUBSTRING(AlphaNumericText, 1, LEN(AlphaNumericText))
```

Both return the same results as Queries 1.1 and 1.2.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
