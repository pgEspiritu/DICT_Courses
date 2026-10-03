# 📊 Day 8 — Preparing Data Visualization I: Charts and Design

> **TESDA Alignment:** Prepare Data Visualization
>
> **Main Workflow:** QUESTION → CHART TYPE → BUILD → CHECK → INTERPRET

---

## 🎯 Learning Objectives

By the end of this lesson, you should be able to:

- Choose an appropriate chart for a question.
- Create bar and line charts.
- Create histograms, box plots, and scatter plots.
- Compare distributions using different bin counts.
- Label charts clearly.
- Improve charts using basic design principles.
- Add direct labels to charts.
- Identify misleading or unnecessary chart elements.
- Interpret visual patterns without overstating conclusions.

---

# 1. 🧭 Choosing the Right Chart

| Question | First Chart to Consider |
|---|---|
| Compare categories | Bar chart |
| Changed over time | Line chart |
| Numeric distribution | Histogram or box plot |
| Two numeric variables relate | Scatter plot |
| Whole composition | Stacked bar chart |
| Simple part-to-whole with 2–3 slices | Pie chart |

### One-Sentence Test

> "I need to show _____ so the reader can decide _____."

The **question** should determine the chart type.

---

# 2. 📦 Load Data for Visualization

```python
import pandas as pd
import matplotlib.pyplot as plt

clean = pd.read_csv(
    "data/service_requests_clean_v1.csv",
    parse_dates=["date_filed"]
)
```

The cleaned dataset should generally be used for visualization unless the task specifically asks for raw data.

---

# 3. 📊 Bar Chart — Compare Categories

A bar chart is useful when comparing independent categories.

```python
by_service = clean["service_type"].value_counts()

plt.figure(figsize=(10, 5))

plt.bar(
    by_service.index,
    by_service.values
)

plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Service Type")
plt.ylabel("Number of requests")

plt.xticks(rotation=45, ha="right")

plt.tight_layout()
plt.show()
```

---

# 4. 🔢 Sort Bars by Value

Sorting makes ranking easier to see.

### Ascending

```python
ordered = by_service.sort_values()
```

Smallest → largest.

### Descending

```python
ordered = by_service.sort_values(ascending=False)
```

Largest → smallest.

### Quick Reference

```text
.sort_values()
→ ascending order

.sort_values(ascending=False)
→ descending order
```

---

# 5. 📈 Line Chart — Change Over Time

Use a line chart when the x-axis has a natural sequence, such as months.

```python
clean["date_filed"] = pd.to_datetime(clean["date_filed"])

clean_by_date = clean.set_index("date_filed")

monthly_volume = clean_by_date.resample("ME").size()

plt.figure(figsize=(10, 5))

plt.plot(
    monthly_volume.index,
    monthly_volume.values,
    marker="o"
)

plt.title("Monthly Service Request Volume, 2025")
plt.xlabel("Month")
plt.ylabel("Number of requests")

plt.tight_layout()
plt.show()
```

### Important

Do not connect unrelated categories with a line simply because the chart looks attractive.

A line implies **sequence or continuity**.

---

# 6. 🏆 Find the Peak Month

### `.max()`

Returns the highest value.

```python
highest_volume = monthly_volume.max()
```

### `.idxmax()`

Returns the index where the highest value occurs.

```python
peak_month = monthly_volume.idxmax()

print(peak_month)
```

### Key Difference

```text
.max()
→ highest value

.idxmax()
→ location/index of highest value
```

---

# 7. 📊 Histogram

A histogram shows the distribution of one numeric variable.

```python
days = clean["days_to_resolve"].dropna()

plt.figure(figsize=(10, 5))

plt.hist(
    days,
    bins=20,
    edgecolor="white"
)

plt.title("Distribution of Resolution Time")
plt.xlabel("Days to resolve")
plt.ylabel("Number of requests")

plt.tight_layout()
plt.show()
```

