# 📊 Python Data Analysis Cheatsheet

## Correlation, Visualization & Simple Linear Regression

> Quick reference for the Python, pandas, Seaborn, Matplotlib, and scikit-learn code encountered in the workshop.

---

# 1. 📦 Import Common Libraries

```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
```

### Machine Learning imports

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.metrics import (
    mean_absolute_error,
    r2_score,
    root_mean_squared_error
)
```

### What they are for

| Library / Function        | Purpose                                                        |
| ------------------------- | -------------------------------------------------------------- |
| `pandas`                  | DataFrames and data manipulation                               |
| `numpy`                   | Numerical operations                                           |
| `seaborn`                 | Statistical visualization and sample datasets                  |
| `matplotlib.pyplot`       | Plot customization and display                                 |
| `train_test_split`        | Split data into training/testing sets                          |
| `LinearRegression`        | Create a linear regression model                               |
| `mean_absolute_error`     | Measure average prediction error                               |
| `r2_score`                | Measure explained variation                                    |
| `root_mean_squared_error` | Measure prediction error with greater penalty for large errors |

---

# 2. 📥 Load a Seaborn Dataset

```python
diamonds = sns.load_dataset("diamonds")
```

Check a categorical column:

```python
diamonds["cut"].unique()
```

### Select specific columns

```python
diamonds = diamonds[
    ["carat", "price", "cut"]
]
```

### Select multiple columns

```python
df[["column1", "column2", "column3"]]
```

### Select one column

```python
df["column"]
```

---

# 3. 🧹 Missing Values

### Count missing values per column

```python
df.isna().sum()
```

### Check whether the entire DataFrame contains missing values

```python
df.isna().any().any()
```

### Remove rows containing missing values

```python
df = df.dropna()
```

### Remove missing values from a specific column

```python
df = df.dropna(subset=["column"])
```

### Fill missing values

```python
df["column"] = df["column"].fillna("Unknown")
```

### Fill numeric missing values with the median

```python
df["column"] = df["column"].fillna(
    df["column"].median()
)
```

---

# 4. 🔍 Inspect a DataFrame

### Number of rows and columns

```python
df.shape
```

Example:

```python
print(df.shape)
```

### First rows

```python
df.head()
```

### Last rows

```python
df.tail()
```

### Data types

```python
df.dtypes
```

### General information

```python
df.info()
```

### Summary statistics

```python
df.describe()
```

### Unique values

```python
df["column"].unique()
```

### Number of unique values

```python
df["column"].nunique()
```

### Frequency of each category

```python
df["column"].value_counts()
```

---

# 5. 🔄 Make a Copy Before Cleaning

Instead of modifying the original data:

```python
clean = raw.copy()
```

This allows you to keep:

```text
raw   → original data
clean → cleaned/modified data
```

### Good practice

```python
raw = pd.read_csv("data.csv")

clean = raw.copy()
```

---

# 6. 🧽 Remove Duplicates

### Remove complete duplicate rows

```python
clean = clean.drop_duplicates()
```

### Remove duplicates based on a specific key

```python
clean = clean.drop_duplicates(
    subset=["request_id"]
)
```

### Count duplicate rows

```python
clean.duplicated().sum()
```

### Count duplicate IDs

```python
clean["request_id"].duplicated().sum()
```

---

# 7. ✂️ Clean Text Categories

### Remove leading/trailing spaces

```python
clean["region"] = clean["region"].str.strip()
```

### Convert to uppercase

```python
clean["region"] = clean["region"].str.upper()
```

### Convert to lowercase

```python
clean["region"] = clean["region"].str.lower()
```

### Convert to title case

```python
clean["column"] = clean["column"].str.title()
```

### Replace a specific category

```python
clean["service_type"] = clean["service_type"].replace({
    "Free Wifi Installation": "Free WiFi Installation"
})
```

### Chain text cleaning operations

```python
clean["region"] = (
    clean["region"]
    .str.strip()
    .str.upper()
)
```

---

# 8. 📅 Convert Dates

For mixed date formats:

```python
clean["date_filed"] = pd.to_datetime(
    clean["date_filed"],
    format="mixed"
)
```

Check the result:

```python
clean["date_filed"].dtype
```

Extract year:

```python
clean["date_filed"].dt.year
```

Extract month:

```python
clean["date_filed"].dt.month
```

Extract day:

```python
clean["date_filed"].dt.day
```

---

# 9. 🔢 Numeric Data

### Minimum

```python
df["column"].min()
```

### Maximum

```python
df["column"].max()
```

### Mean

```python
df["column"].mean()
```

### Median

```python
df["column"].median()
```

### Standard deviation

```python
df["column"].std()
```

### Count

```python
df["column"].count()
```

---

# 10. 🚨 Outliers and IQR

### Calculate Q1 and Q3

```python
Q1 = clean["days_to_resolve"].quantile(0.25)
Q3 = clean["days_to_resolve"].quantile(0.75)
```

### Calculate IQR

```python
IQR = Q3 - Q1
```

### Calculate upper fence

```python
upper_fence = Q3 + 1.5 * IQR
```

### Count values above the upper fence

```python
outlier_count = (
    clean["days_to_resolve"] > upper_fence
).sum()
```

### Count sentinel value such as 999

```python
sentinel_count = (
    clean["days_to_resolve"] == 999
).sum()
```

### Replace sentinel values with missing

```python
clean["days_to_resolve"] = clean[
    "days_to_resolve"
].replace(999, np.nan)
```

---

# 11. 🏷️ Create a Missing-Value Flag

Useful when you want to preserve information that a value was originally missing.

```python
clean["fee_was_missing"] = (
    clean["processing_fee"].isna()
)
```

Then fill the missing value:

```python
clean["processing_fee"] = clean[
    "processing_fee"
].fillna(
    clean["processing_fee"].median()
)
```

### Why?

You now have:

```text
processing_fee
    ↓
