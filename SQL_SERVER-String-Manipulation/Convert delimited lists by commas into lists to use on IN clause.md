# Convert delimited lists by commas into lists to use on IN clause

## 🎯 Exercise
Convert a comma-delimited list of city names into individual row values that can be passed to a `WHERE ... IN (...)` clause — and use it to retrieve all addresses for those cities.

---

## 📝 Note

> This exercise builds directly on the logic from [Create delimited lists by commas from table rows](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Create%20delimited%20lists%20by%20commas%20from%20table%20rows.md). Reading that exercise first is recommended.

---

## 💡 Solution 1 — Using nested subqueries

### Approach
We build a comma-delimited list of all unique city names grouped by country. We then use a **Cartesian Product** and `SUBSTRING()` to slide a window across each character of the list, isolating each city name by detecting the surrounding commas. A `WHERE` clause keeps only the rows where the string starts with a comma — which marks the beginning of each city name. The extracted city names are then passed to a `WHERE ... IN (...)` clause.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `STRING_AGG()` | Concatenates city names into a comma-delimited list |
| `CONVERT(NVARCHAR(max), ...)` | Ensures compatibility with `STRING_AGG()` |
| `CHAR(44)` | Inserts a comma character (ASCII 44) as separator |
| `SUBSTRING()` | Extracts substrings from the delimited list |
| `CHARINDEX()` | Finds the position of the next comma in the string |
| `LEN()` | Returns the length of a string |
| `ROW_NUMBER() OVER (ORDER BY)` | Generates sequential position numbers for iteration |
| `SELECT DISTINCT` | Removes duplicate city names |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Address` |
| `Person` | `StateProvince` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code — Full solution

```sql
SELECT AddressLine1
FROM [AdventureWorks2022].[Person].[Address]
WHERE City IN (
    SELECT
        SUBSTRING(SUBSTDelimCityName, 2,
            CHARINDEX(',', SubstringOne.SUBSTDelimCityName, 2) - 2) AS CityName
    FROM (
        SELECT
            SUBSTRING(Z.DelimListCityName, Iteration.Position,
                LEN(Z.DelimListCityName)) AS SUBSTDelimCityName
        FROM (
            SELECT
                ',' + STRING_AGG(CONVERT(NVARCHAR(max), Y.City), CHAR(44)) + ',' AS DelimListCityName
            FROM (
                SELECT DISTINCT
                    X.City
                  , X.CountryRegionCode
                FROM (
                    SELECT
                        PersonAddress.AddressID
                      , PersonAddress.City
                      , PersonAddress.StateProvinceID
                      , StateID.StateProvinceCode
                      , StateID.CountryRegionCode
                      , StateID.TerritoryID
                      , SalesTerritory.Name AS TerritoryName
                    FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
                    LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
                        ON PersonAddress.StateProvinceID = StateID.StateProvinceID
                    LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
                        ON StateID.TerritoryID = SalesTerritory.TerritoryID
                ) AS X
            ) AS Y
            GROUP BY Y.CountryRegionCode
        ) AS Z,
        (
            SELECT ROW_NUMBER() OVER (ORDER BY AddressID) AS Position
            FROM [AdventureWorks2022].[Person].[Address]
        ) AS Iteration
        WHERE Iteration.Position <= LEN(Z.DelimListCityName)
    ) AS SubstringOne
    WHERE LEN(SUBSTDelimCityName) > 1
      AND SUBSTRING(SUBSTDelimCityName, 1, 1) = ','
)
ORDER BY City
```

---

### Output (truncated)

```
City        AddressLine1
Abingdon    New Millhouse, 2583 Milton Park
Albany      1619 Mills Dr.
Albany      2255 254th Avenue Se
Albany      9098 Story Lane
Albany      Heritage Mall
...
Berlin      8783 Detroit Ave.
Berlin      Alderstr 3955
...
London      1005 Valley Oak Plaza
London      1019 Mt. Davidson Court
...
New York    123 Union Square South
New York    2596 Big Canyon Road
...
York        2050 B Avenue I
Zeeland     855 East Main Avenue
(19614 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Retrieve raw address data (`Subquery X`)
We join three tables to retrieve each address with its city, state, country, and territory. This is the base dataset.

**T-SQL code of Query 1.1**
```sql
SELECT PersonAddress.AddressID					-- OriginalTables1 = X
, PersonAddress.City
, PersonAddress.StateProvinceID
, StateID.StateProvinceCode
, StateID.CountryRegionCode
, StateID.TerritoryID
, SalesTerritory.Name AS TerritoryName
FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
ON PersonAddress.StateProvinceID = StateID.StateProvinceID
LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
ON StateID.TerritoryID = SalesTerritory.TerritoryID	
```

**Output of Query 1.1:** 19,614 rows (Truncated).
```
AddressID	    City			        StateProvinceID		StateProvinceCode	    CountryRegionCode	TerritoryID	    TerritoryName
532		        Ottawa			        57			        ON 			            CA		        	6		        Canada
497		        Burnaby			        7			        BC 		            	CA		        	6	        	Canada
29781		    Dunkerque		        145			        59		            	FR		        	7	        	France
24231		    Verrieres Le Buisson	177		        	91		            	FR		        	7	        	France
19637		    Verrieres Le Buisson	177			        91		            	FR		        	7	        	France
...
15768		    Neunkirchen		        70		        	SL 		            	DE		        	8	        	Germany
17393		    Paderborn		        20		        	HH 		            	DE		        	8	        	Germany
29769		    Berlin			        20		        	HH 		            	DE		        	8	        	Germany
18050		    München			        53		        	NW 		            	DE		        	8	        	Germany
(19614 rows affected)
```

---

### Query 1.2 — Remove duplicate city names (`Subquery Y`)
We apply `SELECT DISTINCT` on city and country to remove duplicate city names, leaving one row per unique city per country.

**T-SQL code of Query 1.2**
```sql
SELECT DISTINCT X.City            				                        -- UniqueCityName2 = Y
	, X.CountryRegionCode
FROM
	(
	SELECT 					                                            -- OriginalTables1 = X
	PersonAddress.AddressID
	, PersonAddress.City
	, PersonAddress.StateProvinceID
	, StateID.StateProvinceCode
	, StateID.CountryRegionCode
	, StateID.TerritoryID
	, SalesTerritory.Name AS TerritoryName
	FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
	LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
		ON PersonAddress.StateProvinceID = StateID.StateProvinceID
	LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
		ON StateID.TerritoryID = SalesTerritory.TerritoryID		    	-- OriginalTables1 = X
	) AS X										                        -- UniqueCityName2 = Y
```


**Output of Query 1.2:** 579 rows — one per unique city (Truncated).
```
City			        CountryRegionCode
Southfield		        US
Seattle			        US
Bendigo			        AU
San Mateo		        US
Saint Matthews        	US
...
Salem			        US
Brampton		        CA
Daly City		        US
Great Falls		        US
(579 rows affected)
```

---

### Query 1.3 — Build comma-delimited lists per country (`Subquery Z`)
We use `STRING_AGG()` grouped by `CountryRegionCode` to concatenate all city names per country into a single comma-delimited string. We add a comma at the start and end of each string (`,CityA,CityB,...,CityN,`) to make every city consistently surrounded by commas — this is essential for the extraction logic that follows.

**T-SQL code of Query 1.3**
```sql
SELECT ',' + STRING_AGG (CONVERT(NVARCHAR(max),Y.City), CHAR(44)) + ',' AS DelimListCityName		-- DelimListCities3 = Z
FROM	
(
	SELECT DISTINCT X.City											-- UniqueCityName2 = Y
		, X.CountryRegionCode
	FROM
		(
		SELECT 														-- OriginalTables1 = X
		PersonAddress.AddressID
		, PersonAddress.City
		, PersonAddress.StateProvinceID
		, StateID.StateProvinceCode
		, StateID.CountryRegionCode
		, StateID.TerritoryID
		, SalesTerritory.Name AS TerritoryName
		FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
		LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
			ON PersonAddress.StateProvinceID = StateID.StateProvinceID
		LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
		ON StateID.TerritoryID = SalesTerritory.TerritoryID			-- OriginalTables1 = X
		) AS X														-- UniqueCityName2 = Y
	) AS Y	
GROUP BY Y.CountryRegionCode										-- DelimitedListCities3 = Z
```

**Output of Query 1.3:** 6 rows — one delimited string per country (AU, CA, DE, FR, GB, US).
```
DelimListCityName
,Bendigo,Cloverdale,Port Macquarie,South Melbourne,Sydney,Matraville,Newcastle,St. Leonards,...,Alexandria,Wollongong,
,Brampton,Quebec,Victoria,Montreal,Waterloo,Langley,Hull,Newton,Shawnee,Weston,Ottawa,Edmonton,...,Chalk Riber,
,Augsburg,Erlangen,Sulzbach Taunus,Solingen,Hamburg,Salzgitter,Ingolstadt,Werne,Saarbrücken,...,Bad Soden,Grevenbroich,
,Dunkerque,Les Ulis,Villeneuve-d'Ascq,Boulogne-Billancourt,Orly,Morangis,Saint Ouen,Bordeaux,...,Roncq,Aujan Mournede,
...
,Cedar City,Auburn,Long Beach,Salem,Daly City,Great Falls,Surprise,Lake George,Waterbury,Tacoma,...,Elk Grove,Carson,
(6 rows affected)
```

---

### Query 1.4 — Slide a window across each string (`SubstringOne`)
We use a **Cartesian Product** between Subquery Z (6 rows) and an `Iteration` subquery that generates 19,614 sequential position numbers. `SUBSTRING()` then extracts a progressively shorter version of each string starting at each position — like sliding a window one character to the right on each row.

The Cartesian Product creates: **6 × 19,614 = 117,684 rows**. After filtering with `WHERE Iteration.Position <= LEN(Z.DelimListCityName)`, this reduces to **5,657 rows** — the total character count across all 6 country strings.

**T-SQL code of Query 1.4**
```sql
SELECT SUBSTRING(Z.DelimListCityName, Iteration.Position, LEN(Z.DelimListCityName)) AS SUBSTDelimCityName		-- SubstringOne4
FROM 
(	
SELECT ',' + STRING_AGG (CONVERT(NVARCHAR(max),Y.City), CHAR(44)) + ',' AS DelimListCityName					-- DelimListCities3 = Z
FROM	
	(
	SELECT DISTINCT X.City												-- UniqueCityName2 = Y
		, X.CountryRegionCode
	FROM
		(
		SELECT 															-- OriginalTables1 = X
		PersonAddress.AddressID
		, PersonAddress.City
		, PersonAddress.StateProvinceID
		, StateID.StateProvinceCode
		, StateID.CountryRegionCode
		, StateID.TerritoryID
		, SalesTerritory.Name AS TerritoryName
		FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
		LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
			ON PersonAddress.StateProvinceID = StateID.StateProvinceID
		LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
			ON StateID.TerritoryID = SalesTerritory.TerritoryID			-- OriginalTables1 = X
		) AS X															-- UniqueCityName2 = Y
	) AS Y	
	GROUP BY Y.CountryRegionCode										-- DelimitedListCities3 = Z 
	) AS Z,
	(
	SELECT ROW_NUMBER() OVER(ORDER BY AddressID) AS Position			-- Iteration3
	FROM [AdventureWorks2022].[Person].[Address]
	) AS Iteration							-- Iteration3
WHERE Iteration.Position <= LEN(Z.DelimListCityName)					-- SubstringOne4
```


**Output of Query 1.3: How the sliding window works (example for Australia):**
```
SUBSTDelimCityName							
,Bendigo,Cloverdale,Port Macquarie,South Melbourne,.........,Wollongong,
Bendigo,Cloverdale,Port Macquarie,South Melbourne,S.........,Wollongong,
endigo,Cloverdale,Port Macquarie,South Melbourne,Sy.........,Wollongong,
ndigo,Cloverdale,Port Macquarie,South Melbourne,Syd.........,Wollongong,
digo,Cloverdale,Port Macquarie,South Melbourne,Sydn.........,Wollongong,
igo,Cloverdale,Port Macquarie,South Melbourne,Sydne.........,Wollongong,
go,Cloverdale,Port Macquarie,South Melbourne,Sydney.........,Wollongong,
o,Cloverdale,Port Macquarie,South Melbourne,Sydney,.........,Wollongong,
,Cloverdale,Port Macquarie,South Melbourne,Sydney,M.........,Wollongong,
Cloverdale,Port Macquarie,South Melbourne,Sydney,Ma.........,Wollongong,
loverdale,Port Macquarie,South Melbourne,Sydney,Mat.........,Wollongong,
overdale,Port Macquarie,South Melbourne,Sydney,Matr.........,Wollongong,
verdale,Port Macquarie,South Melbourne,Sydney,Matra.........,Wollongong,
erdale,Port Macquarie,South Melbourne,Sydney,Matrav.........,Wollongong,
rdale,Port Macquarie,South Melbourne,Sydney,Matravi.........,Wollongong,
dale,Port Macquarie,South Melbourne,Sydney,Matravil.........,Wollongong,
ale,Port Macquarie,South Melbourne,Sydney,Matravill.........,Wollongong,
le,Port Macquarie,South Melbourne,Sydney,Matraville.........,Wollongong,
e,Port Macquarie,South Melbourne,Sydney,Matraville,.........,Wollongong,
...
,Wollongong,								
Wollongong,								
ollongong,								
llongong,								
longong,								
ongong,									
ngong,									
gong,									
ong,									
ng,									
g,									
,
...
(5657 rows affected)
```

---

### Query 1.5 — Extract city names (`SubstringTwo`)
We apply a `WHERE` clause that keeps only rows where:
1. The string has more than 1 character (`LEN > 1`)
2. The first character is a comma (`SUBSTRING(..., 1, 1) = ','`)

This keeps only rows like `,Bendigo,...` and `,Cloverdale,...` — i.e. rows where the window has just landed on a new city boundary. `SUBSTRING()` then extracts the city name between position 2 and the next comma.

**How the filter works:**

```
Row  SUBSTDelimCityName                    Kept?   CityName
1    ,Bendigo,Cloverdale,...               ✓       Bendigo
2    Bendigo,Cloverdale,...                ✗
3    endigo,Cloverdale,...                 ✗
...
9    ,Cloverdale,Port Macquarie,...        ✓       Cloverdale
10   Cloverdale,Port Macquarie,...         ✗
...
```

**Output:** 579 rows — one city name per row, ready for use in an `IN` clause.

```
CityName
Bendigo
Cloverdale
Port Macquarie
South Melbourne
Sydney
...
(579 rows affected)
```

---

### Final Query (Query 1) — Use extracted cities in `WHERE ... IN (...)`
The 579 city names from `SubstringTwo` are passed directly into the `IN` clause of the outer `SELECT`, filtering the `Address` table to return only addresses in those cities.

---

## 💡 Solution 2 — Using CTEs

Same logic as Solution 1 but rewritten with **Common Table Expressions (CTEs)** for clarity. Each step becomes a named CTE instead of a nested subquery.

```sql
WITH
Subquery_X AS
(
    SELECT
        PersonAddress.AddressID
      , PersonAddress.City
      , PersonAddress.StateProvinceID
      , StateID.StateProvinceCode
      , StateID.CountryRegionCode
      , StateID.TerritoryID
      , SalesTerritory.Name AS TerritoryName
    FROM [AdventureWorks2022].[Person].[Address] AS PersonAddress
    LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
        ON PersonAddress.StateProvinceID = StateID.StateProvinceID
    LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
        ON StateID.TerritoryID = SalesTerritory.TerritoryID
),
Subquery_Y AS
(
    SELECT DISTINCT
        Subquery_X.City
      , Subquery_X.CountryRegionCode
    FROM Subquery_X
),
Subquery_Z AS
(
    SELECT
        ',' + STRING_AGG(CONVERT(NVARCHAR(max), Subquery_Y.City), CHAR(44)) + ',' AS DelimListCityName
    FROM Subquery_Y
    GROUP BY Subquery_Y.CountryRegionCode
),
Iteration AS
(
    SELECT ROW_NUMBER() OVER (ORDER BY AddressID) AS Position
    FROM [AdventureWorks2022].[Person].[Address]
),
SubstringOne4 AS
(
    SELECT
        SUBSTRING(Subquery_Z.DelimListCityName, Iteration.Position,
            LEN(Subquery_Z.DelimListCityName)) AS SUBSTDelimCityName
    FROM Subquery_Z, Iteration
    WHERE Iteration.Position <= LEN(Subquery_Z.DelimListCityName)
),
SubstringTwo5 AS
(
    SELECT
        SUBSTRING(SubstringOne4.SUBSTDelimCityName, 2,
            CHARINDEX(',', SubstringOne4.SUBSTDelimCityName, 2) - 2) AS CityName
    FROM SubstringOne4
    WHERE LEN(SUBSTDelimCityName) > 1
      AND SUBSTRING(SUBSTDelimCityName, 1, 1) = ','
)
SELECT AddressLine1
FROM [AdventureWorks2022].[Person].[Address]
WHERE City IN (
    SELECT CityName FROM SubstringTwo5
)
ORDER BY City
```

**Output:** Identical to Solution 1 — 19,614 rows affected.

---

## 💡 Solution 2.1 — Using CTEs with a named variable for the `IN` clause

This is a small variation of Solution 2. The city name column in `SubstringTwo5` is stored as a named variable `CitiesforINclause`, making the final `WHERE ... IN (...)` clause more readable.

The only difference from Solution 2 is in the `SubstringTwo5` CTE and the final `SELECT`:

```sql
-- Inside SubstringTwo5 CTE:
SELECT CitiesforINclause = SUBSTRING(SubstringOne4.SUBSTDelimCityName, 2,
    CHARINDEX(',', SubstringOne4.SUBSTDelimCityName, 2) - 2)
FROM SubstringOne4
WHERE LEN(SUBSTDelimCityName) > 1
  AND SUBSTRING(SUBSTDelimCityName, 1, 1) = ','

-- Final SELECT:
SELECT AddressLine1
FROM [AdventureWorks2022].[Person].[Address]
WHERE City IN (
    SELECT CitiesforINclause
    FROM SubstringTwo5
)
ORDER BY City
```

**Output:** Identical to Solution 1 — 19,614 rows affected.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
