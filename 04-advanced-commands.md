# Advanced SQL Commands

## Timestamps and EXTRACT
`EXTRACT` pulls a specific part (year, month, day, hour, etc.) out of a date/time value.
​```sql
SELECT EXTRACT(YEAR FROM date_column) FROM table_name;
SELECT EXTRACT(MONTH FROM date_column) FROM table_name;
SELECT EXTRACT(DOW FROM date_column) FROM table_name;  -- day of week (0=Sunday)
​```
**Example:**
​```sql
SELECT EXTRACT(YEAR FROM order_date) AS order_year, COUNT(*)
FROM orders
GROUP BY order_year;
​```
**When I'd use it:** Grouping or filtering data by year, month, day of week, etc., instead of the full timestamp.

**Related: AGE()**
​```sql
SELECT AGE(date_column) FROM table_name;  -- how much time has passed since that date
​```

---

## TO_CHAR
Formats a date, timestamp, or number into a specific text format.
​```sql
SELECT TO_CHAR(date_column, 'format') FROM table_name;
​```
**Example:**
​```sql
SELECT TO_CHAR(order_date, 'YYYY-MM-DD') FROM orders;
SELECT TO_CHAR(order_date, 'Month DD, YYYY') FROM orders;  -- "January 05, 2024"
SELECT TO_CHAR(price, 'FM$999,999.00') FROM products;      -- format as currency
​```
**When I'd use it:** Making dates/numbers readable for reports or display, instead of the raw stored format.

---

## Mathematical Functions and Operators
Basic arithmetic can be done directly in a query, plus built-in math functions.
​```sql
+   -   *   /   %       -- addition, subtraction, multiplication, division, modulo

ROUND(number, decimals)  -- rounds a number
CEIL(number)             -- rounds up
FLOOR(number)             -- rounds down
ABS(number)               -- absolute value
​```
**Example:**
​```sql
SELECT price, ROUND(price * 1.1, 2) AS price_with_tax FROM products;
​```

---

## String Functions and Operators
Manipulate and combine text values.
​```sql
LENGTH(string)              -- number of characters
UPPER(string)                -- convert to uppercase
LOWER(string)                -- convert to lowercase
SUBSTRING(string, start, length)  -- extract part of a string
CONCAT(string1, string2)     -- join strings together
string1 || string2           -- also joins strings together (alternative syntax)
TRIM(string)                  -- remove leading/trailing whitespace
​```
**Example:**
​```sql
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM employees;
SELECT UPPER(country) FROM customers;
​```

---

## SubQuery
A query nested inside another query. Runs first, and its result is used by the outer query.
​```sql
SELECT column1
FROM table_name
WHERE column2 = (SELECT MAX(column2) FROM table_name);
​```
**Example:**
​```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
​```
This finds all employees earning more than the company's average salary.

**Gotcha:** If the subquery can return more than one row, you need `IN` instead of `=`:
​```sql
SELECT name
FROM employees
WHERE department_id IN (SELECT id FROM departments WHERE location = 'NYC');
​```