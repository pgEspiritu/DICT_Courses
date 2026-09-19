# 📊 Day 1 — Foundations of Data Analytics and the Colab Environment

**TESDA Alignment:** Data Analytics NC III — Apply Statistical Analysis Concepts
**Dataset:** `service_requests.csv`
**Environment:** Google Colab + Python + Pandas

---

## 📌 1. Data Analytics Workflow

A basic data analytics process follows this sequence:

> **Define the question → Collect data → Clean data → Analyze → Interpret → Report**

| Phase                   | Purpose                                                      |
| ----------------------- | ------------------------------------------------------------ |
| **Define the question** | Identify what you need to know                               |
| **Collect data**        | Obtain the required data                                     |
| **Clean data**          | Fix errors, missing values, duplicates, inconsistent formats |
| **Analyze**             | Apply statistical or analytical methods                      |
| **Interpret**           | Explain what the results mean                                |
| **Report**              | Communicate findings to stakeholders                         |

### Example

**Question:** Which barangays have the longest average service resolution time?

1. Define the question
2. Collect service-request data
3. Clean the dataset
4. Calculate average resolution time by barangay
5. Interpret differences
6. Report the findings

---

# 💻 2. Google Colab Basics

## Run a Cell

```text
Shift + Enter
```

Runs the current cell and moves to the next cell.

### Markdown Cell

Used for:

* Titles
* Headings
* Explanations
* Tables
* Documentation
* Notes

### Code Cell

Used to execute Python code.

Example:

```python
print("Hello, data analytics workshop!")
```

Output:

```text
Hello, data analytics workshop!
```

---

# ☁️ 3. Mount Google Drive

Google Drive allows files to persist between Colab sessions.

```python
from google.colab import drive

drive.mount('/content/drive')
```

Typical output:

```text
Mounted at /content/drive
```

### Save a DataFrame to Drive

```python
df.to_csv('/content/drive/MyDrive/service_requests.csv', index=False)
```

### Important

`/content/` is temporary Colab storage.

`/content/drive/MyDrive/` points to your Google Drive.

---

# 🐼 4. Pandas Basics

Import Pandas:

```python
import pandas as pd
```

Import NumPy:

```python
import numpy as np
```

Load a CSV:

```python
df = pd.read_csv('/content/service_requests.csv')
```

---

# 📂 5. Inspecting a DataFrame

## `.shape`

Returns:

```text
(rows, columns)
```

Example:

```python
df.shape
```

Output:

```text
(500, 12)
```

Meaning:

* **500 rows**
* **12 columns**

---

## `.head()`

Shows the first 5 rows.

```python
df.head()
```

Show a different number:

```python
df.head(10)
```

---

## `.dtypes`

Shows Pandas' storage data type for each column.

```python
df.dtypes
```

Example:

```text
request_id               object
date_filed               object
barangay                 object
service_type             object
requestor_name           object
age                       int64
sex                      object
contact_number            int64
priority_level           object
status                   object
resolution_time_days    float64
satisfaction_rating       int64
```

### Important

> **Pandas dtype ≠ statistical data type**

For example:

```text
contact_number → int64
```

Pandas stores it as a number, but statistically it is a **nominal identifier**, not numerical data.

---

## `.info()`

Provides:

* DataFrame type
* Number of rows
* Column names
* Non-null counts
* Data types
* Memory usage

```python
df.info()
```

Example:

```text
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 500 entries, 0 to 499
Data columns (total 12 columns):
...
dtypes: float64(1), int64(3), object(8)
memory usage: 47.0+ KB
```

### Quick Difference

| Command     | Main Purpose                                  |
| ----------- | --------------------------------------------- |
| `df.shape`  | Rows × columns                                |
| `df.head()` | Preview rows                                  |
| `df.dtypes` | Data type of each column                      |
| `df.info()` | Structure + non-null counts + dtypes + memory |

---

# 🔢 6. Unique Values

Count unique values in one column:

```python
df['barangay'].nunique()
```

Count unique values for every column:

```python
for col in df.columns:
    print(col, '->', df[col].dtype, '| unique values:', df[col].nunique())
```

Example:

```text
request_id -> object | unique values: 500
date_filed -> object | unique values: 125
barangay -> object | unique values: 5
service_type -> object | unique values: 6
requestor_name -> object | unique values: 500
age -> int64 | unique values: 62
sex -> object | unique values: 2
contact_number -> int64 | unique values: 500
priority_level -> object | unique values: 3
status -> object | unique values: 4
resolution_time_days -> float64 | unique values: 125
satisfaction_rating -> int64 | unique values: 5
```