---

# 8. 🔢 Histogram Bin Selection

The number of bins changes how the distribution appears.

### 20 Bins

```python
plt.hist(
    days,
    bins=20,
    edgecolor="white"
)
```

Fewer bins provide a broader view of the distribution and generally reduce visual noise.

### 60 Bins

```python
plt.hist(
    days,
    bins=60,
    edgecolor="white"
)
```

More bins provide greater detail but may make the distribution appear fragmented or noisy.

### Good Practice

Try more than one reasonable bin count before publishing.

Choose the version that communicates important structure clearly without unnecessary noise.

---

# 9. 📦 Box Plot

A box plot summarizes the distribution of a numeric variable.

### Main Components

```text
Box
→ Q1 to Q3

Line inside box
→ Median

Whiskers
→ Values within the usual 1.5 × IQR rule

Points beyond whiskers
→ Potential statistical outliers
```

---

# 10. 📦 Box Plot by Channel

```python
channels = sorted(clean["channel"].dropna().unique())

data_by_channel = [
    clean[clean["channel"] == channel]["days_to_resolve"].dropna()
    for channel in channels
]

plt.figure(figsize=(10, 5))

plt.boxplot(
    data_by_channel,
    tick_labels=channels
)

plt.title("Resolution Time by Channel, 2025")
plt.xlabel("Channel")
plt.ylabel("Days to resolve")

plt.xticks(rotation=45, ha="right")

plt.tight_layout()
plt.show()
```

---

# 11. ⚠️ Box Plot Outliers

Points beyond the whiskers are **not automatically errors**.

They are potential statistical outliers.

Before removing an unusual observation, check:

- Original record.
- Data-entry procedures.
- Definition of the variable.
- Whether the value is plausible.
- Whether there is a documented reason for the unusual value.

### Important Principle

```text
Statistical outlier ≠ automatically data error
```

Do not delete an observation simply because it is far from the others.

---

# 12. 🔵 Scatter Plot

A scatter plot shows the relationship between two numeric variables.

Use it to examine:

- Direction.
- Strength.
- Clusters.
- Unusual observations.

```python
sub = clean.dropna(
    subset=[
        "days_to_resolve",
        "satisfaction_rating"
    ]
)

plt.figure(figsize=(8, 5))

plt.scatter(
    sub["days_to_resolve"],
    sub["satisfaction_rating"],
    alpha=0.2
)

plt.title("Resolution Time vs Satisfaction")
plt.xlabel("Days")
plt.ylabel("Rating")

plt.tight_layout()
plt.show()
```

### Important

```text
Association ≠ causation
```

A scatter plot can show an association or pattern.

It does not prove that one variable causes another.

---

# 13. 🏢 Scatter Plot — Staffing and Workload

The office master data is stored as nested JSON.

First flatten it.

```python
import json

with open("data/regional_offices.json") as f:
    offices = pd.json_normalize(
        json.load(f)["offices"]
    )
```

---

# 14. 🔢 Count Requests by Office

```python
request_counts = (
    clean.groupby("office_code")
    .size()
    .rename("request_count")
    .reset_index()
)
```

This produces a table containing:

```text
office_code
request_count
```

---

# 15. 🔗 Merge Office Data and Request Counts

```python
office_load = offices[
    ["office_code", "staff_count"]
].merge(
    request_counts,
    on="office_code",
    how="left"
)
```

The resulting `office_load` should contain at least:

```text
office_code
staff_count
request_count
```

---

# 16. 🔵 Scatter Plot with Office Code Labels

```python
plt.figure(figsize=(10, 6))

plt.scatter(
    office_load["staff_count"],
    office_load["request_count"]
)

for _, row in office_load.iterrows():
    plt.annotate(
        row["office_code"],
        (row["staff_count"], row["request_count"]),
        xytext=(5, 5),
        textcoords="offset points"
    )

plt.title("Office Staffing and Service Request Workload, 2025")
plt.xlabel("Staff Count")
plt.ylabel("Number of Requests")

plt.tight_layout()
plt.show()
```

