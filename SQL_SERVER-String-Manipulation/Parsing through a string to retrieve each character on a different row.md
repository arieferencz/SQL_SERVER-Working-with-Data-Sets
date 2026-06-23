# Parsing through a string to retrieve each character on a different row

## 🎯 Exercise
Parse through the `FirstName` column and place each individual character on a separate row — so that a name like `JanainaBarreiroGambaro` produces 22 rows, one per character.

---

## 💡 Solution

### Approach
We use a **Cartesian Product** between the `Person` table and an `Iteration` subquery that generates sequential position numbers. `SUBSTRING()` then extracts one character at a time by using the position number as the starting index. A `WHERE` clause stops the iteration once the position exceeds the length of each name, removing empty rows.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `REPLACE()` | Removes spaces from first names |
| `SUBSTRING()` | Extracts one character at a time by position |
| `LEN()` | Returns the length of each name to control the iteration |
| `ROW_NUMBER() OVER (ORDER BY)` | Generates sequential position numbers for iteration |

### Table used

| Schema | Table |
|---|---|
| `Person` | `Person` |

---

### T-SQL code — Full solution

```sql
SELECT
    Person.BusinessEntityID
  , Person.FirstNameNoSpaces
  , SUBSTRING(Person.FirstNameNoSpaces, Iteration.Position, 1) AS FirstNameCharacter
FROM (
    SELECT
        BusinessEntityID
      , REPLACE(FirstName, ' ', '') AS FirstNameNoSpaces
    FROM [AdventureWorks2022].[Person].[Person]
) AS Person,
(
    SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(FirstName, ' ', ''))) AS Position
    FROM [AdventureWorks2022].[Person].[Person]
) AS Iteration
WHERE Iteration.Position <= LEN(REPLACE(Person.FirstNameNoSpaces, ' ', ''))
ORDER BY LEN(Person.FirstNameNoSpaces) DESC
```

---

### Output (truncated)

```
BusinessEntityID		FirstNameNoSpaces			FirstNameCharacter
565						JanainaBarreiroGambaro		J
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	n
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	i
565						JanainaBarreiroGambaro  	n
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	B
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	r
565						JanainaBarreiroGambaro  	r
565						JanainaBarreiroGambaro  	e
565						JanainaBarreiroGambaro  	i
565						JanainaBarreiroGambaro  	r
565						JanainaBarreiroGambaro  	o
565						JanainaBarreiroGambaro  	G
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	m
565						JanainaBarreiroGambaro  	b
565						JanainaBarreiroGambaro  	a
565						JanainaBarreiroGambaro  	r
565						JanainaBarreiroGambaro  	o
...
829						Ed                          E
829						Ed							d
39						Ed							E
39						Ed							d
231						Jo							J
231						Jo							o
539						Jo							J
539						Jo							o
2324					Jo							J
2324					Jo							o
27						Jo							J
27						Jo							o
(117894 rows affected)
```

---

## 🔍 Step-by-step explanation

### How the Cartesian Product works
The `Person` subquery returns 19,972 rows — one per person with spaces removed from their first name. The `Iteration` subquery also returns 19,972 sequential position numbers using `ROW_NUMBER()`.

The Cartesian Product between these two subqueries creates: **19,972 × 19,972 = 398,880,784 rows**.

For each combination, `SUBSTRING(FirstNameNoSpaces, Position, 1)` extracts the character at that position. Without the `WHERE` clause, every name would be paired with all 19,972 positions — producing mostly empty characters beyond the name's length.

---

### Query 1.1 — How the iteration works (single name example)
To illustrate how the iteration steps through a string, here is the result for `FirstName = 'Janaina Barreiro Gambaro'` (22 characters after removing spaces).

**T-SQL code of Query 1.1**
```sql
SELECT
    Person.FirstNameNoSpaces
  , Iteration.Position AS IterationPosition
  , SUBSTRING(Person.FirstNameNoSpaces, Iteration.Position, 1) AS FirstNameCharacter
FROM (
    SELECT REPLACE(FirstName, ' ', '') AS FirstNameNoSpaces
    FROM [AdventureWorks2022].[Person].[Person]
    WHERE REPLACE(FirstName, ' ', '') = 'JanainaBarreiroGambaro'
) AS Person,
(
    SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(FirstName, ' ', ''))) AS Position
    FROM [AdventureWorks2022].[Person].[Person]
) AS Iteration
WHERE Iteration.Position <= LEN(REPLACE(Person.FirstNameNoSpaces, ' ', ''))
```