### Why inspect unique values?

A column with:

* Few unique values → likely categorical
* Many numeric values → may be numerical
* Unique labels → may be an identifier

But **meaning matters more than dtype**.

---

# 📊 7. Statistical Data Types

The four major classifications used in this activity are:

| Type           | Meaning                                   | Example                     |
| -------------- | ----------------------------------------- | --------------------------- |
| **Nominal**    | Categories/labels with no order           | Sex, barangay               |
| **Ordinal**    | Categories with meaningful order          | Low/Medium/High             |
| **Discrete**   | Countable numerical values                | Number of household members |
| **Continuous** | Measurements that can take decimal values | Processing time             |

A **temporal** variable represents dates/time.

---

# 🏷️ 8. Nominal Data

Nominal data are categories or labels with **no meaningful ranking**.

Examples:

```text
Male
Female
Poblacion
San Isidro
Health Consultation
Business Permit
```

### Dataset examples

| Column           | Type    | Why?                       |
| ---------------- | ------- | -------------------------- |
| `request_id`     | Nominal | Unique label               |
| `barangay`       | Nominal | Name/category              |
| `service_type`   | Nominal | Service category           |
| `requestor_name` | Nominal | Person's name              |
| `sex`            | Nominal | Category                   |
| `contact_number` | Nominal | Identifier, not a quantity |

### Important

A phone number may contain digits but **digits do not automatically make a variable numerical**.

You cannot meaningfully calculate:

```text
average phone number
```

Therefore:

```text
contact_number → Nominal
```

---

# 📈 9. Ordinal Data

Ordinal data have categories with a meaningful order/rank.

Example:

```text
Low < Medium < High
```

Dataset examples:

| Column                | Type     |
| --------------------- | -------- |
| `priority_level`      | Ordinal  |
| `status`              | Ordinal* |
| `satisfaction_rating` | Ordinal  |

### Satisfaction rating

```text
1 < 2 < 3 < 4 < 5
```

The values have an order, but the difference between ratings is not necessarily a measurable equal interval.

---

# 🔢 10. Discrete Data

Discrete data are countable numerical values.

Examples:

```text
1 household member
2 household members
3 household members
...
```

Dataset:

```text
age → Discrete
```

An age recorded in whole years is treated as discrete in this activity.

### Example

**Number of household members**

```text
1
2
3
4
5
```

→ Numerical → **Discrete**

---

# 📏 11. Continuous Data

Continuous data represent measurements and can take decimal values.

Example:

```text
2.5 hours
3.75 hours
7.2 days
```

Dataset:

```text
resolution_time_days → Continuous
```

Example:

```text
7.2 days
5.4 days
0.6 days
5.6 days
```

---

# 📅 12. Temporal Data

Temporal data represent dates or times.

Dataset:

```text
date_filed
```

Example:

```text
2025-01-01
2025-01-02
2025-02-12
```

Even if Pandas initially reads the column as:

```text
object
```

its **statistical meaning is temporal**.

---

# 🧾 13. Complete Dataset Classification

| Column                 | Pandas dtype | Statistical Type | Reason                      |
| ---------------------- | ------------ | ---------------- | --------------------------- |
| `request_id`           | object       | Nominal          | Unique label                |
| `date_filed`           | object       | Temporal         | Date                        |
| `barangay`             | object       | Nominal          | Name/category               |
| `service_type`         | object       | Nominal          | Service category            |
| `requestor_name`       | object       | Nominal          | Name of requestor           |
| `age`                  | int64        | Discrete         | Age recorded as whole years |
| `sex`                  | object       | Nominal          | Category                    |
| `contact_number`       | int64        | Nominal          | Identifier                  |
| `priority_level`       | object       | Ordinal          | Low → Medium → High         |
| `status`               | object       | Ordinal*         | Request-status categories   |
| `resolution_time_days` | float64      | Continuous       | Time measurement            |
| `satisfaction_rating`  | int64        | Ordinal          | Rating from 1–5             |

> **Key idea:** Always classify variables according to their **meaning**, not merely their Pandas dtype.

---

# 🔐 14. Data Privacy

The activity introduces the **Data Privacy Act of 2012 (RA 10173)**.

When analyzing government data, ask:

1. Does this identify a person?
2. Is the information necessary for the analysis?
3. Can the information be removed?
4. Can it be masked or aggregated?
5. Could combinations of fields allow re-identification?

---

# 👤 15. Personal / Sensitive Data Audit