### Interpretation Caution

There are only **nine offices**.

Therefore:

```text
Observed pattern
→ reasonable to discuss

Causal staffing policy conclusion
→ overreach
```

The scatter can show how the nine offices differ.

It does not establish that staffing causes workload differences.

---

# 17. 🎨 Good Chart Design

Good chart design is mostly **subtraction**.

## ✅ Do

- Give the chart a descriptive title.
- State the finding only when the data supports it.
- Label axes.
- Include units.
- Use one color unless color has a purpose.
- Sort bars by value unless there is a natural order.
- Start bar axes at zero.
- Remove decorative elements that do not help interpretation.

---

# 18. ❌ Avoid Chart Junk

Avoid:

- 3-D effects that distort comparisons.
- Rainbow palettes for ordinary categories.
- Pie charts with too many slices.
- Legends when direct labels are clearer.
- Heavy borders.
- Unnecessary gridlines.
- Truncated bar axes that exaggerate differences.
- Decorative effects that do not improve interpretation.

---

# 19. ♿ Accessibility

Never make color the **only signal** for meaning.

A reader should still understand the chart if color discrimination is limited.

Use labels, position, patterns, or other visual cues when necessary.

---

# 20. 🏷️ Descriptive Chart Titles

### Poor Title

```python
plt.title("Chart 1")
```

This does not tell the reader what the chart represents.

### Better Title

```python
plt.title("Service Requests by Service Type, 2025")
```

A useful title tells the reader:

- What is being measured.
- What is being compared.
- Relevant time period.

---

# 21. ⚠️ Finding-Based Titles

A finding-based title can be useful, but it must remain supported by the data.

### Risk

A finding-based title may make the chart appear to support a stronger conclusion than the data actually demonstrates.

### Keep It Honest

The title should:

- Describe what the chart directly shows.
- Avoid unsupported causal claims.
- Include relevant context.
- Avoid implying more evidence than is available.

---

# 22. 📏 Bar Charts Should Start at Zero

For a horizontal bar chart:

```python
plt.xlim(left=0)
```

For a vertical bar chart:

```python
plt.ylim(bottom=0)
```

### Why?

A truncated bar axis can exaggerate relatively small differences.

---

# 23. ↔️ Horizontal Bar Chart

Horizontal bars are useful when category names are long.

```python
ordered = by_service.sort_values()

plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values
)

plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Number of requests")

plt.xlim(left=0)

plt.tight_layout()
plt.show()
```

---

# 24. 🎯 One Main Color + One Highlight

Use one main color for ordinary categories and a second color only when the highlight has a purpose.

```python
plt.barh(
    ordered.index,
    ordered.values,
    color="steelblue"
)

plt.barh(
    ordered.index[-1],
    ordered.values[-1],
    color="orange"
)
```

### Purpose

```text
Main color
→ ordinary categories

Highlight
→ category being specifically discussed
```

Do not use multiple colors simply for decoration.

---

# 25. 🏷️ Direct Labels

Direct labels make exact values easier to read without requiring the reader to estimate from the axis.

```python
for i, value in enumerate(ordered.values):
    plt.text(
        value,
        i,
        f"{value}",
        va="center",
        ha="left",
        fontsize=10
    )
```

For horizontal bars:

```text
Service A  ───────────── 245
Service B  ──────────────── 310
Service C  ─────────────────── 350
```

---

# 26. 🧹 Removing Chart Junk

Example:

```python
plt.grid(False)
```

Avoid unnecessary:

- Legends.
- Borders.
- Gridlines.
- Colors.
- 3-D effects.
- Decorative elements.

### Principle

> If an element does not help the reader interpret the data, consider removing it.

---

# 27. 🛠️ Complete Redesigned Horizontal Bar Chart

