# Creating Databases and Tables

## Data Types
Defines what kind of value a column can store. Choosing the right type matters for storage size and data integrity.
​```sql
INTEGER / INT       -- whole numbers
BIGINT               -- large whole numbers
NUMERIC(p, s)        -- exact decimal, p=total digits, s=digits after decimal
REAL / DOUBLE PRECISION  -- floating point (less precise, faster)
VARCHAR(n)           -- variable-length text, max n characters
CHAR(n)              -- fixed-length text, padded with spaces
TEXT                 -- variable-length text, no max length
BOOLEAN              -- true/false
DATE                 -- date only (YYYY-MM-DD)
TIMESTAMP            -- date + time
TIME                 -- time only
​```
**Example:**
​```sql
CREATE TABLE products (
    price NUMERIC(10, 2),   -- up to 10 digits total, 2 after the decimal
    name VARCHAR(100),
    in_stock BOOLEAN
);
​```

---

## Primary Keys and Foreign Keys

**Primary Key** — uniquely identifies each row in a table. Can't be `NULL`, must be unique.
​```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);
​```

**Foreign Key** — a column that references the primary key of another table, creating a relationship between tables.
​```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    order_date DATE
);
​```
**When I'd use it:** Linking related data across tables — e.g. every order belongs to a customer.

---

## Constraints
Rules enforced on columns to maintain data integrity. Can be applied at column-level or table-level.

**Types of constraints:**
​```sql
NOT NULL      -- column cannot be empty
UNIQUE        -- all values in the column must be different
PRIMARY KEY   -- combination of NOT NULL + UNIQUE, identifies each row
FOREIGN KEY   -- links to another table's primary key
CHECK         -- value must satisfy a custom condition
DEFAULT       -- provides a default value if none is given
​```
**Example:**
​```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    email VARCHAR(100) UNIQUE NOT NULL,
    salary NUMERIC CHECK (salary > 0),
    department VARCHAR(50) DEFAULT 'Unassigned'
);
​```

---

## CREATE Table
Creates a new table, defining its columns, data types, and constraints.
​```sql
CREATE TABLE table_name (
    column1 datatype constraint,
    column2 datatype constraint,
    ...
);
​```
**Example:**
​```sql
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    age INT,
    enrolled_date DATE DEFAULT CURRENT_DATE
);
​```

---

## REFERENCES
Shorthand way to create a foreign key constraint directly inline, when defining a column — instead of writing a separate `FOREIGN KEY` clause.
​```sql
column_name datatype REFERENCES other_table(other_column)
​```
**Example:**
​```sql
CREATE TABLE account_job (
    user_id INTEGER REFERENCES account(user_id),
    job_id INTEGER REFERENCES job(job_id),
    hire_date TIMESTAMP
);
​```
This means: `user_id` must match a value that already exists in `account.user_id`, and `job_id` must match a value that already exists in `job.job_id`. If you try to insert a `user_id` that doesn't exist in `account`, Postgres will reject it.

**When I'd use it:** Building a table that links two other tables together (a "junction" or "join" table) — like `account_job` connecting `account` and `job` in a many-to-many relationship.

**Gotcha:** `REFERENCES` on its own is just a shorter way of writing this equivalent, more explicit version:
​```sql
CREATE TABLE account_job (
    user_id INTEGER,
    job_id INTEGER,
    hire_date TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES account(user_id),
    FOREIGN KEY (job_id) REFERENCES job(job_id)
);
​```
Both do the same thing — the explicit `FOREIGN KEY` version is more useful when you want to name the constraint yourself (for easier `ALTER`/`DROP` later) or reference multiple columns at once (a composite key).


## INSERT
Adds new rows to a table.
​```sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2);
​```
**Example:**
​```sql
INSERT INTO students (name, age) VALUES ('Maria', 22);

-- inserting multiple rows at once
INSERT INTO students (name, age)
VALUES ('Maria', 22), ('Carlos', 25);
​```
**Gotcha:** Column order in `VALUES` must match the column order you listed — if you skip listing columns, it assumes *all* columns, in table order.

**Insert values from another table:**
Instead of manually typing values, you can insert the *results of a query* directly into a table. Uses `SELECT` instead of `VALUES`.
​```sql
INSERT INTO table_name (column1, column2)
SELECT column1, column2
FROM another_table
WHERE condition;
​```
**Example:**
​```sql
INSERT INTO archived_customers (id, name)
SELECT id, name
FROM customers
WHERE last_order_date < '2020-01-01';
​```
This copies matching rows from `customers` straight into `archived_customers`, without needing to select and retype the data yourself.

**When I'd use it:** Migrating/archiving data between tables, or populating a new table using data that already exists elsewhere in the database.

**Gotcha:** The columns listed after `SELECT` must match the number and order of columns listed after `INSERT INTO table_name (...)`, and their data types must be compatible.

---

## UPDATE
Modifies existing rows in a table.
​```sql
UPDATE table_name
SET column1 = value1
    column2 = value2
WHERE condition;
​```
**Example:**
​```sql
UPDATE students
SET age = 23
WHERE name = 'Maria';
​```
**Gotcha:** If you forget the `WHERE` clause, it updates **every row** in the table. Always double-check before running.

**Update using another table's values (UPDATE join):**
Lets you update rows in one table based on matching data from another table — instead of a fixed value, `SET` pulls from a joined table.
​```sql
UPDATE table_a
SET column_to_update = table_b.new_value
FROM table_b
WHERE table_a.id = table_b.id;
​```
**Example:**
​```sql
UPDATE employees
SET department_name = departments.name
FROM departments
WHERE employees.department_id = departments.id;
​```
This updates `department_name` in `employees`, pulling the correct value from `departments` for each matching row.

