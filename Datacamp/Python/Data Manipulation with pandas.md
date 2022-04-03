---
layout: article
title: Data Manipulation with pandas
tags: Python Pandas DataScience
pageview: false
aside:
  toc: true
---

Base on DataCamp.

<!--more-->

## 1. Transforming DataFrames

### 1.1. Introducing DataFrames
Exploring a DataFrame:
- `.head()` returns the first few rows (the “head” of the DataFrame).
- `.info()` shows information on each of the columns, such as the data type and number of missing values.
- `.shape` returns the number of rows and columns of the DataFrame.
- `.describe()` calculates a few summary statistics for each column.

Components of a DataFrame:
- `.values`: A two-dimensional NumPy array of values.
- `.columns`: An index of columns: the column names.
- `.index`: An index for the rows: either row numbers or row names.

#### Inspecting a DataFrame

```py
# Print the head of the homelessness data
print(homelessness.head())

# Print information about homelessness
print(homelessness.info())

# Print the shape of homelessness
print(homelessness.shape)

# Print a description of homelessness
print(homelessness.describe())
```

#### Parts of a DataFrame

```py
# Import pandas using the alias pd
import pandas as pd

# Print the values of homelessness
print(homelessness.values)

# Print the column index of homelessness
print(homelessness.columns)

# Print the row index of homelessness
print(homelessness.index)
```

### 1.2. Sorting and subsetting

Sorting
You can sort the rows by passing a column name to `.sort_values()`
- Sorting: 
`df.sort_values("column_name_1")`
- Sorting in descending order: 
`df.sort_values("column_name_1", ascending=False)`
- Sorting b ymultiple variables:  
`df.sort_values(["column_name_1","column_name_2"])`
`df.sort_values(["column_name_1", "column_name_2"], ascending=[True, False])`

By combining .sort_values() with .head(), you can answer questions in the form, "What are the top cases where…?".

Subsetting columns
Square brackets ([]) can be used to select only the columns that matter to you in an order that makes sense to you. 
- Subsetting columns: `df["col_a"]`
- Subsetting multiple columns: `df[["col_a", "col_b"]]`
- Subsetting rows: `df[df["col_a"] > 60]`
- Subsetting based on text data: `df[df["color"] == "tan"]`
- Subsetting based on dates: `df[df["date_of_birth"] < "2015-01-01"]`
- Subsetting based on multiple conditions: `df[(df["breed"] == "Labrador") & (df["color"] == "Brown")]`
- Subsetting using `.isin()`: `df[df["color"].isin(["Black", "Brown"])]`

#### Sorting rows

```py
# Sort homelessness by individuals
homelessness_ind = homelessness.sort_values('individuals')

# Print the top few rows
print(homelessness_ind.head())
```

```py
# Sort homelessness by descending family members
homelessness_fam = homelessness.sort_values('family_members',ascending=False)

# Print the top few rows
print(homelessness_fam.head())
```

```py
# Sort homelessness by region, then descending family members
homelessness_reg_fam = homelessness.sort_values(['region','family_members'], ascending=[True,False])

# Print the top few rows
print(homelessness_reg_fam.head())
```

#### Subsetting columns

```py
# Select the individuals column
individuals = homelessness['individuals']

# Print the head of the result
print(individuals.head())
```

```py
# Select the state and family_members columns
state_fam = homelessness[['state','family_members']]

# Print the head of the result
print(state_fam.head())
```

```py
# Select only the individuals and state columns, in that order
ind_state = homelessness[['individuals','state']]

# Print the head of the result
print(ind_state.head())
```

#### Subsetting rows

```py
# Filter for rows where individuals is greater than 10000
ind_gt_10k = homelessness[homelessness['individuals']>10000]

# See the result
print(ind_gt_10k)
```

```py
# Filter for rows where region is Mountain
mountain_reg = homelessness[homelessness['region']=='Mountain']

# See the result
print(mountain_reg)
```

```py
# Filter for rows where family_members is less than 1000 
# and region is Pacific
fam_lt_1k_pac = homelessness[(homelessness['family_members']<1000) & (homelessness['region']=='Pacific')]

# See the result
print(fam_lt_1k_pac)
```

#### Subsetting rows by categorical variables

```py
# Subset for rows in South Atlantic or Mid-Atlantic regions
south_mid_atlantic = homelessness[(homelessness['region']=='South Atlantic') | (homelessness['region']=='Mid-Atlantic')]

# See the result
print(south_mid_atlantic)
```

```py
# The Mojave Desert states
canu = ["California", "Arizona", "Nevada", "Utah"]

# Filter for rows in the Mojave Desert states
mojave_homelessness = homelessness[homelessness['state'].isin(canu)]

# See the result
print(mojave_homelessness)
```

### 1.3. New columns

#### Adding a new column
`df["height_m"] = df["height_cm"] / 100`

```py
# Add total col as sum of individuals and family_members
homelessness['total']=homelessness['individuals'] + homelessness['family_members']

# Add p_individuals col as proportion of total that are individuals
homelessness['p_individuals']=homelessness['individuals'] / homelessness['total']

# See the result
print(homelessness)
```

