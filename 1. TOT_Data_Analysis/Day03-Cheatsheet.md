# 📊 Day 3 — SQL for Analytics in Colab Cheatsheet

> **DICT Data Analytics — Train the Trainer**
> SQLite + pandas + Colab
> **Focus:** SELECT, WHERE, ORDER BY, LIMIT, GROUP BY, HAVING, JOIN, validation

---

# 🧠 1. Core Mental Model

Think of a SQL table like an Excel worksheet:

| SQL        | Think of it as               |
| ---------- | ---------------------------- |
| Table      | Excel sheet                  |
| Row        | One record                   |
| Column     | One field/detail             |
| `SELECT`   | Which columns do I want?     |
| `FROM`     | Which table?                 |
| `WHERE`    | Which rows?                  |
| `GROUP BY` | How should I group rows?     |
| `HAVING`   | Which groups should I keep?  |
| `ORDER BY` | How should I sort?           |
| `LIMIT`    | How many rows should I show? |
| `JOIN`     | Combine related tables       |

### Basic SQL sentence

```sql
SELECT columns
FROM table
WHERE condition
GROUP BY columns
HAVING condition
ORDER BY column DESC
LIMIT 5;
```

---

# 🐍 2. Run SQL from pandas

The main pattern used throughout Day 3:

```python
pd.read_sql("""
    SELECT ...
    FROM ...
    WHERE ...
""", conn)
```

### What each part means

```python
pd.read_sql(SQL_QUERY, conn)
```

| Part            | Purpose                                                  |
| --------------- | -------------------------------------------------------- |
| `pd.read_sql()` | Sends SQL to the database and returns a pandas DataFrame |
| SQL query       | Tells SQLite what data to retrieve                       |
| `conn`          | SQLite database connection                               |

Example:

```python
df = pd.read_sql("""
    SELECT *
    FROM service_requests
    LIMIT 5
""", conn)
```

---

# 🔌 3. SQLite Connection

```python
import sqlite3

conn = sqlite3.connect("dict_analytics.db")
```

Then query the database:

```python
pd.read_sql("""
    SELECT *
    FROM service_requests
""", conn)
```

### Remember

```text
SQLite database
      ↓
    conn
      ↓
pd.read_sql()
      ↓
pandas DataFrame
```

---

# 🔎 4. The Four-Query First Look

Before querying an unfamiliar table, inspect it first.

## 4.1 See sample rows

```python
pd.read_sql("""
    SELECT *
    FROM service_requests
    LIMIT 5
""", conn)
```

### Key idea

```sql
SELECT *
```

means:

> Give me all columns.

```sql
LIMIT 5
```

means:

> Show only 5 rows.

---

## 4.2 Count rows

```python
n = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM service_requests
""", conn)

n
```

Or directly extract the number:

```python
total_rows = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM service_requests
""", conn)["n"].iloc[0]
```

Result:

```text
1222
```

### `AS`

Creates an alias:

```sql
COUNT(*) AS n
```

Instead of a long column name, the result is simply called `n`.

---

# 🔍 5. Inspect SQLite Table Structure

```python
columns = pd.read_sql("""
    PRAGMA table_info(service_requests)
""", conn)

columns[["name", "type"]]
```

### `PRAGMA table_info()`

Returns metadata about a table.

Important fields:

| Field        | Meaning                 |
| ------------ | ----------------------- |
| `cid`        | Column position         |
| `name`       | Column name             |
| `type`       | Data type               |
| `notnull`    | Whether NULL is allowed |
| `dflt_value` | Default value           |
| `pk`         | Primary-key indicator   |

---

# 🏷️ 6. Find Unique Values

```python
pd.read_sql("""
    SELECT DISTINCT status
    FROM service_requests
    ORDER BY status
""", conn)
```

Another example:

```python
pd.read_sql("""
    SELECT DISTINCT region
    FROM service_requests
    ORDER BY region
""", conn)
```

### `DISTINCT`

Returns unique values only.

Useful for checking categorical data before analysis.

### Example discovery

The region column contained:

```text
NCR
ncr
Region III
Region IV-A
Region VI
Region VII
Region XI
```

⚠️ `NCR` and `ncr` are separate values.

This is a **data-quality issue**, not a SQL query error.

---

# 🎯 7. SELECT — Choose Columns

Instead of:

```sql
SELECT *
```

choose only what you need:

