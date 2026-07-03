# SQL Statement Fundamentals

## SELECT
Retrieves data from a table. The most basic and common SQL command.
​```sql
SELECT column1, column2 FROM table_name;
SELECT * FROM table_name;   -- * means "all columns"
​```
**When I'd use it:** Any time you want to read data.

---

## SELECT DISTINCT
Returns only unique values, removing duplicates.
​```sql
SELECT DISTINCT column1 FROM table_name;
​```
**Example:**
​```sql
SELECT DISTINCT country FROM customers;
​```
**Gotcha:** If you select multiple columns, DISTINCT looks at the *combination* of values, not each column separately.

---

## COUNT
Returns the number of rows that match a query.
​```sql
SELECT COUNT(*) FROM table_name;
SELECT COUNT(column_name) FROM table_name;  -- ignores NULLs
SELECT COUNT(DISTINCT column_name) FROM table_name;
​```
**Example:**
​```sql
SELECT COUNT(*) FROM orders WHERE status = 'shipped';
​```

---

## SELECT WHERE
Filters rows based on a condition.
​```sql
SELECT * FROM table_name WHERE condition;
​```
**Common operators:**
​```sql
=      -- equal
!=     -- not equal  (also <>)
>      -- greater than
<      -- less than
>=     -- greater than or equal
<=     -- less than or equal
AND    -- both conditions must be true
OR     -- either condition can be true
NOT    -- negates a condition
​```
**Example:**
​```sql
SELECT * FROM employees WHERE department = 'Sales' AND salary > 50000;
​```

---

## ORDER BY
Sorts the result set by one or more columns.
​```sql
SELECT * FROM table_name ORDER BY column1 ASC;   -- ascending (default)
SELECT * FROM table_name ORDER BY column1 DESC;  -- descending
SELECT * FROM table_name ORDER BY column1, column2 DESC;  -- multiple columns
​```
**Example:**
​```sql
SELECT name, salary FROM employees ORDER BY salary DESC;
​```

---

## LIMIT
Restricts the number of rows returned. Goes at the end of the query.
​```sql
SELECT * FROM table_name LIMIT 5;
​```
**When I'd use it:** Previewing data, or combined with `ORDER BY` to get "top N" results.
​```sql
SELECT * FROM products ORDER BY price DESC LIMIT 3;  -- 3 most expensive products
​```

---

## BETWEEN
Filters within a range (inclusive on both ends).
​```sql
SELECT * FROM table_name WHERE column BETWEEN value1 AND value2;
​```
**Example:**
​```sql
SELECT * FROM orders WHERE order_date BETWEEN '2024-01-01' AND '2024-01-31';
​```
**Gotcha:** It's inclusive — both `value1` and `value2` are included in the results.

---

## IN
Checks if a value matches any value in a list — shorthand for multiple `OR` conditions.
​```sql
SELECT * FROM table_name WHERE column IN (value1, value2, value3);
​```
**Example:**
​```sql
SELECT * FROM customers WHERE country IN ('USA', 'Canada', 'Mexico');
​```
Equivalent to:
​```sql
WHERE country = 'USA' OR country = 'Canada' OR country = 'Mexico';
​```

---

## LIKE and ILIKE
Pattern matching for text. `LIKE` is case-sensitive, `ILIKE` is case-insensitive (PostgreSQL specific).
​```sql
SELECT * FROM table_name WHERE column LIKE 'pattern';
​```
**Wildcards:**
​```sql
%   -- matches any sequence of characters (including none)
_   -- matches exactly one character
​```
**Example:**
​```sql
SELECT * FROM customers WHERE name LIKE 'A%';      -- starts with A
SELECT * FROM customers WHERE name LIKE '%son';    -- ends with son
SELECT * FROM customers WHERE name ILIKE '%smith%'; -- contains "smith", any case
​```