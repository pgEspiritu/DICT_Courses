# 📊 Day 8 — Preparing Data Visualization I: Charts and Design
## 🧠 Complete Cheatsheet with Line-by-Line Code Explanations

---

# 🎯 1. Learning Objectives

By the end of Day 8, you should be able to:

- 📊 Choose an appropriate chart for a question.
- 📈 Create bar and line charts.
- 📉 Create histograms and box plots.
- 🔵 Create scatter plots.
- 🔗 Combine data from CSV and JSON sources.
- 🎨 Apply basic chart-design principles.
- 🏷️ Add descriptive titles, labels, and annotations.
- 👀 Interpret charts without overclaiming what the data proves.

---

# 🔄 2. Visualization Workflow

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

### 🧠 One-Sentence Test

> "I need to show _____ so the reader can decide _____."

Always start with the question before choosing the chart.

---

# 📚 3. Choosing the Right Chart

| Question | 📊 Chart to Consider | Example |
|---|---|---|
| Compare categories | Bar chart | Requests by service type |
| Show change over time | Line chart | Monthly request volume |
| Show one numeric distribution | Histogram | Distribution of resolution days |
| Compare distributions | Box plot | Resolution time by channel |
| Show relationship between two numeric variables | Scatter plot | Staff count vs request count |
| Show composition | Stacked bar | Status composition by region |

### ⚠️ Important

Do not choose a chart simply because it looks attractive.

Choose the chart that makes the **question easiest to answer**.

---

# 🛠️ 4. Import the Libraries

### 💻 Code

```python
import pandas as pd
import matplotlib.pyplot as plt
```

### 🔍 Line-by-Line Explanation

```python
import pandas as pd
```

- 📦 Imports the `pandas` library.
- Pandas is used for working with tables and datasets.
- `as pd` creates a short nickname.
- Therefore, instead of `pandas.read_csv()`, we can use `pd.read_csv()`.

```python
import matplotlib.pyplot as plt
```

- 📦 Imports Matplotlib's plotting functions.
- `matplotlib.pyplot` contains functions for creating charts.
- `as plt` gives it the shorter name `plt`.
- Most chart commands therefore begin with `plt.`.

### 🧠 Remember

```text
pandas       → data manipulation
matplotlib   → data visualization
pd           → pandas shortcut
plt          → matplotlib.pyplot shortcut
```

---

# 📥 5. Load the Clean Dataset

### 💻 Code

```python
clean = pd.read_csv(
    "data/service_requests_clean_v1.csv",
    parse_dates=["date_filed"]
)
```

### 🔍 Line-by-Line Explanation

```python
clean = pd.read_csv(
```

- 📂 `pd.read_csv()` reads a CSV file.
- The resulting DataFrame is assigned to `clean`.
- `clean` will contain the cleaned service-request dataset.

```python
    "data/service_requests_clean_v1.csv",
```

- 📍 Specifies the location of the CSV file.
- `data/` means the file is inside the `data` folder.
- `service_requests_clean_v1.csv` is the filename.

```python
    parse_dates=["date_filed"]
```

- 📅 Tells pandas to interpret `date_filed` as dates.
- This is especially useful for time-based analysis.
- It allows us to perform operations such as monthly resampling.

```python
)
```

- 🔚 Closes the `pd.read_csv()` function.

---

# 📊 6. Bar Chart — Requests by Service Type

## 🎯 Purpose

Use a **bar chart** when comparing categories.

Example question:

> Which service types received the most requests?

### 💻 Code

```python
by_service = clean["service_type"].value_counts()
by_service = by_service.sort_values(ascending=False)

print("Requests by Service Type:")
print(by_service)

plt.figure(figsize=(10, 5))
plt.bar(by_service.index, by_service.values)
plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Service Type")
plt.ylabel("Number of Requests")
plt.xticks(rotation=45, ha="right")
plt.tight_layout()
plt.show()
```

## 🔍 Line-by-Line Explanation

```python
by_service = clean["service_type"].value_counts()
```

- 🔎 `clean["service_type"]` selects the `service_type` column.
- `.value_counts()` counts how many times each service type appears.
- The result is stored in `by_service`.

Example result:

```text
Digital Literacy Training       220
Free WiFi Installation          210
Business Permit Assistance      205
...
```

---

```python
by_service = by_service.sort_values(ascending=False)
```

- 🔃 `.sort_values()` sorts the values.
- `ascending=False` means largest values come first.
- This makes the ranking easier to see.

### 🧠 Remember

```python
ascending=True
```

➡️ Smallest → largest

```python
ascending=False
```

➡️ Largest → smallest

---

```python
print("Requests by Service Type:")
```

- 🖨️ Displays a descriptive heading in the output.

---

```python
print(by_service)
```

- 🖨️ Displays the service-type counts.

---

```python
plt.figure(figsize=(10, 5))
```

- 🖼️ Creates a new chart figure.
- `figsize=(10, 5)` sets the chart size.
- `10` = width.
- `5` = height.

---

```python
plt.bar(by_service.index, by_service.values)
```

- 📊 Creates a vertical bar chart.
- `by_service.index` supplies the category names.
- `by_service.values` supplies the numerical values.

### 🧠 Remember

```text
index  → category names
values → numbers
```

---

```python
plt.title("Service Requests by Service Type, 2025")
```

- 🏷️ Adds a descriptive title.
- The title identifies:
  - what is being measured;
  - how it is grouped;
  - the relevant year.

---

```python
plt.xlabel("Service Type")
```

- 🏷️ Labels the x-axis.
- The x-axis contains the service categories.

---

```python
plt.ylabel("Number of Requests")
```

- 🏷️ Labels the y-axis.
- It tells the reader what the numerical values represent.

---

```python
plt.xticks(rotation=45, ha="right")
```

- 🔄 Changes the appearance of x-axis labels.
- `rotation=45` rotates labels by 45 degrees.
- `ha="right"` right-aligns the labels.
- This helps when category names are long.

---

```python
plt.tight_layout()
```

