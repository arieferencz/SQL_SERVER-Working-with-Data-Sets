# Finding duplicate records on multiple tables

## 🎯 Exercise
Find all persons in the database who have more than one address associated to their `BusinessEntityID` — returning each address as a separate row with its full details.

---

## 💡 Solution

### Approach
We use a subquery with `HAVING COUNT(*) > 1` on the `BusinessEntityAddress` table to identify all `BusinessEntityID` values that have more than one address. We then pass those IDs to a `WHERE ... IN (...)` clause on the main query, which joins six tables to return the full address details for each of those persons.

### T-SQL functions and clauses used

| Function / Clause | Purpose |
|---|---|
| `HAVING COUNT(*) > 1` | Identifies `BusinessEntityID` values with more than one address record |
| `WHERE ... IN (subquery)` | Filters the main query to keep only persons identified by the subquery |
| `LEFT JOIN` | Joins all address-related tables while preserving all person records |
| `GROUP BY` | Groups by `BusinessEntityID` in the subquery to enable the `COUNT()` check |

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
    Person.BusinessEntityID
  , Person.FirstName
  , Person.MiddleName
  , Person.LastName
  , BEAddress.AddressID
  , PersonAddress.AddressID
  , PersonAddress.AddressLine1
  , PersonAddress.AddressLine2
  , PersonAddress.PostalCode
  , PhoneNumber.PhoneNumber
  , PersonAddress.City
  , PersonAddress.StateProvinceID
  , StateID.StateProvinceID
  , StateID.StateProvinceCode
  , StateID.Name          AS StateProvinceName
  , StateID.CountryRegionCode
  , StateID.TerritoryID
  , SalesTerritory.TerritoryID
  , SalesTerritory.Name   AS TerritoryName
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
ORDER BY Person.BusinessEntityID
```

---

### Output (truncated)

```
BusinessEntityID  FirstName    MiddleName   LastName   AddressID  AddressLine1       AddressLine2  PostalCode  PhoneNumber   City         StateProvinceCode  StateProvinceName  TerritoryName  CountryRegionCode
2996              Amanda       S            Cook       238        4098 Woodcrest Dr. NULL          98201       252-555-0177  Everett      WA                 Washington         Northwest      US
2996              Amanda       S            Cook       12018      9187 Vista Del Sol NULL          98201       252-555-0177  Everett      WA                 Washington         Northwest      US
4073              Miguel       NULL         Miller     17         6696 Anchor Drive  NULL          98011       397-555-0155  Bothell      WA                 Washington         Northwest      US
4073              Miguel       NULL         Miller     13107      4293 Concord Ct.   NULL          98201       397-555-0155  Everett      WA                 Washington         Northwest      US
4388              Osarumwense  Uwaifiokun   Agbonile   18         1873 Lion Circle   NULL          98011       592-555-0152  Bothell      WA                 Washington         Northwest      US
4388              Osarumwense  Uwaifiokun   Agbonile   13423      3858 Vista Diablo  Unit C        98027       592-555-0152  Issaquah     WA                 Washington         Northwest      US
5124              Karl         V            Xie        15         4912 La Vuelta     NULL          98011       508-555-0163  Bothell      WA                 Washington         Northwest      US
5124              Karl         V            Xie        14159      4039 Elkwood Dr.   NULL          98107       508-555-0163  Ballard      WA                 Washington         Northwest      US
5479              Luis         NULL         Shan       11384      500 35th Ave NE    NULL          90012       716-555-0173  Los Angeles  CA                 California         Southwest      US
5479              Luis         NULL         Shan       14514      3993 Jabber Place  NULL          90012       716-555-0173  Los Angeles  CA                 California         Southwest      US
...
(48 rows affected)
```

---

## 🔍 Step-by-step explanation

### The subquery — identify persons with duplicate addresses
The inner subquery groups `BusinessEntityAddress` by `BusinessEntityID` and uses `HAVING COUNT(*) > 1` to keep only those IDs that appear more than once — meaning they have more than one address linked to them.

**Output of subquery:** 24 unique `BusinessEntityID` values.

### The main query — retrieve full address details
The main query joins all six tables to retrieve the complete address record for each person. By filtering with `WHERE BusinessEntityID IN (subquery)`, only the 24 persons with multiple addresses are returned — with each of their addresses on a separate row.

**Key observations in the output:**

- Each of the 24 persons appears **twice** — once per address — giving 48 total rows.
- The duplicate nature is visible in columns `AddressID` and `AddressLine1`, which differ between the two rows for the same person.
- Columns like `PhoneNumber`, `City`, `StateProvinceCode`, and `CountryRegionCode` are the same for both rows since they belong to the person rather than the address.
- `BusinessEntityID 4388` (Osarumwense Agbonile) is notable as one address includes an `AddressLine2` value (`Unit C`) while the other does not.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
