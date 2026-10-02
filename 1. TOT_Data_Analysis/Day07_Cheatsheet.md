# 📊 Day 7 — Summarizing Data Sets: Aggregation and Pivot Analysis

> **TESDA Alignment:** Summarize data sets
> **Main topics:** `groupby()`, `.agg()`, pivot tables, percentages, frequency analysis, binning, and crosstabs.

---

# 🧰 1. Load the Cleaned Dataset

```python
import pandas as pd

clean = pd.read_csv("data/service_requests_clean_v1.csv")
```

---

# 🔄 2. Split, Apply, Combine

Aggregation follows the **Split → Apply → Combine** pattern.

| Step    | Meaning                               |
| ------- | ------------------------------------- |
| Split   | Divide rows into groups               |
| Apply   | Calculate a statistic for each group  |
| Combine | Return the results as a summary table |

## Basic `groupby()`

```python
clean.groupby("region")["days_to_resolve"].mean()
```

This calculates the **average resolution time per region**.

---

# 📌 3. `groupby()` with `.agg()`

Use `.agg()` when you need several calculations at the same time.

```python
clean.groupby("region").agg(
    requests=("request_id", "count"),
    avg_days=("days_to_resolve", "mean")
)
```

### Common aggregation functions

| Function | Purpose                      |
| -------- | ---------------------------- |
| `count`  | Number of non-missing values |
| `mean`   | Average                      |
| `median` | Middle value                 |
| `sum`    | Total                        |
| `min`    | Smallest value               |
| `max`    | Largest value                |

---

# 🔢 4. Group by Multiple Columns

```python
clean.groupby(["region", "status"]).agg(
    request_count=("request_id", "count")
)
```

This creates a summary for every **region + status** combination.

The result may have a MultiIndex.

Use:

```python
.reset_index()
```

to turn the grouped index back into ordinary columns.

Example:

```python
two_level = (
    clean.groupby(["region", "status"])
    .agg(
        request_count=("request_id", "count")
    )
    .reset_index()
)
```

---

# 🏢 5. Grouped Summary — Service Type

```python
by_service = (
    clean.groupby("service_type")
    .agg(
        request_count=("request_id", "count"),
        mean_days_to_resolve=("days_to_resolve", "mean"),
        mean_satisfaction=("satisfaction_rating", "mean")
    )
    .sort_values("request_count", ascending=False)
)
```

### What this produces

One row per `service_type` containing:

* Request count
* Mean resolution days
* Mean satisfaction

The results are sorted from **highest request volume to lowest**.

---

# 🏢 6. Grouped Summary — Office

```python
by_office = (
    clean.groupby("office_code")
    .agg(
        request_count=("request_id", "count"),
        median_days_to_resolve=("days_to_resolve", "median")
    )
)
```

### Why use median?

`days_to_resolve` may be skewed by unusually long cases.

The median is less affected by extreme values.

---

# 🥇 7. Find the Busiest Group

If you already have a grouped table:

```python
busiest_pair = (
    two_level
    .sort_values("request_count", ascending=False)
    .iloc[0]
)
```

### Useful pattern

```python
.sort_values("column", ascending=False)
```

→ Sort highest to lowest.

```python
.iloc[0]
```

→ Select the first row.

---

# 📊 8. Pivot Tables

A pivot table is a **two-dimensional grouped summary**.

Basic structure:

```python
pd.pivot_table(
    clean,
    values="days_to_resolve",
    index="region",
    columns="service_type",
    aggfunc="mean",
    fill_value=0
)
```

## Pivot parameters

| Parameter    | Meaning                           |
| ------------ | --------------------------------- |
| `values`     | Column being summarized           |
| `index`      | Rows                              |
| `columns`    | Columns                           |
| `aggfunc`    | Calculation                       |
| `fill_value` | Value used for empty combinations |

---

# 🔢 9. Pivot Table — Request Counts

```python
pivot_counts = pd.pivot_table(
    clean,
    values="request_id",
    index="region",
    columns="channel",
    aggfunc="count",
    fill_value=0
)
```

This answers:

> How many requests did each region receive through each channel?

---

# 📈 10. Percentage of Each Region's Total

Start with the count pivot:

```python
pivot_pct = (
    pivot_counts
    .div(pivot_counts.sum(axis=1), axis=0)
    .mul(100)
    .round(1)
)
```

