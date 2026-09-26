# 📊 Day 4 — Preparing Data Sets I: Importing from Multiple Sources

> **Main goal:** Import, inspect, diagnose, flatten, merge, and validate data coming from CSV, JSON, and SQL sources.

---

# 1. 📥 Importing a CSV

## Basic CSV import

```python
import pandas as pd

df = pd.read_csv("data/service_requests.csv")
```

### First five minutes checklist

Run these after **every import**:

```python
df.shape
df.head()
df.dtypes
df.isna().sum()
df.columns.tolist()
```

### What each checks

| Code                  | Purpose                    |
| --------------------- | -------------------------- |
| `df.shape`            | Number of rows and columns |
| `df.head()`           | Preview the data           |
| `df.dtypes`           | Check data types           |
| `df.isna().sum()`     | Count missing values       |
| `df.columns.tolist()` | Check column names         |

---

# 2. 🔍 Diagnosing CSV Import Problems

## Problem 1 — Everything is in one column

### Broken import

```python
semicolon_broken = pd.read_csv(
    "data/service_requests_semicolon.csv"
)
```

Check:

```python
semicolon_broken.shape
```

Output:

```text
(27, 1)
```

### Why?

The file uses:

```text
;
```

instead of the default:

```text
,
```

### Fix

Use `sep=`:

```python
semicolon_fixed = pd.read_csv(
    "data/service_requests_semicolon.csv",
    sep=";"
)
```

### Key idea

```python
pd.read_csv("file.csv", sep=";")
```

means:

> "Read the CSV using semicolon as the column separator."

---

# 3. 🔤 Encoding Problems

Sometimes a CSV was saved using a different text encoding.

For example, the file contains characters such as:

```text
Café
Peñalosa
Ñoñoy
```

### Broken import

```python
broken = pd.read_csv(
    "data/service_requests_latin1.csv"
)
```

This can produce:

```text
UnicodeDecodeError
```

### Fix

Specify the encoding:

```python
latin1_fixed = pd.read_csv(
    "data/service_requests_latin1.csv",
    encoding="latin-1"
)
```

Another commonly encountered encoding is:

```python
encoding="utf-8"
```

### Key idea

```python
pd.read_csv(
    "file.csv",
    encoding="latin-1"
)
```

means:

> "Interpret the characters in this file using Latin-1 encoding."

---

# 4. 📑 Extra Rows Before the Header

Sometimes a CSV contains a title before the actual column header.

Example:

```text
Q3 2026 Service Requests Export - Internal Use Only
request_id,date_submitted,office_id,...
REQ-0001,2026-07-03,OFF-001,...
```

### Broken import

```python
extrarow_broken = pd.read_csv(
    "data/service_requests_extrarow.csv"
)
```

Result:

```text
(28, 1)
```

The title was interpreted as the header.

### Fix with `skiprows`

```python
extrarow_fixed = pd.read_csv(
    "data/service_requests_extrarow.csv",
    skiprows=1
)
```

### Key idea

```python
skiprows=1
```

means:

> Skip the first row before reading the actual header/data.

---

# 5. 🧰 Common `read_csv()` Parameters

```python
pd.read_csv(
    "file.csv",
    sep=",",
    encoding="utf-8",
    skiprows=0
)
```

| Parameter  | Purpose                         |
| ---------- | ------------------------------- |
| `sep`      | Column delimiter                |
| `encoding` | Character encoding              |
| `skiprows` | Number of rows to skip          |
| `header`   | Which row contains column names |
| `dtype`    | Specify data types              |
| `usecols`  | Import selected columns         |
| `nrows`    | Import selected number of rows  |

---

# 6. 🧠 CSV Import Troubleshooting

If an imported DataFrame looks wrong:

### Step 1

```python
df.shape
```

### Step 2

```python
df.head()
```

### Step 3

```python
df.columns.tolist()
```

### Step 4

```python
df.dtypes
```

### Step 5

```python
df.isna().sum()
```

### Common symptoms

| Symptom                   | Likely cause                        |
| ------------------------- | ----------------------------------- |
| Everything in one column  | Wrong delimiter                     |
| `UnicodeDecodeError`      | Wrong encoding                      |
| Title becomes column name | Extra row before header             |
| Unexpected missing values | Import/parsing problem              |
| Wrong column names        | Header problem                      |
| Unexpected number of rows | Extra/missing rows or parsing issue |

---

# 7. 🗂️ Working with JSON

Import the JSON module:

```python
import json
```

Open a JSON file:

```python
with open("data/regional_offices.json") as f:
    offices_data = json.load(f)
```

Now:

```python
type(offices_data)
```

returns a Python dictionary.

---

# 8. 👀 Inspect JSON Structure

Pretty-print JSON:

```python
print(json.dumps(offices_data, indent=2))
```

Only print the first 800 characters:

```python
print(json.dumps(offices_data, indent=2)[:800])
```

### Why?

