# 🧹 Day 5 — Data Cleaning and Data Integrity Cheatsheet

> **TESDA Alignment:** Prepare Data Sets — Clean Data, Filter Data
> **Main workflow:** **Profile → Clean → Validate → Version**

---

## 📌 1. Load the Raw Dataset

```python
import pandas as pd
import numpy as np

raw = pd.read_csv("data/service_requests.csv")
```

### Keep the raw data untouched

```python
clean = raw.copy()
```

**Rule:** Never modify `raw` directly. Perform cleaning on `clean`.

---

# 🔍 2. Profile the Data Before Cleaning

## Check shape

```python
raw.shape
```

or:

```python
print("Shape:", raw.shape)
```

Returns:

```text
(rows, columns)
```

---

## Check data types

```python
raw.dtypes
```

More detailed:

```python
raw.info()
```

---

# ❓ 3. Check Missing Values

## Count missing values per column

```python
raw.isna().sum()
```

## Show only columns with missing values

```python
missing_report = raw.isna().sum()[raw.isna().sum() > 0]

print(missing_report)
```

### Useful pattern

```python
raw["column"].isna().sum()
```

Counts missing values in one column.

---

# 🔁 4. Check Duplicates

## Full duplicate rows

```python
raw.duplicated().sum()
```

Store the result:

```python
dup_count = raw.duplicated().sum()
```

## Remove full duplicate rows

```python
clean = clean.drop_duplicates()
```

---

## Duplicate keys

Check duplicate `request_id` values:

```python
raw["request_id"].duplicated().sum()
```

Check whether IDs are completely unique:

```python
raw["request_id"].is_unique
```

Remove duplicate keys:

```python
clean = clean.drop_duplicates(subset=["request_id"])
```

### ⚠️ Important distinction

```text
Full duplicate
→ Entire row is identical

Duplicate key
→ Same request_id appears more than once
```

They are **not necessarily the same problem**.

---

# 🏷️ 5. Inspect Categorical Values

## Unique values

```python
raw["region"].unique()
```

```python
raw["service_type"].unique()
```

```python
raw["status"].unique()
```

## Count unique values

```python
raw["region"].nunique()
```

Example:

```python
region_variants = raw["region"].nunique()
```

### Difference

```python
.unique()
```

→ shows the actual values

```python
.nunique()
```

→ counts the number of unique values

---

# ✂️ 6. Standardize Text Categories

## Remove whitespace

```python
clean["region"] = clean["region"].str.strip()
```

Example:

```text
" ncr " → "ncr"
```

---

## Convert to uppercase

```python
clean["region"] = clean["region"].str.upper()
```

Combined:

```python
clean["region"] = clean["region"].str.strip().str.upper()
```

Example:

```text
" ncr " → "NCR"
```

---

## Convert to title case

```python
clean["column"] = clean["column"].str.title()
```

---

## Replace known spelling variants

```python
clean["service_type"] = clean["service_type"].replace(
    {"Free Wifi Installation": "Free WiFi Installation"}
)
```

### General pattern

```python
df["column"] = df["column"].replace(
    {"incorrect value": "correct value"}
)
```

---

# 📅 7. Parse Dates

The workshop dataset contains mixed date formats.

Use:

```python
clean["date_filed"] = pd.to_datetime(
    clean["date_filed"],
    format="mixed"
)
```

### Check the result

```python
clean["date_filed"].dtype
```

---

# 🩹 8. Handle Missing Values

There are three main strategies.

```text
DROP
FILL
FLAG
```

---

## Drop missing rows

```python
df.dropna()
```

Or for a specific column:

```python
df.dropna(subset=["column"])
```

Use only when dropping the rows is defensible.

---

## Fill with a literal value

Example:

```python
clean["citizen_age_group"] = clean["citizen_age_group"].fillna("Unknown")
```

---

## Fill numeric values with the median

```python
median_fee = clean["processing_fee"].median()

clean["processing_fee"] = clean["processing_fee"].fillna(median_fee)
```