```python
# ============================================================
# REDESIGNED SERVICE REQUEST CHART
# ============================================================

# STEP 1 — Sort the service counts.
ordered = by_service.sort_values()


# STEP 2 — Create the horizontal bar chart.
plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values,
    color="steelblue"
)


# STEP 3 — Highlight the highest-value category.
plt.barh(
    ordered.index[-1],
    ordered.values[-1],
    color="orange"
)


# STEP 4 — Add direct value labels.
for i, value in enumerate(ordered.values):
    plt.text(
        value,
        i,
        f"{value}",
        va="center",
        ha="left",
        fontsize=10
    )


# STEP 5 — Add descriptive title.
plt.title("Service Requests by Service Type, 2025")


# STEP 6 — Add axis label and unit.
plt.xlabel("Number of requests")


# STEP 7 — Start value axis at zero.
plt.xlim(left=0)


# STEP 8 — Remove unnecessary chart junk.
plt.grid(False)


# STEP 9 — Adjust spacing.
plt.tight_layout()


# STEP 10 — Display chart.
plt.show()


# STEP 11 — Mark redesign complete.
redesign_done = True

print("Chart redesign complete:", redesign_done)
```

---

# 28. 🧠 Five-Second Test

Ask:

> **Can a colleague state the chart's main point in five seconds without you explaining it?**

If not, consider improving:

- Title.
- Chart type.
- Sorting.
- Labels.
- Axis scale.
- Color use.
- Unnecessary elements.

---

# 29. 🔍 Interpretation Guide

## Histogram

Ask:

- Is the distribution symmetric or skewed?
- Are there clusters?
- Are there unusual values?
- Does the bin choice reveal or hide structure?

## Box Plot

Ask:

- Which group has the higher median?
- Which group has greater spread?
- Are there potential outliers?
- Are distributions similar or different?

## Scatter Plot

Ask:

- Is the relationship positive or negative?
- Is the relationship weak or strong?
- Are there clusters?
- Are there unusual observations?

---

# 30. ⚠️ Avoid Overclaiming

### Avoid:

```text
Higher staffing causes more requests.
```

if the chart only shows an association.

### Better:

```text
The nine offices show an observed pattern between staff count and request workload.
```

Similarly, avoid:

```text
Longer resolution times cause lower satisfaction.
```

A scatter plot alone cannot establish causation.

### Better:

```text
The chart can be used to examine whether resolution time and satisfaction are associated.
```

---

# 31. 📊 Count vs Percentage

When comparing groups of different sizes, ask whether **counts** or **percentages/rates** are more appropriate.

### Count

```python
clean["region"].value_counts()
```

Shows the number of records.

### Percentage

```python
clean["region"].value_counts(normalize=True) * 100
```

Shows the percentage of records.

The choice depends on the analytical question.

---

# 32. 🧰 Essential Pandas Commands

### Count categories

```python
clean["service_type"].value_counts()
```

### Sort ascending

```python
series.sort_values()
```

### Sort descending

```python
series.sort_values(ascending=False)
```

### Group and count

```python
clean.groupby("office_code").size()
```

### Reset index

```python
.reset_index()
```

### Remove missing values

```python
series.dropna()
```

### Convert to dates

```python
pd.to_datetime(clean["date_filed"])
```

### Find maximum value

```python
series.max()
```

### Find location of maximum

```python
series.idxmax()
```

---

# 33. 🧰 Essential Matplotlib Commands

### Create figure

```python
plt.figure(figsize=(10, 5))
```

### Bar chart

```python
plt.bar(x, y)
```

### Horizontal bar chart

```python
plt.barh(x, y)
```

### Line chart

```python
plt.plot(x, y, marker="o")
```

### Histogram

```python
plt.hist(values, bins=20)
```

### Box plot

```python
plt.boxplot(data)
```

### Scatter plot

```python
plt.scatter(x, y)
```

