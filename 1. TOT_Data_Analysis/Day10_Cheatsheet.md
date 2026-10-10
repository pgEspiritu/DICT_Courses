# Flood Control Project Dataset — Lab 1 & Lab 2 Cheatsheet

**Workshop:** Data Processing and Exploratory Data Analysis  
**Source:** BetterGovPH Flood Control Project Dataset  
**Notebook:** `Day10_Workshop_STUDENT_NAME.ipynb`

---

## Table of Contents

1. [Lab 1 — Read and Process Data](#lab-1--read-and-process-data)
2. [Lab 2 — Exploratory Data Analysis](#lab-2--exploratory-data-analysis)
3. [Pandas Code Patterns](#pandas-code-patterns)
4. [Important Notes](#important-notes)

---

# Lab 1 — Read and Process Data

## Step 1: Import Libraries

```python
# STEP 1: IMPORT LIBRARIES
# ------------------------------------------------------------
import json
import datetime as dt
import requests
import pandas as pd

print("STEP 1: Libraries imported successfully.")
```

- `json`: reads and writes JSON data.
- `datetime`: works with dates and times.
- `requests`: downloads data from a URL.
- `pandas`: stores and processes tabular data.

## Step 2: Configure Pandas Display Settings

```python
# STEP 2: CONFIGURE PANDAS DISPLAY SETTINGS
# ------------------------------------------------------------
pd.set_option("display.max_columns", None)
pd.set_option("display.max_rows", 200)
pd.options.display.float_format = '{:,.2f}'.format

print("STEP 2: Pandas display settings configured successfully.")
```

**Remember:** `'{:,.2f}'.format` displays floating-point values with commas and two decimal places. A year stored as a float can therefore appear as `2,018.00`.

## Step 3: Define the Epoch Timestamp Conversion Function

```python
# STEP 3: DEFINE EPOCH TIMESTAMP CONVERSION FUNCTION
# ------------------------------------------------------------
def epoch_to_timestamp(ts):
    try:
        return dt.datetime.fromtimestamp(ts / 1000)
    except:
        return None

print("STEP 3: Epoch timestamp conversion function defined successfully.")
```

This function converts Unix epoch timestamps in milliseconds to Python datetime values. Dividing by `1000` converts milliseconds to seconds.

## Step 4: Download the Flood Control Dataset

```python
# STEP 4: DOWNLOAD FLOOD CONTROL DATASET
# ------------------------------------------------------------
flood_control_dataset = "https://raw.githubusercontent.com/bettergovph/bettergov/refs/heads/main/src/data/flood_control/flood_control.json"
response = requests.get(flood_control_dataset)

if response.ok:
    print("STEP 4: Flood control dataset downloaded successfully.")
else:
    print("STEP 4: Failed to download the flood control dataset.")
    print("HTTP status code:", response.status_code)
```

The code downloads the JSON source. Continue with the next steps only if the request succeeds.

## Step 5: Save the Raw JSON File

```python
# STEP 5: SAVE RAW JSON FILE
# ------------------------------------------------------------
data = response.json()

with open("flood_control.json", "w") as fp:
    json.dump(data, fp)

print("STEP 5: Raw JSON file saved successfully.")
```

**Output:** `flood_control.json`

## Step 6: Extract Data and Save the Raw CSV

```python
# STEP 6: EXTRACT DATA AND SAVE RAW CSV
# ------------------------------------------------------------
df_raw = pd.DataFrame([f["attributes"] for f in data["features"]])
df_raw.to_csv("flood_control_raw.csv", index=False)

print("STEP 6: Raw dataset extracted and saved successfully.")
print("Raw dataset shape:", df_raw.shape)
display(df_raw.head())
```

- `data["features"]`: records in the JSON source.
- `f["attributes"]`: extracts the attributes for each feature.
- `pd.DataFrame(...)`: converts records into a table.
- `.shape`: returns `(rows, columns)`.
- `.head()`: displays the first five rows by default.
- `index=False`: prevents the DataFrame index from being written as an extra CSV column.

**Output:** `flood_control_raw.csv`

## Step 7: Create a Copy for Processing

```python
# STEP 7: CREATE A COPY FOR DATA PROCESSING
# ------------------------------------------------------------
df_cleaned = df_raw.copy()

print("STEP 7: Working copy of the raw dataset created successfully.")
```

The raw dataset remains in `df_raw`; cleaning operations are applied to `df_cleaned`.

## Step 8: Convert Epoch Timestamps

```python
# STEP 8: CONVERT EPOCH TIMESTAMPS
# ------------------------------------------------------------
df_cleaned["CompletionDateOriginal"] = df_cleaned["CompletionDateOriginal"].map(epoch_to_timestamp)
df_cleaned["CreationDate"] = df_cleaned["CreationDate"].map(epoch_to_timestamp)
df_cleaned["EditDate"] = df_cleaned["EditDate"].map(epoch_to_timestamp)

print("STEP 8: Epoch timestamps converted successfully.")
```

`.map(function)` applies a function to each value in a Series.

## Step 9: Convert `StartDate` to Datetime

```python
# STEP 9: CONVERT START DATE TO DATETIME
# ------------------------------------------------------------
df_cleaned["StartDate"] = pd.to_datetime(
    df_cleaned["StartDate"],
    format="%m/%d/%Y",
    errors="coerce"
)

print("STEP 9: StartDate converted to datetime successfully.")
```

- `format="%m/%d/%Y"`: month/day/four-digit year.
- `errors="coerce"`: invalid date values become `NaT` (missing datetime).

## Step 10: Remove Unwanted Columns

```python
# STEP 10: REMOVE UNWANTED COLUMNS
# ------------------------------------------------------------
df_cleaned.drop(
    labels=["ABC_String", "ContractCost_String"],
    axis=1,
    inplace=True
)

print("STEP 10: Unwanted columns removed successfully.")
```

- `axis=1`: select columns.
- `inplace=True`: modify `df_cleaned` directly.
- This follows the lecture demo by removing the redundant string-formatted columns while retaining `ContractCost`.

## Step 11: Review the Cleaned Dataset

```python
# STEP 11: REVIEW CLEANED DATASET
# ------------------------------------------------------------
print("\nSTEP 11: CLEANED DATASET REVIEW")
print("=" * 60)

print("Cleaned dataset shape:", df_cleaned.shape)
print("\nData types:")
print(df_cleaned.dtypes)
print("\nMissing values per column:")
print(df_cleaned.isnull().sum())
print("\nFirst five records:")
display(df_cleaned.head())

print("STEP 11: Cleaned dataset review completed successfully.")
```

## Step 12: Save the Cleaned Dataset

```python
# STEP 12: SAVE CLEANED DATASET
# ------------------------------------------------------------
df_cleaned.to_csv("flood_control_cleaned.csv", index=False)

print("STEP 12: Cleaned dataset saved successfully.")
print("Output file: flood_control_cleaned.csv")
print("Final dataset shape:", df_cleaned.shape)
```

**Output:** `flood_control_cleaned.csv`

---

# Lab 2 — Exploratory Data Analysis

## Step 1: Read the Cleaned Dataset

```python
# STEP 1: READ THE CLEANED DATASET
# ------------------------------------------------------------
df_cleaned = pd.read_csv("flood_control_cleaned.csv")

print("STEP 1: Cleaned dataset loaded successfully.")
print("Dataset shape:", df_cleaned.shape)
```

## Step 2: Review the Dataset

```python
# STEP 2: REVIEW THE DATASET
# ------------------------------------------------------------
print("\nSTEP 2: DATASET OVERVIEW")
print("=" * 60)

print("\nColumn names:")
print(df_cleaned.columns.tolist())
print("\nFirst five records:")
display(df_cleaned.head())
print("\nDataset information:")
df_cleaned.info()

print("STEP 2: Dataset overview completed successfully.")
```

## Step 3: Top Awarded Contractors by Total Contract Cost

```python
# STEP 3: TOP AWARDED CONTRACTORS
# ------------------------------------------------------------
# Group projects by contractor.
# Calculate total contract cost and number of project records.
# Sort from highest to lowest total contract cost.

df_contractors = (
    df_cleaned
    .groupby("Contractor", as_index=False)
    .agg({
        "ContractCost": "sum",
        "ProjectID": "count"
    })
    .sort_values(by="ContractCost", ascending=False)
)

print("\nSTEP 3: TOP AWARDED CONTRACTORS")
print("=" * 60)
display(df_contractors.head(10))
print("STEP 3: Contractor ranking completed successfully.")
```

This ranks contractors by the sum of their recorded contract costs. It may not represent unique awards if the source contains repeated project records.

## Step 4: Top Provinces by Number of Projects

```python
# STEP 4: TOP PROVINCES WITH FLOOD CONTROL PROJECTS
# ------------------------------------------------------------
df_provinces = (
    df_cleaned
    .groupby("Province", as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .sort_values(by="ProjectID", ascending=False)
)

print("\nSTEP 4: TOP PROVINCES BY NUMBER OF PROJECTS")
print("=" * 60)
display(df_provinces.head(10))
print("STEP 4: Provincial analysis completed successfully.")
```

## Step 5: Top Municipalities by Number of Projects

```python
# STEP 5: TOP MUNICIPALITIES WITH FLOOD CONTROL PROJECTS
# ------------------------------------------------------------
df_municipalities = (
    df_cleaned
    .groupby(["Province", "Municipality"], as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .sort_values(by="ProjectID", ascending=False)
)

print("\nSTEP 5: TOP MUNICIPALITIES BY NUMBER OF PROJECTS")
print("=" * 60)
display(df_municipalities.head(10))
print("STEP 5: Municipal analysis completed successfully.")
```

Grouping by both province and municipality avoids combining similarly named municipalities in different provinces.

## Step 6: Top Municipalities by Total Contract Cost

```python
# STEP 6: TOP MUNICIPALITIES BY TOTAL CONTRACT COST
# ------------------------------------------------------------
df_municipality_cost = (
    df_cleaned
    .groupby(["Province", "Municipality"], as_index=False)
    .agg({
        "ContractCost": "sum",
        "ProjectID": "count"
    })
    .sort_values(by="ContractCost", ascending=False)
)

print("\nSTEP 6: TOP MUNICIPALITIES BY TOTAL CONTRACT COST")
print("=" * 60)
display(df_municipality_cost.head(10))
print("STEP 6: Municipal contract cost analysis completed successfully.")
```

## Step 7: Contractors with the Most Project Records

```python
# STEP 7: CONTRACTORS WITH THE MOST PROJECTS
# ------------------------------------------------------------
df_contractor_count = (
    df_cleaned
    .groupby("Contractor", as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .sort_values(by="ProjectID", ascending=False)
)

print("\nSTEP 7: CONTRACTORS WITH THE MOST PROJECTS")
print("=" * 60)
display(df_contractor_count.head(10))
print("STEP 7: Contractor project count analysis completed successfully.")
```

**Difference from Step 3:** Step 3 ranks by total contract cost; Step 7 ranks by the count of non-missing `ProjectID` values.

## Step 8: Contractors Operating in Multiple Provinces

```python
# STEP 8: CONTRACTORS WORKING IN MULTIPLE PROVINCES
# ------------------------------------------------------------
df_contractor_provinces = (
    df_cleaned
    .groupby("Contractor", as_index=False)
    .agg({
        "Province": "nunique",
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .rename(columns={
        "Province": "NumberOfProvinces",
        "ProjectID": "NumberOfProjects"
    })
    .sort_values(by="NumberOfProvinces", ascending=False)
)

print("\nSTEP 8: CONTRACTORS OPERATING IN MULTIPLE PROVINCES")
print("=" * 60)
display(
    df_contractor_provinces[
        df_contractor_provinces["NumberOfProvinces"] > 1
    ].head(10)
)
print("STEP 8: Multi-province contractor analysis completed successfully.")
```

`nunique()` counts distinct non-missing values. Operating in multiple provinces is an exploratory observation, not proof of a relationship between contractors.

## Step 9: Contractors by Project Location

```python
# STEP 9: CONTRACTORS SHARING THE SAME PROJECT LOCATION
# ------------------------------------------------------------
df_location_contractors = (
    df_cleaned
    .groupby(["Province", "Municipality", "Contractor"], as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .sort_values(by="ProjectID", ascending=False)
)

print("\nSTEP 9: CONTRACTORS BY PROJECT LOCATION")
print("=" * 60)
display(df_location_contractors.head(10))
print("STEP 9: Location-based contractor analysis completed successfully.")
```

This shows each contractor's recorded project count and combined cost in each province–municipality combination.

## Step 10: Projects by Start Year

```python
# STEP 10: PROJECTS BY YEAR
# ------------------------------------------------------------
# Convert StartDate to datetime and extract the year.
# Invalid dates are treated as missing.

df_cleaned["StartDate"] = pd.to_datetime(
    df_cleaned["StartDate"],
    errors="coerce"
)

# Extract the year from StartDate.
df_cleaned["StartYear"] = df_cleaned["StartDate"].dt.year

# Convert to nullable integer so years display as 2018, not 2,018.00.
df_cleaned["StartYear"] = df_cleaned["StartYear"].astype("Int64")

print("\nSTEP 10: PROJECTS BY START YEAR")
print("=" * 60)

df_projects_year = (
    df_cleaned
    .groupby("StartYear", as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .sort_values(by="StartYear")
)

display(df_projects_year)
print("STEP 10: Yearly project analysis completed successfully.")
```

- `.dt.year`: extracts the year from a datetime column.
- `.astype("Int64")`: uses a nullable integer type and preserves missing years as `<NA>`.
- The global float display format does not need to be changed.

## Step 11: Contract Cost Summary Statistics

```python
# STEP 11: DISTRIBUTION OF CONTRACT COSTS
# ------------------------------------------------------------
print("\nSTEP 11: CONTRACT COST SUMMARY")
print("=" * 60)

display(df_cleaned["ContractCost"].describe())
print("STEP 11: Contract cost summary completed successfully.")
```

`describe()` reports count, mean, standard deviation, minimum, quartiles, and maximum for numeric data.

## Step 12: Highest-Cost Individual Project Records

```python
# STEP 12: HIGHEST-COST INDIVIDUAL PROJECTS
# ------------------------------------------------------------
df_highest_cost = df_cleaned.sort_values(
    by="ContractCost",
    ascending=False
)

print("\nSTEP 12: TOP 10 HIGHEST-COST PROJECTS")
print("=" * 60)

display(
    df_highest_cost[[
        "ProjectID",
        "Province",
        "Municipality",
        "Contractor",
        "ContractCost"
    ]].head(10)
)

print("STEP 12: Highest-cost project analysis completed successfully.")
```

## Step 13: Contractors with Multiple Project Records

```python
# STEP 13: CONTRACTORS WITH MULTIPLE PROJECTS
# ------------------------------------------------------------
df_multiple_projects = (
    df_cleaned
    .groupby("Contractor", as_index=False)
    .agg({
        "ProjectID": "count",
        "ContractCost": "sum"
    })
)

df_multiple_projects = df_multiple_projects[
    df_multiple_projects["ProjectID"] > 1
].sort_values(
    by="ProjectID",
    ascending=False
)

print("\nSTEP 13: CONTRACTORS WITH MULTIPLE PROJECTS")
print("=" * 60)
display(df_multiple_projects.head(20))
print("STEP 13: Multiple-project contractor analysis completed successfully.")
```

## Step 14: Municipalities with the Most Distinct Contractors

```python
# STEP 14: COMPARE CONTRACTORS WITHIN THE SAME MUNICIPALITY
# ------------------------------------------------------------
# Count distinct contractors in each municipality.
# Rank municipalities from the most contractors to the fewest.

df_contractor_comparison = (
    df_cleaned
    .groupby(["Province", "Municipality"], as_index=False)
    .agg({
        "Contractor": "nunique",
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .rename(columns={
        "Contractor": "NumberOfContractors",
        "ProjectID": "NumberOfProjects"
    })
    .sort_values(
        by="NumberOfContractors",
        ascending=False
    )
)

print("\nSTEP 14: MUNICIPALITIES WITH THE MOST DISTINCT CONTRACTORS")
print("=" * 60)
display(df_contractor_comparison.head(10))
print("STEP 14: Contractor comparison completed successfully.")
```

## Step 15: Municipalities with the Least Distinct Contractors

```python
# STEP 15: MUNICIPALITIES WITH THE LEAST DISTINCT CONTRACTORS
# ------------------------------------------------------------
# Use the same grouping as Step 14, but sort in ascending order.

df_least_contractor_comparison = (
    df_cleaned
    .groupby(["Province", "Municipality"], as_index=False)
    .agg({
        "Contractor": "nunique",
        "ProjectID": "count",
        "ContractCost": "sum"
    })
    .rename(columns={
        "Contractor": "NumberOfContractors",
        "ProjectID": "NumberOfProjects"
    })
    .sort_values(
        by="NumberOfContractors",
        ascending=True
    )
)

print("\nSTEP 15: MUNICIPALITIES WITH THE LEAST DISTINCT CONTRACTORS")
print("=" * 60)
display(df_least_contractor_comparison.head(10))
print("STEP 15: Least distinct contractor analysis completed successfully.")
```

**Note:** Municipalities with missing contractor names may have a distinct contractor count of zero. A low count alone does not prove restricted competition or wrongdoing.

## Step 16: Final EDA Summary

```python
# STEP 16: FINAL SUMMARY
# ------------------------------------------------------------
print("\nSTEP 16: EXPLORATORY DATA ANALYSIS COMPLETED")
print("=" * 60)

print("Total project records:", len(df_cleaned))
print("Distinct contractors:", df_cleaned["Contractor"].nunique())
print("Distinct provinces:", df_cleaned["Province"].nunique())
print("Distinct municipalities:", df_cleaned["Municipality"].nunique())
print(
    "Total contract cost:",
    f"{df_cleaned['ContractCost'].sum():,.2f}"
)

print("\nAll exploratory analysis steps completed successfully.")
```

---

# Pandas Code Patterns

## Read, Copy, and Save CSV Files

```python
# Read a CSV file
df_raw = pd.read_csv("flood_control_raw.csv")

# Create a working copy
df_cleaned = df_raw.copy()

# Save without exporting the DataFrame index
df_cleaned.to_csv("flood_control_cleaned.csv", index=False)
```

## Inspect a DataFrame

```python
df_cleaned.head()                 # First five rows
df_cleaned.shape                  # Number of rows and columns
df_cleaned.columns.tolist()       # List of column names
df_cleaned.dtypes                 # Data types
df_cleaned.info()                 # Column types and non-null counts
df_cleaned.isnull().sum()         # Missing values per column
df_cleaned.duplicated().sum()     # Exact duplicate row count
```

## Group and Aggregate

```python
df_cleaned.groupby("Contractor", as_index=False).agg({
    "ContractCost": "sum",
    "ProjectID": "count"
})
```

Common aggregation methods:

| Method | Meaning |
|---|---|
| `"sum"` | Total |
| `"count"` | Count of non-missing values |
| `"nunique"` | Count of distinct non-missing values |
| `"mean"` | Average |
| `"min"` | Minimum |
| `"max"` | Maximum |

## Sort Results

```python
# Highest to lowest
df_results.sort_values(by="ContractCost", ascending=False)

# Lowest to highest
df_results.sort_values(by="NumberOfContractors", ascending=True)
```

## Select the Top 10 Records

```python
df_results.head(10)
```

## Filter Rows

```python
# Keep contractors appearing in more than one province
df_contractor_provinces[
    df_contractor_provinces["NumberOfProvinces"] > 1
]
```

## Rename Columns

```python
df_results.rename(columns={
    "ProjectID": "NumberOfProjects",
    "Contractor": "NumberOfContractors"
})
```

## Convert Dates and Extract Year

```python
df_cleaned["StartDate"] = pd.to_datetime(
    df_cleaned["StartDate"],
    errors="coerce"
)

df_cleaned["StartYear"] = (
    df_cleaned["StartDate"].dt.year.astype("Int64")
)
```

---

# Important Notes

1. **Run cells in order.** Lab 1 must create `flood_control_cleaned.csv` before Lab 2 reads it.
2. **Preserve raw data.** Use `df_cleaned = df_raw.copy()` before processing.
3. **Missing values:** `isnull().sum()` reports missing values; it does not fill them.
4. **Date conversion:** `errors="coerce"` turns invalid dates into `NaT`.
5. **Project counts:** `ProjectID: "count"` counts non-missing IDs, not necessarily unique projects. Check repeated IDs before interpreting counts as unique projects.
6. **Contractor relationships:** Shared locations or repeated project awards are exploratory clues, not proof of shared ownership or affiliation.
7. **Display formatting:** `pd.options.display.float_format = '{:,.2f}'.format` changes how floats display, not the underlying values. Convert year columns to nullable integers for display as `2018`, `2019`, and so on.
8. **Success messages:** A printed success message means execution reached that line; it does not guarantee data accuracy or that every conversion is appropriate.
9. **Expected files:** `flood_control.json`, `flood_control_raw.csv`, and `flood_control_cleaned.csv`.
10. **Notebook submission:** Save as `Day10_Workshop_STUDENT_NAME.ipynb`, replacing `STUDENT_NAME` with the required name.


# Detailed Code Explanations — Lab 1 and Lab 2

This section explains why the code is written this way and how to interpret its output. The runnable code remains in the lab steps above.

## Lab 1 — Read and Process Data

### Step 1 — Import Libraries

- `import json` loads Python's tools for reading and writing JSON data.
- `import datetime as dt` imports date-and-time tools and gives the module the shorter name `dt`.
- `import requests` allows Python to send an HTTP request to download the dataset.
- `import pandas as pd` imports pandas, the library used to work with tables. `pd` is its common short alias.
- `print(...)` is a checkpoint message. It tells you the code reached that line; it does not prove the downloaded data is correct.

### Step 2 — Configure Pandas Display Settings

- `pd.set_option("display.max_columns", None)` tells pandas to display all columns instead of hiding some.
- `pd.set_option("display.max_rows", 200)` allows up to 200 rows to appear in a DataFrame display.
- `pd.options.display.float_format = '{:,.2f}'.format` displays floating-point numbers with comma separators and two decimal places. It changes the on-screen format, not the underlying values.
- Because this setting applies globally, years stored as floating-point numbers may appear like `2,018.00`. Lab 2 converts `StartYear` to a nullable integer to avoid this.

### Step 3 — Define the Epoch Timestamp Conversion Function

- `def epoch_to_timestamp(ts):` defines a reusable function that accepts one value, `ts`. Defining the function does not run it yet.
- `try:` attempts to convert the value.
- `ts / 1000` converts milliseconds to seconds because `datetime.fromtimestamp()` expects seconds.
- `dt.datetime.fromtimestamp(...)` returns a Python datetime value, interpreted using the computer's local time zone.
- `except: return None` means an unconvertible value returns `None`. This broad exception matches the demo, but it does not tell you why the conversion failed.

### Step 4 — Download the Dataset

- `flood_control_dataset` stores the source URL in a variable.
- `requests.get(flood_control_dataset)` requests the dataset and stores the server response in `response`.
- `if response.ok:` checks whether the HTTP response indicates success. Continue processing only if the request succeeded.
- `response.status_code` reports the HTTP status code when the request fails, which can help diagnose the issue.
- If the download fails, later steps that expect `data` to exist should not be run.

### Step 5 — Save the Raw JSON File

- `response.json()` parses the response body into Python dictionaries and lists and stores it in `data`.
- `with open("flood_control.json", "w") as fp:` opens or creates a file for writing. The `with` statement automatically closes the file afterward.
- `json.dump(data, fp)` writes the parsed data to the file in JSON format.
- This preserves the downloaded structure locally. It is a raw source copy, not the cleaned tabular dataset.

### Step 6 — Extract Attributes and Save Raw CSV

- `data["features"]` selects the list of features from the JSON structure.
- `[f["attributes"] for f in data["features"]]` loops through the features and extracts each feature's `attributes` dictionary. This is a list comprehension.
- `pd.DataFrame(...)` turns those dictionaries into a table called `df_raw`.
- `df_raw.to_csv("flood_control_raw.csv", index=False)` saves the table as a CSV. `index=False` prevents pandas' row index from being written as an extra column.
- `df_raw.shape` returns `(number_of_rows, number_of_columns)`. `df_raw.head()` previews the first five rows by default.

### Step 7 — Create a Working Copy

- `df_raw.copy()` creates a separate DataFrame named `df_cleaned`.
- This keeps the extracted raw data available for comparison while transformations are applied to the copy.
- Copying does not clean anything by itself; it creates a safer working version.

### Step 8 — Convert Epoch Timestamp Columns

- `df_cleaned["CompletionDateOriginal"]` selects the first date column to transform; the other two lines select `CreationDate` and `EditDate`.
- `.map(epoch_to_timestamp)` applies the function to each value in the selected column.
- Each of the three columns is converted separately because the source stores those values as epoch timestamps.
- Values that cannot be converted become `None`. The missing-value review in Step 11 helps reveal whether conversion failures occurred.

### Step 9 — Convert `StartDate`

- `pd.to_datetime(...)` converts supported date strings into pandas datetime values.
- `format="%m/%d/%Y"` specifies the expected input format: month/day/four-digit year.
- `errors="coerce"` means an invalid or unrecognized date becomes `NaT`, pandas' missing datetime value, instead of stopping the cell with an error.
- Converting to datetime makes later operations such as extracting the year possible.

### Step 10 — Remove Unwanted Columns

- `drop(labels=["ABC_String", "ContractCost_String"], ...)` names the columns to remove.
- `axis=1` means the labels refer to columns. `axis=0` would refer to rows.
- `inplace=True` modifies `df_cleaned` directly rather than returning a separate DataFrame.
- The code removes the redundant string versions while keeping `ContractCost` for numeric summaries.

### Step 11 — Review the Cleaned Dataset

- `df_cleaned.shape` reports the number of rows and columns.
- `df_cleaned.dtypes` shows the data type of each column, helping check whether dates and numeric fields have appropriate types.
- `df_cleaned.isnull().sum()` counts missing values per column. It only reports missingness; it does not fill or delete missing values.
- `df_cleaned.head()` previews the first five records for a visual check before saving.

### Step 12 — Save the Cleaned CSV

- `df_cleaned.to_csv("flood_control_cleaned.csv", index=False)` saves the processed table.
- `index=False` prevents pandas' row index from becoming an extra CSV column.
- The shape printed afterward reports the in-memory DataFrame dimensions. A success message means execution reached that line; it is not a substitute for checking the saved file if something seems wrong.

## Lab 2 — Exploratory Data Analysis

### Step 1 — Read the Cleaned Dataset

- `pd.read_csv(...)` loads the saved CSV into `df_cleaned`.
- This allows Lab 2 to start from the saved output of Lab 1 rather than relying on variables from earlier cells.
- `shape` prints the row and column counts. If the file is not in the notebook's working directory, pandas will report a file-not-found error.

### Step 2 — Review the Dataset

- `df_cleaned.columns.tolist()` turns the column names into a regular Python list.
- `.head()` previews the first five rows.
- `.info()` reports the number of entries, non-missing count per column, data types, and memory use. It is useful for spotting unexpected text/numeric/date types.
- These checks provide context before calculating rankings or summaries.

### Step 3 — Top Contractors by Total Contract Cost

- `.groupby("Contractor", as_index=False)` puts rows with the same contractor name into one group. `as_index=False` keeps `Contractor` as a regular output column.
- `.agg({...})` calculates more than one summary: `ContractCost: sum` adds the costs, while `ProjectID: count` counts non-missing project ID values.
- `.sort_values(by="ContractCost", ascending=False)` sorts from highest total cost to lowest.
- `.head(10)` displays the first ten ranked contractors. This is a ranking of recorded totals, not necessarily unique awards.

### Step 4 — Top Provinces by Number of Projects

- `.groupby("Province", as_index=False)` creates one group per province.
- The aggregation counts non-missing `ProjectID` values and sums `ContractCost` in each province.
- Sorting by `ProjectID` descending puts the largest record counts first.
- The output is the top ten provinces by counted records, not necessarily by distinct project IDs.

### Step 5 — Top Municipalities by Number of Projects

- `.groupby(["Province", "Municipality"], ...)` groups on both fields. This avoids combining same-named municipalities in different provinces.
- The aggregation counts project ID records and sums contract costs for each province–municipality combination.
- Sorting by `ProjectID` descending ranks the combinations with the most records.
- `.head(10)` displays the top ten combinations.

### Step 6 — Top Municipalities by Total Contract Cost

- This uses the same province–municipality grouping as Step 5.
- `ContractCost: sum` adds all recorded costs for the location; `ProjectID: count` gives the record-count context.
- Sorting by `ContractCost` descending ranks locations by total recorded cost rather than number of records.
- Compare this with Step 5: a municipality may have fewer records but a larger combined contract cost.

### Step 7 — Contractors with the Most Project Records

- Rows are grouped by contractor name.
- The aggregation counts non-missing project ID values and sums contract costs for each contractor.
- Sorting by `ProjectID` descending ranks contractors by record count.
- This differs from Step 3, which ranks by total cost. The `count` operation does not guarantee unique project IDs.

### Step 8 — Contractors in Multiple Provinces

- The data is grouped by contractor.
- `Province: nunique` counts distinct non-missing province values associated with each contractor.
- `.rename(...)` gives the summary columns clearer names.
- The filter `NumberOfProvinces > 1` keeps contractors associated with more than one distinct province.
- This identifies geographic spread for further review; it does not establish that different contractors are related or that anything improper occurred.

### Step 9 — Contractors by Project Location

- Grouping by province, municipality, and contractor creates a separate row for each observed combination.
- `ProjectID: count` counts non-missing project ID records in the combination; `ContractCost: sum` totals its costs.
- Sorting by project count descending shows combinations with the most records first.
- Use this as a location-based descriptive summary, not as proof of coordination or misconduct.

### Step 10 — Projects by Start Year

- `pd.to_datetime(..., errors="coerce")` ensures the date column uses datetime values; invalid dates become `NaT`.
- `.dt.year` extracts the year from each date. Missing dates lead to missing years.
- `.astype("Int64")` converts the year values to pandas' nullable integer type, which can store whole-number years and `<NA>` together. It also avoids the global float display format showing years like `2,018.00`.
- Grouping by `StartYear` counts non-missing project IDs and sums contract costs per year. Sorting by year puts the summary in chronological order.

### Step 11 — Contract Cost Summary Statistics

- `df_cleaned["ContractCost"]` selects the cost column.
- `.describe()` reports count, mean, standard deviation (`std`), minimum, quartiles (25%, 50%, 75%), and maximum.
- The 50% value is the median. It is often less affected by very large or small values than the mean.
- The maximum can flag a record for closer inspection, but it should be checked against source documentation rather than assumed to be an error.

### Step 12 — Highest-Cost Project Records

- `.sort_values(by="ContractCost", ascending=False)` orders rows from highest to lowest cost.
- The double-bracket selection `[["ProjectID", ...]]` chooses the fields to display while keeping the result as a DataFrame.
- `.head(10)` displays the ten highest-cost records.
- These are records, so duplicated source records could affect the ranking.

### Step 13 — Contractors with Multiple Project Records

- The group-by aggregation calculates a record count and total contract cost for each contractor.
- `df_multiple_projects["ProjectID"] > 1` filters to contractor groups with more than one non-missing project ID record.
- Sorting in descending order puts the largest counts first; `.head(20)` shows up to 20 results.
- This counts records, not distinct project IDs. If unique projects are required, a distinct-ID calculation would be needed.

### Step 14 — Municipalities with the Most Distinct Contractors

- Grouping by province and municipality creates one summary per location.
- `Contractor: nunique` counts distinct non-missing contractor names; `ProjectID: count` counts records; `ContractCost: sum` adds costs.
- `.rename(...)` replaces the generic aggregation names with more descriptive labels.
- Sorting by `NumberOfContractors` descending puts municipalities with the largest number of distinct names first. Different spellings of the same company can be counted separately.

### Step 15 — Municipalities with the Least Distinct Contractors

- This repeats Step 14's grouping and aggregations to make the results comparable.
- `ascending=True` sorts from the smallest contractor count to the largest.
- `nunique()` excludes missing contractor values by default, so a location may show zero distinct contractors if its contractor field is missing in all its records.
- A low count by itself is not proof of restricted competition or wrongdoing; also consider the number of projects, missing values, and source quality.

### Step 16 — Final EDA Summary

- `len(df_cleaned)` returns the number of rows in the DataFrame.
- `.nunique()` counts distinct non-missing contractor, province, and municipality values. These are distinct labels, not necessarily verified unique legal entities or official geographic units.
- `.sum()` adds the values in `ContractCost`.
- The f-string format `:,.2f` displays the total with comma separators and two decimal places.
- Treat the summary as an overview of the processed dataset, not as validation against an authoritative source.

## Common Pandas Patterns — Quick Reference

| Code | Explanation |
|---|---|
| `df["Column"]` | Select one column as a Series. |
| `df[["A", "B"]]` | Select multiple columns as a DataFrame. |
| `.groupby("Column", as_index=False)` | Group rows by values while keeping the group key as a normal column. |
| `.agg({"Cost": "sum"})` | Apply a summary operation to each group. |
| `"sum"` | Add values within each group. |
| `"count"` | Count non-missing values in the selected column; not necessarily unique IDs. |
| `"nunique"` | Count distinct non-missing values. |
| `.sort_values(by="Cost", ascending=False)` | Sort largest to smallest. Use `ascending=True` for smallest to largest. |
| `.head(10)` | Show the first ten rows after the preceding operation. |
| `.isnull().sum()` | Count missing values in each column. |
| `.copy()` | Create a separate working copy. |
| `.dt.year` | Extract the year from datetime values. |
| `.to_csv("file.csv", index=False)` | Save to CSV without adding the DataFrame index as a column. |

## Final Interpretation Reminders

- A row is a record; it is not automatically a unique project.
- A `count` counts non-missing values. Use distinct counting when the question specifically asks for unique IDs.
- Missing data is not the same as zero. The code above reports missing values but does not automatically impute them.
- Rankings and summaries describe the available records. They do not, on their own, prove misconduct, collusion, or a causal explanation.
- A success `print()` confirms the code reached that point, not that the data or analysis is necessarily correct.