```python
pd.read_sql("""
    SELECT request_id, region, service_type, days_to_resolve
    FROM service_requests
""", conn)
```

### Mental translation

```text
SELECT request_id, region, service_type, days_to_resolve
FROM service_requests
```

=

> Get these four columns from the service_requests table.

---

# 🚦 8. WHERE — Filter Rows

Example:

```python
pd.read_sql("""
    SELECT request_id, service_type, channel
    FROM service_requests
    WHERE status = 'Pending'
""", conn)
```

### `WHERE`

Filters **rows before grouping**.

---

# 🔗 9. Multiple Conditions with AND

Example:

```python
pending_r7 = pd.read_sql("""
    SELECT request_id, service_type, channel
    FROM service_requests
    WHERE status = 'Pending'
      AND region = 'Region VII'
""", conn)
```

Result:

```text
40 rows
```

### Important

SQL string values use single quotes:

```sql
status = 'Pending'
```

Not:

```sql
status = Pending
```

### `AND`

Both conditions must be true.

```sql
WHERE condition_1
  AND condition_2
```

---

# ↕️ 10. ORDER BY — Sort Results

### Ascending

```sql
ORDER BY processing_fee ASC
```

Smallest → largest.

### Descending

```sql
ORDER BY processing_fee DESC
```

Largest → smallest.

---

## Example: Highest Processing Fees

```python
top_fees = pd.read_sql("""
    SELECT request_id, region, processing_fee
    FROM service_requests
    ORDER BY processing_fee DESC
    LIMIT 5
""", conn)
```

All five results had:

```text
processing_fee = 200.0
```

### Pattern

```text
SELECT
FROM
ORDER BY
LIMIT
```

---

# 🔢 11. LIMIT — Restrict Number of Rows

```sql
LIMIT 5
```

means:

> Return only the first 5 rows after the other operations.

Common use:

```sql
SELECT *
FROM service_requests
LIMIT 10;
```

or:

```sql
ORDER BY processing_fee DESC
LIMIT 5;
```

---

# 🐢 12. Finding the Slowest Requests

```python
pd.read_sql("""
    SELECT request_id,
           region,
           service_type,
           days_to_resolve
    FROM service_requests
    WHERE status = 'Escalated'
    ORDER BY days_to_resolve DESC
    LIMIT 5
""", conn)
```

### Read it as a sentence

> Select request ID, region, service type, and days to resolve from service requests where status is Escalated, sort by days to resolve from largest to smallest, and show five rows.

---

# 🧮 13. COUNT()

## Count all rows

```sql
COUNT(*)
```

Counts every row, including rows containing NULL values.

Example:

```python
count_star = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM service_requests
""", conn)["n"].iloc[0]
```

---

## Count a specific column

```sql
COUNT(days_to_resolve)
```

Counts only rows where `days_to_resolve` is **not NULL**.

Example:

```python
count_days = pd.read_sql("""
    SELECT COUNT(days_to_resolve) AS n
    FROM service_requests
""", conn)["n"].iloc[0]
```

Results:

```text
COUNT(*)                 = 1222
COUNT(days_to_resolve)  = 824
```

Difference:

```python
count_diff = count_star - count_days
```

```text
398
```

### ⭐ Important rule

```text
COUNT(*)          → counts rows
COUNT(column)     → counts non-NULL values
```

---

# 📦 14. GROUP BY — Summarize Groups

Example:

```python
by_service = pd.read_sql("""
    SELECT
        service_type,
        COUNT(*) AS request_count,
        ROUND(AVG(days_to_resolve), 2) AS avg_days_to_resolve
    FROM service_requests
    GROUP BY service_type
    ORDER BY request_count DESC
""", conn)
```

### What happens?

Instead of 1,222 rows, SQL produces one row per `service_type`.

---

# 📊 15. GROUP BY + COUNT

Basic pattern:

```sql
SELECT category,
       COUNT(*) AS count
FROM table
GROUP BY category
ORDER BY count DESC;
```

Example:

```python
pd.read_sql("""
    SELECT region,
           COUNT(*) AS total_requests
    FROM service_requests
    GROUP BY region
    ORDER BY total_requests DESC
""", conn)
```

---

# 📐 16. AVG()

Calculate an average:

```sql
AVG(days_to_resolve)
```

Example:

```sql
ROUND(AVG(days_to_resolve), 2)
```