| Column                 | Classification                | Recommended Action           |
| ---------------------- | ----------------------------- | ---------------------------- |
| `request_id`           | System-generated identifier   | Keep with caution            |
| `barangay`             | Low-risk indirect information | Keep; consider combinations  |
| `service_type`         | Low risk                      | Keep                         |
| `requestor_name`       | Direct identifier             | Remove or anonymize          |
| `age`                  | Personal information          | Aggregate/report carefully   |
| `sex`                  | Personal information          | Disclose only when necessary |
| `contact_number`       | Direct identifier             | Mask/anonymize               |
| `priority_level`       | Non-personal                  | Keep                         |
| `status`               | Non-personal                  | Keep                         |
| `resolution_time_days` | Non-personal                  | Keep                         |
| `satisfaction_rating`  | Non-personal                  | Keep                         |

---

# 🛡️ 16. Anonymization

## Option A — Remove the identifier

```python
df_anon = df.copy()

df_anon = df_anon.drop(
    columns=['requestor_name']
)
```

This completely removes the direct identifier from the analysis dataset.

---

## Option B — Mask the contact number

Keep only the last four digits:

```python
df_anon['contact_number'] = df_anon['contact_number'].apply(
    lambda x: 'XXXXXXX' + str(x)[-4:]
)
```

Example:

```text
09233719871
```

becomes approximately:

```text
XXXXXXX9871
```

### Why mask?

You preserve limited verification capability without exposing the complete phone number.

---

# 🔑 17. Pseudonymization

Pseudonymization replaces an identifier with a code.

Example:

```python
lookup = df[['request_id', 'requestor_name']].copy()
```

Create a pseudonym:

```python
df_anon['requestor_code'] = (
    'RESIDENT-' + df_anon.index.astype(str)
)
```

Example:

```text
Resident 0
```

becomes something like:

```text
RESIDENT-0
```

### Important

The real names should be stored separately:

```text
Analysis dataset
        ↓
requestor_code
        ↓
Separate restricted lookup table
        ↓
real requestor name
```

The lookup table should **not** be distributed with the analysis dataset.

---

# ⚠️ 18. Residual Re-identification Risk

Removing names does **not automatically guarantee anonymity**.

For example:

```text
barangay
+
service_type
+
date_filed
```

could potentially narrow a record down to a very small number of people.

### Safer reporting

Instead of publishing row-level records:

```text
Resident + exact date + exact service
```

publish aggregate information such as:

```text
Barangay-level request counts
Average resolution time
Percentage of completed requests
```

### Key Principle

> **Collect and process only information necessary for the stated purpose.**

---

# 💾 19. Save the Anonymized Dataset

```python
df_anon.to_csv(
    '/content/service_requests_anonymized.csv',
    index=False
)
```

Verify:

```python
print("Saved:", df_anon.shape)
```

---

# 🧪 20. Course Dataset Generation

The course dataset contains **500 rows and 12 columns**.

```python
np.random.seed(42)
n = 500
```

Using:

```python
np.random.seed(42)
```

makes the random dataset reproducible.

### Main categories

**Barangays:**

```text
Poblacion
San Isidro
Santa Cruz
Malinis
Bagong Silang
```

**Service types:**

```text
Business Permit
Barangay Clearance
Indigency Certificate
Health Consultation
Social Welfare Assistance
Road Repair Report
```

**Statuses:**

```text
Pending
In Progress
Completed
Cancelled
```

**Sex:**

```text
Male
Female
```

**Priority:**

```text
Low
Medium
High
```

---

# 🧱 21. Creating the DataFrame

Basic structure:

```python
data = {
    'column_name': values
}

df = pd.DataFrame(data)
```

Example:

```python
df = pd.DataFrame(data)
```

Save:

```python
df.to_csv(
    '/content/service_requests.csv',
    index=False
)
```

Preview:

```python
df.head()
```

---

# 🧠 22. Descriptive vs Predictive Analytics

## Descriptive Analytics

Answers:

> **What happened?**

Examples:

* Number of service requests
* Average resolution time
* Unemployment rate per LGU
* Number of completed requests

### Government example

**CBMS data showing unemployment rates per LGU** can be used descriptively to summarize observed conditions.

---

## Predictive Analytics

Answers:

> **What might happen?**

Uses historical/current data to estimate future outcomes.

Example:

```text
Historical voter registration data
            ↓
Pattern analysis
            ↓
Estimated future voter population
```

### Important

Predictive analytics produces an **estimate**, not certainty.

---

# 📝 23. Exit Ticket — Answers

## 1. Descriptive vs Predictive Analytics

**Descriptive:**

> Community-Based Monitoring System (CBMS) data showing the unemployment rate per LGU.

**Predictive:**