- 📐 Automatically adjusts the spacing.
- Helps prevent labels from being cut off.

---

```python
plt.show()
```

- 👀 Displays the completed chart.

---

# 📈 7. Line Chart — Monthly Request Volume

## 🎯 Purpose

Use a **line chart** when showing change over time.

Example question:

> How did monthly request volume change during 2025?

### 💻 Code

```python
clean["date_filed"] = pd.to_datetime(clean["date_filed"])

clean_by_date = clean.set_index("date_filed")

monthly_volume = clean_by_date.resample("ME").size()

print("Monthly Request Volume:")
print(monthly_volume)

plt.figure(figsize=(10, 5))
plt.plot(monthly_volume.index, monthly_volume.values, marker="o")
plt.title("Monthly Service Request Volume, 2025")
plt.xlabel("Month")
plt.ylabel("Number of requests")
plt.tight_layout()
plt.show()
```

## 🔍 Line-by-Line Explanation

```python
clean["date_filed"] = pd.to_datetime(clean["date_filed"])
```

- 📅 Converts `date_filed` into pandas datetime format.
- Useful when dates need to be grouped or plotted over time.

---

```python
clean_by_date = clean.set_index("date_filed")
```

- 📅 Makes `date_filed` the DataFrame index.
- A datetime index allows time-based operations.

---

```python
monthly_volume = clean_by_date.resample("ME").size()
```

- 📅 `.resample()` groups records into time periods.
- `"ME"` means month-end frequency.
- `.size()` counts the records in each month.
- The result is monthly request volume.

### 🧠 Think of it as:

```text
Daily records
     ↓
Group by month
     ↓
Count records
     ↓
Monthly request volume
```

---

```python
print("Monthly Request Volume:")
```

- 🖨️ Displays a heading.

---

```python
print(monthly_volume)
```

- 🖨️ Displays the monthly counts.

---

```python
plt.figure(figsize=(10, 5))
```

- 🖼️ Creates a 10 × 5 chart.

---

```python
plt.plot(monthly_volume.index, monthly_volume.values, marker="o")
```

- 📈 Creates the line chart.
- `monthly_volume.index` = months.
- `monthly_volume.values` = request counts.
- `marker="o"` adds a circular marker to each month.

---

```python
plt.title("Monthly Service Request Volume, 2025")
```

- 🏷️ Adds a descriptive title.

---

```python
plt.xlabel("Month")
```

- 🏷️ Labels the x-axis.

---

```python
plt.ylabel("Number of requests")
```

- 🏷️ Labels the y-axis.

---

```python
plt.tight_layout()
```

- 📐 Adjusts spacing.

---

```python
plt.show()
```

- 👀 Displays the chart.

---

# 🔝 8. Find the Peak Month with `.idxmax()`

### 💻 Code

```python
peak_month = monthly_volume.idxmax()
print(peak_month)
```

### 🔍 Explanation

```python
peak_month = monthly_volume.idxmax()
```

- 🔎 `.idxmax()` finds the **index associated with the largest value**.
- Since the index contains months, it returns the month with the highest request volume.

```python
print(peak_month)
```

- 🖨️ Displays the peak month.

### 🧠 VERY IMPORTANT

```text
.max()
   ↓
Highest value

.idxmax()
   ↓
Location/index of highest value
```

### Example

If:

```text
January     90
February   120
March      150
```

Then:

```python
monthly_volume.max()
```

returns:

```text
150
```

while:

```python
monthly_volume.idxmax()
```

returns:

```text
March
```

---

# 📅 9. One Year Does Not Prove Seasonality

A single year of monthly observations can show **variation over time**, but it cannot by itself establish a reliable seasonal pattern.

### ⚠️ Avoid saying:

> "Requests are seasonal."

### ✅ Better:

> "Request volume varied across the months of 2025."

For stronger evidence of seasonality, examine:

- 📅 Multiple years
- 🔁 Repeated patterns
- 📊 Additional contextual evidence

---

# 📊 10. Histogram — Distribution of Resolution Time

## 🎯 Purpose

A histogram shows the **distribution of one numeric variable**.

Example question:

> How are resolution times distributed?

### 💻 Code

```python
clean = pd.read_csv("data/service_requests_clean_v1.csv")
days = clean["days_to_resolve"].dropna()

plt.figure(figsize=(10, 5))
plt.hist(days, bins=20, edgecolor="white")
plt.title("Distribution of Resolution Time — 20 Bins")
plt.xlabel("Days to resolve")
plt.ylabel("Number of requests")
plt.tight_layout()
plt.show()
```

## 🔍 Line-by-Line Explanation

```python
clean = pd.read_csv("data/service_requests_clean_v1.csv")
```

- 📂 Loads the cleaned dataset.
- Stores it in `clean`.

---

```python
days = clean["days_to_resolve"].dropna()
```

- 🔎 Selects the `days_to_resolve` column.
- `.dropna()` removes missing values.
- The resulting data is stored in `days`.

---

```python
plt.figure(figsize=(10, 5))
```

- 🖼️ Creates the chart area.

---

```python
plt.hist(days, bins=20, edgecolor="white")
```

- 📊 Creates a histogram.
- `days` = numeric data being displayed.
- `bins=20` = divides the data into 20 intervals.
- `edgecolor="white"` makes the boundaries between bars easier to distinguish.

---

```python
plt.title("Distribution of Resolution Time — 20 Bins")
```

- 🏷️ Adds a descriptive title.

---

```python
plt.xlabel("Days to resolve")
```

- 🏷️ Labels the x-axis.

---

```python
plt.ylabel("Number of requests")
```

- 🏷️ Labels the y-axis as frequency/count.

---

```python
plt.tight_layout()
```

- 📐 Adjusts spacing.

---

```python
plt.show()
```

- 👀 Displays the histogram.

---

# 🔢 11. Histogram Bins

The number of bins changes how a histogram looks.

### 🟦 Fewer bins

```python
plt.hist(days, bins=20)
```