### Important

```python
pivot_counts.sum(axis=1)
```

calculates the total for each **row/region**.

```python
.div(..., axis=0)
```

divides each row by its row total.

```python
.mul(100)
```

converts proportions into percentages.

```python
.round(1)
```

rounds to one decimal place.

Each row should total approximately **100%**.

---

# ⏱️ 11. Pivot Table — Mean Resolution Time

```python
pivot_days = pd.pivot_table(
    clean,
    values="days_to_resolve",
    index="region",
    columns="status",
    aggfunc="mean"
).round(1)
```

This shows the average resolution time for each:

**Region × Status**

### Important

Do not automatically replace missing means with `0`.

For example, if `Pending` requests have no resolution time, `0` would incorrectly imply that they were resolved in zero days.

---

# ➕ 12. Pivot Table with Totals

```python
with_totals = pd.pivot_table(
    clean,
    values="request_id",
    index="region",
    columns="channel",
    aggfunc="count",
    fill_value=0,
    margins=True,
    margins_name="Total"
)
```

### Important parameters

```python
margins=True
```

Adds row and column totals.

```python
margins_name="Total"
```

Changes the default `All` label to `Total`.

---

# 🔢 13. Frequency Analysis

Frequency analysis answers:

> How often does each value occur?

## One column — `value_counts()`

```python
clean["service_type"].value_counts()
```

Count how many times each service type occurs.

### Sort categories

```python
clean["service_type"].value_counts().sort_index()
```

---

# 🔀 14. Crosstab

`pd.crosstab()` summarizes the frequency of combinations between two categorical variables.

```python
pd.crosstab(
    clean["channel"],
    clean["speed_band"]
)
```

### Structure

```text
pd.crosstab(ROWS, COLUMNS)
```

Example:

```python
pd.crosstab(
    clean["region"],
    clean["speed_band"]
)
```

Rows → regions
Columns → speed bands
Values → counts

---

# 📦 15. Binning with `pd.cut()`

Binning converts a continuous numeric variable into meaningful categories.

Example:

```python
clean["speed_band"] = pd.cut(
    clean["days_to_resolve"],
    bins=[-0.1, 3, 10, 30, 1000],
    labels=[
        "0-3 days",
        "4-10 days",
        "11-30 days",
        "Over 30 days"
    ]
)
```

### Result

| Original value | Category       |
| -------------: | -------------- |
|              1 | `0-3 days`     |
|              3 | `0-3 days`     |
|              7 | `4-10 days`    |
|             15 | `11-30 days`   |
|             45 | `Over 30 days` |

---

# ⚠️ 16. Why `-0.1`?

The dataset can contain `0` days.

Using:

```python
bins=[-0.1, 3, 10, 30, 1000]
```

ensures that `0` is included in the first category.

---

# 📊 17. Count the Bins

```python
band_counts = (
    clean["speed_band"]
    .value_counts()
    .sort_index()
)
```

### Why `sort_index()`?

It keeps the categories in their logical order:

```text
0-3 days
4-10 days
11-30 days
Over 30 days
```

instead of sorting them by frequency.

---

# 🔀 18. Crosstab of Region and Speed Band

```python
band_by_region = pd.crosstab(
    clean["region"],
    clean["speed_band"]
)
```

This shows the **number of requests** in each speed band for every region.

---

# 📈 19. Crosstab as Row Percentages

```python
band_pct = (
    pd.crosstab(
        clean["region"],
        clean["speed_band"],
        normalize="index"
    )
    .mul(100)
    .round(1)
)
```

### `normalize="index"`

Means:

> Calculate percentages within each row.

Therefore, each region's row should add up to approximately **100%**.

---

# 🏆 20. Find the Region with the Highest Delayed Share

```python
worst_region = band_pct["Over 30 days"].idxmax()
```

### Breakdown

```python
band_pct["Over 30 days"]
```

Selects the percentage of requests taking over 30 days.

```python
.idxmax()
```

Returns the row label with the highest value.

Therefore:

```python
worst_region
```

contains the region with the **highest share** of requests taking over 30 days.

---

# 📊 21. `pd.cut()` vs `pd.qcut()`

## `pd.cut()`

Uses manually defined boundaries.

```python
pd.cut(
    clean["days_to_resolve"],
    bins=[0, 3, 10, 30, 1000]
)
```

