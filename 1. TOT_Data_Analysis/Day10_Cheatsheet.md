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