`ROUND(..., 2)` means round to two decimal places.

Example:

```python
ROUND(AVG(days_to_resolve), 2) AS avg_days
```

---

# 🧮 17. Common SQL Aggregate Functions

| Function        | Purpose               |
| --------------- | --------------------- |
| `COUNT(*)`      | Count rows            |
| `COUNT(column)` | Count non-NULL values |
| `AVG(column)`   | Average               |
| `SUM(column)`   | Total                 |
| `MIN(column)`   | Smallest              |
| `MAX(column)`   | Largest               |

Example:

```sql
SELECT
    COUNT(*) AS total,
    AVG(processing_fee) AS avg_fee,
    MIN(processing_fee) AS min_fee,
    MAX(processing_fee) AS max_fee
FROM service_requests;
```

---

# 🚧 18. WHERE vs HAVING

This is one of the most important Day 3 concepts.

## WHERE

Filters **individual rows**.

```sql
WHERE status = 'Resolved'
```

Use it **before grouping**.

---

## HAVING

Filters **groups after aggregation**.

```sql
HAVING COUNT(*) > 130
```

Use it when the condition involves an aggregate.

---

# ❌ 19. Common HAVING Mistake

This is wrong:

```sql
SELECT office_code,
       COUNT(*) AS n
FROM service_requests
WHERE COUNT(*) > 130
GROUP BY office_code;
```

SQLite gives an error such as:

```text
misuse of aggregate: COUNT()
```

### Why?

`WHERE` operates before `GROUP BY`.

At that point, the grouped `COUNT(*)` does not exist yet.

---

# ✅ 20. Correct HAVING Query

```python
busy_offices = pd.read_sql("""
    SELECT
        office_code,
        COUNT(*) AS request_count
    FROM service_requests
    GROUP BY office_code
    HAVING COUNT(*) > 130
    ORDER BY request_count DESC
""", conn)
```

Result:

```text
DICT-R6-01     218
DICT-R3-01     208
DICT-R11-01    198
```

### Easy rule

```text
WHERE  → filter rows
HAVING → filter groups
```

---

# 🧠 21. SQL Writing Order vs Execution Order

### You normally WRITE SQL like this:

```text
SELECT
FROM
WHERE
GROUP BY
HAVING
ORDER BY
LIMIT
```

### But SQL logically EXECUTES roughly like this:

```text
FROM
WHERE
GROUP BY
HAVING
SELECT
ORDER BY
LIMIT
```

### ⭐ Remember

```text
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT
```

This explains why:

```sql
WHERE COUNT(*) > 130
```

doesn't work.

`COUNT(*)` is created by grouping, which happens after `WHERE`.

---

# 🧹 22. Data-Quality Problems Can Affect GROUP BY

The dataset intentionally contains:

```text
Free WiFi Installation
Free Wifi Installation
```

These are treated as **different categories**.

Therefore:

```sql
GROUP BY service_type
```

produced **7 groups**, even though there are only 6 intended service types.

### Important lesson

SQL does exactly what you tell it.

It does not automatically know that:

```text
Free Wifi Installation
```

and:

```text
Free WiFi Installation
```

mean the same thing.

Data cleaning is a separate step.

---

# 🐼 23. Cross-Check SQL Against pandas

Read the CSV independently:

```python
raw_df = pd.read_csv("data/service_requests.csv")
```

Count rows:

```python
pandas_rowcount = len(raw_df)
```

Compare:

```python
print("pandas rowcount :", pandas_rowcount)
print("SQL rowcount    :", total_rows)
print("Match?           :", pandas_rowcount == total_rows)
```

Expected:

```text
pandas rowcount : 1222
SQL rowcount    : 1222
Match?          : True
```

### ⭐ Validation habit

> Never assume your SQL table contains what you think it contains. Cross-check important counts against another source.

---

# 🔗 24. Working with JSON

Load the JSON:

```python
with open("data/regional_offices.json", "r") as f:
    data = json.load(f)
```

Get the records:

```python
office_records = data["offices"]
```

---

# 📐 25. Flatten Nested JSON

Use:

```python
offices = pd.json_normalize(office_records)
```

This converts nested structures such as:

```text
location
    region
    city
    coordinates
        lat
        lon
```

into columns such as:

```text
location.region
location.city
location.coordinates.lat
location.coordinates.lon
```

### Mental model

```text
Nested JSON
     ↓
pd.json_normalize()
     ↓
Flat DataFrame
```

