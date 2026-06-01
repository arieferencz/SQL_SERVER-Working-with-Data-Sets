# Concatenating multiple addresses for a BusinessEntityID into one row

## 🎯 Exercise
Some persons in the database have more than one address. Retrieve only those persons and concatenate all their addresses into a single semicolon-separated string on one row — instead of displaying each address on a separate row.

**Before (multiple rows per person):**
```
BusinessEntityID  FirstName  LastName  AddressLine1
2996              Amanda     Cook      4098 Woodcrest Dr., Everett, WA
2996              Amanda     Cook      9187 Vista Del Sol, Everett, WA
```

**After (one row per person):**
```
BusinessEntityID  FirstName  LastName  Addresses
2996              Amanda     Cook      4098 Woodcrest Dr., Everett, WA ;9187 Vista Del Sol, Everett, WA
```

---

## 💡 Solution 1 — Concatenate addresses in their original order

### Approach
We first identify all `BusinessEntityID` values that have more than one address using a subquery with `HAVING COUNT(*) > 1`. We then join five tables to build a formatted address string per row (`AddressLine1 + City + StateCode`), and use `STRING_AGG()` to concatenate all addresses per person into one semicolon-separated string.

### T-SQL functions used

| Function | Purpose |
|---|---|
| `STRING_AGG(expression, separator)` | Concatenates multiple address rows into one string per person, separated by `';'` |
| `HAVING COUNT(*) > 1` | Filters to keep only persons who have more than one address |
| `STRING concatenation (+)` | Builds the formatted address string: `AddressLine1 + ', ' + City + ', ' + StateCode` |

### Tables used

| Schema | Table |
|---|---|
| `Person` | `Person` |
| `Person` | `BusinessEntityAddress` |
| `Person` | `Address` |
| `Person` | `PersonPhone` |
| `Person` | `StateProvince` |
| `Sales` | `SalesTerritory` |

---

### T-SQL code

```sql
SELECT
    X.BusinessEntityID
  , X.FirstName
  , X.MiddleName
  , X.LastName
  , STRING_AGG(X.AddressLine1CityProvinceName, ';') AS Addresses
  , X.CountryRegionCode
FROM (
    SELECT
        Person.BusinessEntityID
      , Person.FirstName
      , Person.MiddleName
      , Person.LastName
      , PersonAddress.AddressID
      , PersonAddress.AddressLine1
      , PhoneNumber.PhoneNumber
      , PersonAddress.City
      , PersonAddress.AddressLine1 + ', ' + PersonAddress.City + ', '
          + StateID.StateProvinceCode AS AddressLine1CityProvinceName
      , PersonAddress.StateProvinceID
      , StateID.StateProvinceCode
      , StateID.Name AS StateProvinceName
      , StateID.TerritoryID
      , SalesTerritory.Name AS TerritoryName
      , SalesTerritory.CountryRegionCode
    FROM [AdventureWorks2022].[Person].[Person] AS Person
    LEFT JOIN [AdventureWorks2022].[Person].[BusinessEntityAddress] AS BEAddress
        ON Person.BusinessEntityID = BEAddress.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[Person].[Address] AS PersonAddress
        ON BEAddress.AddressID = PersonAddress.AddressID
    LEFT JOIN [AdventureWorks2022].[Person].[PersonPhone] AS PhoneNumber
        ON Person.BusinessEntityID = PhoneNumber.BusinessEntityID
    LEFT JOIN [AdventureWorks2022].[Person].[StateProvince] AS StateID
        ON PersonAddress.StateProvinceID = StateID.StateProvinceID
    LEFT JOIN [AdventureWorks2022].[Sales].[SalesTerritory] AS SalesTerritory
        ON StateID.TerritoryID = SalesTerritory.TerritoryID
    WHERE Person.BusinessEntityID IN (
        SELECT Person.BusinessEntityID
        FROM [AdventureWorks2022].[Person].[Person] AS Person
        LEFT JOIN [AdventureWorks2022].[Person].[BusinessEntityAddress] AS BEAddress
            ON Person.BusinessEntityID = BEAddress.BusinessEntityID
        GROUP BY Person.BusinessEntityID
        HAVING COUNT(*) > 1
    )
) AS X
GROUP BY X.BusinessEntityID, X.FirstName, X.MiddleName, X.LastName, X.CountryRegionCode
```

---

### Output

