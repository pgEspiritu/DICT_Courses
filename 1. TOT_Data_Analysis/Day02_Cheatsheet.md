# 🐼 Day 2 — pandas Fundamentals Cheatsheet

### DICT Data Analytics — Train the Trainer · Participant Notebook

**Python for Analytics: Series, DataFrames, filtering, sorting, and derived columns**

**TESDA Alignment:** *Prepare data sets* — import data and apply basic data manipulation.

---

## 📚 Table of Contents

1. [Day 2 Learning Goals](#-day-2-learning-goals)
2. [Setup and Loading Data](#-setup-and-loading-data)
3. [The Four-Command First Look](#-the-four-command-first-look)
4. [Section 1 — Python Fundamentals](#-section-1--python-fundamentals)
5. [Functions](#-functions)
6. [List Comprehensions](#-list-comprehensions)
7. [Dictionary Comprehensions](#-dictionary-comprehensions)
8. [Counting Values](#-counting-values)
9. [Section 2 — Filtering and Sorting](#-section-2--filtering-and-sorting)
10. [Boolean Filtering](#-boolean-filtering)
11. [Multiple Conditions](#-multiple-conditions)
12. [`.isin()`](#-isin)
13. [`.nlargest()`](#-nlargest)
14. [Missing Values](#-missing-values)
15. [Section 3 — Derived Columns](#-section-3--derived-columns)
16. [Date Parsing](#-date-parsing)
17. [Extracting Month Names](#-extracting-month-names)
18. [Cleaning Text](#-cleaning-text)
19. [Applying Functions to Columns](#-applying-functions-to-columns)
20. [`.loc` for Conditional Assignment](#-loc-for-conditional-assignment)
21. [Exporting a DataFrame](#-exporting-a-dataframe)
22. [Complete Task Answers](#-complete-task-answers)
23. [Quick Reference](#-quick-reference)

---

# 🎯 Day 2 Learning Goals

By the end of Day 2, you should be able to:

* Import a CSV file into pandas.
* Understand the difference between a **DataFrame** and a **Series**.
* Inspect a dataset before analyzing it.
* Write basic Python functions.
* Use list and dictionary comprehensions.
* Filter DataFrames using Boolean conditions.
* Combine multiple filtering conditions.
* Use `.isin()` to match multiple values.
* Sort or select extreme values with `.nlargest()`.
* Detect missing values with `.isna()`.
* Parse text dates into real datetime values.
* Create derived columns.
* Clean text values with `.str.strip()` and `.str.upper()`.
* Apply a custom function to a pandas column.
* Assign values conditionally with `.loc`.
* Export a cleaned working subset to CSV.

---

# 🐼 Setup and Loading Data

## Import pandas

```python
import pandas as pd
```

`pandas` is the primary Python library used for working with tabular data.

Common convention:

```python
import pandas as pd
```

---

## Import NumPy

```python
import numpy as np
```

NumPy provides numerical operations and tools such as:

```python
np.nan
```

`np.nan` represents a missing numerical value.

---

## Import `os`

```python
import os
```

The `os` module allows Python to interact with the operating system.

Example:

```python
os.path.exists("file.csv")
```

Checks whether a file exists.

---

## Read a CSV file

```python
df = pd.read_csv("citizen_service_requests.csv")
```

This loads the CSV into a pandas **DataFrame**.

Think of:

```text
CSV file
   ↓
pd.read_csv()
   ↓
DataFrame
```

---

## Check the DataFrame shape

```python
df.shape
```

Example:

```text
(1200, 10)
```

The result is:

```text
(rows, columns)
```

Therefore:

```text
1200 rows
10 columns
```

---

# 🔍 The Four-Command First Look

> **Never analyze a file you have not looked at.**

Run these commands before doing analysis:

```python
df.head()
df.info()
df.describe()
df["status"].value_counts()
```

---

## 1. `df.head()`

```python
df.head()
```

Shows the first 5 rows.

Useful for checking:

* column names
* sample values
* data structure
* obvious formatting problems

You can specify the number of rows:

```python
df.head(10)
```

---

## 2. `df.info()`

```python
df.info()
```

Shows:

* number of rows
* column names
* number of non-null values
* data types
* memory usage

Example:

```text
RangeIndex: 1200 entries
Data columns: 10
```

Important data types:

| dtype        | Meaning                  |
| ------------ | ------------------------ |
| `object`     | Usually text/string data |
| `int64`      | Integer                  |
| `float64`    | Decimal/numeric data     |
| `bool`       | True/False               |
| `datetime64` | Date/time                |

### Important observation

If a date column appears as:

```text
object
```

it may still be text rather than a real date.

---

## 3. `df.describe()`

```python
df.describe()
```

Provides descriptive statistics for numeric columns.

Common outputs:

| Statistic | Meaning                      |
| --------- | ---------------------------- |
| `count`   | Number of non-missing values |
| `mean`    | Average                      |
| `std`     | Standard deviation           |
| `min`     | Minimum                      |
| `25%`     | First quartile               |
| `50%`     | Median                       |
| `75%`     | Third quartile               |
| `max`     | Maximum                      |

---

## 4. `value_counts()`

```python
df["status"].value_counts()
```

Counts how many times each distinct value appears.

Example:

```text
Resolved       523
Closed         210
Pending        177
In Progress    162
Escalated       86
Withdrawn       42
```

This is especially useful for categorical columns.

---

# 🐍 Section 1 — Python Fundamentals

## Functions

A function packages reusable logic.

Basic structure:

```python
def function_name(parameter):
    # calculation
    return result
```

Example:

```python
def classify_speed(days):
    if pd.isna(days):
        return "unknown"
    elif days <= 3:
        return "fast"
    elif days <= 10:
        return "normal"
    else:
        return "slow"
```

---

# ⚡ `if`, `elif`, and `else`

Python evaluates conditions from top to bottom.

```python
if condition1:
    ...
elif condition2:
    ...
else:
    ...
```

For `classify_speed()`:

```text
Missing       → unknown
3 days or less → fast
4–10 days     → normal
More than 10  → slow
```

### Important

The order of conditions matters.

For example:

```python
if days <= 10:
    return "normal"
elif days <= 3:
    return "fast"
```

would be wrong because values such as `2` would already satisfy `days <= 10`.

---

# 🧩 Missing Values

Use:

```python
pd.isna(value)
```

or:

```python
pd.isnull(value)
```

Example:

```python
pd.isna(np.nan)
```

returns:

```text
True
```

For a DataFrame column:

```python
df["processing_fee"].isna()
```

returns a Boolean Series.

---

# 📝 Task 1.1 — `classify_speed(days)`

### Question

Turn a resolution time into a service level label:

* 3 days or fewer → `'fast'`
* 10 days or fewer → `'normal'`
* more than 10 → `'slow'`
* missing value → `'unknown'`

### Answer

```python
def classify_speed(days):
    if pd.isna(days):
        return "unknown"
    elif days <= 3:
        return "fast"
    elif days <= 10:
        return "normal"
    else:
        return "slow"
```

### Test

```python
classify_speed(2)
```

Output:

```text
fast
```

```python
classify_speed(7)
```

Output:

```text
normal
```

```python
classify_speed(30)
```

Output:

```text
slow
```

```python
classify_speed(np.nan)
```

Output:

```text
unknown
```

---

# 📋 List Comprehensions

A list comprehension is a compact way to create a list.

General structure:

```python
[expression for item in iterable]
```

With a condition:

```python
[expression for item in iterable if condition]
```

---

## Normal loop

```python
big_sales = []

for s in SALES:
    if s > 5000:
        big_sales.append(s)
```

---

## List comprehension

The same operation can be written:

```python
big_sales = [s for s in SALES if s > 5000]
```

This means:

```text
for every s in SALES
       ↓
check whether s > 5000
       ↓
keep s if True
```

---

# 📝 Task 1.2 — Test `classify_speed`

Create:

```python
speed_labels = [
    classify_speed(d)
    for d in [2, 7, 30, np.nan]
]
```

Result:

```python
['fast', 'normal', 'slow', 'unknown']
```

---

# 📝 Task 1.3 — Keep Sales Above 5000

Given:

```python
SALES = [4200, 7100, 3900, 8800, 5000, 6250, 12000]
```

Use a list comprehension:

```python
big_sales = [s for s in SALES if s > 5000]
```

Output:

```text
[7100, 8800, 6250, 12000]
```

### Important

The condition is:

```python
s > 5000
```

not:

```python
s >= 5000
```

Therefore `5000` is excluded.

---

# 📚 Dictionary Comprehensions

A dictionary comprehension creates a dictionary compactly.

General structure:

```python
{key: value for item in iterable}
```

Example:

```python
{fee: ("free" if fee == 0 else "paid") for fee in FEES}
```

---

# 📝 Task 1.4 — Build a Fee Bucket Dictionary

Given:

```python
FEES = [0, 50, 0, 150, 50, 0, 500, 100, 0]
```

Create:

```python
fee_buckets = {
    fee: ("free" if fee == 0 else "paid")
    for fee in FEES
}
```

Result:

```python
{
    0: "free",
    50: "paid",
    150: "paid",
    500: "paid",
    100: "paid"
}
```

### Why does the dictionary have fewer entries?

`FEES` contains **9 items**, but the dictionary contains only **5 entries** because dictionary keys must be unique.

Repeated values such as:

```text
0
50
0
50
0
```

become one key each.

Check:

```python
print(len(fee_buckets))
print(len(FEES))
```

Output:

```text
5
9
```

---

# 📝 Task 1.5 — Count Free Services

The question asks for the number of **fees in `FEES` that are free**.

Use:

```python
n_free = sum(fee == 0 for fee in FEES)
```

Result:

```text
4
```

### Why does this work?

The expression:

```python
fee == 0
```

produces either:

```text
True
False
```

Python treats:

```text
True  = 1
False = 0
```

Therefore:

```python
sum(fee == 0 for fee in FEES)
```

counts the number of zero-fee items.

---

# 🔎 Section 2 — Filtering and Sorting

Filtering means selecting only the rows that satisfy a condition.

Basic pattern:

```python
df[condition]
```

Example:

```python
df[df["region"] == "NCR"]
```

This returns rows where `region` is exactly `NCR`.

---

# 📝 Task 2.1 — Count the NCR Requests

### Question

Set `ncr_count` to the number of rows where `region` is exactly `'NCR'`.

### Answer

```python
ncr_count = (df["region"] == "NCR").sum()
print(ncr_count)
```

For the generated dataset, the original exact-match count is:

```text
203
```

### Why use `.sum()`?

The comparison creates:

```text
True
False
True
False
...
```

Since:

```text
True = 1
False = 0
```

`.sum()` counts the matching rows.

---

# 🔀 Multiple Conditions

Use:

```python
& 
```

for **AND**.

Use:

```python
|
```

for **OR**.

Use parentheses around each condition.

### AND

```python
df[
    (df["status"] == "Escalated") &
    (df["channel"] == "Online Portal")
]
```

Means:

```text
status is Escalated
AND
channel is Online Portal
```

### OR

```python
df[
    (df["status"] == "Escalated") |
    (df["status"] == "Pending")
]
```

Means:

```text
status is Escalated
OR
status is Pending
```

### Important

Do not use Python's:

```python
and
or
```

for pandas Series filtering.

Use:

```python
&
|
```

---

# 📝 Task 2.2 — Escalated Requests from the Online Portal

### Answer

```python
escalated_online = df[
    (df["status"] == "Escalated") &
    (df["channel"] == "Online Portal")
]

print(len(escalated_online))
```

For the generated dataset:

```text
27
```

---

# 🎯 `.isin()`

`.isin()` checks whether values belong to a list of allowed values.

Syntax:

```python
df["column"].isin(["value1", "value2"])
```

Example:

```python
df["status"].isin(["Pending", "Escalated"])
```

This returns `True` for rows whose status is either:

```text
Pending
Escalated
```

---

# 📝 Task 2.3 — Lowest Satisfaction Ratings

Select ratings of `1` or `2`.

Use `.isin()`:

```python
low_rated = df[
    df["satisfaction_rating"].isin([1, 2])
]
```

Count them:

```python
len(low_rated)
```

For the generated dataset:

```text
109
```

### Why `.isin()`?

Instead of:

```python
(df["satisfaction_rating"] == 1) |
(df["satisfaction_rating"] == 2)
```

you can write:

```python
df["satisfaction_rating"].isin([1, 2])
```

This is cleaner, especially when there are many values.

---

# 🏆 `.nlargest()`

Use `.nlargest()` to select the rows with the largest values in a column.

Syntax:

```python
df.nlargest(n, "column")
```

Example:

```python
df.nlargest(10, "days_to_resolve")
```

---

# 📝 Task 2.4 — Ten Slowest Requests

```python
top10_slowest = df.nlargest(
    10,
    "days_to_resolve"
)
```

Inspect:

```python
print(top10_slowest["days_to_resolve"])
```

### Important dataset issue

The generated dataset intentionally contains:

```text
999
```

in `days_to_resolve`.

These are deliberately unrealistic/extreme values inserted as data-quality problems.

Therefore, the ten largest values can be:

```text
999
```

rather than realistic resolution times.

### Data-quality lesson

Always inspect extreme values.

A large value is not automatically a legitimate value.

---

# ❌ Missing Values

Missing values are different from zero.

For example:

```text
processing_fee = 0
```

means:

> A fee was recorded and it is zero.

But:

```text
processing_fee = NaN
```

means:

> No fee value was recorded.

These should not be treated as the same thing.

---

# 🔍 `.isna()`

Check missing values:

```python
df["processing_fee"].isna()
```

Count missing values:

```python
df["processing_fee"].isna().sum()
```

---

# 📝 Task 2.5 — Count Missing Fees

### Answer

```python
missing_fee_count = df["processing_fee"].isna().sum()
print(missing_fee_count)
```

For the generated dataset:

```text
18
```

### Important distinction

Do not use:

```python
df["processing_fee"] == 0
```

because that finds **recorded zero fees**, not missing fees.

Use:

```python
df["processing_fee"].isna()
```

for missing values.

---

# 🧮 Section 3 — Derived Columns

A **derived column** is a new column calculated from existing data.

Examples:

```text
date_filed
    ↓
filed
    ↓
month_name
```

or:

```text
days_to_resolve
    ↓
classify_speed()
    ↓
speed_category
```

or:

```text
processing_fee
    ↓
conditional rules
    ↓
fee_bracket
```

---

# 📅 Task 3.1 — Parse the Dates

The original `date_filed` column contains two date formats.

The task requires:

* create a **new** column
* keep `date_filed` unchanged
* convert the new column to real dates

For modern pandas versions:

```python
df["filed"] = pd.to_datetime(
    df["date_filed"],
    format="mixed"
)
```

This allows pandas to handle the mixed date formats.

### Why parse dates?

Text:

```text
2025-01-15
```

is not necessarily treated as a true date.

A datetime value allows operations such as:

```python
df["filed"].dt.year
df["filed"].dt.month
df["filed"].dt.day
df["filed"].dt.day_name()
```

---

# 📅 Datetime `.dt` Accessor

Once a column is datetime:

```python
df["filed"].dt
```

provides date/time components.

Examples:

```python
df["filed"].dt.year
```

```python
df["filed"].dt.month
```

```python
df["filed"].dt.day
```

```python
df["filed"].dt.day_name()
```

```python
df["filed"].dt.month_name()
```

---

# 📝 Task 3.2 — Add Month Name

Create:

```python
df["month_name"] = df["filed"].dt.month_name()
```

Example:

```text
2025-01-20 → January
2025-02-15 → February
2025-03-03 → March
```

The result is a new text column containing month names.

---

# 🧹 Task 3.3 — Clean the Region Column

The dataset intentionally contains inconsistent NCR values such as:

```text
"NCR"
" NCR"
"NCR "
" ncr "
"ncr"
"Ncr"
"  NCR"
```

These represent the same logical region but are not identical strings.

---

## `.str.strip()`

Removes leading and trailing spaces.

```python
df["region"].str.strip()
```

Example:

```text
" NCR " → "NCR"
```

---

## `.str.upper()`

Converts text to uppercase.

```python
df["region"].str.upper()
```

Example:

```text
"ncr" → "NCR"
"Ncr" → "NCR"
```

---

## Combine them

```python
df["region_clean"] = (
    df["region"]
    .str.strip()
    .str.upper()
)
```

Now all NCR variants become:

```text
NCR
```

---

## Count cleaned NCR records

```python
ncr_clean_count = (
    df["region_clean"] == "NCR"
).sum()
```

For the generated dataset:

```text
243
```

### Important lesson

Original exact-match result:

```text
203
```

Cleaned result:

```text
243
```

The difference comes from the intentionally dirty NCR values.

---

# ⚙️ Task 3.4 — Categorize Resolution Speed

We already created:

```python
classify_speed()
```

Now apply it to an entire pandas column.

Use:

```python
df["speed_category"] = df["days_to_resolve"].apply(
    classify_speed
)
```

This applies the function to every value in:

```python
df["days_to_resolve"]
```

---

# 🔄 `.apply()`

General pattern:

```python
df["new_column"] = df["old_column"].apply(function)
```

Example:

```python
df["speed_category"] = df["days_to_resolve"].apply(
    classify_speed
)
```

Conceptually:

```text
days_to_resolve
       ↓
classify_speed()
       ↓
speed_category
```

---

# ❓ Finding `unknown`

Because `classify_speed()` returns:

```python
"unknown"
```

for missing values:

```python
df["speed_category"].value_counts()
```

can show how many records fall into each category.

Specifically:

```python
(df["speed_category"] == "unknown").sum()
```

The unknown values correspond to missing `days_to_resolve`.

Since unresolved statuses have no resolution time in the generated dataset, inspect:

```python
df.loc[
    df["speed_category"] == "unknown",
    "status"
].value_counts()
```

This helps identify which statuses produce unknown resolution speed.

---

# 💰 Task 3.5 — Bracket the Processing Fee

The required categories are:

| Condition   | `fee_bracket`                |
| ----------- | ---------------------------- |
| Fee = 0     | `none`                       |
| Fee 1–100   | `low`                        |
| Fee > 100   | `high`                       |
| Missing fee | Should be handled separately |

---

# 📍 `.loc`

Use `.loc` for conditional assignment.

General syntax:

```python
df.loc[condition, "column"] = value
```

Example:

```python
df.loc[df["processing_fee"] == 0, "fee_bracket"] = "none"
```

---

## Complete fee-bracket solution

First create the column:

```python
df["fee_bracket"] = pd.NA
```

Then assign each category:

```python
df.loc[
    df["processing_fee"] == 0,
    "fee_bracket"
] = "none"

df.loc[
    df["processing_fee"].between(1, 100, inclusive="both"),
    "fee_bracket"
] = "low"

df.loc[
    df["processing_fee"] > 100,
    "fee_bracket"
] = "high"
```

Missing fees remain:

```text
<NA>
```

---

# ⚠️ Why Missing Fees Should Not Be `"none"`

This is an important data-quality concept.

Consider:

```text
processing_fee = 0
```

This supports the claim:

> The recorded fee is zero.

But:

```text
processing_fee = NaN
```

only supports:

> No fee was recorded.

Therefore, assigning:

```text
NaN → none
```

would incorrectly imply that the fee was recorded as zero.

A better approach is to leave it missing or explicitly label it:

```text
missing
```

if the analysis requires a categorical label.

For example:

```python
df.loc[
    df["processing_fee"].isna(),
    "fee_bracket"
] = "missing"
```

This preserves the distinction between:

```text
none     = recorded fee of 0
missing  = no fee recorded
```

---

# 📤 Task 3.6 — Export the Working Subset

The working subset must:

* contain all 1,200 rows
* include `request_id`
* include `region_clean`
* include `status`
* include `speed_category`
* include `fee_bracket`

You can choose additional columns.

Example:

```python
working_subset = df[
    [
        "request_id",
        "filed",
        "month_name",
        "region_clean",
        "status",
        "service_type",
        "channel",
        "days_to_resolve",
        "speed_category",
        "processing_fee",
        "fee_bracket",
        "satisfaction_rating"
    ]
]
```

Check the shape:

```python
working_subset.shape
```

Expected:

```text
(1200, 12)
```

---

# 💾 Export to CSV

```python
working_subset.to_csv(
    "working_subset_day2.csv",
    index=False
)
```

### Why `index=False`?

Pandas normally writes the DataFrame index as an additional column.

Using:

```python
index=False
```

prevents the pandas index from becoming an unwanted CSV column.

---

# 🧪 Verify the Export

After writing the file:

```python
working_subset_check = pd.read_csv(
    "working_subset_day2.csv"
)

print(working_subset_check.shape)
```

You should still have:

```text
1200 rows
```

---

# ✅ Complete Task Answers

## Section 1

### Task 1.1

```python
def classify_speed(days):
    if pd.isna(days):
        return "unknown"
    elif days <= 3:
        return "fast"
    elif days <= 10:
        return "normal"
    else:
        return "slow"
```

### Task 1.2

```python
speed_labels = [
    classify_speed(d)
    for d in [2, 7, 30, np.nan]
]

print(speed_labels)
```

Expected:

```text
['fast', 'normal', 'slow', 'unknown']
```

### Task 1.3

```python
big_sales = [s for s in SALES if s > 5000]

print(big_sales)
```

Expected:

```text
[7100, 8800, 6250, 12000]
```

### Task 1.4

```python
fee_buckets = {
    fee: ("free" if fee == 0 else "paid")
    for fee in FEES
}

print(fee_buckets)
print(len(fee_buckets))
print(len(FEES))
```

Expected dictionary:

```python
{
    0: "free",
    50: "paid",
    150: "paid",
    500: "paid",
    100: "paid"
}
```

Lengths:

```text
5
9
```

### Task 1.5

```python
n_free = sum(fee == 0 for fee in FEES)

print(n_free)
```

Expected:

```text
4
```

---

# 🔎 Section 2 Complete Answers

## Task 2.1

```python
ncr_count = (df["region"] == "NCR").sum()

print(ncr_count)
```

Expected:

```text
203
```

---

## Task 2.2

```python
escalated_online = df[
    (df["status"] == "Escalated") &
    (df["channel"] == "Online Portal")
]

print(len(escalated_online))
```

Expected:

```text
27
```

---

## Task 2.3

```python
low_rated = df[
    df["satisfaction_rating"].isin([1, 2])
]

print(len(low_rated))
```

Expected:

```text
109
```

---

## Task 2.4

```python
top10_slowest = df.nlargest(
    10,
    "days_to_resolve"
)

print(top10_slowest["days_to_resolve"])
```

The generated dataset intentionally includes extreme `999` values, so inspect these values rather than assuming they are legitimate.

---

## Task 2.5

```python
missing_fee_count = df["processing_fee"].isna().sum()

print(missing_fee_count)
```

Expected:

```text
18
```

---

# 🧮 Section 3 Complete Answers

## Task 3.1 — Parse dates

```python
df["filed"] = pd.to_datetime(
    df["date_filed"],
    format="mixed"
)
```

---

## Task 3.2 — Month name

```python
df["month_name"] = df["filed"].dt.month_name()
```

---

## Task 3.3 — Clean regions

```python
df["region_clean"] = (
    df["region"]
    .str.strip()
    .str.upper()
)

ncr_clean_count = (
    df["region_clean"] == "NCR"
).sum()

print(ncr_clean_count)
```

Expected:

```text
243
```

---

## Task 3.4 — Resolution speed

```python
df["speed_category"] = (
    df["days_to_resolve"]
    .apply(classify_speed)
)
```

Count unknown:

```python
unknown_count = (
    df["speed_category"] == "unknown"
).sum()

print(unknown_count)
```

Check statuses:

```python
print(
    df.loc[
        df["speed_category"] == "unknown",
        "status"
    ].value_counts()
)
```

---

## Task 3.5 — Fee bracket

```python
df["fee_bracket"] = pd.NA

df.loc[
    df["processing_fee"] == 0,
    "fee_bracket"
] = "none"

df.loc[
    df["processing_fee"].between(1, 100, inclusive="both"),
    "fee_bracket"
] = "low"

df.loc[
    df["processing_fee"] > 100,
    "fee_bracket"
] = "high"

df.loc[
    df["processing_fee"].isna(),
    "fee_bracket"
] = "missing"
```

---

## Task 3.6 — Working subset

```python
working_subset = df[
    [
        "request_id",
        "filed",
        "month_name",
        "region_clean",
        "status",
        "service_type",
        "channel",
        "days_to_resolve",
        "speed_category",
        "processing_fee",
        "fee_bracket",
        "satisfaction_rating"
    ]
]
```

Check:

```python
print(working_subset.shape)
```

Expected:

```text
(1200, 12)
```

Export:

```python
working_subset.to_csv(
    "working_subset_day2.csv",
    index=False
)
```

---

# 📌 Important pandas Concepts

## DataFrame vs Series

### DataFrame

A DataFrame is a two-dimensional table:

```text
rows × columns
```

Example:

```python
df
```

### Series

A single column is usually a Series:

```python
df["status"]
```

Think:

```text
DataFrame
┌───────────────┐
│ column A      │
│ column B      │
│ column C      │
└───────────────┘

        ↓ select one column

Series
┌───────────────┐
│ column A      │
└───────────────┘
```

---

# 🧠 Filtering Cheat Sheet

| Goal            | Code                           |
| --------------- | ------------------------------ |
| Equal           | `df["status"] == "Pending"`    |
| Not equal       | `df["status"] != "Pending"`    |
| Greater than    | `df["fee"] > 100`              |
| Less than       | `df["fee"] < 100`              |
| Greater/equal   | `df["fee"] >= 100`             |
| Less/equal      | `df["fee"] <= 100`             |
| AND             | `(condition1) & (condition2)`  |
| OR              | `(condition1) \| (condition2)` |
| Multiple values | `df["status"].isin([...])`     |
| Missing         | `df["column"].isna()`          |
| Not missing     | `df["column"].notna()`         |

---

# 🧹 String Cleaning Cheat Sheet

| Operation                 | Code              | Example               |
| ------------------------- | ----------------- | --------------------- |
| Remove surrounding spaces | `.str.strip()`    | `" NCR "` → `"NCR"`   |
| Uppercase                 | `.str.upper()`    | `"ncr"` → `"NCR"`     |
| Lowercase                 | `.str.lower()`    | `"NCR"` → `"ncr"`     |
| Replace text              | `.str.replace()`  | Replace unwanted text |
| Check contains            | `.str.contains()` | Find matching text    |

Example:

```python
df["region"].str.strip().str.upper()
```

---

# 📅 Date Cheat Sheet

Convert text to datetime:

```python
df["filed"] = pd.to_datetime(
    df["date_filed"],
    format="mixed"
)
```

Extract year:

```python
df["filed"].dt.year
```

Extract month number:

```python
df["filed"].dt.month
```

Extract month name:

```python
df["filed"].dt.month_name()
```

Extract day:

```python
df["filed"].dt.day
```

Extract weekday:

```python
df["filed"].dt.day_name()
```

---

# 🔢 Useful Counting Patterns

## Count rows

```python
len(df)
```

or:

```python
df.shape[0]
```

---

## Count condition matches

```python
(df["status"] == "Pending").sum()
```

---

## Count missing values

```python
df["column"].isna().sum()
```

---

## Count distinct values

```python
df["status"].nunique()
```

---

## Count each category

```python
df["status"].value_counts()
```

---

# 🏆 Sorting and Extreme Values

Largest:

```python
df.nlargest(10, "days_to_resolve")
```

Smallest:

```python
df.nsmallest(10, "days_to_resolve")
```

Sort ascending:

```python
df.sort_values("days_to_resolve")
```

Sort descending:

```python
df.sort_values(
    "days_to_resolve",
    ascending=False
)
```

---

# 🧮 Derived Column Patterns

## Direct calculation

```python
df["total"] = df["price"] * df["quantity"]
```

## Function

```python
df["category"] = df["value"].apply(my_function)
```

## String transformation

```python
df["clean_name"] = (
    df["name"]
    .str.strip()
    .str.upper()
)
```

## Date extraction

```python
df["month"] = df["date"].dt.month_name()
```

## Conditional assignment

```python
df.loc[df["score"] >= 80, "result"] = "Pass"
```

---

# 📍 `.loc` Cheat Sheet

General form:

```python
df.loc[row_condition, column] = value
```

Example:

```python
df.loc[
    df["processing_fee"] == 0,
    "fee_bracket"
] = "none"
```

Multiple conditions:

```python
df.loc[
    (df["status"] == "Escalated") &
    (df["channel"] == "Online Portal"),
    "priority"
] = "high"
```

---

# 💾 Import / Export Cheat Sheet

## CSV → DataFrame

```python
df = pd.read_csv("file.csv")
```

## DataFrame → CSV

```python
df.to_csv(
    "output.csv",
    index=False
)
```

## JSON → DataFrame

```python
df = pd.read_json("file.json")
```

---

# ⚠️ Common Mistakes

## Mistake 1 — Using `and` instead of `&`

❌ Incorrect:

```python
df[
    (df["status"] == "Pending") and
    (df["region"] == "NCR")
]
```

✅ Correct:

```python
df[
    (df["status"] == "Pending") &
    (df["region"] == "NCR")
]
```

---

## Mistake 2 — Forgetting parentheses

❌ Avoid:

```python
df[df["status"] == "Pending" & df["region"] == "NCR"]
```

✅ Use:

```python
df[
    (df["status"] == "Pending") &
    (df["region"] == "NCR")
]
```

---

## Mistake 3 — Confusing zero with missing

```python
df["processing_fee"] == 0
```

means:

> Recorded fee is zero.

Whereas:

```python
df["processing_fee"].isna()
```

means:

> Fee is missing.

---

## Mistake 4 — Modifying the original column when the task requires a new one

If the task says keep `date_filed` unchanged:

❌

```python
df["date_filed"] = pd.to_datetime(df["date_filed"])
```

✅

```python
df["filed"] = pd.to_datetime(
    df["date_filed"],
    format="mixed"
)
```

---

## Mistake 5 — Treating dirty text as different categories

These may all represent the same region:

```text
"NCR"
" NCR"
"NCR "
"ncr"
"Ncr"
```

Clean them:

```python
df["region_clean"] = (
    df["region"]
    .str.strip()
    .str.upper()
)
```

---

## Mistake 6 — Assuming extreme values are valid

If:

```python
df.nlargest(10, "days_to_resolve")
```

returns:

```text
999
```

do not automatically accept it.

Investigate whether it represents:

* a genuine observation
* a data-entry error
* a placeholder
* an impossible value
* an intentionally inserted data-quality defect

---

# 🧪 Day 2 Data-Quality Lessons

The practice dataset intentionally contains several problems.

### Dirty region values

Examples:

```text
" NCR"
"NCR "
" ncr "
"ncr"
"Ncr"
"  NCR"
```

Solution:

```python
.str.strip().str.upper()
```

---

### Mixed date formats

Examples:

```text
2025-01-15
01/15/2025
```

Solution:

```python
pd.to_datetime(
    column,
    format="mixed"
)
```

---

### Missing resolution times

Unresolved requests have:

```text
NaN
```

Solution:

```python
.isna()
```

or:

```python
pd.isna()
```

---

### Missing processing fees

There are deliberately missing fee values.

Detect them with:

```python
df["processing_fee"].isna()
```

---

### Extreme resolution values

Some completed requests contain:

```text
999
```

These should be investigated rather than blindly accepted.

---

### Missing satisfaction ratings

Satisfaction ratings are only available for appropriate completed requests, and additional ratings are intentionally missing.

Detect:

```python
df["satisfaction_rating"].isna()
```

---

# 🧭 Recommended Data-Preparation Workflow

A useful Day 2 workflow is:

```text
1. LOAD
   ↓
2. INSPECT
   ↓
3. FILTER
   ↓
4. CLEAN
   ↓
5. DERIVE
   ↓
6. VALIDATE
   ↓
7. EXPORT
```

More specifically:

```text
pd.read_csv()
      ↓
df.head()
df.info()
df.describe()
value_counts()
      ↓
Boolean filtering
      ↓
Missing-value checks
      ↓
Date parsing
      ↓
Text cleaning
      ↓
Derived columns
      ↓
Check unusual values
      ↓
working_subset
      ↓
to_csv()
```

---

# 📊 Key Expected Values from the Practice Dataset

| Task | Variable                  |                         Expected Result |
| ---- | ------------------------- | --------------------------------------: |
| 1.2  | `speed_labels`            | `['fast', 'normal', 'slow', 'unknown']` |
| 1.3  | `big_sales`               |             `[7100, 8800, 6250, 12000]` |
| 1.4  | `len(fee_buckets)`        |                                     `5` |
| 1.4  | `len(FEES)`               |                                     `9` |
| 1.5  | `n_free`                  |                                     `4` |
| 2.1  | `ncr_count`               |                                   `203` |
| 2.2  | Escalated + Online Portal |                                    `27` |
| 2.3  | Low ratings 1 or 2        |                                   `109` |
| 2.5  | `missing_fee_count`       |                                    `18` |
| 3.3  | Cleaned NCR count         |                                   `243` |
| 3.6  | Working subset rows       |                                  `1200` |

> **Note:** Some outputs can depend on the exact dataset file being used. The values above correspond to the synthetic Day 2 dataset generated with `seed=2026`.

---

# 🧠 Final Day 2 Mental Model

Remember these core patterns:

### Inspect

```python
df.head()
df.info()
df.describe()
df["column"].value_counts()
```

### Filter

```python
df[df["column"] == value]
```

### Multiple conditions

```python
df[
    (condition1) &
    (condition2)
]
```

### Multiple allowed values

```python
df[
    df["column"].isin([value1, value2])
]
```

### Missing values

```python
df["column"].isna()
```

### Count

```python
(condition).sum()
```

### Top values

```python
df.nlargest(10, "column")
```

### Clean text

```python
df["column"].str.strip().str.upper()
```

### Parse dates

```python
pd.to_datetime(
    df["column"],
    format="mixed"
)
```

### Extract date components

```python
df["date"].dt.month_name()
```

### Apply a function

```python
df["new_column"] = df["column"].apply(function)
```

### Conditional assignment

```python
df.loc[condition, "new_column"] = value
```

### Export

```python
df.to_csv("output.csv", index=False)
```

---

# 🚀 One-Page Day 2 Cheat Sheet

```python
# ============================
# LOAD
# ============================

import pandas as pd
import numpy as np

df = pd.read_csv("file.csv")


# ============================
# INSPECT
# ============================

df.head()
df.info()
df.describe()
df.shape
df["status"].value_counts()


# ============================
# FUNCTIONS
# ============================

def classify_speed(days):
    if pd.isna(days):
        return "unknown"
    elif days <= 3:
        return "fast"
    elif days <= 10:
        return "normal"
    else:
        return "slow"


# ============================
# LIST COMPREHENSION
# ============================

big_sales = [
    s for s in SALES
    if s > 5000
]


# ============================
# DICTIONARY COMPREHENSION
# ============================

fee_buckets = {
    fee: ("free" if fee == 0 else "paid")
    for fee in FEES
}


# ============================
# COUNT
# ============================

count = (
    df["status"] == "Pending"
).sum()


# ============================
# FILTER
# ============================

pending = df[
    df["status"] == "Pending"
]


# ============================
# MULTIPLE CONDITIONS
# ============================

result = df[
    (df["status"] == "Escalated") &
    (df["channel"] == "Online Portal")
]


# ============================
# ISIN
# ============================

low_rated = df[
    df["satisfaction_rating"].isin([1, 2])
]


# ============================
# MISSING
# ============================

missing = df[
    df["processing_fee"].isna()
]

missing_count = (
    df["processing_fee"].isna()
).sum()


# ============================
# LARGEST / SMALLEST
# ============================

top10 = df.nlargest(
    10,
    "days_to_resolve"
)

bottom10 = df.nsmallest(
    10,
    "days_to_resolve"
)


# ============================
# DATE PARSING
# ============================

df["filed"] = pd.to_datetime(
    df["date_filed"],
    format="mixed"
)


# ============================
# DATE COMPONENT
# ============================

df["month_name"] = (
    df["filed"]
    .dt.month_name()
)


# ============================
# TEXT CLEANING
# ============================

df["region_clean"] = (
    df["region"]
    .str.strip()
    .str.upper()
)


# ============================
# APPLY FUNCTION
# ============================

df["speed_category"] = (
    df["days_to_resolve"]
    .apply(classify_speed)
)


# ============================
# CONDITIONAL ASSIGNMENT
# ============================

df["fee_bracket"] = pd.NA

df.loc[
    df["processing_fee"] == 0,
    "fee_bracket"
] = "none"

df.loc[
    df["processing_fee"].between(
        1, 100, inclusive="both"
    ),
    "fee_bracket"
] = "low"

df.loc[
    df["processing_fee"] > 100,
    "fee_bracket"
] = "high"

df.loc[
    df["processing_fee"].isna(),
    "fee_bracket"
] = "missing"


# ============================
# SELECT COLUMNS
# ============================

working_subset = df[
    [
        "request_id",
        "region_clean",
        "status",
        "speed_category",
        "fee_bracket"
    ]
]


# ============================
# EXPORT
# ============================

working_subset.to_csv(
    "working_subset_day2.csv",
    index=False
)
```

---

# 🏁 Core Takeaways

### 1. Always inspect before analyzing.

```python
df.head()
df.info()
df.describe()
df["column"].value_counts()
```

### 2. A filter creates a Boolean condition.

```python
df["status"] == "Pending"
```

### 3. Use `&` for AND and `|` for OR.

```python
(condition1) & (condition2)
```

### 4. Use `.isin()` for multiple accepted values.

```python
df["status"].isin(["Pending", "Escalated"])
```

### 5. Missing and zero are different.

```python
.isna()
```

is for missing values.

```python
== 0
```

is for actual zero values.

### 6. Clean text before comparing categories.

```python
.str.strip().str.upper()
```

### 7. Parse dates before doing date analysis.

```python
pd.to_datetime(...)
```

### 8. `.apply()` lets you reuse a function across a column.

```python
df["new"] = df["old"].apply(function)
```

### 9. `.loc` is the standard tool for conditional assignment.

```python
df.loc[condition, "column"] = value
```

### 10. Validate before exporting.

A dataset is not ready simply because the code ran successfully. Check:

* row count
* missing values
* unusual values
* categories
* dates
* derived columns
* whether your transformations make logical sense

**The goal of data preparation is not merely to make Python accept the data. The goal is to produce data that you can defend.**