Or directly:

```python
clean["processing_fee"] = clean["processing_fee"].fillna(
    clean["processing_fee"].median()
)
```

### Why median?

Median is often useful for skewed numeric data because it is less affected by extreme values than the mean.

---

# 🚩 9. Flag Missing Values Before Filling

This is an important workshop pattern.

```python
clean["fee_was_missing"] = clean["processing_fee"].isna()
```

Then fill:

```python
clean["processing_fee"] = clean["processing_fee"].fillna(
    clean["processing_fee"].median()
)
```

### Why the order matters

Correct:

```text
1. Identify missing values
2. Create flag
3. Fill missing values
```

If you fill first, you can no longer tell which rows originally had missing values.

---

# 🧮 10. IQR Outlier Detection

The IQR method provides a defensible statistical threshold.

## Step 1 — Select the column

```python
col = clean["days_to_resolve"]
```

## Step 2 — Calculate Q1 and Q3

```python
q1, q3 = col.quantile([0.25, 0.75])
```

## Step 3 — Calculate IQR

```python
iqr = q3 - q1
```

## Step 4 — Calculate upper fence

```python
upper_fence = q3 + 1.5 * iqr
```

### Complete pattern

```python
col = clean["days_to_resolve"]

q1, q3 = col.quantile([0.25, 0.75])
iqr = q3 - q1

upper_fence = q3 + 1.5 * iqr
```

---

# 🔎 11. Count Potential Outliers

```python
outlier_count = (col > upper_fence).sum()
```

Or:

```python
print("Values above fence:", (col > upper_fence).sum())
```

### Remember

An outlier is **not automatically an error**.

```text
Outlier
   ↓
Investigate
   ↓
Legitimate extreme?
   ├── Yes → Keep
   └── No → Correct/remove according to meaning
```

---

# 🚨 12. Detect Sentinel Values

A sentinel value is a placeholder that looks like real data.

Examples:

```text
999
-1
0
1900-01-01
```

For this workshop:

```python
sentinel_count = (clean["days_to_resolve"] == 999).sum()
```

Check:

```python
print("Exactly 999:", sentinel_count)
```

The `999` value in this workshop is intentionally a **sentinel**, not a genuine 999-day resolution time.

---

# 🧹 13. Replace Sentinel Values

Replace `999` with missing:

```python
clean["days_to_resolve"] = clean["days_to_resolve"].replace(
    999, np.nan
)
```

This means:

```text
999 → NaN
```

The value is now treated as missing rather than as a real measurement.

---

# 📊 14. Histogram

Use a histogram to inspect a numeric distribution.

```python
df.hist(
    column="unit_price_php",
    bins=15,
    edgecolor="black"
)
```

For separate categories:

```python
df.hist(
    column="unit_price_php",
    by="product_category",
    bins=15,
    figsize=(12, 8),
    layout=(2, 2),
    edgecolor="black"
)
```

---

# 📦 15. Box Plot

Box plots are useful for comparing distributions and visually identifying potential outliers.

```python
df.boxplot(
    column="unit_price_php",
    figsize=(12, 5),
    grid=False,
    rot=20
)
```

---

# ✂️ 16. Winsorization

Winsorization **caps extreme values instead of deleting observations**.

```python
from scipy.stats.mstats import winsorize

data = df["unit_price_php"]

capped_data = winsorize(
    data,
    limits=[0.10, 0.10]
)
```

This caps:

```text
10% lowest values
10% highest values
```

Compare before and after:

```python
print("Original:", data.min(), data.max())
print("Capped:", capped_data.min(), capped_data.max())
```

### Important

Winsorization is different from fixing a sentinel value.

```text
Statistical outlier
→ May be legitimate
→ Investigate / possibly winsorize

Sentinel value
→ Placeholder
→ Replace according to its meaning
```

---

# 🔐 17. Data Integrity Checks

Cleaning also means checking whether relationships between fields make sense.

