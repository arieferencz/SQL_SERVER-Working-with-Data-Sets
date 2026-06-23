# Create delimited lists by commas from table rows

## 🎯 Exercise
Take city names stored as individual rows in a table and convert them into comma-delimited lists — one list per country — displayed horizontally in a single cell instead of vertically across multiple rows.

**Before (vertical — multiple rows per country):**
```
CountryRegionCode  City
AU                 Bendigo
AU                 Sydney
AU                 Melbourne
CA                 Toronto
CA                 Vancouver
...
```

**After (horizontal — one delimited list per country):**
```
CountryRegionCode  DelimitedListCityName
AU                 Bendigo,Sydney,Melbourne,...
CA                 Toronto,Vancouver,...
```

---

## 💡 Solution 1 — Using nested subqueries

### Approach
We join three tables to get each city with its country code, apply `SELECT DISTINCT` to remove duplicate city names, then use `STRING_AGG()` grouped by country to concatenate all city names into a single comma-delimited string per country.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `STRING_AGG()` | Concatenates city names into a comma-delimited list per group |
| `CONVERT(NVARCHAR(max), ...)` | Ensures compatibility with `STRING_AGG()` |
| `CHAR(44)` | Inserts a comma character (ASCII code 44) as the separator |
| `SELECT DISTINCT` | Removes duplicate city names per country |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Address` |
| `Person` | `StateProvince` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code — Full solution

```sql
SELECT
    Y.CountryRegionCode
  , STRING_AGG(CONVERT(NVARCHAR(max), Y.City), CHAR(44)) AS DelimitedListCityName
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
```

---

### Output

```
CountryRegionCode  DelimitedListCityName
AU                 Bendigo,Cloverdale,Port Macquarie,South Melbourne,Sydney,Matraville,
                   Newcastle,St. Leonards,Lane Cove,Brisbane,Gold Coast,...,Wollongong
CA                 Brampton,Quebec,Victoria,Montreal,Waterloo,Langley,Hull,...,Chalk Riber
DE                 Augsburg,Erlangen,Sulzbach Taunus,...,Berlin,...
FR                 Dunkerque,Les Ulis,...,Paris,...
GB                 ...
US                 ...
(6 rows affected)
```

---

## 🔍 Step-by-step explanation

### Query 1.1 — Retrieve raw address data (`Subquery X`)
We join the three tables to retrieve each address record with its city, state code, country code, and territory name. This is the base dataset.

**T-SQL code of Query 1.1**
```sql
SELECT PersonAddress.AddressID
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


**Output of Query 1.1:** 19,614 rows — one per address record.
```
AddressID	City	                StateProvinceID	    StateProvinceCode	    CountryRegionCode	    TerritoryID	    TerritoryName
532	        Ottawa	                57	                ON 	                    CA	                    6	            Canada
497	        Burnaby	                7	                BC 	                    CA	                    6	            Canada
29781	    Dunkerque	            145	                59 	                    FR	                    7	            France
24231	    Verrieres Le Buisson	177	                91 	                    FR	                    7	            France
19637	    Verrieres Le Buisson	177	                91 	                    FR	                    7	            France
...
17393	    Paderborn	            20	                HH 	                    DE	                    8	            Germany
29769	    Berlin	                20	                HH 	                    DE	                    8	            Germany
18050	    München	                53	                NW 	                    DE	                    8	            Germany
(19614 rows affected)
```

---

### Query 1.2 — Remove duplicate city names (`Subquery Y`)
We apply `SELECT DISTINCT` on city and country to remove duplicate city entries, leaving only one row per unique city per country.

**T-SQL code of Query 1.2**
```sql
SELECT DISTINCT X.City	-- UniqueCityName
, X.CountryRegionCode
FROM
	(
	SELECT PersonAddress.AddressID
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
```


**Output of Query 1.2:** 579 rows — one per unique city-country combination.
```
City				CountryRegionCode
Southfield			US
Seattle				US
Bendigo				AU
San Mateo			US
Saint Matthews		US
...
Brampton			CA
Daly City			US
Great Falls			US
(579 rows affected)
```

---

### Full Solution (Query 1) — Concatenate cities into a delimited list using `STRING_AGG()`
We group by `CountryRegionCode` and use `STRING_AGG()` to concatenate all city names within each country into a single string, separated by `CHAR(44)` (the comma character). `CONVERT(NVARCHAR(max), ...)` is required because `STRING_AGG()` does not accept implicit conversion from some data types.

**Final output:** 6 rows — one comma-delimited list of cities per country.

```
CountryRegionCode	DelimitedListCityName
AU					Bendigo,Cloverdale,Port Macquarie,South Melbourne,Sydney,Matraville,...,Alexandria,Wollongong
CA					Brampton,Quebec,Victoria,Montreal,Waterloo,Langley,Hull,Newton,Shawnee,...,Pnot-Rouge,Chalk Riber
DE					Augsburg,Erlangen,Sulzbach Taunus,Solingen,Hamburg,Salzgitter,Ingolstadt,...,Bad Soden,Grevenbroich
FR					Dunkerque,Les Ulis,Villeneuve-d'Ascq,Boulogne-Billancourt,Orly,Morangis,...,Roncq,Aujan Mournede
GB					Abingdon,Maidenhead,Kirkby,Woolston,W. York,Cambridge,Gloucestershire,...,Oxford,West Sussex
US					Cedar City,Auburn,Long Beach,Salem,Daly City,Great Falls,Surprise,Lake George,...,Elk Grove,Carson
(6 rows affected)
```

---

## 💡 Solution 2 — Using CTEs (alternative approach)

Same logic as Solution 1 but rewritten using **Common Table Expressions (CTEs)** for clarity. Each step becomes a named CTE instead of a nested subquery.

```sql
WITH
FullList AS
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
DistinctValuesList AS
(
    SELECT DISTINCT
        FullList.City AS UniqueCityName
      , FullList.CountryRegionCode
    FROM FullList
)
SELECT
    CountryRegionCode
  , STRING_AGG(CONVERT(NVARCHAR(max), UniqueCityName), CHAR(44)) AS DelimitedListCityName
FROM DistinctValuesList
GROUP BY CountryRegionCode
```

**Output:** Identical to Solution 1 — 6 rows affected.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