filled with median

fee_was_missing
    ↓
True / False
```

This preserves the fact that the original value was missing.

---

# 12. 📊 Correlation

### Pearson correlation between two columns

```python
df["carat"].corr(df["price"])
```

Store it:

```python
carat_price_r = df["carat"].corr(
    df["price"]
)
```

### Correlation matrix

```python
df[["carat", "price"]].corr()
```

### Correlation matrix for numeric columns

```python
df.corr(numeric_only=True)
```

### Correlation with a target variable

```python
correlations = df.corr(
    numeric_only=True
)["target"].sort_values(
    ascending=False
)
```

---

# 13. 📈 Understanding Pearson Correlation

Pearson correlation `r` ranges from:

```text
-1  ←────────  0  ────────→  +1
```

|           `r` | General interpretation              |
| ------------: | ----------------------------------- |
| Close to `+1` | Strong positive linear relationship |
|  Close to `0` | Weak/no linear relationship         |
| Close to `-1` | Strong negative linear relationship |

### Positive

```text
X ↑ → Y ↑
```

### Negative

```text
X ↑ → Y ↓
```

### Important

Correlation measures **association**, not causation.

A strong correlation does not prove:

```text
X causes Y
```

---

# 14. 📉 Scatterplot

### Basic scatterplot

```python
sns.scatterplot(
    data=df,
    x="carat",
    y="price"
)

plt.show()
```

### Make overlapping points easier to see

```python
sns.scatterplot(
    data=df,
    x="carat",
    y="price",
    alpha=0.2
)
```

### Remember

```text
x-axis → predictor / explanatory variable
y-axis → outcome / response variable
```

For the diamonds example:

```text
x = carat
y = price
```

---

# 15. 📐 Regression Plot

Seaborn can add a fitted regression line:

```python
sns.regplot(
    data=df,
    x="carat",
    y="price",
    scatter_kws={"alpha": 0.2}
)

plt.show()
```

### Customize the regression line

```python
sns.regplot(
    data=df,
    x="carat",
    y="price",
    scatter_kws={"alpha": 0.2},
    line_kws={"color": "red"}
)
```

### What to look for

When examining a scatterplot, describe:

1. **Direction**

   * Positive
   * Negative

2. **Shape**

   * Linear
   * Curved
   * Other pattern

3. **Spread**

   * Tight around the line
   * Wide variation
   * Increasing spread

---

# 16. 🤖 Simple Linear Regression

Simple linear regression uses:

```text
ONE predictor
        ↓
ONE outcome
```

Example:

```text
carat → price
```

### Create predictor

```python
X = diamonds[["carat"]]
```

### Create target

```python
y = diamonds["price"]
```

### Important

Use double brackets for `X`:

```python
X = df[["carat"]]
```

rather than:

```python
X = df["carat"]
```

because scikit-learn expects `X` to be two-dimensional.

---

# 17. ✂️ Train/Test Split

```python
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.20,
    random_state=42
)
```

### Meaning

```text
80% → training data
20% → test data
```

The model learns from:

```python
X_train
y_train
```

Then we evaluate it using:

```python
X_test
y_test
```

### Why `random_state`?

```python
random_state=42
```

makes the split reproducible.

---

# 18. 🧠 Create a Linear Regression Model

```python
model = LinearRegression()
```

### Train the model

```python
model.fit(
    X_train,
    y_train
)
```

Think:

```text
.fit()
    ↓
learn from training data
```

---

# 19. 🔮 Make Predictions

```python
predictions = model.predict(X_test)
```

Think:

```text
X_test
   ↓
trained model
   ↓
