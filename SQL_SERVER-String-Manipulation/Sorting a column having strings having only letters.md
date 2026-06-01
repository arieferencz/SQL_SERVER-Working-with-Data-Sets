# Sorting a column having strings having only letters

## 🎯 Exercise
Determine whether a string contains only letters — and retrieve only the full names from the database that contain no characters other than letters (spaces excluded).

---

## 📝 Note

> Out of 19,972 full names in the database, **19,393 contain only letters**. The remaining **579 contain non-letter characters** such as:
> - **Periods:** `K. Saravan`, `A. Francesca Leonetti`, `Abigail J. Gonzalez`
> - **Apostrophes:** `Claire O'Donnell`
> - **Hyphens:** `Juha-Pekka Posti`, `Kok-Ho Loh`, `Julie Taft-Rider`

---

## 💡 Solution

### Approach
We create two derived columns based on `FullName` and compare them in a `WHERE` clause:

- **Column 1 (`FullNameNoSpaces`):** Removes spaces and returns the letter `'z'` repeated as many times as the length of the name without spaces — a baseline of all-letter length.
- **Column 2 (`FullNameLettersOnly`):** Removes spaces, periods, and exclamation marks, then uses `TRANSLATE()` to replace every letter with `'z'`.

If both columns are equal, the name contains only letters. If `Column 2` is shorter than `Column 1`, the name contains non-letter characters (apostrophes, hyphens, digits, etc.) that were not replaced by `TRANSLATE()`.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `TRANSLATE(string, from_chars, to_chars)` | Replaces each character in `from_chars` with the corresponding character in `to_chars` — used here to convert all letters to `'z'` |
| `REPLICATE('z', n)` | Returns the letter `'z'` repeated `n` times |
| `REPLACE()` | Removes spaces, periods, and exclamation marks |
| `LEN()` | Returns the length of the name without spaces |
| `LOWER()` | Converts the name to lowercase before comparison |
| `COALESCE()` | Replaces `NULL` middle names with an empty string |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Person` |
| `Person` | `PersonPhone` |

---

### T-SQL code

```sql
SELECT X.FullName
FROM (
    SELECT
        E.FirstName
      , CASE E.MiddleName WHEN NULL THEN '' ELSE E.MiddleName END AS MiddleName
      , E.LastName
      , REPLACE(REPLACE(REPLACE(E.FirstName + ' ' + COALESCE(E.MiddleName, '') + ' ' + E.LastName,
          ' ', '<>'), '><', ''), '<>', ' ') AS FullName
    FROM [AdventureWorks2022].[Person].[Person] AS E
    INNER JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PN
        ON E.BusinessEntityID = PN.BusinessEntityID
) AS X
WHERE
    TRANSLATE(
        REPLACE(REPLACE(REPLACE(LOWER(X.FullName), ' ', ''), '.', ''), '¡', ''),
        'abcdefghijklmnopqrstuvwxyzáéíóúñãçø',
        REPLICATE('z', 35)
    )
    =
    REPLICATE('z', LEN(REPLACE(LOWER(X.FullName), ' ', '')))
```

---

### Output (truncated)

```
FullName
Ken J Sánchez
Terri Lee Duffy
Roberto Tamburello
...
(19393 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the two columns work

**Column 1 — `FullNameNoSpaces` (the baseline):**
```sql
REPLICATE('z', LEN(REPLACE(LOWER(X.FullName), ' ', '')))
```
This removes spaces from the full name and returns `'z'` repeated as many times as the number of remaining characters. For example:
- `'Ken J Sánchez'` → remove spaces → `'KenJSánchez'` (length 11) → `'zzzzzzzzzzz'`

**Column 2 — `FullNameLettersOnly` (the test):**
```sql
TRANSLATE(
    REPLACE(REPLACE(REPLACE(LOWER(X.FullName), ' ', ''), '.', ''), '¡', ''),
    'abcdefghijklmnopqrstuvwxyzáéíóúñãçø',
    REPLICATE('z', 35)
)
```
This removes spaces, periods, and exclamation marks, then replaces every letter (including accented characters) with `'z'`. For example:
- `'Ken J Sánchez'` → remove spaces/punctuation → `'KenJSánchez'` → translate all letters to `'z'` → `'zzzzzzzzzzz'`

**The WHERE clause comparison:**

| Full name | Column 1 | Column 2 | Equal? | Kept? |
|---|---|---|---|---|
| `Ken J Sánchez` | `zzzzzzzzzzz` | `zzzzzzzzzzz` | ✓ | ✓ |
| `Terri Lee Duffy` | `zzzzzzzzzzzzzz` | `zzzzzzzzzzzzzz` | ✓ | ✓ |
| `Claire O'Donnell` | `zzzzzzzzzzzzzzz` | `zzzzzzzzzzzzzz` | ✗ — apostrophe not replaced | ✗ |
| `A. Francesca Leonetti` | `zzzzzzzzzzzzzzzzzzzz` | `zzzzzzzzzzzzzzzzzzz` | ✗ — period removed but not counted | ✗ |
| `Juha-Pekka Posti` | `zzzzzzzzzzzzzzz` | `zzzzzzzzzzzzzz` | ✗ — hyphen not replaced | ✗ |

When a name contains a non-letter character that `TRANSLATE()` does not replace (like an apostrophe or hyphen), Column 2 ends up shorter than Column 1 — so the `WHERE` clause filters it out.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