**When I'd use it:** Syncing a column's value with data that lives in another table — e.g. copying over a name, price, or status that changed elsewhere.

**Gotcha:** The `WHERE` clause here does double duty — it's both the "join condition" (matching rows between the two tables) and the row filter. If you forget it, every row in `table_a` could get updated with a value from a *random* matching row in `table_b`, since there's nothing telling Postgres how to pair them up correctly.

---

## DELETE
Removes rows from a table.
​```sql
DELETE FROM table_name WHERE condition;
​```
**Example:**
​```sql
DELETE FROM students WHERE name = 'Carlos';
​```
**Gotcha:** Same as `UPDATE` — no `WHERE` clause means it deletes **every row**. The table itself still exists, just empty.

**Delete based on data in another table:**
Sometimes you need to delete rows from one table based on a condition that depends on a *different* table.

Using USING (PostgreSQL-specific, like a join)**
​```sql
DELETE FROM table_a
USING table_b
WHERE table_a.column_id = table_b.id AND condition;
​```
**Example:**
​```sql
DELETE FROM orders
USING customers
WHERE orders.customer_id = customers.id AND customers.country = 'Nowhere';
​```

---

## ALTER Table
Changes the structure of an existing table — adding/removing columns, or changing constraints. Doesn't touch existing data unless the change requires it.

**Add a column:**
​```sql
ALTER TABLE table_name 
ADD COLUMN column_name datatype;
​```
**Example:**
​```sql
ALTER TABLE students 
ADD COLUMN email VARCHAR(100);
​```

**Remove a column:**
​```sql
ALTER TABLE table_name 
DROP COLUMN column_name;
​```
**Example:**
​```sql
ALTER TABLE students 
DROP COLUMN email;
​```

**Add a constraint:**
​```sql
ALTER TABLE table_name 
ADD CONSTRAINT constraint_name UNIQUE (column_name);

ALTER TABLE table_name 
ALTER COLUMN column_name 
SET NOT NULL;
​```
**Example:**
​```sql
ALTER TABLE students 
ADD CONSTRAINT unique_email UNIQUE (email);
​```

**Remove a constraint:**
​```sql
ALTER TABLE table_name 
DROP CONSTRAINT constraint_name;

ALTER TABLE table_name 
ALTER COLUMN column_name 
DROP NOT NULL;
​```
**Example:**
​```sql
ALTER TABLE students 
DROP CONSTRAINT unique_email;
​```
**Gotcha:** Constraints you didn't explicitly name (like an inline `CHECK` or `UNIQUE`) get an auto-generated name by Postgres — you may need to look it up before you can drop it:
​```sql
SELECT conname FROM pg_constraint WHERE conrelid = 'table_name'::regclass;
​```

---

## DROP Table
Permanently deletes a table and all its data — structure included, not just rows.
​```sql
ALTER TABLE table_name 
DROP COLUMN column_name;
​```
Check for evidence:
​```sql
ALTER TABLE table_name 
DROP COLUMN IF EXISTS column_name;
​```

**DROP TABLE ... CASCADE**
If other tables have foreign keys referencing this table, a plain `DROP TABLE` will fail. `CASCADE` forces it through by also dropping any dependent objects (like foreign key constraints in other tables) that rely on it.
​```sql
ALTER TABLE table_name 
DROP COLUMN column_name CASCADE;
​```
**Example:**
​```sql
-- if "orders" has a foreign key referencing "customers"
DROP TABLE customers CASCADE;
-- this drops customers AND removes the dependent foreign key constraint in orders
​```
**Gotcha:** `CASCADE` doesn't just warn you — it silently removes dependent constraints/objects too. Always be sure of what depends on a table before using it, since this can have ripple effects you don't immediately see.

---

## CHECK Constraint
Ensures values in a column meet a specific condition before they can be inserted or updated.
​```sql
column_name datatype CHECK (condition)
​```
**Example (at table creation):**
​```sql
CREATE TABLE products (
    price NUMERIC CHECK (price > 0),
    discount NUMERIC CHECK (discount >= 0 AND discount <= 100)
);
​```
**Example (added later):**
​```sql
ALTER TABLE products ADD CONSTRAINT positive_price CHECK (price > 0);
​```
**When I'd use it:** Enforcing business rules directly in the database — e.g. a price can never be negative — so bad data can't even get inserted, regardless of what app or query tries to.

## RETURNING
Displays the value(s) of the row(s) affected by an `INSERT`, `UPDATE`, or `DELETE`, without needing a separate `SELECT` afterward.
​```sql
INSERT INTO table_name (column1, column2)
VALUES (value1, value2)
RETURNING column1, column2;

UPDATE table_name
SET column1 = value1
WHERE condition
RETURNING *;

DELETE FROM table_name
WHERE condition
RETURNING *;
​```
**Example:**
​```sql
INSERT INTO students (name, age)
VALUES ('Sofia', 21)
RETURNING id;
​```
This inserts the row and immediately gives you back the auto-generated `id` for it — handy when you need that id right away (e.g. to insert a related row elsewhere).

**Example with UPDATE:**
​```sql
UPDATE employees
SET salary = salary * 1.1
WHERE department = 'Sales'
RETURNING name, salary;
​```
Shows the updated names and new salaries, confirming exactly which rows changed and what they changed to.

**When I'd use it:** Confirming what actually happened after a write operation — especially useful for `INSERT` when you need a generated `id`, or for `UPDATE`/`DELETE` when you want to double-check exactly which rows were affected, without running a second query.

**Gotcha:** `RETURNING *` returns every column of the affected rows — fine for checking your work, but if you only need one or two values, naming them explicitly (`RETURNING id`) keeps the output cleaner.