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

### T-SQL code — Full solution

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

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
