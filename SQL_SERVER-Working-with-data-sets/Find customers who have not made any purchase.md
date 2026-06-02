# Find customers who have not made any purchase

## 🎯 Exercise
Retrieve the list of customers who have never placed a sales order.

---

## 💡 Solution

### Approach
We use a `LEFT JOIN` between the `Customer` table and the `SalesOrderHeader` table. A `LEFT JOIN` returns all rows from the left table (`Customer`) regardless of whether a matching row exists in the right table (`SalesOrderHeader`). Where no match exists, the columns from the right table are filled with `NULL`. The `WHERE` clause then filters to keep only rows where `SalesOrderID IS NULL` — identifying customers who have never placed an order.

### T-SQL clauses used

| Clause | Purpose |
|---|---|
| `LEFT JOIN` | Returns all customers — including those with no matching rows in `SalesOrderHeader` |
| `WHERE SalesOrderID IS NULL` | Keeps only customers where no sales order was found — i.e. customers with no purchases |

### Tables used

| Schema | Table |
|---|---|
| `Sales` | `Customer` |
| `Sales` | `SalesOrderHeader` |

---

### T-SQL code

```sql
USE AdventureWorks2022;
GO

SELECT Customers.[CustomerID]
FROM [AdventureWorks2022].[Sales].[Customer] AS Customers
LEFT JOIN [AdventureWorks2022].[Sales].[SalesOrderHeader] AS Orders
    ON Customers.CustomerID = Orders.CustomerID
WHERE Orders.[SalesOrderID] IS NULL
```

---

### Output (truncated)

```
CustomerID
1
2
3
4
5
6
...
695
696
697
698
699
700
701
(701 rows affected)
```

---

## 🔍 Step-by-step explanation

### How `LEFT JOIN` works in this context

A `LEFT JOIN` between `Customer` and `SalesOrderHeader` produces one row per customer-order combination. For customers who have placed at least one order, all matching rows are returned with full data in both tables. For customers who have **never** placed an order, one row is returned with `NULL` in all columns from `SalesOrderHeader` — including `SalesOrderID`.

**Illustration of how the join works:**

```
Customers.CustomerID  Orders.SalesOrderID
11000                 43793               ← customer with a purchase
11000                 51522               ← customer with multiple purchases
11000                 57418
...
1                     NULL                ← customer with no purchases
2                     NULL                ← customer with no purchases
3                     NULL                ← customer with no purchases
```

The `WHERE Orders.SalesOrderID IS NULL` clause keeps only rows from the second category — customers where no sales order exists in `SalesOrderHeader`.

---

### Interpreting the output

The query returns **701 rows** — meaning 701 customers in the `Customer` table have never placed a sales order. These are customers who exist in the system (perhaps registered or imported) but have no purchase history associated with their `CustomerID`.

This type of query is commonly used in business contexts for:
- **Identifying inactive customers** for re-engagement campaigns
- **Data quality checks** — verifying whether customer records without orders should exist
- **Reporting** — understanding what proportion of the customer base has never transacted

The `Customer` table contains **19,820 total customers**. With **701 customers having no purchases**, approximately **3.5%** of the customer base has never placed an order.

---

### Why `LEFT JOIN` and not `INNER JOIN`?

An `INNER JOIN` only returns rows where a match exists in **both** tables. Customers who have never placed an order would be silently excluded — making it impossible to detect them. `LEFT JOIN` is essential here because it preserves all customers regardless of whether they have orders, allowing the `WHERE IS NULL` filter to identify the ones with none.

| Join type | Behaviour | Use when |
|---|---|---|
| `INNER JOIN` | Returns only customers who have placed at least 1 order | Finding customers with purchases |
| `LEFT JOIN` + `WHERE IS NULL` | Returns only customers who have placed no orders | Finding customers without purchases |

---

### Related exercise

This exercise uses the same `LEFT JOIN` + `WHERE IS NULL` pattern as [Finding the departments with zero employees](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets/blob/main/SQL_SERVER-Working-with-data-sets/Finding%20the%20departments%20with%20zero%20employees.md) — the only difference being the tables and business context. This pattern is a standard T-SQL technique for detecting missing or unmatched records across any two related tables.

---

## 🔗 Back to repository

[← Back to SQL_SERVER-Working-with-Data-Sets](https://github.com/arieferencz/SQL_SERVER-Working-with-Data-Sets)
