# Part 11: Data Analytics

---

## Chapter 71: NumPy Fundamentals

### Introduction

NumPy (Numerical Python) is the foundational library for numerical computing in Python. It provides the **ndarray** (n-dimensional array) object — a homogeneous, contiguous block of memory that enables vectorized operations at C speed. Every major data science library (Pandas, Scikit-learn, TensorFlow, PyTorch) builds on NumPy arrays internally.

### Why It Matters

Python lists are flexible but slow for numerical operations because each element is a Python object with reference-counting overhead. NumPy arrays store data in contiguous memory (row-major by default), enabling SIMD vectorization, cache-efficient access, and near-C performance. Understanding NumPy is not optional — it is the substrate of all numerical Python.

### Real World Analogy

A Python list is like a list of paper slips scattered across different drawers — each slip must be retrieved individually. A NumPy array is like a ledger book with all numbers written in a single grid — you can run your finger down entire columns in one motion.

> 💡 **Pro Tip:** Vectorization is the single most important performance optimization in NumPy. A vectorized `a + b` runs at C speed because the loop happens in compiled C code, not in the Python interpreter. Whenever you see a `for` loop over array elements in Python, ask yourself: "Can I vectorize this?"

### Theory

**Pandas** provides the **Series** (1D labeled array) and **DataFrame** (2D labeled table) data structures. It wraps NumPy with row/column labels, heterogeneous column types, missing data handling, and a rich API for data manipulation. Pandas is the lingua franca of data analysis in Python.

> 💡 **Pro Tip:** Use `pd.read_csv()` with `nrows=100` and `usecols` to quickly inspect schema before loading large files. Then load the full dataset with appropriate `dtype` specifications to save memory. Memory can often be reduced by 50-70% by downcasting numeric columns and using categorical types for strings with low cardinality.

### Why It Matters

Raw data never arrives as clean matrices. You need labeled columns, mixed types (strings, floats, ints, dates), missing values, and time indices. Pandas handles all of this natively. Virtually every data pipeline — ETL, exploration, feature engineering, visualization — starts with a DataFrame.

### Real World Analogy

A NumPy array is a spreadsheet with only numbers. A DataFrame is a full Excel workbook: different column types, row labels, formulas, filters, pivot tables, and the ability to merge multiple sheets.

> 📖 **Instructor Note:** The distinction between `.loc` and `.iloc` is one of the most common sources of Pandas bugs. `.loc` uses label-based indexing — it includes the endpoint. `.iloc` uses positional indexing — it excludes the endpoint like Python slicing. Always test small examples to verify your indexing logic before applying to large datasets.

### Theory