predictions
```

---

# 20. 📏 Regression Coefficient / Slope

```python
carat_coefficient = model.coef_[0]
```

### Meaning

The coefficient represents the estimated change in the outcome for a **one-unit increase in the predictor**.

For diamonds:

```text
carat → price
```

The coefficient represents the estimated change in **price in US dollars** for a **1-carat increase**.

### General form

```text
predicted Y = intercept + coefficient × X
```

---

# 21. 🧮 Intercept

Get the intercept:

```python
model.intercept_
```

The intercept is the model's estimated outcome when the predictor equals zero.

For a linear model:

```text
price = intercept + coefficient × carat
```

---

# 22. 📐 Mean Absolute Error — MAE

```python
test_mae = mean_absolute_error(
    y_test,
    predictions
)
```

### Meaning

MAE represents the average absolute prediction error.

If:

```text
MAE = 1,000
```

then predictions are off from the actual values by about:

```text
$1,000 on average
```

MAE is expressed in the **same units as the target variable**.

---

# 23. 🎯 R² Score

```python
test_r2 = r2_score(
    y_test,
    predictions
)
```

R² describes how much variation in the outcome is explained by the model.

For example:

```text
R² = 0.85
```

can be interpreted as the model explaining approximately **85% of the variation** in the outcome for the evaluated data.

### Important

R² is not:

```text
85% prediction accuracy
```

It measures explained variation, not the percentage of predictions that are correct.

---

# 24. 📏 RMSE

```python
test_rmse = root_mean_squared_error(
    y_test,
    predictions
)
```

RMSE measures prediction error while giving **more weight to larger errors**.

### Compare

```text
MAE
→ average absolute error

RMSE
→ error metric that penalizes larger errors more heavily
```

---

# 25. ⚠️ Common Metric Mistake

Correct:

```python
test_mae = mean_absolute_error(
    y_test,
    predictions
)

test_r2 = r2_score(
    y_test,
    predictions
)
```

Think:

```text
actual values → predictions
```

Avoid accidentally reversing them.

---

# 26. 🔥 Correlation Heatmap

```python
plt.figure(figsize=(8, 6))

sns.heatmap(
    df.corr(numeric_only=True),
    annot=True,
    cmap="coolwarm"
)

plt.show()
```

### `annot=True`

Displays the correlation values inside the cells.

---

# 27. 📊 Group Comparison

For comparing a numeric variable across categories:

```python
sns.boxplot(
    data=df,
    x="cut",
    y="price"
)

plt.show()
```

### Useful questions

```text
Do groups have different medians?

How wide is the spread?

Are there outliers?

Do groups overlap?
```

---

# 28. 📦 Grouped Summary

Calculate a statistic by category:

```python
df.groupby("cut")["price"].mean()
```

Multiple statistics:

```python
df.groupby("cut")["price"].agg(
    ["mean", "median", "min", "max"]
)
```

---

# 29. 🔗 Correlation vs Regression

### Correlation

Answers:

> How strongly are two variables linearly associated?

```python
df["carat"].corr(df["price"])
```

Produces:

```text
r
```

### Regression

Answers:

> Can we use one variable to estimate another?

```python
model.fit(X_train, y_train)
```

Produces:

```text
coefficient
intercept
predictions
```

### Remember

```text
Correlation
    ↓
strength + direction of linear association

Regression
    ↓
prediction / estimated relationship
```

Neither alone establishes causation.

---

# 30. 🧪 Complete Regression Workflow

The common workflow is:

```text
1. Load data
       ↓
2. Select relevant columns
       ↓
3. Check missing values
       ↓
4. Clean data
       ↓
5. Explore relationship
       ↓
6. Calculate correlation
       ↓
7. Create X and y
       ↓
8. Train/test split
       ↓
9. Fit model
       ↓
10. Predict
       ↓
11. Evaluate
       ↓
12. Interpret results
```

### Compact code pattern

```python
# Load and prepare data
df = sns.load_dataset("diamonds")[
    ["carat", "price", "cut"]
].dropna()

# Correlation
carat_price_r = df["carat"].corr(
    df["price"]
)

# Predictor and target
X = df[["carat"]]
y = df["price"]

# Train/test split
X_train, X_test, y_train, y_test = train_test_split(
    X,
    y,
    test_size=0.20,
    random_state=42
)

# Model
model = LinearRegression()

# Train
model.fit(X_train, y_train)

# Predict
predictions = model.predict(X_test)

# Coefficient
carat_coefficient = model.coef_[0]

# Evaluation
test_mae = mean_absolute_error(
    y_test,
    predictions
)

test_r2 = r2_score(
    y_test,
    predictions
)

test_rmse = root_mean_squared_error(
    y_test,
    predictions
)
```

---

# 31. 🧠 Interpretation Cheat Sheet

## Correlation

```text
r > 0
→ positive relationship