#### Combo-attack!

```py
# Create indiv_per_10k col as homeless individuals per 10k state pop
homelessness["indiv_per_10k"] = 10000 * homelessness['individuals'] / homelessness['state_pop']

# Subset rows for indiv_per_10k greater than 20
high_homelessness = homelessness[homelessness['indiv_per_10k']>20]

# Sort high_homelessness by descending indiv_per_10k
high_homelessness_srt = high_homelessness.sort_values(['indiv_per_10k'],ascending=False)

# From high_homelessness_srt, select the state and indiv_per_10k cols
result = high_homelessness_srt[['state','indiv_per_10k']]

# See the result
print(result)
```

## 2. Aggregating DataFrames

### 2.1. Summary Statistics

Summarizing numerical data
- `.median(),.mode()`
- `.min(),.max()`
- `.var(),.std()`
- `.sum()`
- `.quantile()`

Summarizing dates
`df["date_of_birth"].min()`
`df["date_of_birth"].max()`

The .agg() method: The .agg() method allows you to apply your own custom functions to a DataFrame, as well as apply functions to more than one column of a DataFrame at once, making your aggregations super-efficient `df['column'].agg(function)`
```py
def pct30(column):
  return column.quantile(0.3)
  
dogs["weight_kg"].agg(pct30)
```

Summaries on multiple columns
`dogs[["weight_kg", "height_cm"]].agg(pct30)`

Multiple summaries
```py
def pct40(column):
  return column.quantile(0.4)
  
dogs["weight_kg"].agg([pct30, pct40])
```

Cumulative sum
- `.cummax()`
- `.cummin()`
- `.cumprod()`

#### Mean and median

```py
# Print the head of the sales DataFrame
print(sales.head())

# Print the info about the sales DataFrame
print(sales.info())

# Print the mean of weekly_sales
print(sales['weekly_sales'].mean())

# Print the median of weekly_sales
print(sales['weekly_sales'].median())
```

#### Summarizing dates

```py
# Print the maximum of the date column
print(sales['date'].max())

# Print the minimum of the date column
print(sales['date'].min())
```

#### Efficient summaries

```py
# A custom IQR function
def iqr(column):
    return column.quantile(0.75) - column.quantile(0.25)
    
# Print IQR of the temperature_c column
print(sales['temperature_c'].agg(iqr))
```

```py
# A custom IQR function
def iqr(column):
    return column.quantile(0.75) - column.quantile(0.25)

# Update to print IQR of temperature_c, fuel_price_usd_per_l, & unemployment
print(sales[["temperature_c", 'fuel_price_usd_per_l', 'unemployment']].agg(iqr))
```

```py
# Import NumPy and create custom IQR function
import numpy as np
def iqr(column):
    return column.quantile(0.75) - column.quantile(0.25)

# Update to print IQR and median of temperature_c, fuel_price_usd_per_l, & unemployment
print(sales[["temperature_c", "fuel_price_usd_per_l", "unemployment"]].agg([iqr,np.median]))
```

#### Cumulative statistics

```py
# Sort sales_1_1 by date
sales_1_1 = sales_1_1.sort_values("date",ascending=True)

# Get the cumulative sum of weekly_sales, add as cum_weekly_sales col
sales_1_1['cum_weekly_sales'] = sales_1_1['weekly_sales'].cumsum()

# Get the cumulative max of weekly_sales, add as cum_max_sales col
sales_1_1['cum_max_sales'] = sales_1_1['weekly_sales'].cummax()

# See the columns you calculated
print(sales_1_1[["date", "weekly_sales", "cum_weekly_sales", "cum_max_sales"]])
```

### 2.2. Counting

Dropping duplicate names: `vet_visits.drop_duplicates(subset="name")`
Dropping duplicate pairs: `unique_dogs = vet_visits.drop_duplicates(subset=["name", "breed"])`
Easy as 1,2,3:
`unique_dogs["breed"].value_counts()`
`unique_dogs["breed"].value_counts(sort=True)`
Proportions: `unique_dogs["breed"].value_counts(normalize=True)`

#### Dropping duplicates

```py
# Drop duplicate store/type combinations
store_types = sales.drop_duplicates(subset=['store','type'])
print(store_types.head())

# Drop duplicate store/department combinations
store_depts = sales.drop_duplicates(subset=['store','department'])
print(store_depts.head())

# Subset the rows where is_holiday is True and drop duplicate dates
holiday_dates = sales[sales['is_holiday']].drop_duplicates('date')

# Print date col of holiday_dates
print(holiday_dates['date'])
```

#### Counting categorical variables

```py
# Count the number of stores of each type
store_counts = store_types['type'].value_counts()
print(store_counts)

# Get the proportion of stores of each type
store_props = store_types['type'].value_counts(normalize=True)
print(store_props)

# Count the number of each department number and sort
dept_counts_sorted = store_depts['department'].value_counts(sort=True)
print(dept_counts_sorted)

# Get the proportion of departments of each number and sort
dept_props_sorted = store_depts['department'].value_counts(sort=True, normalize=True)
print(dept_props_sorted)
```

