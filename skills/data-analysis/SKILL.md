---
name: data-analysis
description: >-
  Data analysis workflows with pandas, Polars, and DuckDB for exploration,
  cleaning, and transformation. Use when: performing exploratory data analysis,
  cleaning messy datasets, computing aggregations and statistics, writing
  Jupyter/Marimo notebook workflows, choosing between pandas and Polars,
  querying Parquet files with DuckDB, building reproducible analysis notebooks,
  auditing existing analysis code for performance or correctness, or designing
  post-pipeline transform steps.
tags:
    - analyst
    - researcher
    - developer
---

# Data Analysis Standards

## When to Use

- Exploring a new dataset (shape, distributions, quality)
- Cleaning and transforming data after pipeline ingestion
- Computing aggregations, groupings, and statistical summaries
- Writing Jupyter or Marimo notebooks for analysis
- Querying Parquet or CSV files with DuckDB SQL
- Choosing between pandas, Polars, and DuckDB for a task
- Building reproducible analysis workflows
- Auditing existing notebooks or analysis scripts for correctness, performance, or readability

---

## Tool Selection

| Tool | Best For | Install |
|------|----------|---------|
| **Polars** | Large datasets, performance-critical transforms, lazy evaluation, type safety | `uv add polars` |
| **pandas** | Quick exploration, broad ecosystem compatibility, small-to-medium data | `uv add pandas` |
| **DuckDB** | SQL-based analytics on Parquet/CSV without loading into memory | `uv add duckdb` |

### Decision Guide

- **Default to Polars** for any repeatable analysis or transform — faster, stricter types, lower memory.
- **Use pandas** when a downstream library requires it (sklearn, certain plotting libraries) or for quick one-off exploration.
- **Use DuckDB SQL** when the question is naturally expressed in SQL (aggregations, joins, window functions) and data lives in Parquet files.
- **Combine freely**: Polars and DuckDB share Apache Arrow as the memory format — zero-copy conversion between them.

---

## Exploratory Data Analysis (EDA) Workflow

Follow this sequence when encountering a new dataset:

### 1. Shape and Schema

```python
import polars as pl

df = pl.read_parquet("data/raw/users.parquet")

print(f"Rows: {df.height:,}, Columns: {df.width}")
print(df.schema)
print(df.head(5))
```

### 2. Null and Completeness Audit

```python
null_report = df.select(
    pl.all().null_count().name.suffix("_nulls"),
).unpivot().sort("value", descending=True)
print(null_report)

# Percentage complete
completeness = df.select(
    (1 - pl.all().null_count() / df.height).name.suffix("_complete"),
).unpivot().sort("value")
print(completeness)
```

### 3. Descriptive Statistics

```python
# Numeric columns
print(df.select(pl.col(pl.NUMERIC_DTYPES)).describe())

# Categorical value counts
for col in df.select(pl.col(pl.String)).columns:
    print(f"\n{col}:")
    print(df.group_by(col).len().sort("len", descending=True).head(10))
```

### 4. Distributions and Outliers

```python
# Quick histogram via DuckDB for large data
import duckdb

duckdb.sql("""
    SELECT histogram(amount, 20) AS amount_hist
    FROM 'data/raw/orders.parquet'
""").show()

# Outlier detection with IQR
q1, q3 = df.select(
    pl.col("amount").quantile(0.25).alias("q1"),
    pl.col("amount").quantile(0.75).alias("q3"),
).row(0)
iqr = q3 - q1
outliers = df.filter(
    (pl.col("amount") < q1 - 1.5 * iqr) | (pl.col("amount") > q3 + 1.5 * iqr)
)
print(f"Outliers: {outliers.height} rows ({outliers.height / df.height:.1%})")
```

### 5. Temporal Patterns

```python
daily = (
    df.with_columns(pl.col("created_at").cast(pl.Date).alias("date"))
    .group_by("date")
    .agg(pl.len().alias("count"))
    .sort("date")
)
print(daily)
```

---

## Data Cleaning Patterns

### Standard Cleaning Pipeline

```python
def clean_users(df: pl.DataFrame) -> pl.DataFrame:
    """Clean raw user data."""
    return (
        df
        # Drop exact duplicates
        .unique()
        # Normalize text columns
        .with_columns(
            pl.col("email").str.to_lowercase().str.strip_chars(),
            pl.col("name").str.strip_chars(),
        )
        # Filter invalid rows
        .filter(pl.col("email").str.contains(r"^[^@]+@[^@]+\.[^@]+$"))
        # Cast types
        .with_columns(
            pl.col("created_at").str.to_datetime("%Y-%m-%dT%H:%M:%SZ"),
        )
        # Drop columns not needed downstream
        .drop("raw_html", "debug_info")
    )
```

### Handling Missing Data

| Strategy | When |
|----------|------|
| **Drop rows** | Missing data is rare (<5%) and random |
| **Fill with default** | Business logic defines a sensible default (e.g., 0 for missing counts) |
| **Forward/backward fill** | Time-series data with sparse readings |
| **Flag as missing** | Downstream analysis needs to distinguish "0" from "unknown" |