```
BusinessEntityID  FirstName    MiddleName   LastName   Addresses                                                          CountryRegionCode
2996              Amanda       S            Cook       4098 Woodcrest Dr., Everett, WA ;9187 Vista Del Sol, Everett, WA   US
4073              Miguel       NULL         Miller     6696 Anchor Drive, Bothell, WA ;4293 Concord Ct., Everett, WA      US
4388              Osarumwense  Uwaifiokun   Agbonile   1873 Lion Circle, Bothell, WA ;3858 Vista Diablo, Issaquah, WA     US
5124              Karl         V            Xie        4912 La Vuelta, Bothell, WA ;4039 Elkwood Dr., Ballard, WA         US
5479              Luis         NULL         Shan       500 35th Ave NE, Los Angeles, CA ;3993 Jabber Place, Los Angeles,  US
5561              Caleb        NULL         Alexander  4775 Kentucky Dr., Monroe, WA ;1261 Viking Drive, Everett, WA      US
5668              Jose         M            Hayes      793 Crawford Street, Kenmore, WA ;3141 Jabber Place, Ballard, WA   US
5976              Nathan       D            Chen       2095 Sierra Drive, Sammamish, WA ;2238 Pine Street, Issaquah, WA   US
8892              Gilbert      NULL         Xu         1010 Maple, Baltimore, MD ;7779 Merry Drive, Cheektowaga, NY       US
9196              Calvin       A            Raji       5415 San Gabriel Dr., Bothell, WA ;5509 Newcastle Road, Bothell,   US
10781             Sydney       C            Martinez   3770 Viewpoint Ct, Renton, WA ;5165 Cambridge Drive, Renton, WA   US
11072             Julia        NULL         Lee        3148 Rose Street, Bothell, WA ;3443 Centennial Way, Seattle, WA   US
11625             Lauren       NULL         Miller     6202 Seeno St., Sammamish, WA ;6437 Brookview Dr., Redmond, WA    US
15073             Evan         L            Murphy     249 Alexander Pl., Redmond, WA ;6900 Bellord Ct., Redmond, WA     US
16746             Aaron        R            Green      9 Olive Way, Seattle, WA ;8486 Hazelwood Lane, Seattle, WA        US
17243             Terry        G            Chander    771 Northridge Drive, Bellevue, WA ;521 Hermosa, Bellevue, WA     US
17298             José         S            Miller     9008 Creekside Drive, Everett, WA ;1946 Bayside Way, Everett, WA  US
17400             Karen        NULL         Wu         9265 La Paz, Bothell, WA ;7057 Striped Maple Court, Bothell, WA   US
17691             Jonathan     L            Jackson    40 Ellis St., Bothell, WA ;769 Algiers Drive, Edmonds, WA         US
20099             Destiny      NULL         Alexander  6097 Mt. McKinley Ct., Redmond, WA ;7432 Corte Valencia, Redmond, US
20305             Chloe        NULL         Patterson  1960 Via Catanzaro, Redmond, WA ;3175 Olivera Rd., Redmond, WA    US
20419             Julia        NULL         Simmons    7723 Firestone Drive, Redmond, WA ;2701 Sierra Rd, Redmond, WA    US
20550             Karen        NULL         Sanchez    7469 Paradise Ct., Newport Hills, WA ;6036 Park Glenn, Issaquah,  US
20759             Isabella     NULL         Reed       5297 Algiers Drive, Renton, WA ;6351 22nd Ave., Renton, WA        US
(24 rows affected)
```

---

## 🔍 Step-by-step explanation

### Subquery — Identify persons with more than one address
The inner subquery uses `HAVING COUNT(*) > 1` on `BusinessEntityAddress` to find all `BusinessEntityID` values that have more than one address record. This filters the outer query to return only those 24 persons.

### Build the formatted address string
Each address is formatted as a single string by concatenating:
```sql
AddressLine1 + ', ' + City + ', ' + StateProvinceCode
```
For example: `4098 Woodcrest Dr., Everett, WA`

### Concatenate with `STRING_AGG()`
`STRING_AGG(AddressLine1CityProvinceName, ';')` groups all address rows per person and concatenates them into one semicolon-separated string. The outer `GROUP BY` ensures one row per person.

---

## 💡 Solution 2 — Concatenate addresses in alphabetical order

This solution is identical to Solution 1 except that `STRING_AGG()` uses `WITHIN GROUP (ORDER BY AddressLine1CityProvinceName ASC)` to sort the addresses alphabetically before concatenating them.

```sql
STRING_AGG(X.AddressLine1CityProvinceName, ';')
    WITHIN GROUP (ORDER BY X.AddressLine1CityProvinceName ASC) AS Addresses
```

**Difference in output — comparing Solution 1 vs Solution 2 for `BusinessEntityID = 4073`:**

```
Solution 1 (original order):
6696 Anchor Drive, Bothell, WA ;4293 Concord Ct., Everett, WA

Solution 2 (alphabetical order):
4293 Concord Ct., Everett, WA ;6696 Anchor Drive, Bothell, WA
```

**Full output:** 24 rows — identical persons as Solution 1, with addresses sorted alphabetically within each person's concatenated string.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