### Title

```python
plt.title("Descriptive Title")
```

### X-axis label

```python
plt.xlabel("X-axis label")
```

### Y-axis label

```python
plt.ylabel("Y-axis label")
```

### Rotate labels

```python
plt.xticks(rotation=45, ha="right")
```

### Set x-axis minimum

```python
plt.xlim(left=0)
```

### Set y-axis minimum

```python
plt.ylim(bottom=0)
```

### Disable grid

```python
plt.grid(False)
```

### Improve spacing

```python
plt.tight_layout()
```

### Display chart

```python
plt.show()
```

---

# 34. 📌 Quick Chart Selection Cheat Sheet

```text
CATEGORY COMPARISON
        ↓
   BAR CHART

CHANGE OVER TIME
        ↓
   LINE CHART

ONE NUMERIC VARIABLE
        ↓
HISTOGRAM / BOX PLOT

TWO NUMERIC VARIABLES
        ↓
 SCATTER PLOT

PART OF A WHOLE
        ↓
STACKED BAR / SIMPLE PIE
```

---

# 35. 🧠 Important Lessons from Day 8

### 1. Start with the question

```text
QUESTION
   ↓
CHART TYPE
   ↓
BUILD
   ↓
CHECK
   ↓
INTERPRET
```

### 2. More visual detail is not always better

A 60-bin histogram provides more detail than a 20-bin histogram, but excessive detail can create noise.

### 3. Outliers require investigation

An unusual value is not automatically an error.

### 4. Small samples require caution

Nine offices are enough to describe observed differences, but the small number of offices limits what can reasonably be inferred.

### 5. Association is not causation

A visible relationship does not prove that one variable causes another.

### 6. Design should support interpretation

Use color, labels, sorting, and formatting only when they help the reader understand the data.

### 7. Good chart design is mostly subtraction

Remove anything that does not help answer the question.

---

# 36. 🚀 Complete Day 8 Visualization Workflow

```python
# ============================================================
# DAY 8 — BASIC DATA VISUALIZATION WORKFLOW
# ============================================================

# STEP 1 — Import libraries.
import pandas as pd
import matplotlib.pyplot as plt


# STEP 2 — Load cleaned data.
clean = pd.read_csv(
    "data/service_requests_clean_v1.csv",
    parse_dates=["date_filed"]
)


# STEP 3 — Ask the analytical question.
# Example:
# Which service types receive the most requests?


# STEP 4 — Prepare the data.
by_service = clean["service_type"].value_counts()
ordered = by_service.sort_values()


# STEP 5 — Build the chart.
plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values
)


# STEP 6 — Add descriptive labels.
plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Number of requests")


# STEP 7 — Start bar axis at zero.
plt.xlim(left=0)


# STEP 8 — Improve spacing.
plt.tight_layout()


# STEP 9 — Display the chart.
plt.show()


# STEP 10 — Apply the five-second test.
# Ask:
# "Can a colleague understand the chart's main point quickly?"
```

---

# 37. 📝 Final Chart Checklist

Before publishing a chart, check:

- [ ] Does the chart answer a clear question?
- [ ] Is the chart type appropriate?
- [ ] Is the title descriptive?
- [ ] Are the axes labeled?
- [ ] Are units included?
- [ ] Are categories sorted when appropriate?
- [ ] Does a bar chart start at zero?
- [ ] Is color being used for a purpose?
- [ ] Is the chart accessible without relying only on color?
- [ ] Are unnecessary legends removed?
- [ ] Are unnecessary borders/gridlines removed?
- [ ] Are labels readable?
- [ ] Are unusual observations investigated?
- [ ] Does the chart avoid unsupported conclusions?
- [ ] Does it pass the five-second test?

---

# 🎯 Day 8 Key Takeaway

> **A good visualization is not the chart with the most decoration or detail. It is the chart that communicates the intended data message clearly, honestly, and efficiently.**