**Output of Query 1.1:** 22 rows — one per character, stopping exactly at position 22.
```
FirstNameNoSpaces       	IterationPosition  		FirstNameCharacter
JanainaBarreiroGambaro  	1						J
JanainaBarreiroGambaro  	2						a
JanainaBarreiroGambaro  	3						n
JanainaBarreiroGambaro  	4						a
JanainaBarreiroGambaro  	5						i
JanainaBarreiroGambaro  	6						n
JanainaBarreiroGambaro  	7						a
JanainaBarreiroGambaro  	8						B
JanainaBarreiroGambaro  	9						a
JanainaBarreiroGambaro  	10						r
JanainaBarreiroGambaro  	11						r
JanainaBarreiroGambaro  	12						e
JanainaBarreiroGambaro  	13						i
JanainaBarreiroGambaro  	14						r
JanainaBarreiroGambaro  	15						o
JanainaBarreiroGambaro  	16						G
JanainaBarreiroGambaro  	17						a
JanainaBarreiroGambaro  	18						m
JanainaBarreiroGambaro  	19						b
JanainaBarreiroGambaro  	20						a
JanainaBarreiroGambaro  	21						r
JanainaBarreiroGambaro  	22						o
(22 rows affected)
```

---

### Query 1.2 — What happens without the `WHERE` clause
Removing the `WHERE` clause lets the iteration continue all the way to position 19,972, producing empty characters for all positions beyond the name's length.

```sql
SELECT                                                -- This is the same query as Query 1.1 but WITHOUT the WHERE clause
    Person.FirstNameNoSpaces
  , Iteration.Position AS IterationPosition
  , SUBSTRING(Person.FirstNameNoSpaces, Iteration.Position, 1) AS FirstNameCharacter
FROM (
    SELECT REPLACE(FirstName, ' ', '') AS FirstNameNoSpaces
    FROM [AdventureWorks2022].[Person].[Person]
    WHERE REPLACE(FirstName, ' ', '') = 'JanainaBarreiroGambaro'
) AS Person,
(
    SELECT ROW_NUMBER() OVER (ORDER BY LEN(REPLACE(FirstName, ' ', ''))) AS Position
    FROM [AdventureWorks2022].[Person].[Person]
) AS Iteration
```

**Output:** 19,972 rows — 22 valid characters followed by 19,950 empty rows.

```
FirstNameNoSpaces           IterationPosition       FirstNameCharacter
JanainaBarreiroGambaro      1		    	        J
JanainaBarreiroGambaro		2        	    		a
JanainaBarreiroGambaro		3		            	n
JanainaBarreiroGambaro		4		            	a
JanainaBarreiroGambaro		5	            		i
JanainaBarreiroGambaro		6	            		n
JanainaBarreiroGambaro		7	            		a
JanainaBarreiroGambaro		8	            		B
JanainaBarreiroGambaro		9	            		a
JanainaBarreiroGambaro		10	            		r
JanainaBarreiroGambaro		11	            		r
JanainaBarreiroGambaro		12	               		e
JanainaBarreiroGambaro		13	            		i
JanainaBarreiroGambaro		14		            	r
JanainaBarreiroGambaro		15		            	o
JanainaBarreiroGambaro		16		            	G
JanainaBarreiroGambaro		17		            	a
JanainaBarreiroGambaro		18		            	m
JanainaBarreiroGambaro		19		            	b
JanainaBarreiroGambaro		20		            	a
JanainaBarreiroGambaro		21		            	r
JanainaBarreiroGambaro		22		            	o
JanainaBarreiroGambaro		23		            	(empty)
JanainaBarreiroGambaro		24		            	(empty)
JanainaBarreiroGambaro		25	            		(empty)
JanainaBarreiroGambaro		26		            	(empty)
...
JanainaBarreiroGambaro		19970                   (empty)
JanainaBarreiroGambaro		19971                   (empty)
JanainaBarreiroGambaro		19972                   (empty)
(19972 rows affected)
```

> **Key takeaway:** The `WHERE` clause `Iteration.Position <= LEN(...)` is what controls the iteration — it acts as the stop condition, keeping only the rows where the position is within the actual length of the name. Without it, the query returns thousands of empty rows per person.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