---

# 📝 26. Convert Python Lists Before SQLite

The JSON contains:

```text
services_offered
```

as a Python list.

SQLite cannot directly store a Python list.

Convert it:

```python
offices["services_offered"] = (
    offices["services_offered"].astype(str)
)
```

Then:

```python
offices.to_sql(
    "offices",
    conn,
    if_exists="replace",
    index=False
)
```

### Important

```text
Python list
   ↓
.astype(str)
   ↓
SQLite-compatible text
```

---

# 🗃️ 27. Load a DataFrame into SQLite

General pattern:

```python
df.to_sql(
    "table_name",
    conn,
    if_exists="replace",
    index=False
)
```

### Parameters

| Parameter             | Meaning                             |
| --------------------- | ----------------------------------- |
| `"table_name"`        | SQLite table name                   |
| `conn`                | Database connection                 |
| `if_exists="replace"` | Replace existing table              |
| `index=False`         | Don't save pandas index as a column |

---

# 🔗 28. JOIN — Combine Tables

The two tables:

```text
service_requests
        |
        | office_code
        ↓
offices
```

Both contain:

```text
office_code
```

This is the key used to connect them.

---

# 🔑 29. INNER JOIN

Basic syntax:

```sql
SELECT columns
FROM table1 AS a
INNER JOIN table2 AS b
    ON a.key = b.key;
```

Day 3 example:

```python
joined = pd.read_sql("""
    SELECT
        sr.request_id,
        sr.region,
        o.office_name,
        o.staff_count
    FROM service_requests AS sr
    INNER JOIN offices AS o
        ON sr.office_code = o.office_code
""", conn)
```

---

# 🏷️ 30. Table Aliases

Instead of repeatedly writing:

```sql
service_requests.office_code
```

use an alias:

```sql
service_requests AS sr
```

Then:

```sql
sr.office_code
```

Likewise:

```sql
offices AS o
```

Then:

```sql
o.office_code
```

### Common pattern

```sql
FROM service_requests AS sr
INNER JOIN offices AS o
    ON sr.office_code = o.office_code
```

---

# 🧪 31. JOIN Validation — The Critical Habit

Before joining:

```python
total_rows = ...
```

After joining:

```python
join_rows = len(joined)
```

Compare:

```python
print("Total rows :", total_rows)
print("Joined rows:", join_rows)
print("Do they match?", join_rows == total_rows)
```

Expected:

```text
Total rows : 1222
Joined rows: 1222
Do they match? True
```

### ⭐ Core rule

```text
COUNT BEFORE JOIN
       ↓
JOIN
       ↓
COUNT AFTER JOIN
       ↓
COMPARE
```

Never assume a join is correct just because SQL ran successfully.

---

# ⚠️ 32. What If JOIN Row Counts Change?

Suppose:

```text
Before JOIN = 1222
After JOIN  = 1100
```

Possible issue:

> Some service requests had no matching office.

An `INNER JOIN` removes unmatched rows.

---

Suppose:

```text
Before JOIN = 1222
After JOIN  = 1500
```

Possible issue:

> The join key is not unique on the other table, causing rows to multiply.

This is a potentially dangerous **many-to-many or one-to-many join problem**.

### Key lesson

```text
Rows disappear → unmatched keys may exist

Rows increase → duplicate/non-unique join keys may exist

Rows unchanged → investigate, but this alone does not prove the join is correct
```

---

# ↔️ 33. INNER JOIN vs LEFT JOIN

## INNER JOIN

Keeps only rows with a match in both tables.

```sql
FROM service_requests sr
INNER JOIN offices o
    ON sr.office_code = o.office_code
```

Conceptually:

```text
Table A ∩ Table B
```

---

## LEFT JOIN

Keeps **all rows from the left table**, even if there is no match.

```sql
FROM service_requests sr
LEFT JOIN offices o
    ON sr.office_code = o.office_code
```

If an office doesn't match:

```text
office_name = NULL
staff_count = NULL
```

### Quick comparison

| JOIN         | Keeps unmatched left rows? |
| ------------ | -------------------------- |
| `INNER JOIN` | ❌ No                       |
| `LEFT JOIN`  | ✅ Yes                      |

---

# 👥 34. GROUP BY Multiple Columns

You can group by more than one column:

```sql
GROUP BY region, service_type
```

Example:

```python
escalated_by_service = pd.read_sql("""
    SELECT
        region,
        service_type,
        COUNT(*) AS escalated_count
    FROM service_requests
    WHERE status = 'Escalated'
    GROUP BY region, service_type
    ORDER BY escalated_count DESC
""", conn)
```

This produces one group for each:

```text
region + service_type
```

combination.

---

# 👨‍💼 35. Workload per Staff Member

The Day 3 workload calculation:

```python
workload = pd.read_sql("""
    SELECT
        o.office_name,
        o.staff_count,
        COUNT(sr.request_id) AS request_count,
        ROUND(
            COUNT(sr.request_id) * 1.0 / o.staff_count,
            2
        ) AS requests_per_staff
    FROM offices AS o
    INNER JOIN service_requests AS sr
        ON o.office_code = sr.office_code
    GROUP BY
        o.office_code,
        o.office_name,
        o.staff_count
    ORDER BY requests_per_staff DESC
""", conn)
```

---

# ⚠️ 36. SQLite Integer Division

SQLite can perform integer division.

For example:

```sql
COUNT(sr.request_id) / o.staff_count
```

may truncate decimal results when both values are integers.

### ❌ Risky

```sql
COUNT(sr.request_id) / o.staff_count
```

### ✅ Force decimal division

```sql
COUNT(sr.request_id) * 1.0 / o.staff_count
```

Then round:

```sql
ROUND(
    COUNT(sr.request_id) * 1.0 / o.staff_count,
    2
)
```

Expected top result:

```text
12.25
```

not:

```text
12
```

### Alternative

Use `CAST`:

```sql
CAST(COUNT(sr.request_id) AS REAL) / o.staff_count
```

---

# 📊 37. Complete Workload Pattern

```sql
SELECT
    o.office_name,
    o.staff_count,
    COUNT(sr.request_id) AS request_count,
    ROUND(
        COUNT(sr.request_id) * 1.0 / o.staff_count,
        2
    ) AS requests_per_staff
FROM offices AS o
INNER JOIN service_requests AS sr
    ON o.office_code = sr.office_code
GROUP BY
    o.office_code,
    o.office_name,
    o.staff_count
ORDER BY requests_per_staff DESC;
```

### Think step-by-step

```text
1. Join offices + requests
2. Group by office
3. Count requests
4. Divide requests by staff
5. Round to 2 decimals
6. Sort highest → lowest
```

---

# 🧩 38. The Complete SQL Pattern

For many analytics tasks, start with:

```sql
SELECT
    grouping_column,
    COUNT(*) AS count,
    ROUND(AVG(numeric_column), 2) AS average
FROM table
WHERE row_condition
GROUP BY grouping_column
HAVING COUNT(*) > threshold
ORDER BY count DESC
LIMIT 10;
```

Not every clause is required.

---

# 🧭 39. SQL Clause Cheat Sheet

| Clause     | Question it answers            |
| ---------- | ------------------------------ |
| `SELECT`   | What do I want to see?         |
| `FROM`     | Where is the data?             |
| `WHERE`    | Which rows should I keep?      |
| `GROUP BY` | How should I summarize?        |
| `HAVING`   | Which groups should I keep?    |
| `ORDER BY` | How should I sort?             |
| `LIMIT`    | How many rows do I want?       |
| `JOIN`     | Which tables should I combine? |
| `ON`       | How are the tables related?    |

---

# 🧠 40. WHERE vs HAVING — Memorize This

```text
WHERE
  ↓
Filters ROWS
  ↓
GROUP BY
  ↓
Creates GROUPS
  ↓
HAVING
  ↓
Filters GROUPS
```

### Example

Filter rows:

```sql
WHERE status = 'Resolved'
```

Filter groups:

```sql
HAVING COUNT(*) > 130
```

---

# 🔢 41. COUNT(*) vs COUNT(column)

```text
COUNT(*) 
→ counts every row

COUNT(column)
→ counts only non-NULL values
```

Example:

```sql
COUNT(*)
```

= `1222`

```sql
COUNT(days_to_resolve)
```

= `824`

Difference:

```text
398
```

---

# 🧪 42. Validation Checklist

Whenever you perform SQL analysis, ask:

### Before query

```text
☐ What table am I querying?
☐ How many rows?
☐ What columns exist?
☐ What data types?
☐ What categorical values exist?
```

### After filtering