**Label-based vs Position-based Indexing**: `.loc` uses the index label (even if it's an integer), `.iloc` uses the integer position (0, 1, 2, ...). This distinction is critical when the index is non-integer or non-unique.

**Alignment**: When you add two DataFrames, they align by index (rows) and columns. Missing intersections produce NaN. This is a core Pandas design principle — data always aligns on labels, not position.

**Internal Working**: A DataFrame is a dict of Series (each column is a Series). Each Series contains a NumPy array (values) and an Index object (labels). The Index is hash-mapped for O(1) label lookup.

### Code Examples

```python
import pandas as pd
import numpy as np

# === SERIES ===
s1 = pd.Series([10, 20, 30, 40], name='scores')
s2 = pd.Series({'a': 1, 'b': 2, 'c': 3})
s3 = pd.Series(5, index=range(4))
s4 = pd.Series(np.random.randn(5))
print(s1.index, s1.values, s1.name)

# === DATAFRAME CREATION ===
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [70000, 80000, 90000]
})

df2 = pd.DataFrame([
    {'city': 'NYC', 'temp': 72},
    {'city': 'LA', 'temp': 85},
])

df3 = pd.DataFrame(np.random.randn(100, 4), columns=list('ABCD'))

# === INSPECTION ===
print(df.head(2))
print(df.tail(2))
print(df.info())
print(df.describe())
print(df.dtypes)
print(df.shape)
print(df.columns)
print(df.index)

# === INDEX TYPES ===
idx_range = pd.RangeIndex(0, 100, 1)
idx_int = pd.Int64Index([1, 2, 3])
idx_date = pd.DatetimeIndex(pd.date_range('2024-01-01', periods=5))
idx_cat = pd.CategoricalIndex(['a', 'b', 'c', 'a', 'b'])

# MultiIndex
arrays = [['A', 'A', 'B', 'B'], [1, 2, 1, 2]]
midx = pd.MultiIndex.from_arrays(arrays, names=('group', 'id'))
df_multi = pd.DataFrame(np.random.randn(4, 3), index=midx)

df = df.set_index('name')
df = df.reset_index()

# === SELECTION ===
print(df['age'])
print(df[['age', 'salary']])
print(df.loc['Alice'])
print(df.iloc[0])
print(df.at['Alice', 'age'])
print(df.iat[0, 0])
print(df_multi.xs('A'))
print(df.query('age > 25 and salary > 75000'))

# === BOOLEAN INDEXING ===
print(df[df['age'] > 25])
print(df[(df['age'] > 25) & (df['salary'] > 75000)])
print(df[(df['age'] < 30) | (df['name'] == 'Charlie')])
print(df[~df['name'].isin(['Bob'])])
print(df[df['age'].between(25, 30)])

# === SORTING ===
print(df.sort_values('salary', ascending=False))
print(df.sort_values(['age', 'salary'], ascending=[True, False]))

# === MISSING DATA ===
data = pd.DataFrame({'A': [1, np.nan, 3], 'B': [np.nan, np.nan, 6]})
print(data.isnull())
print(data.notnull())
print(data.dropna())
print(data.dropna(axis='columns'))
print(data.dropna(how='all'))
print(data.dropna(thresh=2))
print(data.fillna(0))
print(data.fillna(method='ffill'))
print(data.fillna(method='bfill', limit=1))
print(data.interpolate(method='linear'))

# === DUPLICATES ===
data2 = pd.DataFrame({'id': [1, 1, 2, 2, 3], 'val': [10, 10, 20, 25, 30]})
print(data2.duplicated())
print(data2.drop_duplicates())
print(data2.drop_duplicates(keep='last'))
print(data2.drop_duplicates(keep=False))

# === APPLYING FUNCTIONS ===
df = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})
print(df.apply(np.sum, axis=0))
print(df.apply(np.sum, axis=1))
print(df.applymap(lambda x: x * 2))
print(df['A'].map(lambda x: x ** 2))

# === VECTORIZED STRING OPERATIONS ===
str_data = pd.Series(['hello world', 'foo bar', 'hello foo'])
print(str_data.str.upper())
print(str_data.str.split())
print(str_data.str.replace('hello', 'hi'))
print(str_data.str.contains('foo'))
print(str_data.str.extract(r'(\w+)\s(\w+)'))
print(str_data.str.strip())

# === VECTORIZED DATETIME ===
dates = pd.to_datetime(['2024-01-01', '2024-02-15', '2024-03-20'])
s_dates = pd.Series(dates)
print(s_dates.dt.year)
print(s_dates.dt.month)
print(s_dates.dt.day)
print(s_dates.dt.weekday)
print(s_dates.dt.quarter)

print(pd.date_range('2024-01-01', periods=5, freq='M'))
print(pd.bdate_range('2024-01-01', periods=5))

delta = pd.Timedelta('3 days 4 hours')
print(delta.days, delta.seconds)

ts = pd.Series(np.random.randn(100), index=pd.date_range('2024-01-01', periods=100, freq='h'))
print(ts.resample('D').mean())

# === CATEGORICAL DATA ===
cat_data = pd.Categorical(['small', 'large', 'medium', 'small'],
                          categories=['small', 'medium', 'large'],
                          ordered=True)
s_cat = pd.Series(cat_data)
print(s_cat.cat.categories)
print(s_cat.cat.codes)
print(pd.cut(np.random.randn(100), bins=4))
print(pd.qcut(np.random.randn(100), q=4))

# === MEMORY OPTIMIZATION ===
df_int = df.select_dtypes(include=['int'])
for col in df_int.columns:
    df[col] = pd.to_numeric(df[col], downcast='integer')
df_float = df.select_dtypes(include=['float'])
for col in df_float.columns:
    df[col] = pd.to_numeric(df[col], downcast='float')

for col in df.select_dtypes(include=['object']).columns:
    if df[col].nunique() / len(df[col]) < 0.5:
        df[col] = df[col].astype('category')

print(f'Memory: {df.memory_usage(deep=True).sum() / 1024**2:.2f} MB')
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Chained assignment `df['a'][0]=5` | May modify copy, not original | Use `df.loc[0, 'a'] = 5` |
| `pd.read_csv` without `parse_dates` | Dates stored as strings | `parse_dates=['col']` |
| Forgetting `axis` in `dropna` | Drops rows instead of columns | `df.dropna(axis='columns')` |
| Using `==` to compare floats | Floating-point precision issues | Use `np.isclose` |
| Not specifying `dtype` on read | Memory bloat (float64 for ints) | `dtype={'col': 'int32'}` |
| `inplace=True` everywhere | Deprecated, confusing | `df = df.operation()` |

### Best Practices

1. Always specify `dtype` when reading large CSVs to save memory
2. Use `usecols` in `read_csv` to only load needed columns
3. Chain operations with `.pipe()` for readability
4. Prefer `pd.to_numeric` with `downcast` for memory efficiency
5. Use `copy()` explicitly when you need an independent DataFrame
6. Set `low_memory=False` in `read_csv` when dtypes are inconsistent
7. Use `pd.concat` with `keys` to track source when stacking DataFrames

### Interview Questions

1. Explain the difference between `.loc` and `.iloc` with a non-integer index
2. How does Pandas handle alignment in arithmetic operations?
3. What is the difference between `apply` on axis=0 vs axis=1?
4. How would you detect and handle memory issues in a 100 GB CSV?
5. Explain how `fillna(method='ffill')` works under the hood

### Practical Exercises

1. Load a CSV with mixed types, inspect memory usage, and downcast everything
2. Create a MultiIndex DataFrame and select data using `.xs` and `.loc`
3. Given a DataFrame with missing values, compare `dropna`, `fillna` (mean/median), and `interpolate`
4. Convert a messy date column (multiple formats) into a clean DatetimeIndex

### Mini Project

**Data Quality Report Generator**: Write a function or class that takes any DataFrame and generates a report containing: column name, dtype, non-null count, unique count, min/max, missing percentage, memory usage, and a flag for high-cardinality categoricals.

### Revision Notes

| Concept | Key Tool | Syntax Example |
|---------|---------|----------------|
| Series creation | `pd.Series()` | `pd.Series([1,2,3], name='x')` |
| DataFrame creation | `pd.DataFrame()` | `pd.DataFrame({'a': [1,2]})` |
| Label selection | `.loc` | `df.loc[0:5, ['a','b']]` |
| Position selection | `.iloc` | `df.iloc[:, 0:2]` |
| Missing data | `dropna, fillna` | `df.fillna(method='ffill')` |
| Strings | `.str` accessor | `df['col'].str.lower()` |
| Dates | `.dt` accessor | `df['date'].dt.year` |
| Categoricals | `pd.Categorical` | `df['col'].astype('category')` |

---

## Chapter 73: Pandas — Aggregation and Grouping

### Introduction

The `groupby` operation (split-apply-combine) is the most powerful tool in Pandas for exploratory analysis. It partitions data into groups, applies a function to each, and combines results. Combined with `pivot_table`, `crosstab`, and `melt`, it covers 90% of analytical aggregation needs.

### Why It Matters

Raw data is transactional: one row per event. Analysis requires summaries by category (average sales by region, total revenue by month, count of users by country). `groupby` is the engine for this transformation. Pivot tables and cross-tabulations are simply different views of the same grouped data.

### Real World Analogy

You have a deck of playing cards. `groupby` means separate them into suits, then compute something (e.g., average value) for each suit. `pivot_table` then arranges those results back into a 2D grid with suits as rows and something else as columns.

> ✅ **Best Practice:** Use `transform` instead of `apply` when you need to preserve the original DataFrame shape. `transform` returns a result with the same index as the original, making it ideal for adding group-wise statistics (e.g., group means, ranks) as new columns without collapsing rows.

### Theory

**Split-Apply-Combine**:
1. **Split**: A DataFrame is split into groups based on key(s)
2. **Apply**: A function is applied to each group independently
3. **Combine**: Results are assembled into a new DataFrame or Series

**Internal**: `groupby` creates a `GroupBy` object that lazily stores the splitting plan. It doesn't compute until you call `agg`, `transform`, `filter`, or `apply`. The actual split uses hash-based grouping — O(n) time.

### Code Examples

```python
import pandas as pd
import numpy as np

df = pd.DataFrame({
    'department': ['IT', 'HR', 'IT', 'HR', 'IT', 'HR', 'Fin'],
    'team': ['A', 'A', 'B', 'B', 'A', 'A', 'A'],
    'salary': [70000, 60000, 80000, 65000, 90000, 55000, 72000],
    'bonus': [5000, 3000, 4000, 2000, 7000, 1000, 3500],
    'years': [2, 3, 5, 1, 7, 4, 6]
})

# === GROUPBY BASICS ===
grouped = df.groupby('department')
print(grouped['salary'].mean())
print(grouped[['salary', 'bonus']].mean())

grouped2 = df.groupby(['department', 'team'])
print(grouped2['salary'].mean())

for name, group in df.groupby('department'):
    print(f'{name}: {group.shape[0]} rows')

# === AGGREGATION ===
print(df.groupby('department')['salary'].agg('sum'))

print(df.groupby('department').agg({
    'salary': ['mean', 'std'],
    'bonus': 'sum',
    'years': 'max'
}))

print(df.groupby('department').agg(
    avg_salary=('salary', 'mean'),
    total_bonus=('bonus', 'sum'),
    max_years=('years', 'max')
))

print(df.groupby('department')['salary'].agg(lambda x: x.max() - x.min()))

# === TRANSFORM ===
df['salary_rank_in_dept'] = df.groupby('department')['salary'].transform('rank')
df['pct_of_dept_salary'] = df.groupby('department')['salary'].transform(
    lambda x: x / x.sum()
)
df['dept_mean_salary'] = df.groupby('department')['salary'].transform('mean')

# === FILTER ===
print(df.groupby('department').filter(lambda g: g['salary'].mean() > 65000))

# === PIVOT TABLES ===
pivot = pd.pivot_table(
    df,
    values='salary',
    index='department',
    columns='team',
    aggfunc='mean',
    fill_value=0,
    margins=True
)

pivot2 = pd.pivot_table(
    df,
    values=['salary', 'bonus'],
    index='department',
    columns='team',
    aggfunc={'salary': 'mean', 'bonus': 'sum'},
    margins=True
)

# === CROSS-TABULATION ===
print(pd.crosstab(df['department'], df['team']))

print(pd.crosstab(
    df['department'],
    df['team'],
    normalize='index',
    margins=True
))

print(pd.crosstab(
    df['department'],
    df['team'],
    values=df['salary'],
    aggfunc='mean'
))

# === MELTING ===
wide = pd.DataFrame({
    'id': [1, 2, 3],
    'Q1': [100, 200, 150],
    'Q2': [110, 190, 160],
    'Q3': [120, 180, 170],
})

long = pd.melt(
    wide,
    id_vars=['id'],
    value_vars=['Q1', 'Q2', 'Q3'],
    var_name='quarter',
    value_name='revenue'
)

# === STACK / UNSTACK ===
stacked = pivot.stack()
unstacked = stacked.unstack()
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Applying `mean()` on string columns | Error or silent type coercion | Select numeric columns first |
| Forgetting `reset_index()` after groupby | Result has MultiIndex | `df.groupby().agg().reset_index()` |
| Using `apply` when `transform` is needed | Wrong shape / group membership lost | Use `transform` for same-length results |
| `agg` with list of functions flattens columns | MultiIndex column headers | Rename with named aggregation |
| Not handling NaN in group keys | Groups dropped silently | Use `dropna=False` in `groupby()` |

### Best Practices

1. Use named aggregation (`.agg(name=(col, func))`) for clean column names
2. Chain `.reset_index()` or use `as_index=False` in `groupby`
3. Prefer `transform` over `apply` for group-wise normalization
4. Use `observed=True` in `groupby` with categoricals to avoid empty groups
5. Set `margins=True` in pivot tables for sub-totals

### Interview Questions

1. Explain split-apply-combine with a concrete example
2. How does `transform` differ from `agg` in terms of output shape?
3. What is the difference between `melt` and `stack`?
4. How would you compute the percentage of total per group for each row?
5. When would you use `pivot_table` vs `groupby` + `unstack`?

### Practical Exercises

1. From an e-commerce dataset, compute monthly revenue per customer segment
2. Create a pivot table showing average order value by region and quarter
3. Use `transform` to normalize each product's sales within its category
4. Convert a wide DataFrame of test scores (student, test1, test2, test3) to long format

### Mini Project

**Sales Report Pipeline**: Take a transactional sales CSV (date, region, product, quantity, price), produce: daily revenue by region (pivot table), month-over-month growth rate per product (groupby + transform + pct_change), a melted DataFrame suitable for plotting with seaborn, and top 10% of products by revenue contribution.

### Revision Notes

| Concept | Key Function | Syntax |
|---------|-------------|--------|
| GroupBy | `df.groupby()` | `df.groupby('col')['val'].mean()` |
| Aggregation | `.agg()` | `.agg({'col': ['sum', 'mean']})` |
| Transform | `.transform()` | `.transform('mean')` |
| Filter | `.filter()` | `.filter(lambda g: g.sum() > 10)` |
| Pivot Table | `pd.pivot_table` | `pd.pivot_table(df, values=..., index=..., columns=...)` |
| Crosstab | `pd.crosstab` | `pd.crosstab(df['a'], df['b'])` |
| Melt | `pd.melt` | `pd.melt(df, id_vars=['id'])` |
| Stack | `.stack()` | `df.stack()` |

---

## Chapter 74: Pandas — Merging, Joining, Concatenating

### Introduction

Real-world analysis requires combining data from multiple sources. Pandas provides three families of operations: `concat` (stacking), `merge` (SQL-style joins), and `join` (convenience for index-based joins). Understanding the differences and when to use each is critical for building data pipelines.

### Why It Matters

Data is rarely in a single table. You have customers in one file, orders in another, and payments in a third. Relational joins are the backbone of data integration. Pandas merge/join/concat are direct analogs of SQL JOIN and UNION operations.

### Real World Analogy

Concatenation is like stacking two decks of cards (adding rows) or taping two tables side by side (adding columns). Merging is like finding matching entries in two ledgers: match on a key (customer ID, order number) and bring the relevant info together.

> ⚠️ **Warning:** Merge with duplicate keys on both sides produces a Cartesian product, which can explode your DataFrame size unexpectedly. Always use the `validate` parameter to check relationships: `pd.merge(a, b, on='key', validate='many_to_one')`. This catches data quality issues before they impact downstream processing.

### Theory

**Concatenation**: `pd.concat` stacks DataFrames along an axis. It doesn't align on keys — it simply glues them together. You can pass `keys` to create a hierarchical index identifying the source.

**Merge**: `pd.merge` performs relational joins (inner, left, right, outer, cross) on columns or indices. It uses hash-based matching and handles one-to-one, one-to-many, many-to-one, and many-to-many relationships.

**Join**: `df.join` is a shorthand for merge on the index. It's optimized for index-based lookups.

### Code Examples

```python
import pandas as pd
import numpy as np

# === CONCAT ===
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})
df3 = pd.DataFrame({'C': [9, 10], 'D': [11, 12]})

stacked = pd.concat([df1, df2], axis=0)
print(stacked)

stacked_keys = pd.concat([df1, df2], keys=['source1', 'source2'])
print(stacked_keys)

side_by_side = pd.concat([df1, df3], axis=1)
print(side_by_side)

df4 = pd.DataFrame({'A': [1, 2], 'E': [3, 4]})
inner_concat = pd.concat([df1, df4], axis=0, join='inner')
print(inner_concat)

df_mismatch = pd.concat([df1, df2], ignore_index=True)
print(df_mismatch)

# === MERGE ===
customers = pd.DataFrame({
    'customer_id': [1, 2, 3, 4],
    'name': ['Alice', 'Bob', 'Charlie', 'David'],
    'city': ['NYC', 'LA', 'SF', 'NYC']
})

orders = pd.DataFrame({
    'order_id': [101, 102, 103, 104, 105],
    'customer_id': [1, 1, 2, 3, 5],
    'amount': [100, 200, 150, 300, 250]
})

inner = pd.merge(customers, orders, on='customer_id', how='inner')
print(inner)

left = pd.merge(customers, orders, on='customer_id', how='left')
print(left)

right = pd.merge(customers, orders, on='customer_id', how='right')
print(right)

outer = pd.merge(customers, orders, on='customer_id', how='outer')
print(outer)

cross = pd.merge(customers, orders, how='cross')
print(cross.shape)

orders2 = orders.rename(columns={'customer_id': 'cust_id'})
merged_diff = pd.merge(customers, orders2, left_on='customer_id', right_on='cust_id', how='inner')
print(merged_diff)

customers2 = customers.copy()
customers2['amount'] = [0, 0, 0, 0]
merged_suffix = pd.merge(customers2, orders, on='customer_id', suffixes=('_cust', '_ord'))
print(merged_suffix.columns)

merged_ind = pd.merge(customers, orders, on='customer_id', how='outer', indicator=True)
print(merged_ind['_merge'].value_counts())

customers_indexed = customers.set_index('customer_id')
orders_indexed = orders.set_index('customer_id')
merged_idx = pd.merge(customers_indexed, orders_indexed, left_index=True, right_index=True, how='left')

# === MERGE_ASOF ===
trades = pd.DataFrame({
    'time': pd.to_datetime(['2024-01-01 09:30:01', '2024-01-01 09:30:05']),
    'price': [100.1, 100.3],
    'qty': [100, 200]
})

quotes = pd.DataFrame({
    'time': pd.to_datetime(['2024-01-01 09:30:00', '2024-01-01 09:30:02', '2024-01-01 09:30:04']),
    'bid': [99.9, 100.0, 100.2],
    'ask': [100.2, 100.3, 100.5]
})

asof = pd.merge_asof(trades, quotes, on='time', direction='backward')
print(asof)

# === JOIN ===
df_left = pd.DataFrame({'val': [1, 2, 3]}, index=['a', 'b', 'c'])
df_right = pd.DataFrame(['x', 'y', 'z'], index=['a', 'b', 'd'], columns=['letter'])
joined = df_left.join(df_right, how='left')
print(joined)

joined_inner = df_left.join(df_right, how='inner')
print(joined_inner)

df_extra = pd.DataFrame({'extra': [10, 20, 30]}, index=['a', 'b', 'c'])
joined_multi = df_left.join([df_right, df_extra])
print(joined_multi)
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `pd.concat` default axis=0 when you want axis=1 | Stacks instead of side-by-side | Explicit `axis=1` |
| Merge with duplicate keys on both sides | Result is Cartesian product | Deduplicate first or use `validate` |
| Forgetting `suffixes` | Column name conflicts | `suffixes=('_x', '_y')` |
| `how='outer'` when you meant `how='left'` | Extra rows from unexpected matches | Check expected key sets |
| Not specifying `on` when multiple common columns | Merge uses intersection of all common cols | Specify `on` explicitly |
| Using `join` when you need column-based merge | Only works on index | Use `pd.merge` with `on` |

### Best Practices

1. Always `validate` relationships (one_to_one, one_to_many, many_to_one, many_to_many)
2. Use `indicator=True` in merges to debug row provenance
3. Set index on join keys before `join` for faster index-based merges
4. Deduplicate keys before merge if you expect unique matches
5. Use `merge_asof` for time-series joins instead of complex window-based filtering
6. Prefer `pd.concat` with `keys` when stacking multiple files to preserve source info

### Interview Questions

1. What is the difference between `merge` and `join` in Pandas?
2. How does `merge` handle duplicate keys on both sides?
3. When would you use `merge_asof` instead of a regular merge?
4. Explain the behavior of `pd.concat` with `axis=0` and `join='inner'`
5. How would you perform a right anti-join in Pandas?

### Practical Exercises

1. Given a customers table and an orders table, produce a report of all customers who have never placed an order
2. Merge stock trades with the most recent quote (use `merge_asof`)
3. Stack 12 monthly CSV files into a single DataFrame, preserving source month
4. Join three DataFrames (users, purchases, products) into a single denormalized table

### Mini Project

**Relational Data Pipeline**: Create three DataFrames — customers (10K rows), orders (50K rows), and order_items (200K rows) with realistic distributions (some customers have many orders, some have zero). Write a pipeline that: merges orders to customers (left join), filters to only active customers, aggregates total spend per customer, joins back the order details for the top 10% of customers, and outputs the final DataFrame with row counts tracked at each stage.

### Revision Notes

| Concept | Key Function | Syntax |
|---------|-------------|--------|
| Row stack | `pd.concat(axis=0)` | `pd.concat([a, b])` |
| Column stack | `pd.concat(axis=1)` | `pd.concat([a, b], axis=1)` |
| Inner join | `pd.merge(how='inner')` | `pd.merge(left, right, on='key')` |
| Left join | `pd.merge(how='left')` | `pd.merge(left, right, on='key', how='left')` |
| Outer join | `pd.merge(how='outer')` | `pd.merge(left, right, on='key', how='outer')` |
| Nearest merge | `pd.merge_asof` | `pd.merge_asof(a, b, on='time')` |
| Index join | `df.join` | `df_left.join(df_right)` |

---

## Chapter 75: Polars (Modern Alternative)

### Introduction

Polars is a DataFrame library written in Rust with a Python binding, designed for speed and memory efficiency on datasets larger than RAM. It uses Apache Arrow as its memory format, supports lazy evaluation, and has a clean expression-based API. It is the most serious competitor to Pandas in the Python ecosystem.

### Why It Matters

Pandas struggles with datasets larger than RAM (spills to disk are painful), has a complex internals (reference chains), and is single-threaded for many operations. Polars is multi-threaded by default, lazy-evaluated (query optimization), and can process datasets larger than memory via streaming. For modern data workloads, Polars is increasingly the better choice.

### Real World Analogy

Pandas is like a manual transmission car — powerful but requires careful handling. Polars is like an electric car with instant torque and regenerative braking — faster, more efficient, and smarter about energy (memory).

> 💡 **Pro Tip:** Polars lazy mode is where it truly shines. By building a query plan and optimizing it before execution (predicate pushdown, projection pushdown), Polars can process datasets larger than RAM efficiently. Always use the lazy API (`scan_csv`, `scan_parquet`) for production data pipelines.

### Theory

**Eager vs Lazy Execution**:
- **Eager**: Each operation executes immediately. You can inspect results at every step.
- **Lazy**: Build a query plan and execute only when `.collect()` is called. Polars can optimize the plan (predicate pushdown, projection pushdown, join reordering).

**Arrow Memory Format**: Data is stored in columnar format using Apache Arrow, which enables zero-copy sharing between libraries (Pandas, DuckDB, Arrow-native tools).

**Expressions**: Instead of method chaining on DataFrames, Polars uses expressions (`pl.col('col').mean().alias('avg')`) that are composable and reusable.

### Code Examples

```python
import polars as pl
import pandas as pd

# === CREATION ===
df = pl.DataFrame({
    'name': ['Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 35],
    'salary': [70000, 80000, 90000]
})

# From Pandas
pdf = pd.DataFrame({'a': [1, 2, 3]})
pdf_polars = pl.from_pandas(pdf)

# === SERIES ===
s = pl.Series('values', [1, 2, 3, 4, 5])
print(s)

# === BASIC OPERATIONS ===
print(df.describe())
print(df.schema)
print(df.shape)
print(df.head(2))

# === EXPRESSIONS ===
result = df.select(
    pl.col('name'),
    pl.col('age'),
    pl.col('salary').alias('income')
)
print(result)

result = df.with_columns(
    (pl.col('salary') / 12).alias('monthly_salary'),
    (pl.col('age') - 20).alias('years_after_20')
)

result = df.filter(pl.col('age') > 28)
result = df.filter((pl.col('age') > 25) & (pl.col('salary') > 75000))

result = df.group_by('name').agg([
    pl.col('salary').mean().alias('avg_salary'),
    pl.col('age').max().alias('max_age')
])

# === LAZY FRAME ===
lf = (
    pl.scan_csv('data.csv')
    .filter(pl.col('date') > pl.lit('2024-01-01'))
    .group_by('region')
    .agg(pl.col('revenue').sum().alias('total_revenue'))
    .sort('total_revenue', descending=True)
)

df_result = lf.collect()
print(lf.explain())

# === JOINS ===
customers = pl.DataFrame({'id': [1, 2, 3], 'name': ['A', 'B', 'C']})
orders = pl.DataFrame({'cust_id': [1, 1, 2], 'amount': [100, 200, 150]})

joined = customers.join(orders, left_on='id', right_on='cust_id', how='inner')
print(joined)

# === COMPLEX PIPELINE ===
result = (
    df.lazy()
    .filter(pl.col('age') > 25)
    .with_columns(
        (pl.col('salary') / pl.col('salary').mean()).alias('salary_ratio')
    )
    .group_by('name')
    .agg([
        pl.col('salary').mean(),
        pl.col('salary_ratio').last()
    ])
    .sort('salary_mean', descending=True)
    .collect()
)
print(result)

# === WINDOW FUNCTIONS ===
df.with_columns(
    pl.col('salary').mean().over('name').alias('avg_salary_by_name')
)

df.select(pl.all().mean())
df.select(pl.exclude('name'))
```

### Performance Comparison

| Operation | Pandas (10M rows) | Polars (10M rows) | Dask (10M rows) |
|-----------|-------------------|-------------------|-----------------|
| GroupBy sum | ~2.5s | ~0.3s | ~2.1s (4 workers) |
| Filter | ~1.2s | ~0.08s | ~1.0s |
| Sort | ~4.0s | ~0.9s | ~3.5s |
| Join (inner) | ~6.0s | ~0.7s | ~5.0s |
| CSV read + parse | ~8.0s | ~0.8s | ~3.0s (parallel chunks) |
| Memory usage | ~800 MB | ~350 MB | ~900 MB |

### When to Use Polars vs Pandas

| Criteria | Choose Pandas | Choose Polars |
|----------|--------------|---------------|
| Dataset size | < 2 GB (in memory) | 2 GB — 100 GB+ (streaming) |
| Ecosystem compatibility | Need scikit-learn, statsmodels | Standalone analysis, DuckDB pipeline |
| Team familiarity | Everyone knows Pandas | Willing to learn new API |
| Single-threaded guarantees | Debugging, teaching | Production pipelines |
| Complex groupby/transform | Mature, well-understood | Faster, expression-based |
| Time series | Excellent (resample, date_range) | Good (growing support) |

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Calling `.collect()` too early | Loses optimization opportunity | Keep lazy as long as possible |
| Using Pandas-style chained operations | Not idiomatic Polars | Use expression-based approach |
| Not using `streaming=True` for large data | Out of memory error | Add `streaming=True` to `.collect()` |
| Assuming Pandas datetime behavior | Different API | Use `pl.date_range`, `pl.dt` namespace |
| `.groupby` vs `.group_by` | Deprecated syntax | Use `group_by` (v0.19+) |

### Best Practices

1. Stay lazy as long as possible — build entire query before `.collect()`
2. Use `.explain()` to inspect and optimize query plans
3. Use `streaming=True` for datasets > available RAM
4. Write reusable expressions as variables: `expr = pl.col('x').mean().alias('avg_x')`
5. Pre-define schema with `pl.scan_csv(schema_overrides=...)` to avoid type inference
6. Use `pl.lit` for scalar values in expressions

### Interview Questions

1. What is the difference between eager and lazy execution in Polars?
2. How does Polars achieve better performance than Pandas?
3. Explain the streaming mode in Polars
4. What is an expression in Polars and how is it different from Pandas method chaining?
5. When would you choose Polars over Pandas for a production pipeline?

### Practical Exercises

1. Load a 5 GB CSV with Polars lazy API, apply filters and aggregations, and collect
2. Recreate a complex Pandas groupby + transform pipeline in Polars
3. Compare memory usage of Pandas vs Polars on the same dataset
4. Build a multi-step ETL pipeline entirely in Polars lazy mode

### Mini Project

**ETL Pipeline with Polars**: Build a pipeline that reads 50 Parquet files (simulated), filters recent records, joins with a dimension table, computes aggregations, and writes the result to a single Parquet file. Use lazy evaluation and streaming. Compare runtime with an equivalent Pandas pipeline.

### Revision Notes

| Concept | Polars | Pandas Equivalent |
|---------|--------|-------------------|
| DataFrame | `pl.DataFrame` | `pd.DataFrame` |
| Lazy | `pl.scan_csv().collect()` | N/A |
| Select | `df.select(pl.col('a'))` | `df[['a']]` |
| Filter | `df.filter(pl.col('a') > 0)` | `df[df['a'] > 0]` |
| Add column | `df.with_columns(...)` | `df.assign(...)` |
| GroupBy | `df.group_by('a').agg(...)` | `df.groupby('a').agg(...)` |
| Join | `df.join(other, on='key')` | `pd.merge(df, other, on='key')` |
| Custom function | `df.with_columns(pl.col('a').map_elements(f))` | `df['a'].apply(f)` |

---


## Chapter 76: Data Visualization — Matplotlib

### Introduction

Matplotlib is the foundational visualization library in Python. Every other library (Seaborn, Plotly, Pandas built-in plotting) is built on top of Matplotlib's rendering engine. Understanding Matplotlib's architecture — Figure, Axes, Artists — gives you full control over every visual element.

### Why It Matters

Seaborn handles 80% of statistical plots. Plotly handles interactive. But for technical publications, custom layouts, intricate annotations, and pixel-perfect control, you need Matplotlib. It is to visualization what NumPy is to numerical computing.

### Real World Analogy

Matplotlib's Figure is like a canvas (piece of paper), Axes are the individual charts drawn on that canvas, and Artists are every element you draw — lines, points, text, legends. The pyplot interface is like having a robot that works on "the default canvas," while the OOP API is like painting each chart yourself with full control.

> ✅ **Best Practice:** Always use the OOP API (`fig, ax = plt.subplots()`; `ax.plot()`) instead of pyplot state-machine (`plt.plot()`). The OOP API is explicit about which axes you're plotting on, essential for multi-subplot figures, and avoids the confusing "current axes" state that causes bugs in complex scripts.

### Theory

**Architecture hierarchy**:
- `Figure`: The top-level container (the entire image or page)
- `Axes`: An individual plot within the figure (has x-axis, y-axis, data area)
- `Artist`: Everything visible on the plot (lines, text, patches, collections)
- `Spines`: The borders of the axes
- `Ticks`: Markers along the axes
- `Axis`: The scale and labels along an edge

**pyplot vs OOP**: `plt.plot()` operates on "the current axes" (gca). It's simpler for quick plots but doesn't scale to multiple subplots. The OOP API (`fig, ax = plt.subplots()`; `ax.plot()`) is explicit and necessary for complex layouts.

### Code Examples

```python
import matplotlib.pyplot as plt
import numpy as np

# === PYPLOT BASICS ===
x = np.linspace(0, 10, 100)
plt.plot(x, np.sin(x), label='sin(x)')
plt.plot(x, np.cos(x), label='cos(x)')
plt.xlabel('x')
plt.ylabel('y')
plt.title('Trig Functions')
plt.legend()
plt.grid(True)
plt.show()

# === OOP API ===
fig, ax = plt.subplots(figsize=(8, 5))
ax.plot(x, np.sin(x), label='sin')
ax.plot(x, np.cos(x), label='cos')
ax.set_xlabel('x', fontsize=12)
ax.set_ylabel('y', fontsize=12)
ax.set_title('Trig Functions (OOP)', fontsize=14)
ax.legend()
ax.grid(alpha=0.3)
plt.tight_layout()
plt.show()

# === BASIC PLOT TYPES ===
fig, axes = plt.subplots(2, 3, figsize=(15, 8))
x = np.linspace(0, 10, 100)
axes[0, 0].plot(x, np.sin(x))
axes[0, 0].set_title('Line Plot')

n = 200
x_scat = np.random.randn(n)
y_scat = np.random.randn(n)
colors = np.random.rand(n)
sizes = np.random.randint(20, 200, n)
sc = axes[0, 1].scatter(x_scat, y_scat, c=colors, s=sizes, alpha=0.6)
axes[0, 1].set_title('Scatter Plot')
plt.colorbar(sc, ax=axes[0, 1])

categories = ['A', 'B', 'C', 'D']
values = [3, 7, 2, 5]
axes[0, 2].bar(categories, values, color=['red', 'blue', 'green', 'orange'])
axes[0, 2].set_title('Bar Chart')

axes[1, 0].barh(categories, values)
axes[1, 0].set_title('Horizontal Bar')

data = np.random.randn(1000)
axes[1, 1].hist(data, bins=30, density=True, alpha=0.7, edgecolor='black')
axes[1, 1].set_title('Histogram')

data_list = [np.random.randn(100) for _ in range(4)]
bp = axes[1, 2].boxplot(data_list, patch_artist=True)
axes[1, 2].set_title('Box Plot')

plt.tight_layout()
plt.show()

# === CUSTOMIZATION ===
fig, ax = plt.subplots(figsize=(10, 6), dpi=150)
ax.plot(x, np.sin(x), color='#FF5733', linewidth=2.5, linestyle='--',
        marker='o', markersize=4, markerfacecolor='white')
ax.set_xlim(0, 10)
ax.set_ylim(-1.5, 1.5)
ax.set_xticks(np.arange(0, 11, 2))
ax.set_yticks([-1, -0.5, 0, 0.5, 1])
ax.tick_params(axis='both', labelsize=10, direction='in', length=5)
ax.spines['top'].set_visible(False)
ax.spines['right'].set_visible(False)
ax.grid(alpha=0.3, linestyle=':')
ax.set_title('Customized Plot', fontweight='bold', pad=15)

# === COLORS ===
fig, ax = plt.subplots()
x = np.linspace(0, 10, 20)
for i in range(5):
    ax.plot(x, np.sin(x + i * 0.5) * (5 - i), label=f'Line {i}',
            color=plt.cm.viridis(i / 4))
ax.legend()
plt.show()

# === SUBPLOTS ===
fig, axes = plt.subplots(2, 2, figsize=(10, 8), sharex=True, sharey=True)

gs = plt.GridSpec(3, 3, figure=plt.figure(figsize=(10, 8)))
ax1 = plt.subplot(gs[0, :])
ax2 = plt.subplot(gs[1, 0])
ax3 = plt.subplot(gs[1, 1:])
ax4 = plt.subplot(gs[2, :])

fig, ax = plt.subplots()
ax.plot(x, np.sin(x))
inset_ax = fig.add_axes([0.6, 0.6, 0.3, 0.25])
inset_ax.plot(x[:20], np.sin(x[:20]), color='red')
inset_ax.set_title('Zoomed')
plt.show()

# === ANNOTATIONS ===
fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 100)
ax.plot(x, np.sin(x))
ax.annotate('Local Maximum', xy=(np.pi/2, 1), xytext=(np.pi/2 + 1.5, 0.5),
            arrowprops=dict(arrowstyle='->', color='black', lw=1.5),
            fontsize=12, bbox=dict(boxstyle='round', facecolor='wheat', alpha=0.5))

ax.text(2, -0.5, 'Text box', fontsize=10, style='italic',
        bbox={'facecolor': 'lightblue', 'alpha': 0.5, 'pad': 5})
ax.arrow(2, 0.5, 0.5, 0.3, head_width=0.1, head_length=0.1, fc='red', ec='red')
ax.axhline(0, color='gray', linestyle=':', lw=0.8)
ax.axvline(np.pi, color='green', linestyle='--', lw=0.8)
plt.show()

# === TEXT AND LATEX ===
fig, ax = plt.subplots()
ax.text(0.5, 0.5, r'$\alpha > \beta$ and $\sum_{i=0}^{n} x_i^2$',
        fontsize=16, ha='center', transform=ax.transAxes)
plt.show()

# === SAVING ===
fig, ax = plt.subplots()
ax.plot(x, np.sin(x))
plt.savefig('sin_plot.png', dpi=300, bbox_inches='tight', transparent=False)
plt.savefig('sin_plot.pdf', format='pdf', dpi=150)
plt.savefig('sin_plot.svg', format='svg')

# === STYLES ===
print(plt.style.available)
plt.style.use('seaborn-v0_8-darkgrid')

# === 3D PLOTTING ===
from mpl_toolkits.mplot3d import Axes3D
fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')
X, Y = np.meshgrid(np.linspace(-5, 5, 50), np.linspace(-5, 5, 50))
Z = np.sin(np.sqrt(X**2 + Y**2))
ax.plot_surface(X, Y, Z, cmap='viridis', edgecolor='none')
ax.set_xlabel('X')
ax.set_ylabel('Y')
ax.set_zlabel('Z')
ax.set_title('3D Surface')
plt.show()

# === ANIMATION ===
from matplotlib.animation import FuncAnimation
fig, ax = plt.subplots()
x = np.linspace(0, 2 * np.pi, 100)
line, = ax.plot(x, np.sin(x))

def update(frame):
    line.set_ydata(np.sin(x + frame * 0.1))
    return line,

anim = FuncAnimation(fig, update, frames=100, interval=50, blit=True)
plt.show()
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| `plt.plot()` + `plt.show()` in a loop | Creates multiple windows | Create subplots once, plot in loop |
| Not calling `plt.tight_layout()` | Overlapping axes/labels | Call `plt.tight_layout()` or `fig.tight_layout()` |
| Mixing pyplot and OOP randomly | Confusing state machine | Choose one; use OOP for >1 subplot |
| Forgetting `.copy()` in animations | All frames show same data | Update in place, don't reassign |
| Using `plt.figure()` inside a loop | Creates hundreds of figures | Reuse figure or close with `plt.close()` |
| Saving before `plt.show()` | Blank or different image | Save before show, or save after closing |

### Best Practices

1. Prefer OOP API for anything with multiple subplots
2. Set `figsize` and `dpi` explicitly — don't rely on defaults
3. Use `tight_layout()` before saving or showing
4. Set vector formats (PDF, SVG) for publications, PNG for web
5. Use colorblind-friendly palettes (e.g., `plt.cm.colorblind.colors`)
6. Keep aspect ratio appropriate to data (use `ax.set_aspect('equal')` for maps)
7. Label everything — axes, titles, legends, annotations

### Interview Questions

1. What is the difference between Figure, Axes, and Axis in Matplotlib?
2. When would you use pyplot vs OOP API?
3. How would you create a figure with 3 plots (top row spans 2 columns, bottom row is full width)?
4. Explain the difference between `plt.close()`, `plt.clf()`, and `plt.cla()`
5. How would you create an animated scatter plot of random walks?

### Practical Exercises

1. Create a 2x2 subplot with four different chart types (line, scatter, bar, histogram)
2. Customize spines, ticks, and grid for publication-quality output
3. Create a 3D surface plot of a mathematical function
4. Animate a sine wave with a moving phase

### Mini Project

**Financial Dashboard**: Create a Matplotlib figure with 4 subplots: (1) Top: Line chart of stock price with 50-day and 200-day moving averages, (2) Middle-left: Volume bar chart, (3) Middle-right: RSI indicator plot, (4) Bottom: Daily returns histogram with overlaid normal distribution. Use proper tick formatting, grid lines, legends, and color-coding (green for up, red for down).

### Revision Notes

| Concept | Key Function | Syntax |
|---------|-------------|--------|
| Figure creation | `plt.subplots()` | `fig, ax = plt.subplots(2, 2)` |
| Basic plot | `ax.plot()` | `ax.plot(x, y, 'r--')` |
| Scatter | `ax.scatter()` | `ax.scatter(x, y, c=z, s=size)` |
| Bar | `ax.bar()` | `ax.bar(cats, vals)` |
| Histogram | `ax.hist()` | `ax.hist(data, bins=30)` |
| Customize | `ax.set_xlabel()` | `ax.set_title('Title', fontsize=14)` |
| Save | `plt.savefig()` | `plt.savefig('file.png', dpi=300)` |
| Subplots | `plt.GridSpec` | `ax = plt.subplot(gs[0, :])` |

---

## Chapter 77: Data Visualization — Seaborn and Plotly

### Introduction

Seaborn extends Matplotlib with statistical plot types (kernel density, violin, joint distributions, heatmaps, pairplots) and sensible defaults for colors, themes, and aesthetics. Plotly provides interactive, web-native visualizations with hover, zoom, pan, and animation. Together, they cover static publication and interactive dashboard needs.

### Why It Matters

Seaborn reduces 20 lines of Matplotlib to 2 lines of high-level API. It automatically computes statistics, handles categorical aggregation, and applies attractive themes. Plotly enables data exploration that static plots cannot: zoom into clusters, hover for exact values, animate over time, and embed in web applications.

### Real World Analogy

Seaborn is like a professional photographer with good presets — you get great results quickly. Matplotlib is like a darkroom where you develop each photo by hand. Plotly is like an interactive museum exhibit — visitors can touch, zoom, and explore.

> ✅ **Best Practice:** For data exploration, start with Seaborn's high-level API to quickly understand your data's distribution, then use Plotly for interactive dashboards and Matplotlib for publication-quality figures. Each tool has a clear niche — use the right one for the job.

> 💡 **Pro Tip:** Use Seaborn's `kdeplot` with `fill=True` and `alpha=0.3` for beautiful overlapping distributions. For publications, use `sns.set_context('paper')` to scale fonts appropriately. Always set `sns.set_theme()` at the beginning of your script for consistent styling across all plots.

### Theory

**Seaborn** works on "tidy" (long-form) DataFrames. Each column is a variable, each row is an observation. It automatically aggregates and computes confidence intervals. It internally uses Matplotlib for rendering.

**Plotly Express** is a high-level API that generates Plotly Graph Objects (traces + layout JSON). Plotly.js renders these in the browser using WebGL for performance. Plotly Graph Objects give full control over every interactive detail.

### Code Examples

```python
import seaborn as sns
import matplotlib.pyplot as plt
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import pandas as pd
import numpy as np

# === SEABORN ===
tips = sns.load_dataset('tips')
iris = sns.load_dataset('iris')
flights = sns.load_dataset('flights')
penguins = sns.load_dataset('penguins')

# === relplot ===
sns.relplot(data=tips, x='total_bill', y='tip', hue='time', style='sex', size='size')
plt.title('Tips by total bill')
plt.show()

sns.relplot(data=flights, x='year', y='passengers', hue='month', kind='line', ci=None)
plt.show()

# === displot ===
sns.displot(data=tips, x='total_bill', hue='sex', kind='hist', kde=True, bins=30)
plt.show()

sns.displot(data=tips, x='total_bill', kind='ecdf')
plt.show()

# === catplot ===
sns.catplot(data=tips, x='day', y='total_bill', hue='sex', kind='box')
plt.show()

sns.catplot(data=tips, x='day', y='total_bill', hue='sex', kind='violin', split=True)
plt.show()

sns.catplot(data=tips, x='day', y='total_bill', kind='strip', jitter=True)
plt.show()

# === lmplot ===
sns.lmplot(data=tips, x='total_bill', y='tip', hue='sex', ci=95, order=2)
plt.show()

# === pairplot ===
sns.pairplot(iris, hue='species', diag_kind='kde', markers=['o', 's', 'D'])
plt.show()

# === jointplot ===
sns.jointplot(data=iris, x='sepal_length', y='sepal_width', kind='hex')
plt.show()

# === heatmap ===
corr = tips.select_dtypes('number').corr()
sns.heatmap(corr, annot=True, fmt='.2f', cmap='coolwarm', vmin=-1, vmax=1,
            square=True, linewidths=0.5)
plt.title('Correlation Matrix')
plt.show()

# === clustermap ===
sns.clustermap(corr, annot=True, fmt='.2f', cmap='coolwarm', standard_scale=1)
plt.show()

# === Themes and Palettes ===
sns.set_theme(style='whitegrid', palette='muted', font='monospace', font_scale=1.2)
sns.set_palette('colorblind')
sns.set_context('talk')

# === Statistical Plots ===
sns.kdeplot(data=tips, x='total_bill', hue='sex', fill=True, alpha=0.3)
plt.show()

sns.ecdfplot(data=tips, x='total_bill', hue='sex')
plt.show()

sns.rugplot(data=tips, x='total_bill', hue='sex')
plt.show()

# ===========================================
# === PLOTLY EXPRESS ===
# ===========================================
df_iris = px.data.iris()

fig = px.scatter(df_iris, x='sepal_length', y='sepal_width', color='species',
                 size='petal_length', hover_data=['sepal_width'],
                 title='Iris Dataset')
fig.show()

df_gap = px.data.gapminder()
fig = px.line(df_gap, x='year', y='lifeExp', color='country',
              title='Life Expectancy Over Time')
fig.show()

df_tips = px.data.tips()
fig = px.bar(df_tips, x='day', y='total_bill', color='sex',
             title='Total Bill by Day and Gender', barmode='group')
fig.show()

fig = px.histogram(df_tips, x='total_bill', color='sex', nbins=30,
                   marginal='box', title='Distribution of Total Bill')
fig.show()

fig = px.box(df_tips, x='day', y='total_bill', color='sex',
             points='all', title='Box Plot of Tips')
fig.show()

fig = px.violin(df_tips, x='day', y='total_bill', color='sex',
                box=True, points='all')
fig.show()

fig = px.density_heatmap(df_tips, x='total_bill', y='tip', nbinsx=20, nbinsy=20,
                         title='Density Heatmap')
fig.show()

df_gap = px.data.gapminder()
fig = px.choropleth(df_gap.query("year==2007"), locations='iso_alpha',
                    color='lifeExp', hover_name='country',
                    title='World Life Expectancy 2007')
fig.show()

fig = px.scatter(df_gap, x='gdpPercap', y='lifeExp', size='pop', color='continent',
                 log_x=True, size_max=60, animation_frame='year',
                 title='Gapminder Over Time (animated)')
fig.show()

# ===========================================
# === PLOTLY GRAPH OBJECTS ===
# ===========================================
fig = go.Figure()

fig.add_trace(go.Scatter(
    x=[1, 2, 3, 4],
    y=[10, 11, 12, 13],
    mode='lines+markers',
    name='Series A',
    line=dict(color='firebrick', width=2, dash='dash'),
    marker=dict(size=8, symbol='circle')
))

fig.add_trace(go.Bar(
    x=['A', 'B', 'C'],
    y=[20, 14, 23],
    name='Bar Series',
    marker_color='lightsalmon'
))

fig.update_layout(
    title='Graph Objects Example',
    xaxis=dict(title='X Axis', showgrid=True, gridcolor='lightgray'),
    yaxis=dict(title='Y Axis', showgrid=True, gridcolor='lightgray'),
    legend=dict(x=0.02, y=0.98, bgcolor='rgba(255,255,255,0.8)'),
    template='plotly_white',
    hovermode='x unified',
    width=900,
    height=500
)
fig.show()

# === SUBPLOTS IN PLOTLY ===
fig = make_subplots(
    rows=2, cols=2,
    subplot_titles=('Scatter', 'Bar', 'Heatmap', 'Box'),
    specs=[[{}, {}], [{'type': 'heatmap'}, {'type': 'box'}]]
)

fig.add_trace(go.Scatter(x=[1,2,3], y=[4,5,6], mode='markers'), row=1, col=1)
fig.add_trace(go.Bar(x=['A','B','C'], y=[10,11,12]), row=1, col=2)
fig.add_trace(go.Heatmap(z=[[1,20,30],[20,1,60],[30,60,1]]), row=2, col=1)
fig.add_trace(go.Box(y=[1,2,3,4,5,6,7,8,9]), row=2, col=2)

fig.update_layout(height=600, showlegend=False, title='Plotly Subplots')
fig.show()
```

### Dash (Interactive Dashboards)

```python
import dash
from dash import dcc, html, Input, Output
import plotly.express as px

df = px.data.iris()

app = dash.Dash(__name__)

app.layout = html.Div([
    html.H1('Iris Dashboard'),
    dcc.Dropdown(
        id='x-variable',
        options=[{'label': c, 'value': c} for c in df.columns[:4]],
        value='sepal_length'
    ),
    dcc.Graph(id='scatter-plot')
])

@app.callback(
    Output('scatter-plot', 'figure'),
    Input('x-variable', 'value')
)
def update_chart(x_var):
    fig = px.scatter(df, x=x_var, y='sepal_width', color='species')
    return fig

if __name__ == '__main__':
    app.run_server(debug=True)
```

### When to Use Static vs Interactive

| Use Case | Static (Seaborn/Matplotlib) | Interactive (Plotly) |
|----------|---------------------------|---------------------|
| Publication/PDF | Yes | Rare (PDF export, lose interactivity) |
| Data exploration | No | Yes (zoom, filter, hover) |
| Dashboards | No | Yes (Dash app) |
| Large datasets | Faster (bitmap) | Use datashader + Plotly |
| Statistical rigor | Yes (Seaborn does stats) | Manual computations |
| ML presentations | Both | Interactive for demos |

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Using Seaborn on wide-form data | Wrong orientation | `melt()` to long form first |
| Calling `plt.show()` before `sns.jointplot` | Last plot overwritten | Show immediately after each plot |
| Plotly in Jupyter without `fig.show()` | Nothing appears | Always call `fig.show()` |
| Too many traces in one Plotly figure | Slow rendering | Use subplots or subset data |
| Forgetting `template='plotly_dark'` | Default template is plain | Choose appropriate template |

### Best Practices

1. Use long-form (tidy) data for Seaborn
2. Use colorblind-friendly palettes
3. Set appropriate figure sizes for the output medium
4. Limit legend entries to 6-8 categories
5. Use log scales for heavily skewed distributions
6. Annotate outliers in exploratory plots
7. Use `facet_col` in Plotly Express for small multiples

### Interview Questions

1. What is the difference between Seaborn's `relplot` and `lmplot`?
2. How do you create an interactive dashboard with Plotly Dash?
3. Explain the data format required by Seaborn (tidy vs wide)
4. When would you use `go.Figure` instead of `px.scatter`?
5. How would you animate a scatter plot over time in Plotly?

### Practical Exercises

1. Create a Seaborn pairplot with different diagonal and off-diagonal plots
2. Build a Plotly dashboard with 3 linked plots (scatter, histogram, box)
3. Reproduce a Seaborn violin plot using Plotly Graph Objects
4. Animate the gapminder dataset with Plotly Express

### Mini Project

**Exploratory Data Analysis Dashboard**: Using a dataset of your choice (e.g., tips, iris, penguins, or custom), create: (1) A static (Seaborn) EDA report PDF containing pairplot, correlation heatmap, distribution of all variables, box plots by category. (2) An interactive (Plotly) single-page web dashboard with scatter with dropdown for x/y variables, histogram with slider for bins, and a summary stats table.

### Revision Notes

| Concept | Seaborn | Plotly |
|---------|---------|--------|
| Scatter | `sns.relplot(kind='scatter')` | `px.scatter()` |
| Distribution | `sns.displot(kind='hist')` | `px.histogram()` |
| KDE | `sns.kdeplot()` | `px.histogram(marginal='violin')` |
| Box | `sns.catplot(kind='box')` | `px.box()` |
| Heatmap | `sns.heatmap()` | `px.imshow()` or `go.Heatmap` |
| Pairplot | `sns.pairplot()` | `px.scatter_matrix()` |
| Map | N/A | `px.choropleth()` |
| Animation | Static only | `px.scatter(animation_frame=...)` |
| Dashboard | N/A | Dash |

---

## Chapter 78: Data Cleaning and Preprocessing

### Introduction

Real-world data is messy: missing values, inconsistent formats, outliers, duplicates, type mismatches, and encoding errors. Data cleaning is the process of detecting and fixing these issues. It typically consumes 60-80% of a data scientist's time. This chapter covers systematic approaches to data quality.

### Why It Matters

Garbage in, garbage out. Models trained on dirty data produce unreliable results. Misleading visualizations from unclean data lead to wrong business decisions. A well-defined cleaning pipeline is the most impactful thing you can build for downstream analysis.

### Real World Analogy

Data cleaning is like preparing ingredients before cooking. You would not use bruised vegetables, unwashed produce, or expired ingredients for a gourmet meal. Similarly, you cannot produce high-quality analysis from dirty data — you must inspect, clean, and validate before cooking (analysis/modeling).

> ⚠️ **Warning:** Data cleaning decisions impact model performance significantly. Never drop all rows with missing values — this can introduce bias if missingness is not random. Always analyze the pattern of missingness (MCAR, MAR, MNAR) before choosing an imputation strategy.

### Theory

**Data Quality Dimensions**:
1. **Completeness**: Are all required values present?
2. **Consistency**: Are values in the same format across records?
3. **Accuracy**: Do values reflect reality?
4. **Validity**: Do values conform to schema (type, range, format)?
5. **Timeliness**: Are data sufficiently current?

**Missing Data Mechanisms**:
- **MCAR (Missing Completely At Random)**: No systematic relationship between missingness and data values
- **MAR (Missing At Random)**: Missingness depends on observed data but not the missing values
- **MNAR (Missing Not At Random)**: Missingness depends on the missing values themselves

### Code Examples

```python
import pandas as pd
import numpy as np
from sklearn.impute import KNNImputer, SimpleImputer
from sklearn.ensemble import IsolationForest
from scipy import stats
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer
import re

# === MISSING DATA HANDLING ===
df = pd.DataFrame({
    'age': [25, 30, np.nan, 35, 28, np.nan, 40],
    'salary': [50000, np.nan, 60000, 65000, 55000, 62000, np.nan],
    'experience': [2, 5, 3, np.nan, 4, 6, 8],
    'department': ['IT', 'HR', 'IT', np.nan, 'HR', 'IT', 'HR']
})

print(df.isnull().sum())
print(df.isnull().sum() / len(df) * 100)

# Deletion
df_drop_row = df.dropna()
df_drop_col = df.dropna(axis=1, thresh=5)
df_drop_subset = df.dropna(subset=['age'])

# Mean imputation
num_cols = df.select_dtypes(include=[np.number]).columns
imputer_mean = SimpleImputer(strategy='mean')
df[num_cols] = imputer_mean.fit_transform(df[num_cols])

# Mode imputation for categorical
cat_imputer = SimpleImputer(strategy='most_frequent')
df['department'] = cat_imputer.fit_transform(df[['department']]).ravel()

# KNN Imputation
df_knn = df.copy()
knn_imputer = KNNImputer(n_neighbors=3)
df_knn[num_cols] = knn_imputer.fit_transform(df_knn[num_cols])

# MICE imputation
mice_imputer = IterativeImputer(max_iter=10, random_state=42)
df_mice = df.copy()
df_mice[num_cols] = mice_imputer.fit_transform(df_mice[num_cols])

# Forward/Backward fill (time series)
ts = pd.Series([1, np.nan, np.nan, 4, 5, np.nan, 7],
               index=pd.date_range('2024-01-01', periods=7))
print(ts.ffill())
print(ts.bfill())
print(ts.interpolate(method='linear'))

# === OUTLIER DETECTION ===
data = pd.DataFrame({'value': [10, 12, 11, 13, 100, 12, 11, 14, -50, 13, 12]})

# IQR method
Q1 = data['value'].quantile(0.25)
Q3 = data['value'].quantile(0.75)
IQR = Q3 - Q1
lower_bound = Q1 - 1.5 * IQR
upper_bound = Q3 + 1.5 * IQR
outliers_iqr = data[(data['value'] < lower_bound) | (data['value'] > upper_bound)]
print(f'IQR outliers: {outliers_iqr}')

# Z-score method
z_scores = np.abs(stats.zscore(data['value'], nan_policy='omit'))
outliers_z = data[z_scores > 3]
print(f'Z-score outliers: {outliers_z}')

# Modified Z-score (robust)
median = data['value'].median()
mad = np.median(np.abs(data['value'] - median))
modified_z = 0.6745 * (data['value'] - median) / mad
outliers_mz = data[np.abs(modified_z) > 3.5]
print(f'Modified Z-score outliers: {outliers_mz}')

# Isolation Forest
iso_forest = IsolationForest(contamination=0.1, random_state=42)
outlier_pred = iso_forest.fit_predict(data[['value']])
data['is_outlier_if'] = outlier_pred == -1
print(data[data['is_outlier_if']])

# === DATA TYPE VALIDATION ===
df = pd.DataFrame({
    'id': ['001', '002', '003'],
    'price': ['$12.50', '$15.00', 'error'],
    'quantity': ['10', '20', '30'],
    'date': ['2024-01-01', '2024-13-01', '2024-01-15']
})

df['price_clean'] = pd.to_numeric(df['price'].str.replace('$', ''), errors='coerce')
df['quantity_clean'] = pd.to_numeric(df['quantity'], errors='coerce')
df['date_clean'] = pd.to_datetime(df['date'], errors='coerce')
print(df.dtypes)
print(df)

# === STRING CLEANING ===
messy = pd.Series([
    '  Alice Smith  ',
    'BOB JOHNSON',
    'charlie brown ',
    '   ',
    None,
    'david@email.com'
])

print(messy.str.strip())
print(messy.str.lower())
print(messy.str.title())
print(messy.str.extract(r'(\w+)@(\w+)\.(\w+)'))

# === DEDUPLICATION ===
df = pd.DataFrame({
    'name': ['Alice', 'Bob', 'Alice', 'Bob', 'Charlie'],
    'age': [25, 30, 25, 31, 35],
    'city': ['NYC', 'LA', 'NYC', 'Los Angeles', 'SF']
})

print(df.duplicated(subset=['name']))
print(df.drop_duplicates(subset=['name'], keep='first'))

# === CLEANING PIPELINE ===
def clean_pipeline(df):
    df = df.copy()
    df = df.dropna(axis=1, thresh=len(df) * 0.5)
    num_cols = df.select_dtypes(include=[np.number]).columns
    medians = df[num_cols].median()
    df[num_cols] = df[num_cols].fillna(medians)
    cat_cols = df.select_dtypes(include=['object', 'category']).columns
    for col in cat_cols:
        df[col] = df[col].fillna(df[col].mode()[0] if not df[col].mode().empty else 'Unknown')
    for col in num_cols:
        Q1, Q3 = df[col].quantile(0.25), df[col].quantile(0.75)
        IQR = Q3 - Q1
        lower, upper = Q1 - 1.5*IQR, Q3 + 1.5*IQR
        df[col] = df[col].clip(lower, upper)
    for col in cat_cols:
        df[col] = df[col].astype(str).str.strip().str.title()
    return df
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Dropping all rows with any NaN | Loses too much data | Use threshold or impute |
| Filling NaN with mean before splitting train/test | Data leakage | Compute mean on train, apply to all |
| Not distinguishing MCAR/MAR/MNAR | Wrong imputation strategy | Analyze missingness patterns |
| Removing outliers without domain knowledge | Loses legitimate extreme values | Cap/winsorize instead of delete |
| Applying imputation without checking distribution | Distorts data distribution | Visualize before/after imputation |
| Ignoring `errors` parameter in `pd.to_numeric` | Silent type coercion failure | Use `errors='coerce'` and check |

> 💡 **Pro Tip:** Build a reusable data cleaning pipeline using sklearn's `Pipeline` and `ColumnTransformer`. This ensures your cleaning steps (imputation, scaling, encoding) are applied consistently to train and test data, preventing data leakage. Chain transformations with `.pipe()` in Pandas or use custom `TransformerMixin` classes for full reproducibility.

### Best Practices

1. Document missingness decisions — log why you chose mean vs median vs KNN
2. Validate after cleaning — build automated checks (Great Expectations)
3. Keep original data — never overwrite raw inputs; use copies and reproducible scripts
4. Impute before scaling — imputation should precede normalization
5. Split before cleaning transforms — compute imputation params on train, apply to test
6. Use domain knowledge — consult subject matter experts for outlier thresholds
7. Build cleaning pipelines — chain operations with `.pipe()` or sklearn `Pipeline`

### Interview Questions

1. Explain MCAR, MAR, and MNAR with examples
2. How would you handle a dataset with 30% missing values across different columns?
3. What is the difference between winsorizing and trimming?
4. How do you detect and handle outliers in a multi-modal distribution?
5. What is data leakage in the context of imputation?

### Practical Exercises

1. Load a real-world dataset, identify all data quality issues, and document them
2. Compare mean, median, KNN, and MICE imputation on a dataset with synthetic missingness
3. Build a robust outlier detection function that handles multiple distributions
4. Create a Great Expectations suite for a customer dataset

### Mini Project

**Data Cleaning Automation**: Build a class `DataCleaner` that takes a DataFrame and configuration: config (which columns to impute with which strategy, outlier method, string standardization rules), `.fit(df)` learns parameters, `.transform(df)` applies cleaning, `.fit_transform(df)` does both. Include logging (track how many rows changed per operation) and validation report generation.

### Revision Notes

| Concept | Methods | Syntax |
|---------|---------|--------|
| Missing detection | `isnull(), info()` | `df.isnull().sum()` |
| Deletion | `dropna()` | `df.dropna(thresh=5)` |
| Imputation | `fillna, SimpleImputer, KNNImputer` | `df.fillna(df.median())` |
| Outliers (IQR) | `quantile(), 1.5*IQR` | `Q3 + 1.5*IQR` |
| Outliers (Z-score) | `stats.zscore` | `z > 3` |
| Data types | `to_numeric, to_datetime` | `pd.to_numeric(s, errors='coerce')` |
| String cleaning | `.str.strip(), .str.replace()` | `s.str.lower().str.strip()` |
| Deduplication | `duplicated, drop_duplicates` | `df.drop_duplicates(subset=['id'])` |

---

## Chapter 79: Feature Engineering

### Introduction

Feature engineering is the process of transforming raw data into features that better represent the underlying problem to predictive models. It is the most effective way to improve model performance — often more impactful than algorithm selection or hyperparameter tuning.

### Why It Matters

Models are only as good as their inputs. Raw data rarely contains the signal in a form that models can use directly. Feature engineering encodes domain knowledge, exposes latent patterns, and creates representations that simplify the learning task. It separates good practitioners from great ones.

### Real World Analogy

If modeling is cooking, feature engineering is the preparation and seasoning. Raw ingredients (raw data) need to be cut, marinated, and combined before cooking (modeling). The same ingredient prepared differently yields an entirely different dish.

> 💡 **Pro Tip:** Feature engineering often yields more improvement than model selection. Start with simple encodings and transformations, then iteratively add more complex features while validating on a hold-out set. If a new feature doesn't improve validation metrics, discard it — more features aren't always better.

### Theory

**Three levels of feature engineering**:
1. **Encoding**: Converting non-numeric data to numeric (categorical, text, dates)
2. **Transformation**: Changing the distribution or scale of numeric features
3. **Creation**: Building new features from existing ones (interactions, aggregations, domain-specific)

**Feature Selection**: Filter methods (statistical tests), wrapper methods (iterative model evaluation), and embedded methods (model-internal feature importance).

> ⚠️ **Warning:** Never perform feature selection or scaling before splitting data into train and test sets. This causes data leakage — information from the test set influences the training process, leading to overly optimistic performance estimates. Always fit transformers on training data only, then transform test data.

### Code Examples

```python
import pandas as pd
import numpy as np
from sklearn.preprocessing import (OneHotEncoder, LabelEncoder, OrdinalEncoder,
                                   StandardScaler, MinMaxScaler, RobustScaler,
                                   MaxAbsScaler, PowerTransformer, QuantileTransformer,
                                   Normalizer, KBinsDiscretizer, PolynomialFeatures)
from sklearn.feature_selection import (SelectKBest, chi2, mutual_info_classif,
                                       RFE, SelectFromModel)
from sklearn.linear_model import Lasso, LogisticRegression
from sklearn.ensemble import RandomForestClassifier
from sklearn.datasets import make_classification
import category_encoders as ce
from scipy import stats

# ===========================================
# === ENCODING METHODS ===
# ===========================================

df = pd.DataFrame({
    'color': ['red', 'blue', 'green', 'red', 'blue'],
    'size': ['S', 'M', 'L', 'M', 'S'],
    'price': [10, 20, 15, 25, 30],
    'target': [0, 1, 0, 1, 1]
})

# One-Hot
ohe = pd.get_dummies(df['color'], prefix='color', drop_first=False)
print(ohe)

encoder = OneHotEncoder(drop='first', sparse_output=False)
encoded = encoder.fit_transform(df[['color']])
print(encoder.get_feature_names_out(['color']))

# Label Encoding
le = LabelEncoder()
df['color_label'] = le.fit_transform(df['color'])

# Ordinal Encoding
oe = OrdinalEncoder(categories=[['S', 'M', 'L']])
df['size_ordinal'] = oe.fit_transform(df[['size']])

# Target/Mean Encoding
mean_encoded = df.groupby('color')['target'].mean()
df['color_target_enc'] = df['color'].map(mean_encoded)

# Frequency Encoding
freq = df['color'].value_counts() / len(df)
df['color_freq'] = df['color'].map(freq)

# Binary Encoding
binary_enc = ce.BinaryEncoder(cols=['color'])
df_binary = binary_enc.fit_transform(df)

# ===========================================
# === SCALING METHODS ===
# ===========================================

X = np.array([[   1,   10],
              [1000,   20],
              [  50,   30],
              [ 200,   15],
              [  10, 5000]])

# StandardScaler
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
print(f'StandardScaler mean: {X_scaled.mean(axis=0).round(2)}')

# MinMaxScaler
minmax = MinMaxScaler()
X_minmax = minmax.fit_transform(X)

# RobustScaler
robust = RobustScaler()
X_robust = robust.fit_transform(X)

# PowerTransformer (Box-Cox)
boxcox = PowerTransformer(method='box-cox')
X_boxcox = boxcox.fit_transform(X[:3])

# QuantileTransformer
qt_normal = QuantileTransformer(output_distribution='normal')
X_qt_normal = qt_normal.fit_transform(X)

# Normalizer
normalizer = Normalizer(norm='l2')
X_normalized = normalizer.fit_transform(X)

# ===========================================
# === BINNING ===
# ===========================================

ages = pd.Series([23, 45, 67, 12, 89, 34, 56, 78, 25, 43])

bins_ew = pd.cut(ages, bins=3, labels=['young', 'middle', 'senior'])
print(bins_ew.value_counts())

bins_eq = pd.qcut(ages, q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
print(bins_eq.value_counts())

kbd = KBinsDiscretizer(n_bins=3, encode='ordinal', strategy='uniform')
binned = kbd.fit_transform(ages.values.reshape(-1, 1))

# ===========================================
# === FEATURE INTERACTIONS ===
# ===========================================

df_int = pd.DataFrame({'A': [1, 2, 3], 'B': [4, 5, 6]})

poly = PolynomialFeatures(degree=2, interaction_only=False, include_bias=True)
X_poly = poly.fit_transform(df_int)
print(poly.get_feature_names_out(['A', 'B']))

# ===========================================
# === DATETIME FEATURES ===
# ===========================================

dates = pd.Series(pd.date_range('2024-01-01', periods=10, freq='h'))
df_dates = pd.DataFrame({'timestamp': dates})
df_dates['year'] = dates.dt.year
df_dates['month'] = dates.dt.month
df_dates['day'] = dates.dt.day
df_dates['hour'] = dates.dt.hour
df_dates['weekday'] = dates.dt.weekday
df_dates['quarter'] = dates.dt.quarter
df_dates['day_of_year'] = dates.dt.dayofyear
df_dates['is_weekend'] = df_dates['weekday'].isin([5, 6]).astype(int)

# Cyclical encoding
df_dates['hour_sin'] = np.sin(2 * np.pi * df_dates['hour'] / 24)
df_dates['hour_cos'] = np.cos(2 * np.pi * df_dates['hour'] / 24)

# ===========================================
# === NUMERICAL TRANSFORMATIONS ===
# ===========================================

X = np.array([1, 2, 3, 10, 100, 1000])

X_log = np.log1p(X)
X_sqrt = np.sqrt(X)
X_recip = 1 / X

# ===========================================
# === FEATURE SELECTION ===
# ===========================================

X, y = make_classification(n_samples=1000, n_features=20, n_informative=5,
                           n_redundant=5, n_repeated=5, random_state=42)
X = pd.DataFrame(X, columns=[f'f{i}' for i in range(20)])

# Filter: Correlation
corr = X.apply(lambda col: col.corr(pd.Series(y)))
print('Top 5 by correlation:', corr.abs().sort_values(ascending=False).head(5))

# Filter: Mutual Information
mi = mutual_info_classif(X, y, random_state=42)
mi_series = pd.Series(mi, index=X.columns).sort_values(ascending=False)
print('Top 5 by mutual info:', mi_series.head(5))

# Wrapper: RFE
estimator = LogisticRegression(max_iter=1000)
rfe = RFE(estimator, n_features_to_select=5)
rfe.fit(X, y)
print('RFE selected:', X.columns[rfe.get_support()])

# Embedded: Lasso
lasso = Lasso(alpha=0.01)
lasso.fit(X, y)
for name, coef in zip(X.columns, lasso.coef_):
    if abs(coef) > 0.01:
        print(f'  {name}: {coef:.4f}')

# Embedded: Tree importance
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X, y)
importance = pd.Series(rf.feature_importances_, index=X.columns).sort_values(ascending=False)
print('Top 5 by RF importance:', importance.head(5))

# SelectFromModel
selector = SelectFromModel(rf, threshold='median')
selector.fit(X, y)
print('Model-based selected:', X.columns[selector.get_support()])
```

### Featuretools (Deep Feature Synthesis)

```python
import featuretools as ft

es = ft.EntitySet(id='customer_data')

customers = pd.DataFrame({
    'customer_id': [1, 2, 3],
    'signup_date': pd.to_datetime(['2024-01-01', '2024-02-01', '2024-03-01'])
})
transactions = pd.DataFrame({
    'transaction_id': [101, 102, 103, 104, 105],
    'customer_id': [1, 1, 2, 3, 3],
    'amount': [100, 200, 150, 300, 250],
    'transaction_date': pd.to_datetime(['2024-01-15', '2024-02-15', '2024-02-20', '2024-03-10', '2024-03-20'])
})

es = es.add_dataframe(dataframe_name='customers', dataframe=customers, index='customer_id')
es = es.add_dataframe(dataframe_name='transactions', dataframe=transactions, index='transaction_id')
es = es.add_relationship(ft.Relationship(es['customers']['customer_id'], es['transactions']['customer_id']))

features, feature_defs = ft.dfs(
    entityset=es, target_dataframe_name='customers',
    agg_primitives=['sum', 'mean', 'count', 'max', 'min', 'std'],
    trans_primitives=['day', 'month', 'year', 'weekday'],
    max_depth=2
)
print(features.head())
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| One-hot encoding high-cardinality columns (1000+ categories) | Massive feature explosion | Use target encoding or frequency encoding |
| Scaling before train/test split | Data leakage | Fit scaler on train, transform test separately |
| Feature selection using full dataset (including test) | Over-optimistic performance | Select features only on training set |
| Creating too many irrelevant features | Overfitting, curse of dimensionality | Use feature selection (L1, mutual info) |
| Forgetting `handle_unknown='ignore'` in OneHotEncoder | Error on unseen categories in test | Set parameter appropriately |
| Not binning continuous features when non-linear | Missed non-linear patterns | Try KBinsDiscretizer with models |

### Best Practices

1. Start simple — encode/log-transform only what is needed before complex engineering
2. Validate on validation set — each engineering step should improve validation metrics
3. Use domain knowledge — consult experts for meaningful feature interactions
4. Automate with Featuretools for relational data
5. Keep feature store — cache engineered features for reproducibility
6. Use pipelines — `sklearn.pipeline.Pipeline` to chain transforms
7. Monitor feature distributions over time (drift detection)

### Interview Questions

1. What is the difference between one-hot encoding and label encoding, and when would you use each?
2. How would you handle a categorical feature with 10,000 unique values?
3. Explain the bias-variance tradeoff in feature engineering
4. What is target leakage in feature engineering, and how do you prevent it?
5. Compare filter, wrapper, and embedded feature selection methods

### Practical Exercises

1. Apply 5 different encoding methods to a categorical variable and compare model performance
2. Create cyclic features (sin/cos) for hour of day and day of year
3. Use Featuretools to automatically generate features from a multi-table dataset
4. Implement forward feature selection from scratch (without sklearn)

### Mini Project

**Feature Engineering Pipeline**: Build a complete feature engineering pipeline for a real-world dataset (e.g., House Prices, Titanic, or custom sales data): identify column types, apply appropriate encodings and transformations, generate interaction features (top 2-way based on domain knowledge), create aggregated features from related tables, apply feature selection (filter + embedded), wrap everything in a sklearn Pipeline, and compare model performance with raw vs engineered features.

### Revision Notes

| Concept | Key Functions | Syntax |
|---------|--------------|--------|
| One-hot | `pd.get_dummies, OneHotEncoder` | `pd.get_dummies(df['col'])` |
| Label | `LabelEncoder` | `le.fit_transform(df['col'])` |
| Ordinal | `OrdinalEncoder(categories=[...])` | `oe.fit_transform(df[['col']])` |
| Standardization | `StandardScaler` | `scaler.fit_transform(X)` |
| Normalization | `MinMaxScaler, Normalizer` | `MinMaxScaler().fit_transform(X)` |
| Binning | `pd.cut, pd.qcut, KBinsDiscretizer` | `pd.cut(x, bins=3)` |
| Interactions | `PolynomialFeatures` | `poly.fit_transform(df)` |
| Feature selection | `SelectKBest, RFE, SelectFromModel` | `SelectKBest(chi2, k=10)` |

---

## Chapter 80: Time Series Analysis

### Introduction

Time series analysis deals with data indexed in time order: stock prices, temperature readings, server metrics, sales figures. It requires specialized techniques because observations are **dependent** (autocorrelated), violating the i.i.d. assumption of most statistical methods. This chapter covers decomposition, stationarity, ARIMA/SARIMA, Prophet, and time series cross-validation.

### Why It Matters

Time series data is everywhere: finance, IoT, healthcare, operations, web analytics. Forecasting demand, detecting anomalies, and understanding temporal patterns are core business problems. The field has developed its own rigorous statistical framework distinct from cross-sectional ML.

### Real World Analogy

Cross-sectional ML is like describing a photograph (one moment in time). Time series analysis is like describing a movie — the sequence matters, the past influences the future, and you need to understand the plot (trend), seasonal costumes (seasonality), and one-time events (residuals).

> 📖 **Instructor Note:** Stationarity is the most critical assumption in time series analysis. Before fitting any ARIMA model, always test for stationarity using the ADF test (checks if unit root exists) and KPSS test (checks if series is trend-stationary). Differencing is the most common technique to achieve stationarity.

### Theory

**Components**:
- **Trend**: Long-term direction (upward/downward/flat)
- **Seasonality**: Regular periodic patterns (daily, weekly, yearly)
- **Cyclic**: Longer-term non-fixed-period fluctuations (business cycle)
- **Residual/Irregular**: Random noise after removing trend + seasonality

**Stationarity**: A time series is stationary if its statistical properties (mean, variance, autocorrelation) are constant over time. Most forecasting models require stationarity.

**ARIMA (Autoregressive Integrated Moving Average)**:
- **AR(p)**: Uses p lagged observations as predictors
- **I(d)**: Differencing d times to achieve stationarity
- **MA(q)**: Uses q lagged forecast errors as predictors

### Code Examples

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from statsmodels.tsa.seasonal import seasonal_decompose, STL
from statsmodels.tsa.stattools import adfuller, kpss, acf, pacf
from statsmodels.tsa.holtwinters import ExponentialSmoothing, SimpleExpSmoothing, Holt
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from sklearn.metrics import mean_absolute_error, mean_squared_error
from sklearn.model_selection import TimeSeriesSplit
from pmdarima import auto_arima
import warnings
warnings.filterwarnings('ignore')

np.random.seed(42)
t = np.arange(100)
trend = 0.1 * t
seasonal = 5 * np.sin(2 * np.pi * t / 12)
noise = np.random.randn(100) * 2
series = trend + seasonal + noise

df = pd.DataFrame({'value': series}, index=pd.date_range('2020-01-01', periods=100, freq='M'))

# === COMPONENTS VISUALIZATION ===
fig, axes = plt.subplots(4, 1, figsize=(12, 8), sharex=True)
axes[0].plot(df.index, df['value'])
axes[0].set_title('Original')
axes[1].plot(df.index, trend)
axes[1].set_title('Trend')
axes[2].plot(df.index, seasonal)
axes[2].set_title('Seasonal')
axes[3].plot(df.index, noise)
axes[3].set_title('Residual')
plt.tight_layout()
plt.show()

# === STATIONARITY TESTS ===
result_adf = adfuller(df['value'])
print(f'ADF p-value: {result_adf[1]:.4f}')
print(f'Stationary (ADF): {result_adf[1] < 0.05}')

result_kpss = kpss(df['value'], regression='ct')
print(f'KPSS p-value: {result_kpss[1]:.4f}')
print(f'Stationary (KPSS): {result_kpss[1] >= 0.05}')

# === DIFFERENCING ===
df['diff_1'] = df['value'].diff()
df['diff_seasonal'] = df['value'].diff(12)

# === DECOMPOSITION ===
decomp_add = seasonal_decompose(df['value'], model='additive', period=12)
fig = decomp_add.plot()
plt.suptitle('Additive Decomposition', y=1.02)
plt.show()

stl = STL(df['value'], period=12, robust=True)
result_stl = stl.fit()
fig = result_stl.plot()
plt.suptitle('STL Decomposition', y=1.02)
plt.show()

# === MOVING AVERAGES ===
df['SMA_3'] = df['value'].rolling(window=3).mean()
df['SMA_12'] = df['value'].rolling(window=12).mean()
df['EMA'] = df['value'].ewm(span=3, adjust=False).mean()

# === ACF/PACF ===
fig, axes = plt.subplots(2, 1, figsize=(12, 6))
plot_acf(df['value'].dropna(), lags=20, ax=axes[0])
axes[0].set_title('ACF')
plot_pacf(df['value'].dropna(), lags=20, ax=axes[1])
axes[1].set_title('PACF')
plt.tight_layout()
plt.show()

# === SMOOTHING ===
train = df['value'][:80]
test = df['value'][80:]

ses = SimpleExpSmoothing(train).fit(smoothing_level=0.2)
ses_forecast = ses.forecast(len(test))

holt = Holt(train).fit(smoothing_level=0.8, smoothing_trend=0.2)
holt_forecast = holt.forecast(len(test))

hw = ExponentialSmoothing(train, seasonal_periods=12, trend='add', seasonal='add').fit()
hw_forecast = hw.forecast(len(test))

# === ARIMA ===
arima = ARIMA(train, order=(1, 1, 1))
arima_fit = arima.fit()
print(arima_fit.summary())
arima_forecast = arima_fit.forecast(len(test))

auto_model = auto_arima(train, seasonal=False, stepwise=True, trace=True,
                        suppress_warnings=True)
print(f'Best ARIMA order: {auto_model.order}')
auto_forecast = auto_model.predict(n_periods=len(test))

# === SARIMA ===
sarima = ARIMA(train, order=(1, 1, 1), seasonal_order=(1, 1, 1, 12))
sarima_fit = sarima.fit()
sarima_forecast = sarima_fit.forecast(len(test))

auto_sarima = auto_arima(train, seasonal=True, m=12, stepwise=True, trace=True,
                         suppress_warnings=True, error_action='ignore')
print(f'Best SARIMA order: {auto_sarima.order}, seasonal: {auto_sarima.seasonal_order}')
auto_sarima_forecast = auto_sarima.predict(n_periods=len(test))

# === TIME SERIES CV ===
tscv = TimeSeriesSplit(n_splits=5)
for fold, (train_idx, val_idx) in enumerate(tscv.split(df)):
    print(f'Fold {fold+1}: train {train_idx[0]}-{train_idx[-1]}, val {val_idx[0]}-{val_idx[-1]}')

# === METRICS ===
def forecast_metrics(y_true, y_pred):
    mae = mean_absolute_error(y_true, y_pred)
    mse = mean_squared_error(y_true, y_pred)
    rmse = np.sqrt(mse)
    mape = np.mean(np.abs((y_true - y_pred) / y_true)) * 100
    smape = np.mean(2 * np.abs(y_pred - y_true) / (np.abs(y_true) + np.abs(y_pred))) * 100
    naive = np.roll(y_true, 12)
    naive_mae = mean_absolute_error(y_true[12:], naive[12:])
    mase = mae / naive_mae if naive_mae > 0 else np.nan
    return {'MAE': mae, 'MSE': mse, 'RMSE': rmse, 'MAPE': mape, 'sMAPE': smape, 'MASE': mase}

metrics = forecast_metrics(test.values, hw_forecast.values)
for k, v in metrics.items():
    print(f'{k}: {v:.4f}')
```

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Not checking stationarity before ARIMA | Invalid model (spurious regression) | Always run ADF + KPSS |
| Using default ADF test with wrong regression | Wrong critical values | Choose 'c', 'ct', or 'n' based on trend |
| Overdifferencing | Loses information, increases variance | Test stationarity after each differencing |
| Not using seasonality in ARIMA | Poor forecasts for seasonal data | Check ACF for seasonal spikes; use SARIMA |
| Forecasting without backtesting | Over-optimistic performance | Use TimeSeriesSplit or expanding window |
| Using MAPE when actuals are near zero | Infinite or undefined MAPE | Use sMAPE or MASE |
| Not logging predictions vs actuals | Hard to debug failures | Always plot prediction vs actual |

### Best Practices

1. Always plot the series first — visual inspection reveals trend, seasonality, outliers, structural breaks
2. Test stationarity with 2+ tests (ADF + KPSS) — they complement each other
3. Use AIC/BIC to compare models — lower is better, but prefer simpler model within 2 AIC
4. Backtest systematically — use expanding window CV, not just a single train/test split
5. Check residuals — should be white noise (ACF shows no significant lags)
6. Benchmark against naive — your fancy ARIMA should beat "predict last value"
7. Use Prophet for datasets with strong seasonality and holiday effects

### Interview Questions

1. Explain the difference between ADF and KPSS tests
2. How do you determine the p, d, q parameters for an ARIMA model?
3. What is the difference between additive and multiplicative seasonality?
4. How would you handle multiple seasonalities (hourly, daily, weekly)?
5. What is MASE and why is it preferred over MAPE?

### Practical Exercises

1. Decompose a time series with both additive and multiplicative models and compare residuals
2. Build ARIMA(p,d,q) and SARIMA models for a monthly dataset; compare AIC and forecast accuracy
3. Implement expanding-window cross-validation for model selection
4. Use Prophet to forecast with custom holiday/event effects

### Mini Project

**Sales Forecasting System**: Using a retail sales dataset (or simulated with trend + seasonality + holidays): (1) Visualize time series components, (2) Test and achieve stationarity, (3) Build and tune ARIMA, SARIMA, Holt-Winters, and Prophet models, (4) Compare forecasts using RMSE, MAE, MAPE, MASE, (5) Backtest with TimeSeriesSplit, (6) Select best model and generate 12-month forecast with confidence intervals, (7) Document forecasting assumptions and limitations.

### Revision Notes

| Concept | Key Function | Syntax |
|---------|-------------|--------|
| Decomposition | `seasonal_decompose, STL` | `seasonal_decompose(series, period=12)` |
| Stationarity | `adfuller, kpss` | `adfuller(series)` |
| Differencing | `.diff()` | `series.diff(1)` |
| ACF/PACF | `plot_acf, plot_pacf` | `plot_acf(series)` |
| Smoothing | `SimpleExpSmoothing, Holt, ExponentialSmoothing` | `Holt(series).fit()` |
| ARIMA | `ARIMA(order=(p,d,q))` | `ARIMA(series, order=(1,1,1)).fit()` |
| SARIMA | `ARIMA + seasonal_order` | `ARIMA(series, seasonal_order=(1,1,1,12))` |
| Prophet | `Prophet().fit()` | `Prophet().fit(df).predict(future)` |
| CV | `TimeSeriesSplit` | `TimeSeriesSplit(n_splits=5)` |
| Metrics | `MAE, RMSE, MAPE, MASE` | `mean_absolute_error(y, pred)` |

---

## Chapter 81: SQL for Analytics

### Introduction

SQL (Structured Query Language) is the lingua franca of data. While Pandas and Polars are powerful for in-memory analysis, most production data lives in databases (PostgreSQL, Snowflake, BigQuery, Redshift). Advanced SQL — window functions, CTEs, GROUP BY extensions, and query optimization — is essential for any data analyst or engineer.

### Why It Matters

SQL scales to terabytes and runs in-database, avoiding data movement. Window functions enable running totals, rankings, and moving averages without self-joins. CTEs enable readable, modular queries. Understanding query plans lets you write fast queries. SQL is not optional.

### Real World Analogy

Basic SQL is like being able to read a book. Window functions are like being able to read multiple pages simultaneously, remembering context as you go. CTEs are like bookmarking and referencing other sections. Explain plans are like X-ray vision into the engine.

> ✅ **Best Practice:** Window functions are almost always more efficient than self-joins for running totals, rankings, and moving averages. They process the data in a single pass rather than creating and scanning intermediate results. Master `ROW_NUMBER`, `LAG/LEAD`, and frame specifications for analytics queries.

### Theory

**Window Functions**: Perform a calculation across a set of rows related to the current row, without collapsing rows like GROUP BY. The OVER clause defines the window (partition, order, frame).

**Frame Specification**: `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (default), `RANGE`, `ROWS`, or `GROUPS` with `PRECEDING`, `FOLLOWING`, `UNBOUNDED PRECEDING`, `CURRENT ROW`.

**CTE (Common Table Expression)**: A named temporary result set within a query. Recursive CTEs allow traversal of hierarchical data (org charts, bill of materials).

**Grouping Extensions**: `GROUPING SETS` (explicit combination list), `CUBE` (all combinations), `ROLLUP` (hierarchical subtotals).

### SQL Examples

```sql
-- ===========================================
-- WINDOW FUNCTIONS
-- ===========================================

-- ROW_NUMBER: sequential integer per partition
SELECT
    region,
    sale_date,
    amount,
    ROW_NUMBER() OVER (PARTITION BY region ORDER BY amount DESC) AS rank_in_region
FROM sales;

-- RANK vs DENSE_RANK
SELECT
    product,
    amount,
    RANK() OVER (ORDER BY amount DESC) AS rank_skip,
    DENSE_RANK() OVER (ORDER BY amount DESC) AS rank_no_skip
FROM sales;
-- RANK: ties get same rank, next rank skips (1,1,3)
-- DENSE_RANK: ties get same rank, next rank does not skip (1,1,2)

-- NTILE: distribute rows into N buckets
SELECT
    sale_date,
    amount,
    NTILE(4) OVER (ORDER BY amount) AS quartile
FROM sales;

-- LAG / LEAD: access previous/next row
SELECT
    sale_date,
    amount,
    LAG(amount, 1) OVER (ORDER BY sale_date) AS prev_day_amount,
    LEAD(amount, 1) OVER (ORDER BY sale_date) AS next_day_amount,
    amount - LAG(amount, 1) OVER (ORDER BY sale_date) AS daily_change
FROM sales;

-- LAG with partition
SELECT
    region,
    sale_date,
    amount,
    LAG(amount) OVER (PARTITION BY region ORDER BY sale_date) AS prev_region_sale
FROM sales;

-- FIRST_VALUE / LAST_VALUE
SELECT
    region,
    sale_date,
    amount,
    FIRST_VALUE(amount) OVER (PARTITION BY region ORDER BY sale_date) AS first_sale_amount,
    LAST_VALUE(amount) OVER (
        PARTITION BY region
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS last_sale_amount
FROM sales;

-- ===========================================
-- CTEs (WITH clauses)
-- ===========================================

-- Simple CTE
WITH monthly_sales AS (
    SELECT
        DATE_TRUNC('month', sale_date) AS month,
        region,
        SUM(amount) AS total_sales
    FROM sales
    GROUP BY DATE_TRUNC('month', sale_date), region
)
SELECT
    month,
    region,
    total_sales,
    RANK() OVER (PARTITION BY month ORDER BY total_sales DESC) AS region_rank
FROM monthly_sales
ORDER BY month, region_rank;

-- Multiple CTEs
WITH
region_totals AS (
    SELECT region, SUM(amount) AS total FROM sales GROUP BY region
),
product_totals AS (
    SELECT product, SUM(amount) AS total FROM sales GROUP BY product
)
SELECT 'Regions' AS type, region AS name, total FROM region_totals
UNION ALL
SELECT 'Products' AS type, product AS name, total FROM product_totals;

-- Recursive CTE (hierarchy traversal)
WITH RECURSIVE org_hierarchy AS (
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    UNION ALL
    SELECT e.id, e.name, e.manager_id, oh.level + 1
    FROM employees e
    INNER JOIN org_hierarchy oh ON e.manager_id = oh.id
)
SELECT level, repeat('  ', level - 1) || name AS org_tree
FROM org_hierarchy
ORDER BY level, name;

-- ===========================================
-- ADVANCED GROUP BY
-- ===========================================

-- GROUPING SETS
SELECT region, product, SUM(amount) AS total_sales
FROM sales
GROUP BY GROUPING SETS ((region, product), (region), (product), ())
ORDER BY region, product;

-- CUBE (all combinations)
SELECT
    COALESCE(region, 'All Regions') AS region,
    COALESCE(product, 'All Products') AS product,
    SUM(amount) AS total_sales
FROM sales
GROUP BY CUBE (region, product)
ORDER BY region, product;

-- ROLLUP (hierarchical subtotals)
SELECT
    COALESCE(region, 'All Regions') AS region,
    COALESCE(product, 'All Products') AS product,
    SUM(amount) AS total_sales
FROM sales
GROUP BY ROLLUP (region, product)
ORDER BY region, product;

-- ===========================================
-- MOVING CALCULATIONS WITH FRAMES
-- ===========================================

-- 7-day moving average
SELECT
    sale_date,
    amount,
    AVG(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7d
FROM sales;

-- Running total (cumulative sum)
SELECT
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM sales;

-- ===========================================
-- COHORT ANALYSIS
-- ===========================================

WITH first_purchase AS (
    SELECT
        customer_id,
        MIN(sale_date) AS first_order_date,
        DATE_TRUNC('month', MIN(sale_date)) AS cohort_month
    FROM sales
    GROUP BY customer_id
),
monthly_activity AS (
    SELECT
        fp.cohort_month,
        DATE_TRUNC('month', s.sale_date) AS activity_month,
        COUNT(DISTINCT s.customer_id) AS active_customers
    FROM sales s
    INNER JOIN first_purchase fp ON s.customer_id = fp.customer_id
    GROUP BY fp.cohort_month, DATE_TRUNC('month', s.sale_date)
)
SELECT
    cohort_month,
    activity_month,
    active_customers,
    ROW_NUMBER() OVER (PARTITION BY cohort_month ORDER BY activity_month) - 1 AS period_number
FROM monthly_activity
ORDER BY cohort_month, activity_month;

-- ===========================================
-- STATISTICAL FUNCTIONS
-- ===========================================

SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY amount) AS median_amount,
    PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY amount) AS q1,
    PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY amount) AS q3,
    CORR(quantity, amount) AS correlation_qty_amount,
    STDDEV(amount) AS std_dev_amount,
    VARIANCE(amount) AS variance_amount,
    REGR_SLOPE(amount, quantity) AS slope_amount_vs_qty
FROM sales;

-- ===========================================
-- QUERY PERFORMANCE (PostgreSQL)
-- ===========================================

EXPLAIN ANALYZE
SELECT region, SUM(amount) AS total
FROM sales
WHERE sale_date >= '2024-01-01'
GROUP BY region;

-- Create indexes for performance
CREATE INDEX idx_sales_date ON sales(sale_date);
CREATE INDEX idx_sales_region_date ON sales(region, sale_date);
CREATE INDEX idx_sales_customer ON sales(customer_id);
```

### Query Optimization Strategies

1. **Use EXPLAIN ANALYZE**: Find sequential scans on large tables. Look for "Seq Scan" and "Sort" operations on high-row-count tables.
2. **Create appropriate indexes**: B-tree for equality/range queries, GiST for full-text, BRIN for time-series data. Covering indexes (`INCLUDE` columns) enable index-only scans.
3. **Select only needed columns**: Avoid `SELECT *` which retrieves all columns and prevents index-only scans.
4. **Filter early**: Push WHERE clauses as early as possible in the query plan. Use CTEs with filtering inside.
5. **Use materialized views**: For expensive aggregations queried frequently (e.g., dashboards). Refresh on schedule.
6. **Partition large tables**: Range partitioning by date allows partition pruning — the query planner skips irrelevant partitions.
7. **Analyze regularly**: `ANALYZE` updates table statistics for the query planner. Autovacuum should be configured properly.
8. **Connection pooling**: Use PgBouncer or similar to avoid connection overhead for short queries.
9. **Use `work_mem`**: Increase for queries involving large sorts or hash aggregations (set per-session).
10. **Avoid functions in WHERE**: `WHERE DATE(col) = '2024-01-01'` prevents index usage. Use `WHERE col >= '2024-01-01' AND col < '2024-01-02'`.

### Common Mistakes

| Mistake | Why It's Wrong | Fix |
|---------|---------------|-----|
| Forgetting `PARTITION BY` in window function | Window operates on entire table | Add `PARTITION BY` for grouped windows |
| Using `GROUP BY` and window function on same level | Logical conflict | Use subquery or CTE to separate concerns |
| No index on join columns | Full table scan | Create index on join/where columns |
| `SELECT *` in production | Retrieves unnecessary data, breaks on schema changes | Specify required columns |
| Not filtering before aggregation | Processes all rows unnecessarily | Filter (WHERE) before GROUP BY |
| Recursive CTE without termination condition | Infinite loop | Ensure UNION ALL eventually returns no rows |
| Using `ROLLUP` without COALESCE | NULL in subtotal rows is confusing | `COALESCE(col, 'All')` for display |
| Applying function in WHERE clause | Prevents index usage | Use range conditions instead |

### Best Practices

1. Use CTEs for readability — break complex queries into named steps
2. Add indexes on JOIN, WHERE, and ORDER BY columns
3. Use EXPLAIN ANALYZE before optimizing — assumptions about bottlenecks are often wrong
4. Prefer window functions over self-joins for running totals and rankings
5. Use materialized views for dashboard queries (refresh periodically)
6. Partition large tables by a logical key (date, region) for partition pruning
7. Set a `statement_timeout` to prevent runaway queries in production
8. Normalize to 3NF for OLTP, denormalize for OLAP/analytics

### Interview Questions

1. Explain the difference between ROW_NUMBER, RANK, and DENSE_RANK with an example
2. What is the difference between ROWS, RANGE, and GROUPS in window frame specification?
3. How would you write a query to compute a 7-day moving average?
4. What is a recursive CTE and when would you use it?
5. How would you identify the most common index-related performance issues?

### Practical Exercises

1. Write a query that assigns a rank to each product by sales within each region
2. Use a recursive CTE to traverse an organizational hierarchy
3. Compute a running total of sales by month, partitioned by product category
4. Write a query using CUBE to get totals at every combination of region and product

### Mini Project

**Sales Analytics Queries**: Given a sales table (sale_id, product_id, customer_id, region, sale_date, amount, quantity) and a customers table (customer_id, signup_date, segment): (1) Write a CTE that computes monthly revenue by region and segment, (2) Use window functions to rank products by revenue within each region showing top 3, (3) Compute month-over-month growth rate per region using LAG, (4) Build a cohort retention query (monthly active users by signup cohort), (5) Write a query to detect anomalies (amount > 3 standard deviations from rolling 30-day mean), (6) Compare performance with and without indexes using EXPLAIN ANALYZE.

### Revision Notes

| Concept | Key Function | Syntax |
|---------|-------------|--------|
| Row numbering | `ROW_NUMBER()` | `ROW_NUMBER() OVER (PARTITION BY x ORDER BY y)` |
| Rank | `RANK(), DENSE_RANK()` | `RANK() OVER (ORDER BY y DESC)` |
| Previous row | `LAG()` | `LAG(col, 1) OVER (ORDER BY date)` |
| Next row | `LEAD()` | `LEAD(col, 1) OVER (ORDER BY date)` |
| Quantile | `NTILE()` | `NTILE(4) OVER (ORDER BY y)` |
| CTE | `WITH` | `WITH cte AS (SELECT ...) SELECT * FROM cte` |
| Recursive CTE | `WITH RECURSIVE` | `WITH RECURSIVE cte AS (anchor UNION ALL recursive)` |
| Grouping sets | `GROUPING SETS, CUBE, ROLLUP` | `GROUP BY CUBE (a, b)` |
| Moving average | `AVG() OVER (ROWS BETWEEN N PRECEDING AND CURRENT ROW)` | `AVG(amount) OVER (ORDER BY date ROWS 6 PRECEDING)` |
| Percentile | `PERCENTILE_CONT, PERCENTILE_DISC` | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x)` |
| Correlation | `CORR()` | `CORR(x, y)` |
| Query plan | `EXPLAIN ANALYZE` | `EXPLAIN ANALYZE SELECT ...` |

---

# End of Part 11: Data Analytics