```python
# Fill with default
df = df.with_columns(pl.col("country").fill_null("Unknown"))

# Forward fill time-series
df = df.sort("timestamp").with_columns(pl.col("temperature").forward_fill())

# Flag missing
df = df.with_columns(pl.col("amount").is_null().alias("amount_missing"))
```

---

## Aggregation Patterns

### Group-By Aggregations

```python
summary = (
    df.group_by("category")
    .agg(
        pl.len().alias("count"),
        pl.col("amount").sum().alias("total"),
        pl.col("amount").mean().alias("avg"),
        pl.col("amount").median().alias("median"),
        pl.col("amount").std().alias("std"),
        pl.col("amount").quantile(0.95).alias("p95"),
    )
    .sort("total", descending=True)
)
```

### Window Functions

```python
df = df.with_columns(
    # Running total per user
    pl.col("amount")
    .cum_sum()
    .over("user_id")
    .alias("running_total"),
    # Rank within category
    pl.col("amount")
    .rank(method="dense", descending=True)
    .over("category")
    .alias("rank_in_category"),
)
```

### Time-Series Resampling

```python
monthly = (
    df.sort("date")
    .group_by_dynamic("date", every="1mo")
    .agg(
        pl.col("amount").sum().alias("monthly_total"),
        pl.col("user_id").n_unique().alias("unique_users"),
    )
)
```

---

## DuckDB SQL for Analysis

Query Parquet files directly without loading into Python memory:

```python
import duckdb

# Aggregate across partitioned Parquet files
result = duckdb.sql("""
    SELECT
        date_trunc('month', created_at) AS month,
        country,
        count(*) AS orders,
        sum(amount) AS revenue,
        avg(amount) AS avg_order_value
    FROM 'data/raw/orders/**/*.parquet'
    GROUP BY 1, 2
    ORDER BY 1, 4 DESC
""")

# Convert to Polars for further manipulation
df = result.pl()
```

### DuckDB + Polars Integration

```python
import duckdb
import polars as pl

# Register Polars DataFrame as a DuckDB table
df = pl.read_parquet("data/cleaned/users.parquet")
duckdb.sql("SELECT * FROM df WHERE country = 'US' LIMIT 10").show()
```

DuckDB automatically detects Polars and pandas DataFrames by variable name — no explicit registration needed.

---

## Notebook Best Practices

### Structure

Every analysis notebook should follow this structure:

1. **Title and objective** — Markdown cell stating what question the notebook answers.
2. **Imports and config** — Single cell with all imports; set display options.
3. **Data loading** — Load from Parquet (pipeline output), not raw APIs.
4. **Exploration** — EDA steps (shape, nulls, distributions).
5. **Analysis** — The actual computation answering the objective.
6. **Visualization** — Charts supporting the conclusions (see **data-visualization skill**).
7. **Conclusions** — Markdown cell summarizing findings.

### Config Cell Template

```python
import polars as pl
import duckdb
from pathlib import Path

# Display settings
pl.Config.set_tbl_rows(20)
pl.Config.set_fmt_str_lengths(80)

DATA_DIR = Path("data")
```

### Rules

- **Never call APIs from notebooks** — load from pipeline outputs (Parquet files). Pipelines are the ingestion layer; notebooks are the analysis layer.
- **Use relative paths from the project root**, not absolute paths.
- **Each notebook answers one question** — don't create monolithic analysis notebooks.
- **Clear outputs before committing** — notebooks with output diffs are unreadable in git.
- **Name notebooks descriptively**: `01_user_churn_analysis.ipynb`, not `Untitled3.ipynb`.
- **Pin the Python kernel** to the project's `uv` environment.

---

## Performance Tips

### Polars Lazy Evaluation

For large datasets, use lazy evaluation to let Polars optimize the query plan:

```python
result = (
    pl.scan_parquet("data/raw/events/*.parquet")  # lazy — no data loaded yet
    .filter(pl.col("event_type") == "purchase")
    .group_by("user_id")
    .agg(pl.col("amount").sum().alias("total_spent"))
    .sort("total_spent", descending=True)
    .head(100)
    .collect()  # executes the optimized plan
)
```

### Memory Management

- **Use `scan_parquet` (lazy)** instead of `read_parquet` (eager) for datasets larger than available RAM.
- **Select only needed columns**: `pl.scan_parquet(...).select(["id", "amount"])` avoids loading unused columns.
- **Filter early**: Push filters before aggregations to reduce data volume.
- **Stream large results**: Use `collect(streaming=True)` for datasets that don't fit in memory even after filtering.

---

## Cross-References

- **Data pipelines**: See the **data-pipelines skill** for dlt-based ingestion that produces the Parquet files analyzed here.
- **Data visualization**: See the **data-visualization skill** for plotting analysis results in notebooks.
- **Charts (web)**: See the **charts skill** for React/Recharts visualization when building web dashboards from analysis outputs.
- **Python conventions**: See the **python skill** for type hints, project setup, and Ruff configuration.
- **Database queries**: See the **database skill** when analysis requires querying PostgreSQL or DynamoDB directly.