- Wider intervals.
- Easier to see the overall distribution.
- May hide smaller patterns.

### 🟧 More bins

```python
plt.hist(days, bins=60)
```

- Narrower intervals.
- Shows more detail.
- May make random variation look important.

### 🧠 Practical Rule

Try more than one reasonable bin count before selecting the final chart.

### Example

```python
plt.figure(figsize=(10, 5))
plt.hist(days, bins=20, edgecolor="white")
plt.title("Distribution of Resolution Time — 20 Bins")
plt.xlabel("Days to resolve")
plt.ylabel("Number of requests")
plt.tight_layout()
plt.show()

plt.figure(figsize=(10, 5))
plt.hist(days, bins=60, edgecolor="white")
plt.title("Distribution of Resolution Time — 60 Bins")
plt.xlabel("Days to resolve")
plt.ylabel("Number of requests")
plt.tight_layout()
plt.show()
```

### 📌 Interpretation

A 20-bin version may provide a clearer overall picture, while a 60-bin version may reveal additional detail.

Do not automatically assume that more bins means a better chart.

---

# 📦 12. Box Plot — Resolution Time by Channel

## 🎯 Purpose

A box plot is useful for comparing the distribution of a numeric variable across several categories.

Example question:

> How does resolution time vary across service channels?

### 💻 Code

```python
import pandas as pd
import matplotlib.pyplot as plt

clean = pd.read_csv("data/service_requests_clean_v1.csv")

channels = sorted(clean["channel"].dropna().unique())

data_by_channel = [
    clean[clean["channel"] == channel]["days_to_resolve"].dropna()
    for channel in channels
]

plt.figure(figsize=(10, 5))
plt.boxplot(data_by_channel, tick_labels=channels)
plt.title("Resolution Time by Channel, 2025")
plt.xlabel("Channel")
plt.ylabel("Days to resolve")
plt.xticks(rotation=45, ha="right")
plt.tight_layout()
plt.show()
```

## 🔍 Line-by-Line Explanation

```python
import pandas as pd
```

- 📦 Imports pandas.

---

```python
import matplotlib.pyplot as plt
```

- 📦 Imports Matplotlib.

---

```python
clean = pd.read_csv("data/service_requests_clean_v1.csv")
```

- 📂 Loads the cleaned dataset.

---

```python
channels = sorted(clean["channel"].dropna().unique())
```

- `clean["channel"]` selects the channel column.
- `.dropna()` removes missing channel values.
- `.unique()` finds distinct channel names.
- `sorted()` puts them in alphabetical order.
- The result is stored in `channels`.

### 🧠 Method sequence

```text
column
  ↓
drop missing values
  ↓
find unique values
  ↓
sort values
```

---

```python
data_by_channel = [
```

- 📋 Starts a Python list.
- The list will contain resolution-time data for each channel.

---

```python
    clean[clean["channel"] == channel]["days_to_resolve"].dropna()
```

- `clean["channel"] == channel` checks which rows belong to the current channel.
- `clean[...]` filters the DataFrame.
- `["days_to_resolve"]` selects the resolution-time column.
- `.dropna()` removes missing resolution-time values.

---

```python
    for channel in channels
]
```

- 🔁 Repeats the filtering operation for every channel.
- Produces one group of resolution-time values per channel.

---

```python
plt.figure(figsize=(10, 5))
```

- 🖼️ Creates the figure.

---

```python
plt.boxplot(data_by_channel, tick_labels=channels)
```

- 📦 Creates the box plots.
- `data_by_channel` supplies the numerical data.
- `tick_labels=channels` labels each box with its channel.

---

```python
plt.title("Resolution Time by Channel, 2025")
```

- 🏷️ Adds a descriptive title.

---

```python
plt.xlabel("Channel")
```

- 🏷️ Labels the x-axis.

---

```python
plt.ylabel("Days to resolve")
```

- 🏷️ Labels the y-axis.

---

```python
plt.xticks(rotation=45, ha="right")
```

- 🔄 Rotates the channel names.
- Makes long labels easier to read.

---

```python
plt.tight_layout()
```

- 📐 Adjusts spacing.

---

```python
plt.show()
```

- 👀 Displays the box plot.

---

# 📦 13. Understanding a Box Plot

A simplified box plot looks like:

```text
       ●  ← Possible outlier
       |
   ────┤  ← Upper whisker
       │
    ┌───────┐
    │       │
    │   ─   │ ← Median
    │       │
    └───────┘
       │
   ────┤  ← Lower whisker
       |
```

### 📌 Main Components

| Component | Meaning |
|---|---|
| Box | Middle 50% of observations |
| Bottom of box | Q1 |
| Line inside box | Median |
| Top of box | Q3 |
| Whiskers | Values within the usual 1.5 × IQR rule |
| Points beyond whiskers | Displayed potential outliers |

### ⚠️ Important

A point outside the whiskers is **not automatically a data error**.

Before removing it, investigate:

- 📄 Original record
- 🧑‍💼 Data-entry procedure
- 📐 Definition of the variable
- ✅ Whether the value is plausible
- 📝 Whether there is a documented reason

---

# 🔵 14. Scatter Plot — Staffing and Workload

## 🎯 Purpose

A scatter plot shows the relationship between **two numeric variables**.

Example question:

> Is there an observable relationship between staff count and service-request workload across offices?

The dataset contains:

- `staff_count` from the JSON file
- `request_count` calculated from the service-request data

### 💻 Code

```python
import json
import pandas as pd
import matplotlib.pyplot as plt

with open("data/regional_offices.json") as f:
    offices = pd.json_normalize(json.load(f)["offices"])

request_counts = (
    clean.groupby("office_code")
    .size()
    .rename("request_count")
    .reset_index()
)

office_load = offices[
    ["office_code", "staff_count"]
].merge(
    request_counts,
    on="office_code",
    how="left"
)

print("Office Load:")
print(office_load)

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

---

# 🔍 15. Import JSON

```python
import json
```

- 📦 Imports Python's built-in JSON library.
- JSON files commonly store structured information.
- We use it to read `regional_offices.json`.

---

# 📂 16. Open the JSON File

```python
with open("data/regional_offices.json") as f:
```

- 📂 Opens the JSON file.
- `"data/regional_offices.json"` specifies the file location.
- `as f` stores the opened file in the variable `f`.
- `with` automatically handles closing the file after the block finishes.

---

# 🔄 17. Convert JSON to a DataFrame

```python
    offices = pd.json_normalize(json.load(f)["offices"])