Before flattening JSON, understand its structure.

For this dataset:

```text
regions
    ↓
list of regions
    ↓
offices
    ↓
list of offices
    ↓
contact
    ↓
email
```

---

# 9. 🧭 Navigate Nested JSON Manually

Example:

```python
first_office_email = (
    offices_data["regions"][0]
    ["offices"][0]
    ["contact"]["email"]
)
```

Output:

```text
manila.branch@example.gov.ph
```

### Structure

```python
offices_data["regions"]
```

gets the regions list.

```python
offices_data["regions"][0]
```

gets the first region.

```python
["offices"][0]
```

gets the first office.

```python
["contact"]["email"]
```

gets its email.

### General pattern

```python
dictionary["key"]
```

Access a list item:

```python
list[0]
```

Combine them:

```python
data["regions"][0]["offices"][0]["contact"]["email"]
```

---

# 10. 🔄 `pd.json_normalize()`

Nested JSON is often inconvenient for analysis.

Use:

```python
pd.json_normalize()
```

to flatten nested structures into a DataFrame.

Basic structure:

```python
offices = pd.json_normalize(
    offices_data["regions"],
    record_path="offices",
    meta=["region_code", "region_name"],
    sep="_"
)
```

---

# 11. 🧩 Three Questions Before `json_normalize()`

Always ask:

### 1. What is my record?

For this dataset:

```text
office
```

We want:

> One row per office.

### 2. Where is the array?

```python
record_path="offices"
```

This tells pandas:

> "Each item inside `offices` should become a row."

### 3. What outer fields should be carried down?

```python
meta=["region_code", "region_name"]
```

This adds the region information to every office row.

---

# 12. 📐 `json_normalize()` Key Parameters

```python
pd.json_normalize(
    data,
    record_path="offices",
    meta=["region_code", "region_name"],
    sep="_"
)
```

| Parameter     | Meaning                              |
| ------------- | ------------------------------------ |
| `data`        | Starting JSON structure              |
| `record_path` | Nested list containing the records   |
| `meta`        | Outer fields copied onto each record |
| `sep`         | Separator for nested column names    |

---

# 13. 🪜 Flattening Nested Dictionaries

Original JSON:

```json
{
    "contact": {
        "phone": "8-527-1001",
        "email": "manila.branch@example.gov.ph"
    }
}
```

After normalization:

```text
contact_phone
contact_email
```

because:

```python
sep="_"
```

was used.

---

# 14. 🏢 Validate the Flattened JSON

Use the same checklist:

```python
print(offices.shape)
print(offices.head())
print(offices.dtypes)
print(offices.isna().sum())
print(offices.columns.tolist())
```

For this dataset:

```text
(10, 7)
```

There are:

```text
10 offices
```

and:

```text
7 columns
```

---

# 15. 🔗 Merging DataFrames

Basic merge:

```python
merged = df.merge(
    offices,
    on="office_id",
    how="left"
)
```

### Important parameters

```python
on="office_id"
```

means:

> Merge using `office_id`.

```python
how="left"
```

means:

> Keep every row from the left DataFrame.

---

# 16. 🛡️ Merge With Validation

Recommended pattern:

```python
merged = df.merge(
    offices,
    on="office_id",
    how="left",
    indicator=True,
    validate="many_to_one"
)
```

### `indicator=True`

Creates:

```text
_merge
```

with values such as:

```text
both
left_only
right_only
```

### `validate="many_to_one"`

Means:

> Many rows from the left DataFrame may match one row on the right.

This is appropriate when:

```text
many service requests → one office
```

---

# 17. 📊 Understanding `_merge`

After:

```python
indicator=True
```

you can run:

```python
merged["_merge"].value_counts()
```

Example:

```text
both          24
left_only      3
right_only     0
```

### Meaning

| Value        | Meaning                        |
| ------------ | ------------------------------ |
| `both`       | Match found in both DataFrames |
| `left_only`  | Exists only in left DataFrame  |
| `right_only` | Exists only in right DataFrame |

---

# 18. 🚨 Find Unmatched Rows

```python
unmatched = merged[
    merged["_merge"] == "left_only"
]
```

Inspect:

```python
print(unmatched)
```

Or only important columns:

```python
print(
    unmatched[
        ["request_id", "office_id", "description"]
    ]
)
```

---

# 19. 🔎 Investigating Unmatched Keys

Compare unmatched keys:

```python
print(
    unmatched[
        ["request_id", "office_id", "description"]
    ]
)

print(
    sorted(offices["office_id"].tolist())
)
```

For this exercise:

```text
OFF-999
OFF-02
```

are unmatched.

### Interpretation

`OFF-999`:

> More consistent with a decommissioned/unmigrated office because it is not close to any current office code and the description explicitly refers to a closed satellite office.

`OFF-02`:

> More consistent with a typo because it closely resembles the valid office code `OFF-002`.

---

# 20. 🔀 Types of Joins

## Left join

```p
```
