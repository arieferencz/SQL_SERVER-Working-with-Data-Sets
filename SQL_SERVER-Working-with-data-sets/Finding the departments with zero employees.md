Exercise: Finding the departments with zero employees. 

Solution: We used LEFT JOIN to connect the Department and EmployeeDepartmentHistory tables, to retrieve the list of departments having zero employees.

If the BusinessEntityID of any employee IS NULL, the condition in the WHERE clause evaluates to TRUE, and the query will retrieve the name and ID of the departments with zero employees.

The output displays no values for columns Name and DepartmentID, indicating there are no departments with zero employees.


Tables used
[AdventureWorks2022].[HumanResources].[Department]
[AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory]


T-SQL code
USE AdventureWorks2022;
GO

SELECT Department.[Name]
, Department.[DepartmentID]
FROM [AdventureWorks2022].[HumanResources].[Department] AS Department 
LEFT JOIN [AdventureWorks2022].[HumanResources].[EmployeeDepartmentHistory] AS EmployeeDepartmentHistory
ON Department.DepartmentID = EmployeeDepartmentHistory.DepartmentID
WHERE EmployeeDepartmentHistory.[BusinessEntityID] IS NULL;

Output
Name		DepartmentID
(empty)		(empty)