```text
☐ Does the row count make sense?
☐ Are the conditions correct?
☐ Are string values spelled/cased correctly?
```

### After GROUP BY

```text
☐ Does the number of groups make sense?
☐ Are unexpected categories present?
☐ Are NULL values affecting the aggregate?
```

### After JOIN

```text
☐ How many rows before?
☐ How many rows after?
☐ Did rows disappear?
☐ Did rows multiply?
☐ Is the join key unique?
```

---

# 🛠️ 43. Common SQL Mistakes

## Mistake 1 — Missing quotes

❌

```sql
WHERE status = Pending
```

✅

```sql
WHERE status = 'Pending'
```

---

## Mistake 2 — Using WHERE for aggregate conditions

❌

```sql
WHERE COUNT(*) > 130
```

✅

```sql
HAVING COUNT(*) > 130
```

---

## Mistake 3 — Forgetting GROUP BY

❌

```sql
SELECT region, COUNT(*)
FROM service_requests;
```

If you want a count per region, use:

```sql
GROUP BY region
```

---

## Mistake 4 — Assuming categories are clean

```text
NCR
ncr
```

are different values.

Also:

```text
Free WiFi Installation
Free Wifi Installation
```

are different values.

Check with:

```sql
SELECT DISTINCT column
FROM table;
```

---

## Mistake 5 — Integer division

❌

```sql
COUNT(*) / staff_count
```

✅

```sql
COUNT(*) * 1.0 / staff_count
```

---

## Mistake 6 — Not validating a JOIN

Don't just run:

```sql
JOIN ...
```

Check:

```text
before = row count
after  = row count
```

---

# 🧰 44. Essential pandas + SQL Patterns

## SQL → DataFrame

```python
df = pd.read_sql("""
    SELECT *
    FROM table
""", conn)
```

## DataFrame → SQLite

```python
df.to_sql(
    "table",
    conn,
    if_exists="replace",
    index=False
)
```

## JSON → Python

```python
with open("file.json", "r") as f:
    data = json.load(f)
```

## Nested JSON → DataFrame

```python
df = pd.json_normalize(data["records"])
```

## CSV → pandas

```python
df = pd.read_csv("data/file.csv")
```

## Count DataFrame rows

```python
len(df)
```

---

# 🏆 45. Day 3 Core Code Templates

## Basic query

```python
result = pd.read_sql("""
    SELECT column1, column2
    FROM table
""", conn)
```

## Filter

```python
result = pd.read_sql("""
    SELECT column1, column2
    FROM table
    WHERE status = 'Pending'
""", conn)
```

## Multiple filters

```python
result = pd.read_sql("""
    SELECT column1, column2
    FROM table
    WHERE status = 'Pending'
      AND region = 'Region VII'
""", conn)
```

## Sort + limit

```python
result = pd.read_sql("""
    SELECT column1, column2
    FROM table
    ORDER BY column2 DESC
    LIMIT 5
""", conn)
```

## Count

```python
result = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM table
""", conn)
```

## Group + count

```python
result = pd.read_sql("""
    SELECT category,
           COUNT(*) AS n
    FROM table
    GROUP BY category
    ORDER BY n DESC
""", conn)
```

## Group + average

```python
result = pd.read_sql("""
    SELECT category,
           COUNT(*) AS n,
           ROUND(AVG(value), 2) AS avg_value
    FROM table
    GROUP BY category
""", conn)
```

## WHERE + GROUP BY

```python
result = pd.read_sql("""
    SELECT channel,
           COUNT(*) AS n
    FROM table
    WHERE status = 'Resolved'
    GROUP BY channel
    ORDER BY n DESC
""", conn)
```

## GROUP BY + HAVING

```python
result = pd.read_sql("""
    SELECT office_code,
           COUNT(*) AS n
    FROM table
    GROUP BY office_code
    HAVING COUNT(*) > 130
    ORDER BY n DESC
""", conn)
```

## JOIN

```python
result = pd.read_sql("""
    SELECT
        a.id,
        a.region,
        b.office_name
    FROM table_a AS a
    INNER JOIN table_b AS b
        ON a.office_code = b.office_code
""", conn)
```

## JOIN + GROUP BY