Useful when boundaries have real-world meaning.

Examples:

* Service standards
* Policy thresholds
* Processing targets
* Risk categories

---

## `pd.qcut()`

Creates approximately equal-sized groups based on the data distribution.

```python
pd.qcut(
    clean["days_to_resolve"],
    q=4
)
```

Useful for:

* Quartile analysis
* Ranking
* Distribution-based segmentation

---

# 🧠 22. Good Binning Rules

### Rule 1 — Use meaningful boundaries

Prefer:

```text
0-7 days
8-14 days
15-30 days
Over 30 days
```

if a **7-day service standard** exists.

Avoid arbitrary categories simply because they have equal widths.

### Rule 2 — Use clear names

Bad:

```text
Bin 1
Bin 2
Bin 3
Bin 4
```

Better:

```text
0-7 days
8-14 days
15-30 days
Over 30 days
```

The audience should understand the category without needing an explanation.

---

# ⚠️ 23. What Binning Loses

Binning simplifies continuous data but loses precision.

For example:

```text
11 days
29 days
```

could both become:

```text
11-30 days
```

The original difference between 11 and 29 days is no longer visible.

Use raw values when:

* Exact values matter.
* Small differences are important.
* You need detailed statistical analysis.
* You need to identify extreme values.
* You need to understand the actual distribution.

Useful alternatives:

```python
clean["days_to_resolve"].describe()
```

Histogram:

```python
clean["days_to_resolve"].hist()
```

Boxplot:

```python
clean.boxplot(column="days_to_resolve")
```

---

# ⚠️ 24. When `fill_value=0` Helps

Example:

```python
pd.pivot_table(
    clean,
    values="request_id",
    index="region",
    columns="channel",
    aggfunc="count",
    fill_value=0
)
```

For **counts**, an empty combination can reasonably mean:

> There were zero requests.

Therefore, `fill_value=0` is useful.

---

# 🚨 25. When `fill_value=0` Can Lie

For measurements such as:

```python
days_to_resolve
```

an empty cell may mean:

> There is no valid measurement.

It does **not** necessarily mean:

> The value is zero.

For example, a pending request has no completed resolution time.

Therefore:

```text
Missing ≠ 0
```

Use `fill_value=0` based on what an empty cell actually means.

---

# ⚖️ 26. Count vs Percentage

## Count

Shows the actual volume.

Example:

```text
Region A → 500 walk-in requests
Region B → 200 walk-in requests
```

Useful for:

* Workload
* Staffing
* Resource requirements
* Number of actual cases

---

## Percentage

Shows the share within each region.

Example:

```text
Region A → 20% walk-in
Region B → 50% walk-in
```

Useful for:

* Comparing regions of different sizes
* Understanding channel preferences
* Comparing proportions

### Important

A large region can have:

> High count but low percentage.

A small region can have:

> Low count but high percentage.

Therefore, count and percentage answer **different questions**.

---

# 🧮 27. Common Percentage Pattern

## Percentage of row total

```python
table.div(table.sum(axis=1), axis=0).mul(100).round(1)
```

Use this for:

> What percentage of each region's requests belong to each category?

---

## Percentage of column total

```python
table.div(table.sum(axis=0), axis=1).mul(100).round(1)
```

Use this for:

> What percentage of each category comes from each region?

---

# 📋 28. Complete Day 7 Workflow

