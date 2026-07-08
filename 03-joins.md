# JOINS

## AS Statement (Aliasing)
Renames a column or table temporarily, just for the output of the query. Doesn't change the actual table/column name.
​```sql
SELECT column_name AS alias_name FROM table_name;
SELECT t.column_name FROM table_name AS t;
​```
**Example:**
​```sql
SELECT first_name AS name, salary AS pay FROM employees;

SELECT e.name, d.name AS department_name
FROM employees AS e
JOIN departments AS d ON e.department_id = d.id;
​```
**When I'd use it:** Shortening long table names in joins, or making output column headers more readable.

---

## Inner Join
Returns only the rows that have matching values in **both** tables.
​```sql
SELECT *
FROM table_a
INNER JOIN table_b ON table_a.id = table_b.a_id;
​```
**Example:**
​```sql
SELECT orders.id, customers.name
FROM orders
INNER JOIN customers ON orders.customer_id = customers.id;
​```
**Picture it:** Only the overlapping part of a Venn diagram between the two tables.

---

## Full Outer Join
Returns **all** rows from both tables — matched where possible, with `NULL`s filled in where there's no match on either side.
​```sql
SELECT *
FROM table_a
FULL OUTER JOIN table_b ON table_a.id = table_b.a_id;
​```
**When I'd use it:** When you want to see everything from both tables, including unmatched rows on either side (e.g. finding orders with no customer, and customers with no orders, all in one query).

---

## Left Outer Join
Returns **all** rows from the left table, plus matching rows from the right table. Unmatched right-side columns show as `NULL`.
​```sql
SELECT *
FROM table_a
LEFT JOIN table_b ON table_a.id = table_b.a_id;
​```
**Example:**
​```sql
SELECT customers.name, orders.id
FROM customers
LEFT JOIN orders ON customers.id = orders.customer_id;
​```
This shows *all* customers, even ones who've never placed an order (their `orders.id` will just be `NULL`).

---

## Right Join
Returns **all** rows from the right table, plus matching rows from the left table. Opposite of Left Join. Unmatched left-side columns show as `NULL`.
​```sql
SELECT *
FROM table_a
RIGHT JOIN table_b ON table_a.id = table_b.a_id;
​```
**Gotcha:** A `RIGHT JOIN` can always be rewritten as a `LEFT JOIN` by swapping the table order — most people default to using `LEFT JOIN` for consistency and rarely use `RIGHT JOIN`.

---

## UNION
Combines the results of two separate `SELECT` queries into one result set, stacking rows on top of each other (not side by side like a join).
​```sql
SELECT column1 FROM table_a
UNION
SELECT column1 FROM table_b;
​```
**Example:**
​```sql
SELECT name FROM current_employees
UNION
SELECT name FROM former_employees;
​```
**Gotchas:**
- Both queries must have the **same number of columns**, in the same order, with compatible data types.
- `UNION` removes duplicate rows by default. Use `UNION ALL` to keep duplicates (and it's faster, since it skips the de-duplication step).