```python
result = pd.read_sql("""
    SELECT
        b.office_name,
        b.staff_count,
        COUNT(a.request_id) AS request_count,
        ROUND(
            COUNT(a.request_id) * 1.0 / b.staff_count,
            2
        ) AS requests_per_staff
    FROM table_b AS b
    INNER JOIN table_a AS a
        ON b.office_code = a.office_code
    GROUP BY
        b.office_code,
        b.office_name,
        b.staff_count
    ORDER BY requests_per_staff DESC
""", conn)
```

---

# 🚀 46. Day 3 "Summarize Main Code" Box

```python
# ==========================================
# DAY 3 — SQL FOR ANALYTICS QUICK REFERENCE
# ==========================================

# SQL → pandas DataFrame
result = pd.read_sql("""
    SELECT *
    FROM service_requests
    LIMIT 5
""", conn)

# Count rows
total_rows = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM service_requests
""", conn)["n"].iloc[0]

# Inspect table structure
columns = pd.read_sql("""
    PRAGMA table_info(service_requests)
""", conn)

# Unique values
pd.read_sql("""
    SELECT DISTINCT status
    FROM service_requests
    ORDER BY status
""", conn)

# Filter rows
pending_r7 = pd.read_sql("""
    SELECT request_id, service_type, channel
    FROM service_requests
    WHERE status = 'Pending'
      AND region = 'Region VII'
""", conn)

# Sort + LIMIT
top_fees = pd.read_sql("""
    SELECT request_id, region, processing_fee
    FROM service_requests
    ORDER BY processing_fee DESC
    LIMIT 5
""", conn)

# GROUP BY + COUNT + AVG
by_service = pd.read_sql("""
    SELECT
        service_type,
        COUNT(*) AS request_count,
        ROUND(AVG(days_to_resolve), 2) AS avg_days
    FROM service_requests
    GROUP BY service_type
    ORDER BY request_count DESC
""", conn)

# WHERE before GROUP BY
by_channel = pd.read_sql("""
    SELECT
        channel,
        COUNT(*) AS resolved_requests
    FROM service_requests
    WHERE status = 'Resolved'
    GROUP BY channel
    ORDER BY resolved_requests DESC
""", conn)

# HAVING filters groups
busy_offices = pd.read_sql("""
    SELECT
        office_code,
        COUNT(*) AS request_count
    FROM service_requests
    GROUP BY office_code
    HAVING COUNT(*) > 130
    ORDER BY request_count DESC
""", conn)

# COUNT(*) vs COUNT(column)
count_star = pd.read_sql("""
    SELECT COUNT(*) AS n
    FROM service_requests
""", conn)["n"].iloc[0]

count_days = pd.read_sql("""
    SELECT COUNT(days_to_resolve) AS n
    FROM service_requests
""", conn)["n"].iloc[0]

count_diff = count_star - count_days

# JSON → DataFrame
with open("data/regional_offices.json", "r") as f:
    data = json.load(f)

offices = pd.json_normalize(data["offices"])

# Python list → SQLite-compatible text
offices["services_offered"] = (
    offices["services_offered"].astype(str)
)

# DataFrame → SQLite
offices.to_sql(
    "offices",
    conn,
    if_exists="replace",
    index=False
)

# INNER JOIN
joined = pd.read_sql("""
    SELECT
        sr.request_id,
        sr.region,
        o.office_name,
        o.staff_count
    FROM service_requests AS sr
    INNER JOIN offices AS o
        ON sr.office_code = o.office_code
""", conn)

# Validate JOIN row count
join_rows = len(joined)

print("Before JOIN:", total_rows)
print("After JOIN :", join_rows)
print("Match?     :", total_rows == join_rows)

# JOIN + GROUP BY + workload calculation
workload = pd.read_sql("""
    SELECT
        o.office_name,
        o.staff_count,
        COUNT(sr.request_id) AS request_count,
        ROUND(
            COUNT(sr.request_id) * 1.0 / o.staff_count,
            2
        ) AS requests_per_staff
    FROM offices AS o
    INNER JOIN service_requests AS sr
        ON o.office_code = sr.office_code
    GROUP BY
        o.office_code,
        o.office_name,
        o.staff_count
    ORDER BY requests_per_staff DESC
""", conn)
```

---

# 🧠 47. One-Minute Memory Cheat Sheet