r < 0
→ negative relationship

r ≈ 0
→ little/no linear relationship

|r| close to 1
→ strong linear relationship
```

---

## Regression coefficient

```text
Coefficient
= estimated change in Y
  for a 1-unit increase in X
```

Example:

```text
carat coefficient = 7,700

→ estimated $7,700 increase in price
  for each additional carat
```

Remember:

```text
Association ≠ causation
```

---

## MAE

```text
MAE = average absolute prediction error
```

If:

```text
MAE = $1,000
```

then:

```text
Predictions differ from actual prices
by about $1,000 on average.
```

---

## R²

```text
R² = proportion of variation
     explained by the model
```

Example:

```text
R² = 0.85

→ approximately 85% of the variation
  is explained by the model
```

---

## RMSE

```text
RMSE = prediction error
        with greater sensitivity
        to large errors
```

---

# 32. 🔎 Questions to Ask When Looking at a Scatterplot

Always check:

### Direction

```text
↗ Positive
↘ Negative
→ No obvious direction
```

### Shape

```text
Linear?
Curved?
Clusters?
```

### Strength

```text
Are points close to a line?
Or widely scattered?
```

### Spread

```text
Does the spread stay constant?

Or does it become wider/narrower
as X increases?
```

### Outliers

```text
Are there observations far away
from the main pattern?
```

---

# 33. ⚠️ Correlation Does Not Mean Causation

A strong correlation such as:

```text
r = 0.92
```

does **not** prove:

```text
Increasing X causes Y to increase.
```

For the diamonds example:

```text
carat ↔ price
```

A strong relationship exists, but price is also related to:

```text
cut
color
clarity
and other characteristics
```

Therefore, be careful when using causal language.

Prefer:

```text
"Carat is positively associated with price."

"Carat helps predict price."

"The model estimates..."
```

Avoid:

```text
"Adding one carat causes the price to increase..."
```

unless the study design supports a causal conclusion.

---

# 34. 🧩 Quick Syntax Reference

| Task                 | Code                                           |
| -------------------- | ---------------------------------------------- |
| Load Seaborn dataset | `sns.load_dataset("diamonds")`                 |
| Select columns       | `df[["a", "b"]]`                               |
| One column           | `df["a"]`                                      |
| Missing count        | `df.isna().sum()`                              |
| Remove missing       | `df.dropna()`                                  |
| Copy DataFrame       | `df.copy()`                                    |
| Remove duplicates    | `df.drop_duplicates()`                         |
| Unique values        | `df["a"].unique()`                             |
| Frequency            | `df["a"].value_counts()`                       |
| Mean                 | `df["a"].mean()`                               |
| Median               | `df["a"].median()`                             |
| Minimum              | `df["a"].min()`                                |
| Maximum              | `df["a"].max()`                                |
| Correlation          | `df["a"].corr(df["b"])`                        |
| Scatterplot          | `sns.scatterplot(...)`                         |
| Regression plot      | `sns.regplot(...)`                             |
| Train/test split     | `train_test_split(...)`                        |
| Create model         | `LinearRegression()`                           |
| Train model          | `model.fit(X_train, y_train)`                  |
| Predict              | `model.predict(X_test)`                        |
| Coefficient          | `model.coef_[0]`                               |
| Intercept            | `model.intercept_`                             |
| MAE                  | `mean_absolute_error(y_test, predictions)`     |
| R²                   | `r2_score(y_test, predictions)`                |
| RMSE                 | `root_mean_squared_error(y_test, predictions)` |
| Display plot         | `plt.show()`                                   |

---

# 35. 🧠 Most Important Patterns to Remember

### DataFrame vs Series

```python
df["carat"]
```

→ one column / Series

```python
df[["carat"]]
```

→ one column / DataFrame

This distinction is especially important for:

```python
X = df[["carat"]]
```

because machine-learning predictors should normally be 2-dimensional.

---

### `.fit()` vs `.predict()`

```python
model.fit(X_train, y_train)
```

means:

> Learn from the training data.

```python
model.predict(X_test)
```

means:

> Use what was learned to make predictions for new/test data.

---

### Training vs Testing

```text
Training data
→ model learns

Test data
→ model is evaluated
```

Do not evaluate the model only on the same data used to train it.

---

# 36. 🏁 Final Mental Model

For a typical data-analysis problem:

```text
DATA
 ↓
Inspect
 ↓
Clean
 ↓
Explore
 ↓
Visualize
 ↓
Correlation
 ↓
Choose X and y
 ↓
Train/Test Split
 ↓
Fit Model
 ↓
Predict
 ↓
Evaluate
 ↓
Interpret
```

And always ask:

> **What does the code calculate?**

> **What are the units?**

> **What does the result actually tell me?**

> **What can I conclude—and what can I not conclude?**