```

This line performs several operations.

### Part 1

```python
json.load(f)
```

- 📖 Reads the JSON file.
- Converts the JSON contents into Python objects, usually dictionaries and lists.

### Part 2

```python
json.load(f)["offices"]
```

- 🔎 Selects the `"offices"` section of the JSON.
- This contains the regional-office records.

### Part 3

```python
pd.json_normalize(...)
```

- 🔄 Converts nested JSON data into a tabular DataFrame.
- Nested fields such as location information can become separate columns.

### Result

The result is stored in:

```python
offices
```

---

# 🧮 18. Calculate Request Count by Office

### 💻 Code

```python
request_counts = (
    clean.groupby("office_code")
    .size()
    .rename("request_count")
    .reset_index()
)
```

### 🔍 Line-by-Line Explanation

```python
clean.groupby("office_code")
```

- 🏢 Groups the service-request records by office.
- Every office gets its own group.

---

```python
.size()
```

- 🔢 Counts the number of rows in each group.
- Since each row represents a service request, this gives request volume by office.

---

```python
.rename("request_count")
```

- 🏷️ Renames the resulting count column to `request_count`.
- This makes the purpose of the column clear.

---

```python
.reset_index()
```

- 🔄 Converts the grouped index back into a normal DataFrame column.
- This makes the result easier to merge with another DataFrame.

---

# 🔗 19. Merge Office Staffing and Request Counts

### 💻 Code

```python
office_load = offices[
    ["office_code", "staff_count"]
].merge(
    request_counts,
    on="office_code",
    how="left"
)
```

### 🔍 Line-by-Line Explanation

```python
offices[
    ["office_code", "staff_count"]
]
```

- Selects only the two columns needed from the `offices` DataFrame.
- `office_code` identifies the office.
- `staff_count` provides the staffing value.

---

```python
.merge(
```

- 🔗 Combines two DataFrames.

---

```python
request_counts,
```

- Specifies the second DataFrame to merge.
- It contains:
  - `office_code`
  - `request_count`

---

```python
on="office_code",
```

- 🔑 Specifies the column used to match the two datasets.
- Offices with the same `office_code` are matched together.

---

```python
how="left"
```

- Uses a left join.
- Keeps every office from the `offices` DataFrame.
- If an office has no matching request count, its `request_count` can become missing.

### 🧠 Think of the merge as:

```text
REGIONAL OFFICES
office_code | staff_count
     +
REQUEST COUNTS
office_code | request_count
     ↓
OFFICE LOAD
office_code | staff_count | request_count
```

---

# 🖨️ 20. Check the Merged Data

```python
print("Office Load:")
print(office_load)
```

### Explanation

```python
print("Office Load:")
```

- 🏷️ Prints a heading.

```python
print(office_load)
```

- 👀 Displays the merged DataFrame.
- This is an important validation step before creating the chart.

---

# 🔵 21. Create the Scatter Plot

### 💻 Code

```python
plt.figure(figsize=(10, 6))

plt.scatter(
    office_load["staff_count"],
    office_load["request_count"]
)
```

### Explanation

```python
plt.figure(figsize=(10, 6))
```

- 🖼️ Creates a figure.
- Width = 10.
- Height = 6.

---

```python
plt.scatter(
```

- 🔵 Creates a scatter plot.

---

```python
    office_load["staff_count"],
```

- Provides the x-axis values.
- X-axis = staff count.

---

```python
    office_load["request_count"]
```

- Provides the y-axis values.
- Y-axis = number of requests.

---

```python
)
```

- Closes the scatter-plot function.

### 🧠 Structure

```text
X → staff_count
Y → request_count
```

Each point represents one office.

---

# 🏷️ 22. Add Office Labels

### 💻 Code

```python
for _, row in office_load.iterrows():
    plt.annotate(
        row["office_code"],
        (row["staff_count"], row["request_count"]),
        xytext=(5, 5),
        textcoords="offset points"
    )
```

### 🔍 Explanation

```python
for _, row in office_load.iterrows():
```

- 🔁 Loops through the DataFrame one row at a time.
- `.iterrows()` provides each row.
- `row` contains the current office's information.
- `_` represents the row index, which we do not need.

---

```python
plt.annotate(
```

- 🏷️ Adds text to a chart.

---

```python
row["office_code"],
```

- Specifies the text that should appear.
- The office code becomes the label.

---

```python
(row["staff_count"], row["request_count"]),
```

- 📍 Specifies where the label belongs.
- The coordinates are:
  - x = staff count
  - y = request count

---

```python
xytext=(5, 5),
```

- ↗️ Moves the label slightly away from the point.
- `5, 5` means a small horizontal and vertical offset.

---

```python
textcoords="offset points"
```

- Tells Matplotlib that `(5, 5)` should be interpreted as an offset from the point.

---

```python
)
```

- Closes the annotation function.

---

# 🏷️ 23. Finish the Scatter Plot

```python
plt.title("Office Staffing and Service Request Workload, 2025")
plt.xlabel("Staff Count")
plt.ylabel("Number of Requests")
plt.tight_layout()
plt.show()
```

### Explanation

```python
plt.title("Office Staffing and Service Request Workload, 2025")
```

- 🏷️ Gives the chart a descriptive title.
- Identifies both variables and the year.

```python
plt.xlabel("Staff Count")
```

- 🏷️ Labels the x-axis.

```python
plt.ylabel("Number of Requests")
```

- 🏷️ Labels the y-axis.

```python
plt.tight_layout()
```

- 📐 Adjusts chart spacing.

```python
plt.show()
```

- 👀 Displays the finished chart.

---

# ⚠️ 24. Interpreting the Staffing vs Workload Scatter Plot

There are only **nine offices** in the dataset.

Therefore, the chart can be used to:

- 👀 Describe visible patterns.
- 🔎 Identify unusual offices.
- 💬 Discuss possible relationships.

But avoid saying that the chart proves:

> "More staff causes more requests."

or:

> "Staffing should be increased because of this relationship."

### 🧠 Why?

A scatter plot shows an **association or relationship**, not automatically a causal relationship.

Other factors may affect request volume, such as:

- Population served
- Geographic coverage
- Service availability
- Office responsibilities
- Local demand
- Reporting practices

---

# 🔗 25. Relationship Does Not Automatically Mean Causation

If two variables move together:

```text
X ↗
Y ↗
```

that does not automatically mean:

```text
X causes Y
```

### Example

```text
Staff Count ↔ Request Volume
```

A relationship may exist, but additional evidence is needed before making a causal claim.

---

# 🎨 26. Chart Design Principles

Good visualization is often about **subtraction**.

### ✅ DO

- 🏷️ Use descriptive titles.
- 📏 Label axes.
- 🔢 Include units.
- 🔃 Sort bars when ranking is meaningful.
- 0️⃣ Start bar-chart value axes at zero.
- 🎨 Use color only when it has a purpose.
- 🧹 Remove unnecessary decoration.
- 👀 Make labels readable.
- ♿ Consider accessibility.

### ❌ AVOID

- 🚫 3-D effects.
- 🌈 Rainbow colors without meaning.
- 🥧 Pie charts with too many slices.
- 📚 Unnecessary legends.
- 🖼️ Decorative elements that add no information.
- 📈 Truncated bar-chart axes that exaggerate differences.
- 🔤 Tiny or unreadable labels.

---

# 👎 27. Poor Chart Example

### 💻 Code

```python
by_service = clean["service_type"].value_counts()

plt.figure(figsize=(9, 4.5))
plt.bar(
    by_service.index,
    by_service.values,
    color=["red", "yellow", "green", "blue", "purple", "orange"],
)
plt.title("Chart 1")
plt.ylim(150, by_service.max())
plt.xticks(rotation=55)
plt.show()
```

### 🚨 Problems

The chart contains several design problems.

```python
plt.title("Chart 1")
```

❌ Problem:
- The title does not explain what the chart measures.
- The reader has to inspect the chart to understand it.

### Better:

```python
plt.title("Service Requests by Service Type, 2025")
```

---

```python
color=["red", "yellow", "green", "blue", "purple", "orange"]
```

❌ Problem:
- Multiple bright colors are used without communicating additional information.
- The colors are decorative rather than analytical.

---

```python
plt.ylim(150, by_service.max())
```

❌ Problem:
- The y-axis starts at 150 instead of zero.
- This can exaggerate differences between categories.

### Better:

```python
plt.xlim(left=0)
```

for a horizontal bar chart, or use an appropriate zero-based y-axis for a vertical bar chart.

---

```python
plt.xticks(rotation=55)
```

⚠️ Problem:
- Long category labels may become difficult to read.
- A horizontal bar chart can be easier for long category names.

---

# 🛠️ 28. Redesign the Bar Chart

## 🎯 Task

Create a chart that:

- ✅ Sorts the categories.
- ✅ Uses a horizontal bar chart.
- ✅ Has a descriptive title.
- ✅ Has labeled axes.
- ✅ Starts the value axis at zero.
- ✅ Uses one main color.
- ✅ Highlights one meaningful category.
- ✅ Removes unnecessary chart decoration.

### 💻 Code

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
plt.grid(False)

plt.tight_layout()
plt.show()
```

### 🔍 Explanation

```python
ordered = by_service.sort_values()
```

- 🔃 Sorts the service counts from smallest to largest.
- This makes the horizontal bars appear in an ordered sequence.
- The smallest value appears at the bottom and the largest at the top.

---

```python
plt.figure(figsize=(10, 6))
```

- 🖼️ Creates a larger chart.
- Width = 10.
- Height = 6.

---

```python
plt.barh(
```

- 📊 Creates a horizontal bar chart.
- `barh` means **bar horizontal**.

---

```python
    ordered.index,
```

- Provides the service names.
- These appear on the y-axis.

---

```python
    ordered.values
```

- Provides the request counts.
- These determine the length of each bar.

---

```python
)
```

- Closes the bar chart function.

---

```python
plt.title("Service Requests by Service Type, 2025")
```

- 🏷️ Provides a descriptive title.
- Tells the reader exactly what is being shown.

---

```python
plt.xlabel("Number of requests")
```

- 🏷️ Labels the numerical axis.
- Clearly identifies the measurement.

---

```python
plt.xlim(left=0)
```

- 0️⃣ Forces the x-axis to begin at zero.
- This is important for a bar chart because bar length represents magnitude.

---

```python
plt.grid(False)
```

- 🧹 Turns off the grid.
- Removes unnecessary visual elements.

---

```python
plt.tight_layout()
```

- 📐 Adjusts spacing.

---

```python
plt.show()
```

- 👀 Displays the redesigned chart.

---

# ⭐ 29. Highlight the Highest Category

### 💻 Code

```python
plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values
)

plt.barh(
    ordered.index[-1],
    ordered.values[-1]
)

plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Number of requests")
plt.xlim(left=0)
plt.grid(False)

plt.tight_layout()
plt.show()
```

### 🔍 Important Lines

```python
ordered.index[-1]
```

- 🔝 Gets the last category in the sorted list.
- Because the data was sorted from smallest to largest, this is the highest category.

```python
ordered.values[-1]
```

- 🔝 Gets the highest request count.

```python
plt.barh(
    ordered.index[-1],
    ordered.values[-1]
)
```

- Draws the highest bar again so it can be visually distinguished.

### ⚠️ Design Reminder

Highlighting should have a purpose.

If the difference between the highest and second-highest categories is very small, highlighting the highest category may visually overstate its importance.

---

# 🏷️ 30. Add Direct Data Labels

Direct labels can make exact values easier to read.

### 💻 Code

```python
plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values
)

for i, value in enumerate(ordered.values):
    plt.text(
        value,
        i,
        f"{value}",
        va="center",
        ha="left",
        fontsize=10
    )

plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Number of requests")
plt.xlim(left=0)
plt.grid(False)

plt.tight_layout()
plt.show()
```

### 🔍 Line-by-Line Explanation

```python
for i, value in enumerate(ordered.values):
```

- 🔁 Loops through each request count.
- `enumerate()` provides both:
  - `i` = position/index
  - `value` = actual request count

Example:

```text
i = 0 → value = 150
i = 1 → value = 175
i = 2 → value = 200
```

---

```python
plt.text(
```

- 🏷️ Adds text to the chart.

---

```python
value,
```

- 📍 Sets the x-coordinate.
- Places the text at the end of the bar.

---

```python
i,
```

- 📍 Sets the y-coordinate.
- Matches the text with the correct bar.

---

```python
f"{value}",
```

- 🔤 Converts the value into display text.
- `f""` is an f-string.
- If `value = 250`, it displays `250`.

---

```python
va="center",
```

- ↕️ Vertically centers the label relative to the bar.

---

```python
ha="left",
```

- ↔️ Aligns the text from the left side.

---

```python
fontsize=10
```

- 🔤 Sets the text size.

---

```python
)
```

- Closes `plt.text()`.

---

# 🧹 31. Remove Chart Junk

Chart junk refers to visual elements that do not help communicate the data.

### Examples

❌ Avoid:

```text
3-D effects
Heavy borders
Unnecessary shadows
Rainbow colors
Decorative backgrounds
Excessive gridlines
Unnecessary legends
```

### Prefer:

```text
Clear title
Readable labels
Simple chart
Meaningful color
Appropriate scale
Direct labels when useful
```

---

# ♿ 32. Accessibility

Never make **color the only signal** for meaning.

For example, do not communicate:

```text
Red   = Pending
Green = Resolved
```

using color alone.

Consider adding:

- 🏷️ Direct labels
- 🔤 Text labels
- 📊 Different chart positions
- 🔲 Different patterns or markers where appropriate

The chart should remain understandable even if the reader has difficulty distinguishing colors.

---

# 🧪 33. The Five-Second Test

Ask:

> 👀 "Can a colleague state the chart's main point in five seconds without me explaining it?"

If **no**, consider improving:

- The title
- The chart type
- The ordering
- The labels
- The scale
- The amount of decoration

---

# 🏷️ 34. Descriptive Titles

### ❌ Weak

```python
plt.title("Chart 1")
```

The reader does not know what is being measured.

### ❌ Also weak

```python
plt.title("service_type")
```

This only repeats a variable name.

### ✅ Better

```python
plt.title("Service Requests by Service Type, 2025")
```

This tells the reader:

```text
WHAT     → Service Requests
GROUP BY → Service Type
WHEN     → 2025
```

### ⚠️ Avoid Unsupported Conclusions

Do not write:

```python
plt.title("Poor Service Performance by Region")
```

unless the data and measurement actually support that conclusion.

A safer descriptive title would be:

```python
plt.title("Average Resolution Time by Region, 2025")
```

The title should describe what the data directly shows.

---

# 📏 35. Why Bar Charts Should Start at Zero

### ❌ Potentially misleading

```python
plt.ylim(150, 300)
```

- The chart starts at 150.
- Differences between bars may appear much larger than they actually are.

### ✅ Better

```python
plt.ylim(bottom=0)
```

or for a horizontal chart:

```python
plt.xlim(left=0)
```

### 🧠 Rule

For ordinary bar charts:

```text
Value axis → Start at zero
```

because the **length of the bar represents the magnitude**.

---

# 🔃 36. Sorting Bar Charts

### Largest → Smallest

```python
ordered = by_service.sort_values(ascending=False)
```

### Smallest → Largest

```python
ordered = by_service.sort_values()
```

### Why sort?

Sorting makes rankings easier to see.

### When not to sort?

Alphabetical or natural order may be better when:

- 🔤 Users need to find a specific category quickly.
- 📅 Categories have a natural sequence.
- 🗺️ Geographic order provides meaningful structure.
- 📈 Time order is important.

---

# 🧰 37. Essential Pandas Commands for Visualization

## Select a column

```python
clean["service_type"]
```

➡️ Selects one column.

---

## Count categories

```python
clean["service_type"].value_counts()
```

➡️ Counts each category.

---

## Sort values

```python
by_service.sort_values()
```

➡️ Sorts smallest → largest.

```python
by_service.sort_values(ascending=False)
```

➡️ Sorts largest → smallest.

---

## Remove missing values

```python
clean["days_to_resolve"].dropna()
```

➡️ Removes missing values.

---

## Find unique categories

```python
clean["channel"].unique()
```

➡️ Returns distinct values.

---

## Sort unique categories

```python
sorted(clean["channel"].unique())
```

➡️ Returns unique values in sorted order.

---

## Convert to datetime

```python
pd.to_datetime(clean["date_filed"])
```

➡️ Converts values to pandas datetime format.

---

## Set an index

```python
clean.set_index("date_filed")
```

➡️ Makes `date_filed` the DataFrame index.

---

## Resample by month

```python
clean_by_date.resample("ME")
```

➡️ Groups datetime data by month-end periods.

---

## Count rows

```python
clean_by_date.resample("ME").size()
```

➡️ Counts records in each month.

---

## Group by category

```python
clean.groupby("office_code")
```

➡️ Groups records by office.

---

## Count records in each group

```python
clean.groupby("office_code").size()
```

➡️ Counts records per office.

---

## Rename a result

```python
.rename("request_count")
```

➡️ Gives the resulting series a meaningful name.

---

## Reset the index

```python
.reset_index()
```

➡️ Converts an index back into a normal column.

---

## Merge DataFrames

```python
df1.merge(df2, on="office_code", how="left")
```

➡️ Combines two DataFrames using `office_code`.

---

# 📊 38. Essential Matplotlib Commands

## Create a figure

```python
plt.figure(figsize=(10, 5))
```

➡️ Creates the chart area.

---

## Bar chart

```python
plt.bar(x, y)
```

➡️ Creates a vertical bar chart.

---

## Horizontal bar chart

```python
plt.barh(y, x)
```

➡️ Creates a horizontal bar chart.

---

## Line chart

```python
plt.plot(x, y)
```

➡️ Creates a line chart.

---

## Histogram

```python
plt.hist(data, bins=20)
```

➡️ Creates a histogram.

---

## Box plot

```python
plt.boxplot(data)
```

➡️ Creates a box plot.

---

## Scatter plot

```python
plt.scatter(x, y)
```

➡️ Creates a scatter plot.

---

## Chart title

```python
plt.title("My Chart")
```

➡️ Adds a title.

---

## X-axis label

```python
plt.xlabel("X Variable")
```

➡️ Labels the x-axis.

---

## Y-axis label

```python
plt.ylabel("Y Variable")
```

➡️ Labels the y-axis.

---

## Rotate labels

```python
plt.xticks(rotation=45)
```

➡️ Rotates x-axis labels.

---

## Set horizontal axis minimum

```python
plt.xlim(left=0)
```

➡️ Makes the x-axis start at zero.

---

## Remove gridlines

```python
plt.grid(False)
```

➡️ Removes gridlines.

---

## Adjust spacing

```python
plt.tight_layout()
```

➡️ Prevents labels from being cut off.

---

## Display chart

```python
plt.show()
```

➡️ Displays the chart.

---

# 🔎 39. Important Visualization Concepts

## 📊 Bar Chart

Use when:

```text
Category → Number
```

Example:

```text
Service Type → Request Count
```

---

## 📈 Line Chart

Use when:

```text
Time → Measurement
```

Example:

```text
Month → Request Count
```

---

## 📉 Histogram

Use when:

```text
Numeric variable → Distribution
```

Example:

```text
Days to resolve → Frequency
```

---

## 📦 Box Plot

Use when:

```text
Category → Numeric distribution
```

Example:

```text
Channel → Resolution time
```

---

## 🔵 Scatter Plot

Use when:

```text
Numeric X → Numeric Y
```

Example:

```text
Staff count → Request count
```

---

# ⚠️ 40. Outliers Are Not Automatically Errors

If a box plot displays an unusual point:

```text
Unusual value
     ↓
Investigate
     ↓
Check original record
     ↓
Check definition
     ↓
Check plausibility
     ↓
Determine whether it is an error
```

Do **not** automatically delete it.

An unusual observation may represent:

- A legitimate unusual case.
- A rare event.
- A real operational condition.
- A data-entry error.

---

# 🧠 41. Correlation vs Causation

A scatter plot can help identify a relationship.

It does **not automatically prove causation**.

### Example

```text
Staff Count
     ↕
Request Count
```

A visible relationship could be influenced by other variables.

### Better wording

✅ "The scatter plot shows an observable relationship between staff count and request volume."

### Avoid

❌ "Increasing staff causes request volume to increase."

unless supported by an appropriate causal design and evidence.

---

# 👥 42. Match the Interpretation to the Unit of Analysis

This is extremely important.

### Request-Level Analysis

Example:

```text
Resolution Time vs Satisfaction
```

Each point represents a **service request**.

### Office-Level Analysis

Example:

```text
Staff Count vs Request Count
```

Each point represents an **office**.

Therefore, the conclusions apply to different units.

### 🧠 Remember

```text
Request-level chart
       ↓
Interpret at request level

Office-level chart
       ↓
Interpret at office level
```

Do not automatically generalize one level of analysis to another.

---

# 📋 43. Complete Visualization Workflow

```python
# ============================================================
# 1. IMPORT LIBRARIES
# ============================================================

import pandas as pd
import matplotlib.pyplot as plt


# ============================================================
# 2. LOAD CLEAN DATA
# ============================================================

clean = pd.read_csv(
    "data/service_requests_clean_v1.csv",
    parse_dates=["date_filed"]
)


# ============================================================
# 3. ASK THE QUESTION
# ============================================================

# Example:
# Which service types received the most requests?


# ============================================================
# 4. PREPARE THE DATA
# ============================================================

by_service = clean["service_type"].value_counts()
ordered = by_service.sort_values()


# ============================================================
# 5. BUILD THE CHART
# ============================================================

plt.figure(figsize=(10, 6))

plt.barh(
    ordered.index,
    ordered.values
)


# ============================================================
# 6. ADD DESCRIPTIVE LABELS
# ============================================================

plt.title("Service Requests by Service Type, 2025")
plt.xlabel("Number of requests")


# ============================================================
# 7. CHECK THE SCALE
# ============================================================

plt.xlim(left=0)


# ============================================================
# 8. REMOVE UNNECESSARY DECORATION
# ============================================================

plt.grid(False)


# ============================================================
# 9. ADJUST THE LAYOUT
# ============================================================

plt.tight_layout()


# ============================================================
# 10. DISPLAY THE CHART
# ============================================================

plt.show()
```

---

# 🧠 44. How to Read the Complete Workflow

```text
IMPORT
  ↓
Load the tools

LOAD
  ↓
Read the clean dataset

QUESTION
  ↓
What do I want the reader to understand?

PREPARE
  ↓
Group, count, sort, filter, or transform the data

BUILD
  ↓
Choose the appropriate chart

LABEL
  ↓
Add title and axis labels

CHECK
  ↓
Check scale, readability, outliers, and possible misleading design

SIMPLIFY
  ↓
Remove unnecessary decoration

DISPLAY
  ↓
Show the final chart

INTERPRET
  ↓
State only what the data supports
```

---

# 📝 45. Chart Interpretation Template

When discussing a chart, use:

```text
1. WHAT DOES THE CHART SHOW?
2. WHAT PATTERN IS VISIBLE?
3. WHAT IS UNUSUAL?
4. WHAT DOES THE DATA SUPPORT?
5. WHAT SHOULD WE NOT CLAIM?
```

### Example

> The chart shows the number of service requests by service type in 2025. The categories have different request volumes, with some services receiving more requests than others. The chart describes differences in request volume but does not by itself explain why those differences occurred.

---

# 🚨 46. Common Mistakes to Remember

| ❌ Mistake | ✅ Better Approach |
|---|---|
| Using a line chart for unrelated categories | Use a bar chart |
| Using too many histogram bins | Compare reasonable bin counts |
| Assuming outliers are errors | Investigate first |
| Using many decorative colors | Use color with purpose |
| Starting bar axis at 150 | Start the value axis at zero |
| Using `"Chart 1"` as title | Use a descriptive title |
| Making unsupported causal claims | Describe the observed relationship |
| Ignoring missing values | Check and handle them |
| Using unreadable labels | Rotate or redesign |
| Adding unnecessary chart junk | Simplify |
| Making color the only signal | Add labels or other visual cues |
| Generalizing nine offices to all offices | State the actual population |

---

# ⚡ 47. Quick Code Reference

```python
# 📥 Load CSV
df = pd.read_csv("file.csv")

# 📅 Load dates
df = pd.read_csv("file.csv", parse_dates=["date"])

# 🔢 Count categories
df["column"].value_counts()

# 🔃 Sort ascending
df["column"].value_counts().sort_values()

# 🔃 Sort descending
df["column"].value_counts().sort_values(ascending=False)

# 🧹 Remove missing values
df["column"].dropna()

# 🔎 Find unique values
df["column"].unique()

# 📅 Convert to datetime
pd.to_datetime(df["date"])

# 📅 Set datetime index
df.set_index("date")

# 📅 Monthly count
df.set_index("date").resample("ME").size()

# 🔝 Highest value
series.max()

# 📍 Index of highest value
series.idxmax()

# 🏢 Group by category
df.groupby("office_code")

# 🔢 Count rows per group
df.groupby("office_code").size()

# 🏷️ Rename result
series.rename("new_name")

# 🔄 Reset index
df.reset_index()

# 🔗 Merge DataFrames
df1.merge(df2, on="office_code", how="left")

# 📊 Bar chart
plt.bar(x, y)

# 📊 Horizontal bar chart
plt.barh(y, x)

# 📈 Line chart
plt.plot(x, y)

# 📉 Histogram
plt.hist(data, bins=20)

# 📦 Box plot
plt.boxplot(data)

# 🔵 Scatter plot
plt.scatter(x, y)

# 🏷️ Title
plt.title("Title")

# 🏷️ X-axis
plt.xlabel("X")

# 🏷️ Y-axis
plt.ylabel("Y")

# 🔄 Rotate labels
plt.xticks(rotation=45)

# 0️⃣ Start x-axis at zero
plt.xlim(left=0)

# 🧹 Remove grid
plt.grid(False)

# 📐 Fix spacing
plt.tight_layout()

# 👀 Display chart
plt.show()
```

---

# 🧩 48. Key Python Patterns to Memorize

### 🔢 Count Categories

```python
df["column"].value_counts()
```

### 🔃 Sort a Series

```python
series.sort_values()
```

### 🔝 Find Highest Value

```python
series.max()
```

### 📍 Find Where Highest Value Occurs

```python
series.idxmax()
```

### 📅 Group Dates by Month

```python
df.set_index("date").resample("ME").size()
```

### 🏢 Group and Count

```python
df.groupby("category").size()
```

### 🔗 Merge Two DataFrames

```python
df1.merge(df2, on="key", how="left")
```

### 📊 Create a Bar Chart

```python
plt.bar(x, y)
```

### 📊 Create a Horizontal Bar Chart

```python
plt.barh(y, x)
```

### 📈 Create a Line Chart

```python
plt.plot(x, y)
```

### 📉 Create a Histogram

```python
plt.hist(data, bins=20)
```

### 📦 Create a Box Plot

```python
plt.boxplot(data)
```

### 🔵 Create a Scatter Plot

```python
plt.scatter(x, y)
```

---

# 🏆 49. Day 8 Final Checklist

Before submitting or presenting a chart, check:

```text
☐ Did I start with a clear question?
☐ Did I choose a chart appropriate for that question?
☐ Is the title descriptive?
☐ Are the axes labeled?
☐ Are the units clear?
☐ Are category labels readable?
☐ Are bars sorted when appropriate?
☐ Does a bar chart start at zero?
☐ Did I avoid unnecessary colors?
☐ Did I remove unnecessary decoration?
☐ Is color being used for a meaningful purpose?
☐ Did I check missing values?
☐ Did I investigate unusual observations?
☐ Did I avoid claiming causation from simple relationships?
☐ Did I interpret the chart according to its unit of analysis?
☐ Can someone understand the main point within five seconds?
```

---

# 🧠 50. Day 8 Key Takeaways

### 📊 Bar Chart

> **Compare categories.**

```python
plt.bar(x, y)
```

### 📊 Horizontal Bar

> **Compare categories with long labels or rankings.**

```python
plt.barh(y, x)
```

### 📈 Line Chart

> **Show change over time.**

```python
plt.plot(x, y)
```

### 📉 Histogram

> **Show the distribution of one numeric variable.**

```python
plt.hist(data, bins=20)
```

### 📦 Box Plot

> **Compare numeric distributions across groups.**

```python
plt.boxplot(data)
```

### 🔵 Scatter Plot

> **Examine the relationship between two numeric variables.**

```python
plt.scatter(x, y)
```

### 🎨 Good Visualization

```text
Clear question
     ↓
Appropriate chart
     ↓
Clean design
     ↓
Readable labels
     ↓
Honest interpretation
```

### ⭐ Most Important Principle

> **A good chart does not merely look good. It helps the reader understand the data accurately and quickly.**