---

## Resolved requests should have resolution days

```python
resolved_have_days = clean.loc[
    clean["status"] == "Resolved",
    "days_to_resolve"
].notna().all()
```

Returns:

```python
True
```

if every `Resolved` request has a non-missing resolution time.

---

## Check that all dates are in 2025

```python
dates_in_2025 = (
    clean["date_filed"].dt.year == 2025
).all()
```

---

## Check for duplicate request IDs

```python
no_dup_ids = clean["request_id"].is_unique
```

---

# 📋 18. Store Integrity Results in a Dictionary

The workshop uses:

```python
integrity = {
    "resolved_have_days": clean.loc[
        clean["status"] == "Resolved",
        "days_to_resolve"
    ].notna().all(),

    "dates_in_2025": (
        clean["date_filed"].dt.year == 2025
    ).all(),

    "no_dup_ids": clean["request_id"].is_unique
}
```

Inspect:

```python
print(integrity)
```

Example structure:

```python
{
    "resolved_have_days": True,
    "dates_in_2025": True,
    "no_dup_ids": True
}
```

---

# 💾 19. Versioned Output

Never overwrite the raw file.

Save the cleaned version separately:

```python
clean.to_csv(
    "data/service_requests_clean_v1.csv",
    index=False
)
```

### Versioning idea

```text
service_requests.csv
        ↓
service_requests_clean_v1.csv
        ↓
service_requests_clean_v2.csv
        ↓
service_requests_clean_v3.csv
```

This makes the cleaning process easier to audit and reproduce.

---

# 🔄 20. Complete Day 5 Cleaning Workflow

```python
import pandas as pd
import numpy as np

# Load raw data
raw = pd.read_csv("data/service_requests.csv")

# Work on a copy
clean = raw.copy()

# 1. Remove full duplicate rows
clean = clean.drop_duplicates()

# 2. Standardize region
clean["region"] = clean["region"].str.strip().str.upper()

# 3. Fix known service type variant
clean["service_type"] = clean["service_type"].replace(
    {"Free Wifi Installation": "Free WiFi Installation"}
)

# 4. Parse mixed date formats
clean["date_filed"] = pd.to_datetime(
    clean["date_filed"],
    format="mixed"
)

# 5. Fill missing age groups
clean["citizen_age_group"] = clean["citizen_age_group"].fillna(
    "Unknown"
)

# 6. Flag and fill missing processing fees
clean["fee_was_missing"] = clean["processing_fee"].isna()

clean["processing_fee"] = clean["processing_fee"].fillna(
    clean["processing_fee"].median()
)

# 7. Calculate IQR upper fence
col = clean["days_to_resolve"]

q1, q3 = col.quantile([0.25, 0.75])
iqr = q3 - q1

upper_fence = q3 + 1.5 * iqr

# 8. Count potential outliers
outlier_count = (col > upper_fence).sum()

# 9. Count sentinel values
sentinel_count = (col == 999).sum()

# 10. Replace sentinel values
clean["days_to_resolve"] = clean["days_to_resolve"].replace(
    999, np.nan
)

# 11. Integrity checks
integrity = {
    "resolved_have_days": clean.loc[
        clean["status"] == "Resolved",
        "days_to_resolve"
    ].notna().all(),

    "dates_in_2025": (
        clean["date_filed"].dt.year == 2025
    ).all(),

    "no_dup_ids": clean["request_id"].is_unique
}

# 12. Save versioned output
clean.to_csv(
    "data/service_requests_clean_v1.csv",
    index=False
)

# 13. Display results
print("Shape:", clean.shape)
print("Upper fence:", upper_fence)
print("Outlier count:", outlier_count)
print("Sentinel count:", sentinel_count)
print("Integrity:", integrity)
print("Saved: data/service_requests_clean_v1.csv")
```

---

# 🧠 21. Quick Pandas Cheat Sheet