```text
SELECT   = what columns?
FROM     = which table?
WHERE    = which rows?
GROUP BY = summarize by what?
HAVING   = which groups?
ORDER BY = sort how?
LIMIT    = how many?

COUNT(*)       = all rows
COUNT(column)  = non-NULL values
AVG(column)    = average
SUM(column)    = total
MIN(column)    = minimum
MAX(column)    = maximum

DISTINCT       = unique values
AS             = alias
ASC            = smallest → largest
DESC           = largest → smallest

INNER JOIN     = matching rows only
LEFT JOIN      = keep all left-table rows
ON             = join condition

GROUP BY happens before HAVING.
WHERE filters rows.
HAVING filters groups.

JOIN:
COUNT BEFORE → JOIN → COUNT AFTER → COMPARE

SQLite decimal division:
COUNT(*) * 1.0 / staff_count

JSON:
json.load()
    ↓
data["offices"]
    ↓
pd.json_normalize()
    ↓
astype(str)
    ↓
to_sql()
```

---

# 🎯 48. The Day 3 SQL Formula

When faced with a new analytics question, think:

```text
1. WHAT do I need?
        ↓
     SELECT

2. WHERE is it?
        ↓
      FROM

3. WHICH ROWS?
        ↓
      WHERE

4. DO I NEED GROUPS?
        ↓
     GROUP BY

5. DO I NEED TO FILTER GROUPS?
        ↓
      HAVING

6. HOW SHOULD IT BE SORTED?
        ↓
     ORDER BY

7. HOW MANY RESULTS?
        ↓
      LIMIT

8. AM I COMBINING TABLES?
        ↓
       JOIN

9. DID THE JOIN CHANGE ROW COUNT?
        ↓
      VALIDATE
```

---

# ⭐ 49. Most Important Lessons from Day 3

### 1. SQL is a question language

Don't memorize SQL as random syntax.

Think:

> **What question am I asking the database?**

---

### 2. `WHERE` and `HAVING` are different

```text
WHERE  → rows
HAVING → groups
```

---

### 3. `COUNT(*)` and `COUNT(column)` are different

```text
COUNT(*)       → includes NULL rows
COUNT(column)  → excludes NULL values
```

---

### 4. Dirty data affects analysis

```text
NCR
ncr
```

and:

```text
Free WiFi Installation
Free Wifi Installation
```

can become separate groups.

---

### 5. A successful JOIN is not necessarily a correct JOIN

Always check:

```text
Rows before
Rows after
Join key uniqueness
```

---

### 6. SQLite can silently produce misleading calculations

For ratios:

```sql
COUNT(*) * 1.0 / staff_count
```

forces decimal division.

---

### 7. Validate your results

The Day 3 habit is:

```text
QUERY
  ↓
CHECK
  ↓
CROSS-CHECK
  ↓
INTERPRET
```

Not simply:

```text
QUERY → TRUST RESULT
```

---

# 🏁 Day 3 Final Checklist

Before finishing Day 3, you should be able to:

* [ ] Connect to SQLite with `sqlite3`
* [ ] Use `pd.read_sql()`
* [ ] Use `SELECT`
* [ ] Use `WHERE`
* [ ] Combine conditions with `AND`
* [ ] Use `ORDER BY`
* [ ] Use `ASC` and `DESC`
* [ ] Use `LIMIT`
* [ ] Use `COUNT(*)`
* [ ] Understand `COUNT(column)`
* [ ] Use `AVG()`
* [ ] Use `ROUND()`
* [ ] Use `GROUP BY`
* [ ] Understand `WHERE` vs `HAVING`
* [ ] Use `HAVING COUNT(*)`
* [ ] Inspect unique categories with `DISTINCT`
* [ ] Load JSON with `json.load()`
* [ ] Flatten JSON with `pd.json_normalize()`
* [ ] Convert lists with `.astype(str)`
* [ ] Load DataFrames into SQLite with `.to_sql()`
* [ ] Use `INNER JOIN`
* [ ] Understand `LEFT JOIN`
* [ ] Use table aliases
* [ ] Join using `ON`
* [ ] Validate row counts before and after JOIN
* [ ] Avoid SQLite integer division
* [ ] Cross-check SQL results against pandas
* [ ] Recognize data-quality problems hiding inside categories

---

# 💡 Teaching Principle to Carry Forward

> **Don't just teach people how to write a query. Teach them how to know whether the query answered the right question.**

The most important Day 3 workflow is:

```text
LOOK
 ↓
ASK
 ↓
QUERY
 ↓
CHECK
 ↓
VALIDATE
 ↓
INTERPRET
```