> Historical voter-registration data can be analyzed to estimate the number of voters in future elections.

---

## 2. Classify the Variables

| Variable                           | Classification         |
| ---------------------------------- | ---------------------- |
| Type of ID presented               | Categorical — Nominal  |
| Number of household members        | Numerical — Discrete   |
| Satisfaction score (1–5)           | Categorical — Ordinal  |
| Time to process a request in hours | Numerical — Continuous |

### Memory Trick

```text
NAME / LABEL → NOMINAL
RANK / ORDER → ORDINAL
COUNT → DISCRETE
MEASUREMENT → CONTINUOUS
DATE / TIME → TEMPORAL
```

---

## 3. Correct Analytics Workflow

```text
Define the question
        ↓
Collect data
        ↓
Clean data
        ↓
Analyze
        ↓
Interpret
        ↓
Report
```

---

## 4. Data That Should Not Be Published As-Is

### Contact Number

Should not be published as-is because it is personally identifiable information that can directly identify or contact an individual.

Recommended:

```text
09233719871
```

→

```text
XXXXXXX9871
```

### Request ID

A request ID should also be handled carefully if it can be linked back to a specific person or transaction.

Use:

* An anonymized identifier
* A pseudonym
* An aggregated result

---

# ⚡ 24. Quick Command Cheat Sheet

| Task                   | Command                              |
| ---------------------- | ------------------------------------ |
| Import Pandas          | `import pandas as pd`                |
| Import NumPy           | `import numpy as np`                 |
| Read CSV               | `pd.read_csv('file.csv')`            |
| Save CSV               | `df.to_csv('file.csv', index=False)` |
| Number of rows/columns | `df.shape`                           |
| First 5 rows           | `df.head()`                          |
| First N rows           | `df.head(N)`                         |
| Data types             | `df.dtypes`                          |
| Dataset information    | `df.info()`                          |
| Unique count           | `df['column'].nunique()`             |
| All column names       | `df.columns`                         |
| Copy DataFrame         | `df.copy()`                          |
| Remove columns         | `df.drop(columns=['column'])`        |
| Apply function         | `df['column'].apply(function)`       |
| Mount Drive            | `drive.mount('/content/drive')`      |

---

# 🎯 25. Exam / Practical Review

### Remember These

**`.shape`**

```python
df.shape
```

→ `(rows, columns)`

**`.head()`**

```python
df.head()
```

→ First 5 rows

**`.dtypes`**

```python
df.dtypes
```

→ Pandas storage types

**`.info()`**

```python
df.info()
```

→ Structure, non-null counts, dtypes, memory

**`.nunique()`**

```python
df['column'].nunique()
```

→ Number of unique values

---

# 🧩 26. Most Important Concepts

### 1. Data type ≠ Pandas dtype

```text
Pandas dtype = how data is stored
Statistical type = what the data means
```

---

### 2. Numbers are not always numerical variables

```text
Age              → Numerical
Household count  → Numerical
Phone number     → Nominal
ID number        → Nominal
```

---

### 3. Ordered categories are ordinal

```text
Low → Medium → High
```

---

### 4. Measurements are continuous

```text
2.5 hours
7.2 days
3.75 meters
```

---

### 5. Privacy should be considered before analysis/reporting

Ask:

```text
Can this identify someone?
        ↓
Is it necessary?
        ↓
Can I remove it?
        ↓
Can I mask it?
        ↓
Can I aggregate it?
```

---

# 🚀 27. One-Minute Review

```text
DATA ANALYTICS
│
├── Define question
├── Collect data
├── Clean data
├── Analyze
├── Interpret
└── Report
```

```text
STATISTICAL TYPES
│
├── Nominal      → labels/categories
├── Ordinal      → ordered categories
├── Discrete     → counts
├── Continuous   → measurements
└── Temporal     → dates/time
```

```text
PANDAS INSPECTION
│
├── df.shape      → dimensions
├── df.head()     → preview
├── df.dtypes     → storage types
├── df.info()     → structure
└── nunique()     → unique values
```

```text
PRIVACY
│
├── Identify PII
├── Remove identifiers
├── Mask sensitive values
├── Pseudonymize when necessary
├── Keep lookup tables restricted
└── Aggregate before public reporting
```

---

# 🏁 Final Takeaway

The central lesson of Day 1 is:

> **Good data analytics starts before statistical analysis.**

You first need to understand **what the data represents**, how it is **stored**, whether it is **appropriate for analysis**, and whether it can be **used and shared responsibly**.

A reliable workflow is:

**Understand → Inspect → Classify → Protect → Analyze → Communicate**