| Task                    | Code                                  |
| ----------------------- | ------------------------------------- |
| Load CSV                | `pd.read_csv("file.csv")`             |
| Shape                   | `df.shape`                            |
| Data types              | `df.dtypes`                           |
| Missing count           | `df.isna().sum()`                     |
| Duplicate rows          | `df.duplicated().sum()`               |
| Remove duplicates       | `df.drop_duplicates()`                |
| Duplicate key           | `df["id"].duplicated().sum()`         |
| Unique values           | `df["col"].unique()`                  |
| Number of unique values | `df["col"].nunique()`                 |
| Minimum                 | `df["col"].min()`                     |
| Maximum                 | `df["col"].max()`                     |
| Median                  | `df["col"].median()`                  |
| Q1/Q3                   | `df["col"].quantile([.25, .75])`      |
| Strip whitespace        | `.str.strip()`                        |
| Uppercase               | `.str.upper()`                        |
| Title case              | `.str.title()`                        |
| Replace values          | `.replace({"old": "new"})`            |
| Convert dates           | `pd.to_datetime(..., format="mixed")` |
| Fill missing            | `.fillna(value)`                      |
| Check missing           | `.isna()`                             |
| Replace sentinel        | `.replace(999, np.nan)`               |
| Unique IDs              | `df["id"].is_unique`                  |
| Save CSV                | `df.to_csv("file.csv", index=False)`  |

---

# ⚡ 22. Must-Remember Patterns

### Profile

```python
df.shape
df.dtypes
df.isna().sum()
df.duplicated().sum()
df["column"].unique()
df["column"].min()
df["column"].max()
```

### Clean

```python
clean = raw.copy()

clean = clean.drop_duplicates()

clean["region"] = clean["region"].str.strip().str.upper()

clean["service_type"] = clean["service_type"].replace(
    {"Free Wifi Installation": "Free WiFi Installation"}
)

clean["date_filed"] = pd.to_datetime(
    clean["date_filed"],
    format="mixed"
)
```

### Missing values

```python
clean["citizen_age_group"] = clean[
    "citizen_age_group"
].fillna("Unknown")
```

```python
clean["fee_was_missing"] = clean["processing_fee"].isna()

clean["processing_fee"] = clean["processing_fee"].fillna(
    clean["processing_fee"].median()
)
```

### Outliers

```python
q1, q3 = clean["days_to_resolve"].quantile([0.25, 0.75])
iqr = q3 - q1
upper_fence = q3 + 1.5 * iqr

outlier_count = (
    clean["days_to_resolve"] > upper_fence
).sum()
```

### Sentinel

```python
sentinel_count = (
    clean["days_to_resolve"] == 999
).sum()

clean["days_to_resolve"] = clean[
    "days_to_resolve"
].replace(999, np.nan)
```

### Integrity

```python
integrity = {
    "resolved_have_days": clean.loc[
        clean["status"] == "Resolved",
        "days_to_resolve"
    ].notna().all(),

    "dates_in_2025": (
        clean["date_filed"].dt.year == 2025
    ).all(),

    "no_dup_ids": clean["request_id"].is_unique
}
```

### Version

```python
clean.to_csv(
    "data/service_requests_clean_v1.csv",
    index=False
)
```

---

# 🎯 Day 5 Mental Model

```text
RAW DATA
   │
   ▼
PROFILE
   │
   ├── Shape
   ├── Missing values
   ├── Duplicates
   ├── Categories
   └── Numeric ranges
   │
   ▼
CLEAN
   │
   ├── Remove duplicates
   ├── Standardize categories
   ├── Parse dates
   ├── Handle missing values
   └── Correct known defects
   │
   ▼
VALIDATE
   │
   ├── Outliers
   ├── Sentinel values
   ├── Date rules
   ├── Required fields
   └── Duplicate IDs
   │
   ▼
VERSION
   │
   └── service_requests_clean_v1.csv
```

> **Golden rule:** Don't just make the data look clean. Make the cleaning process **defensible, reproducible, and auditable**.