### 2.3. Grouped summary statistics

Summaries by group: `dogs[dogs["color"] == "Black"]["weight_kg"].mean()`
Grouped summaries: `dogs.groupby("color")["weight_kg"].mean()`
Multiple grouped summaries: `dogs.groupby("color")["weight_kg"].agg([min, max, sum])`
Grouping by multiple variables: `dogs.groupby(["color", "breed"])["weight_kg"].mean()`
Many groups, many summaries: `dogs.groupby(["color", "breed"])[["weight_kg", "height_cm"]].mean()`

#### What percent of sales occurred at each store type?

```py
# Calc total weekly sales
sales_all = sales["weekly_sales"].sum()

# Subset for type A stores, calc total weekly sales
sales_A = sales[sales["type"] == "A"]["weekly_sales"].sum()

# Subset for type B stores, calc total weekly sales
sales_B = sales[sales["type"] == "B"]["weekly_sales"].sum()

# Subset for type C stores, calc total weekly sales
sales_C = sales[sales["type"] == "C"]["weekly_sales"].sum()

# Get proportion for each type
sales_propn_by_type = [sales_A, sales_B, sales_C] / sales_all
print(sales_propn_by_type)
```

#### Calculations with .groupby()

```py
# Group by type; calc total weekly sales
sales_by_type = sales.groupby("type")["weekly_sales"].sum()

# Get proportion for each type
sales_propn_by_type = sales_by_type / sum(sales_by_type)
print(sales_propn_by_type)
```

```py
# From previous step
sales_by_type = sales.groupby("type")["weekly_sales"].sum()

# Group by type and is_holiday; calc total weekly sales
sales_by_type_is_holiday = sales.groupby(["type","is_holiday"])["weekly_sales"].sum()
print(sales_by_type_is_holiday)
```

#### Multiple grouped summaries

```py
# Import numpy with the alias np
import numpy as np

# For each store type, aggregate weekly_sales: get min, max, mean, and median
sales_stats = sales.groupby("type")["weekly_sales"].agg([np.min,np.max,np.mean,np.median])

# Print sales_stats
print(sales_stats)

# For each store type, aggregate unemployment and fuel_price_usd_per_l: get min, max, mean, and median
unemp_fuel_stats = sales.groupby("type")[["unemployment","fuel_price_usd_per_l"]].agg([np.min,np.max,np.mean,np.median])

# Print unemp_fuel_stats
print(unemp_fuel_stats)
```

### 2.4. Pivot tables

Pivot tables are the standard way of aggregating data in spreadsheets. In pandas, pivot tables are essentially just another way of performing grouped calculations. That is, the .pivot_table() method is just an alternative to .groupby().

Group by to pivot table:
`dogs.groupby("color")["weight_kg"].mean()`
`dogs.pivot_table(values="weight_kg", index="color")`

Different statistics: `dogs.pivot_table(values="weight_kg", index="color", aggfunc=np.median)`
Multiple statistics: `dogs.pivot_table(values="weight_kg", index="color", aggfunc=[np.mean, np.median])`
Pivot on two variables: 
`dogs.groupby(["color", "breed"])["weight_kg"].mean()`
`dogs.pivot_table(values="weight_kg", index="color", columns="breed")`
Filling missing values in pivot tables: `dogs.pivot_table(values="weight_kg", index="color", columns="breed", fill_value=0)`
Summing with pivot tables: `dogs.pivot_table(values="weight_kg", index="color", columns="breed", fill_value=0, margins=True)`

#### Pivoting on one variable

```py
# Pivot for mean weekly_sales for each store type
mean_sales_by_type = sales.pivot_table(values="weekly_sales",index="type")

# Print mean_sales_by_type
print(mean_sales_by_type)
```

```py
# Import NumPy as np
import numpy as np

# Pivot for mean and median weekly_sales for each store type
mean_med_sales_by_type = sales.pivot_table(values="weekly_sales",index="type",aggfunc=[np.mean,np.median])

# Print mean_med_sales_by_type
print(mean_med_sales_by_type)
```

```py
# Pivot for mean weekly_sales by store type and holiday 
mean_sales_by_type_holiday = sales.pivot_table(values="weekly_sales",index="type",columns="is_holiday")

# Print mean_sales_by_type_holiday
print(mean_sales_by_type_holiday)
```

#### Fill in missing values and sum values with pivot tables

```py
# Print mean weekly_sales by department and type; fill missing values with 0
print(sales.pivot_table(values="weekly_sales",index="department",columns="type",fill_value=0))
```

```py
# Print the mean weekly_sales by department and type; fill missing values with 0s; sum all rows and cols
print(sales.pivot_table(values="weekly_sales", index="department", columns="type", fill_value=0, aggfunc=sum,margins=True))
```


## 3. Slicing and Indexing DataFrames

```py

```

## 4. Creating and Visualizing DataFrames


