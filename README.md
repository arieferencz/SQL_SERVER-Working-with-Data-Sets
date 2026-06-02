# SQL Server — Working with Data Sets

This repository contains T-SQL exercises using Microsoft's **AdventureWorks2022** database, organised into 4 topic folders.

Each exercise includes a question, one or more solutions, the T-SQL code, the query output, and a step-by-step explanation.

---

## 📋 Table of contents

- [Folder 1 — Hierarchical parent-child relationship](#folder-1--hierarchical-parent-child-relationship)
- [Folder 2 — Pivoting](#folder-2--pivoting)
- [Folder 3 — String manipulation](#folder-3--string-manipulation)
- [Folder 4 — Data sets](#folder-4--data-sets)

---

## 📁 Folder 1 — Hierarchical parent-child relationship

| Exercise | Description |
|---|---|
| [Finding the manager for each employee](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Hierarchical-Parent-Child-relationship/Finding%20the%20manager%20for%20each%20employee%20(Parent-Child%20relationship).md) | Uses CTEs and self-joins to retrieve each employee's manager based on organisational hierarchy |
| [Finding the manager for each employee](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Hierarchical-Parent-Child-relationship/Finding%20the%20manager%20for%20each%20employee%20(Parent-Child%20relationship)%20(.GetAncestor()%20method).md) | Uses .GetAncestor() method to retrieve each employee's manager based on organisational hierarchy |

---

## 📁 Folder 2 — Pivoting

| Exercise | Description |
|---|---|
| [Pivoting employee count by department name](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Pivoting%20employee%20count%20by%20department%20name.md) | Transforms department rows into columns using PIVOT |
| [Pivoting multiple department names under same group name](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Pivoting%20multiple%20department%20names%20under%20same%20group%20name.md) | Groups and pivots departments sharing the same group name |
| [Pivoting query results for year-to-year comparison of sales](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Pivoting%20query%20results%20for%20year-to-year%20comparison%20of%20sales.md) | Creates a year-over-year sales comparison using PIVOT |
| [Reverse pivoting a query's result into only one column](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Reverse%20pivoting%20a%20query's%20result%20(multiple%20columns)%20into%20only%20one%20column.md) | Uses UNPIVOT to collapse multiple columns into a single column |
| [Reverse pivoting employee count by department](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Pivoting/Reverse%20pivoting%20employee%20count%20by%20department%20(columns%20into%20rows).md) | Converts pivoted columns back into rows using UNPIVOT |

---

## 📁 Folder 3 — String manipulation

| Exercise | Description |
|---|---|
| [Alphabetize the individual characters within a string](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Alphabetize%20the%20individual%20characters%20within%20a%20string.md) | Sorts each character in a string alphabetically using T-SQL |
| [Convert delimited lists by commas into lists to use on IN clause](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Convert%20delimited%20lists%20by%20commas%20into%20lists%20to%20use%20on%20IN%20clause.md) | Parses comma-separated strings for use in IN clauses |
| [Create delimited lists by commas from table rows](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Create%20delimited%20lists%20by%20commas%20from%20table%20rows.md) | Concatenates multiple rows into a single comma-separated string |
| [Parsing through a string to retrieve each character on a different row](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Parsing%20through%20a%20string%20to%20retrieve%20each%20character%20on%20a%20different%20row.md) | Splits a string character by character into individual rows |
| [Parsing through a string while changing the parameters of SUBSTRING](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Parsing%20through%20a%20string%20while%20changing%20the%20parameters%20of%20SUBSTRING%20function.md) | Demonstrates SUBSTRING with varying start and length parameters |
| [Sorting a column having mixed AlphaNumeric values](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Sorting%20a%20column%20having%20mixed%20AlphaNumeric%20values.md) | Correctly sorts columns containing both letters and numbers |
| [Sorting a column having strings having only letters](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-String-Manipulation/Sorting%20a%20column%20having%20strings%20having%20only%20letters.md) | Sorts purely alphabetical string columns |

---

## 📁 Folder 4 — Data sets

| Exercise | Description |
|---|---|
| [Calculating the historic subtotal purchasing amount for all vendors](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Calculating%20the%20historic%20subtotal%20purchasing%20amount%20for%20all%20vendors.md) | Aggregates historical purchasing totals across all vendors |
| [Concatenating multiple addresses for a BusinessEntityID into one row](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Concatenating%20multiple%20addresses%20for%20a%20BusinessEntityID%20into%20one%20row.md) | Merges multiple address rows into a single row per entity |
| [Count employees in each department having more than 5 employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Count%20employees%20in%20each%20department%20having%20more%20than%205%20employees.md) | Displays the count of employees for departments having 5 employees or more |
| [Count number of males and females in each department using conditional aggregation](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Count%20number%20of%20males%20and%20females%20in%20each%20department%20using%20conditional%20aggregation.md) | Counts the number of males and females for each department using the COUNT function and CASE WHEN statement |
| [Creating evenly sized groups of vendors](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Creating%20evenly%20size%20groups%20of%20vendors.md) | Divides vendors into equal groups using NTILE |
| [Dividing vendors into 10 groups based on historical purchasing amounts](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Dividing%20vendors%20into%2010%20groups%20based%20on%20historical%20purchasing%20amounts.md) | Ranks and segments vendors into 10 tiers by purchase volume |
| [List all departments and their employee counts](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/List%20all%20departments%20and%20their%20employee%20counts.md) | Displays all departments and their employee counts, including departments with zero employees |
| [Finding beginning and end of a range of consecutive numbers](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Finding%20beginning%20and%20end%20of%20a%20range%20of%20consecutive%20numbers.md) | Identifies start and end points of consecutive number sequences |
| [Find customers who have not made any purchase](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Find%20customers%20who%20have%20not%20made%20any%20purchase.md) | Displays customers who have not made a purchase |
| [Finding departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Finding%20the%20departments%20with%20zero%20employees.md) | Displays list of departments having zero employees |
| [Finding duplicate records on multiple tables](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Finding%20duplicate%20records%20on%20multiple%20tables.md) | Detects duplicate records across joined tables |
| [Finding employees with odd numbers on column BusinessEntityID](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Finding%20employees%20with%20odd%20numbers%20on%20column%20BusinessEntityID.md) | Filters rows where the ID column contains odd numbers |
| [Finding number of employees in each job title](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Find%20the%20number%20of%20employees%20in%20each%20job%20title.md) | Count the number of employees per job title |
| [Generate lists of consecutive numeric values](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Generate%20lists%20of%20consecutives%20numeric%20values.md) | Generates sequential number lists without a numbers table |
| [Removing repeating values from a column](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Data-Sets/Removing%20repeating%20values%20from%20a%20column.md) | Displays a value only on its first occurrence in a sorted result |

---

## 🛠️ Data source

All exercises use Microsoft's **AdventureWorks2022** sample database.
For installation instructions see: [SQL_SERVER-Database-AdventureWorks2022-for-SSMS](https://github.com/arieferencz/SQL_SERVER-Database-AdventureWorks2022-for-SSMS)

---

## 🔗 Back to my profile

[🏠 github.com/arieferencz](https://github.com/arieferencz)