```python
import pandas as pd

# Load data
clean = pd.read_csv("data/service_requests_clean_v1.csv")

# ============================================================
# GROUPED SUMMARIES
# ============================================================

by_service = (
    clean.groupby("service_type")
    .agg(
        request_count=("request_id", "count"),
        mean_days_to_resolve=("days_to_resolve", "mean"),
        mean_satisfaction=("satisfaction_rating", "mean")
    )
    .sort_values("request_count", ascending=False)
)

by_office = (
    clean.groupby("office_code")
    .agg(
        request_count=("request_id", "count"),
        median_days_to_resolve=("days_to_resolve", "median")
    )
)

two_level = (
    clean.groupby(["region", "status"])
    .agg(
        request_count=("request_id", "count")
    )
    .reset_index()
)

busiest_pair = (
    two_level
    .sort_values("request_count", ascending=False)
    .iloc[0]
)

# ============================================================
# PIVOT TABLES
# ============================================================

pivot_counts = pd.pivot_table(
    clean,
    values="request_id",
    index="region",
    columns="channel",
    aggfunc="count",
    fill_value=0
)

pivot_pct = (
    pivot_counts
    .div(pivot_counts.sum(axis=1), axis=0)
    .mul(100)
    .round(1)
)

pivot_days = pd.pivot_table(
    clean,
    values="days_to_resolve",
    index="region",
    columns="status",
    aggfunc="mean"
).round(1)

with_totals = pd.pivot_table(
    clean,
    values="request_id",
    index="region",
    columns="channel",
    aggfunc="count",
    fill_value=0,
    margins=True,
    margins_name="Total"
)

# ============================================================
# BINNING AND FREQUENCY ANALYSIS
# ============================================================

clean["speed_band"] = pd.cut(
    clean["days_to_resolve"],
    bins=[-0.1, 3, 10, 30, 1000],
    labels=[
        "0-3 days",
        "4-10 days",
        "11-30 days",
        "Over 30 days"
    ]
)

band_counts = (
    clean["speed_band"]
    .value_counts()
    .sort_index()
)

band_by_region = pd.crosstab(
    clean["region"],
    clean["speed_band"]
)

band_pct = (
    pd.crosstab(
        clean["region"],
        clean["speed_band"],
        normalize="index"
    )
    .mul(100)
    .round(1)
)

worst_region = band_pct["Over 30 days"].idxmax()

print("Service Summary:")
print(by_service)

print("\nOffice Summary:")
print(by_office)

print("\nTwo-Level Summary:")
print(two_level)

print("\nBusiest Pair:")
print(busiest_pair)

print("\nChannel Pivot:")
print(pivot_counts)

print("\nChannel Percentage:")
print(pivot_pct)

print("\nMean Days Pivot:")
print(pivot_days)

print("\nPivot with Totals:")
print(with_totals)

print("\nSpeed Band Counts:")
print(band_counts)

print("\nSpeed Band by Region:")
print(band_by_region)

print("\nSpeed Band Percentages:")
print(band_pct)

print("\nRegion with Highest Over-30-Day Share:")
print(worst_region)
```

---

# 📝 29. Quick Syntax Reference

| Task                      | Code                                   |
| ------------------------- | -------------------------------------- |
| Group by one column       | `df.groupby("col")`                    |
| Group by multiple columns | `df.groupby(["col1", "col2"])`         |
| Count                     | `.count()`                             |
| Mean                      | `.mean()`                              |
| Median                    | `.median()`                            |
| Sum                       | `.sum()`                               |
| Multiple aggregations     | `.agg(...)`                            |
| Flatten grouped index     | `.reset_index()`                       |
| Sort descending           | `.sort_values("col", ascending=False)` |
| First row                 | `.iloc[0]`                             |
| Create bins               | `pd.cut()`                             |
| Create quartile groups    | `pd.qcut()`                            |
| Count categories          | `.value_counts()`                      |
| Two-way frequency table   | `pd.crosstab()`                        |
| Crosstab percentages      | `normalize="index"`                    |
| Pivot table               | `pd.pivot_table()`                     |
| Add pivot totals          | `margins=True`                         |
| Rename total              | `margins_name="Total"`                 |
| Highest value's label     | `.idxmax()`                            |
| Row total                 | `.sum(axis=1)`                         |
| Column total              | `.sum(axis=0)`                         |

---

# 🎯 30. Key Takeaways

1. **`groupby()`** is the main tool for Split → Apply → Combine.
2. **`.agg()`** allows several summaries in one operation.
3. **`median()`** is useful when numeric data may be skewed or contain extreme values.
4. **Pivot tables** organize grouped summaries into rows and columns.
5. **Count tables** show actual workload; **percentage tables** show proportions.
6. **`value_counts()`** analyzes frequencies for one variable.
7. **`pd.crosstab()`** analyzes frequencies between two variables.
8. **`pd.cut()`** converts continuous values into meaningful categories.
9. **`pd.qcut()`** creates approximately equal-sized groups.
10. **`normalize="index"`** calculates percentages within each row.
11. **`fill_value=0`** is appropriate when an empty combination genuinely means zero.
12. **Missing values should not automatically be treated as zero.**
13. Binning improves readability but **loses numerical detail**.
14. Always choose grouping and binning based on the **question being answered and the audience**.
