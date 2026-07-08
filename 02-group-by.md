# GROUP BY Statements

## Aggregation Functions
Functions that take multiple rows and return a single summary value. Almost always used with `GROUP BY`.
​```sql
COUNT()   -- number of rows
SUM()     -- total of a numeric column
AVG()     -- average of a numeric column
MIN()     -- smallest value
MAX()     -- largest value
​```
**Example:**
​```sql
SELECT AVG(salary) FROM employees;
SELECT SUM(amount) FROM orders;
​```

---

## GROUP BY
Groups rows that share the same value in a column, so aggregation functions apply per group instead of the whole table.
​```sql
SELECT column1, AGG_FUNC(column2)
FROM table_name
GROUP BY column1;
​```
**Example:**
​```sql
SELECT department, AVG(salary)
FROM employees
GROUP BY department;
​```
This gives you the average salary *per department*, instead of one average for the whole company.

**Gotcha:** Every column in your `SELECT` that isn't inside an aggregate function must appear in the `GROUP BY` clause, or the query will error.

**Order matters in the query itself:**
​```sql
SELECT ... FROM ... WHERE ... GROUP BY ... ORDER BY ...
​```

---

## HAVING
Filters groups *after* aggregation — like `WHERE`, but for aggregated results. `WHERE` can't filter on aggregate functions, so `HAVING` exists for that.
​```sql
SELECT column1, AGG_FUNC(column2)
FROM table_name
GROUP BY column1
HAVING condition;
​```
**Example:**
​```sql
SELECT department, COUNT(*)
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
​```
This only shows departments with more than 10 employees.

**Key distinction to remember:**
- `WHERE` filters rows **before** grouping
- `HAVING` filters groups **after** aggregation

​```sql
SELECT department, AVG(salary)
FROM employees
WHERE hire_date > '2020-01-01'   -- filters individual rows first
GROUP BY department
HAVING AVG(salary) > 60000;      -- filters the grouped results after
​```