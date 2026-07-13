# Conditional Expressions and Procedures

## CASE
SQL's version of if/else logic. Evaluates conditions in order and returns a value for the first match. There are two ways to write it.

**Searched CASE (most common) — checks a condition per branch:**
​```sql
CASE
    WHEN condition1 THEN result1
    WHEN condition2 THEN result2
    ELSE default_result
END
​```
**Example:**
​```sql
SELECT name, salary,
    CASE
        WHEN salary >= 80000 THEN 'High'
        WHEN salary >= 50000 THEN 'Medium'
        ELSE 'Low'
    END AS salary_band
FROM employees;
​```

**Simple CASE — compares one expression against fixed values (shorter, but less flexible):**
​```sql
CASE column_name
    WHEN value1 THEN result1
    WHEN value2 THEN result2
    ELSE default_result
END
​```
**Example:**
​```sql
SELECT name,
    CASE department
        WHEN 'HR' THEN 'Human Resources'
        WHEN 'IT' THEN 'Information Technology'
        ELSE department
    END AS department_full_name
FROM employees;
​```
**When to use which:** Simple CASE only works for exact-match comparisons against a single column. Searched CASE is needed for ranges, multiple columns, or more complex conditions (like the `salary >= 80000` example above).

**Gotcha:** `ELSE` is optional — if you skip it and no condition matches, the result is `NULL`.

---

## COALESCE
Returns the first non-`NULL` value from a list of arguments.
​```sql
COALESCE(value1, value2, value3, ...)
​```
**Example:**
​```sql
SELECT name, COALESCE(phone, email, 'No contact info') AS contact
FROM customers;
​```
This shows the phone number if it exists, otherwise falls back to email, otherwise shows a default message.
**When I'd use it:** Providing fallback/default values when a column might be `NULL`.

---

## CAST
Converts a value from one data type to another.
​```sql
CAST(value AS target_type)
value::target_type   -- PostgreSQL shorthand syntax
​```
**Example:**
​```sql
SELECT CAST('123' AS INTEGER);
SELECT price::TEXT FROM products;   -- shorthand equivalent
​```
**When I'd use it:** Comparing or combining values of different types — e.g. treating a stored text field as a number, or formatting a number as text for concatenation.

---

## NULLIF
Returns `NULL` if two values are equal; otherwise returns the first value.
​```sql
NULLIF(value1, value2)
​```
**Example:**
​```sql
SELECT NULLIF(10, 10);   -- returns NULL
SELECT NULLIF(10, 5);    -- returns 10
​```
**When I'd use it:** Avoiding division-by-zero errors:
​```sql
SELECT amount / NULLIF(quantity, 0) FROM orders;
​```
If `quantity` is 0, this returns `NULL` instead of throwing an error.

---

## Views
A saved query that acts like a virtual table — doesn't store data itself, just runs the underlying query each time it's referenced.

**Create a view:**
​```sql
CREATE VIEW view_name AS
SELECT column1, column2
FROM table_name
WHERE condition;
​```
**Example:**
​```sql
CREATE VIEW high_earners AS
SELECT name, salary FROM employees WHERE salary > 80000;
​```
Now you can query it just like a table:
​```sql
SELECT * FROM high_earners;
​```

**Alter a view (change its underlying query):**
​```sql
CREATE OR REPLACE VIEW view_name AS
SELECT column1, column2, column3   -- new/updated query
FROM table_name
WHERE new_condition;
​```
**Gotcha:** `CREATE OR REPLACE` can add columns or change logic, but it **cannot** remove or reorder existing columns — for that, you must `DROP` and recreate the view.

**Rename a view:**
​```sql
ALTER VIEW old_view_name RENAME TO new_view_name;
​```

**Delete a view:**
​```sql
DROP VIEW view_name;
DROP VIEW IF EXISTS view_name;       -- avoids an error if it doesn't exist
DROP VIEW view_name CASCADE;         -- also drops dependent objects (e.g. other views built on top of it)
​```
**When I'd use views:** Simplifying a complex/frequent query into something reusable, or restricting access — e.g. giving someone a view with only the columns they're allowed to see, instead of the full table.

---

## Import and Export
Moving data in and out of a table via CSV files (PostgreSQL syntax). (It can be done right-clicking over the table you want to import/export).

HELPFUL LINKS:
https://stackoverflow.com/questions/2987433/how-to-import-csv-file-data-into-a-postgresql-table
https://www.enterprisedb.com/postgres-tutorials/how-import-and-export-data-using-csv-files-postgresql


**Export a table to CSV:**
​```sql
COPY table_name TO '/path/to/file.csv' DELIMITER ',' CSV HEADER;
​```

**Import a CSV into a table:**
​```sql
COPY table_name FROM '/path/to/file.csv' DELIMITER ',' CSV HEADER;
​```
**Example:**
​```sql
COPY employees TO '/tmp/employees_backup.csv' DELIMITER ',' CSV HEADER;
COPY employees FROM '/tmp/new_employees.csv' DELIMITER ',' CSV HEADER;
​```
**When I'd use it:** Backing up data, or bulk-loading data from an external source instead of writing individual `INSERT` statements.

**Gotcha:** `COPY` runs on the server side, so the file path must be accessible to the PostgreSQL server itself, not just your local machine. If you're using a client tool (like `psql`'s `\copy`), the path is relative to your local machine instead — worth checking which one you're using.