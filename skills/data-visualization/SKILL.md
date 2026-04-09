---
name: data-visualization
description: >-
  Python data visualization for notebooks, scripts, and reports using matplotlib,
  seaborn, and Plotly. Use when: plotting analysis results in Jupyter/Marimo
  notebooks, choosing between static and interactive charts, creating publication-
  quality figures, building dashboard-style multi-panel layouts, exporting charts
  for reports or presentations, auditing existing plots for readability and
  accessibility, or improving visual consistency across an analysis project.
tags:
    - analyst
    - researcher
    - developer
---

# Python Data Visualization Standards

## When to Use

- Plotting analysis results in Jupyter or Marimo notebooks
- Choosing between static (matplotlib/seaborn) and interactive (Plotly) charts
- Creating publication-quality figures for reports or presentations
- Building multi-panel layouts for comparative analysis
- Exporting charts as PNG, SVG, or HTML
- Auditing existing plots for missing labels, poor color choices, or accessibility issues
- Standardizing visual style across a project's analysis outputs

> **For React web application charts** (Recharts, dark mode theming, CSS token integration), see the **charts skill** instead. This skill covers Python-native visualization for data science workflows.

---

## Library Selection

| Library | Best For | Output | Install |
|---------|----------|--------|---------|
| **matplotlib** | Full control, publication figures, custom layouts | Static (PNG, SVG, PDF) | `uv add matplotlib` |
| **seaborn** | Statistical plots, distribution analysis, beautiful defaults | Static (built on matplotlib) | `uv add seaborn` |
| **Plotly** | Interactive exploration, hover tooltips, dashboards | Interactive HTML, static export | `uv add plotly` |
| **Altair** | Declarative interactive exploration, web integration, layered charts | Interactive HTML (Vega-Lite) | `uv add altair` |

### Decision Guide

- **Default to seaborn** for notebook analysis — beautiful defaults, statistical awareness, minimal code.
- **Use matplotlib** when you need fine-grained control (custom annotations, multi-panel layouts, publication formatting).
- **Use Plotly** when interactivity matters for exploration — hover details, zoom, pan. Good for dashboards.
- **Use Altair** for declarative, composable interactive charts — excellent for quick exploratory plots and web integration without heavy dependencies.
- **Never use matplotlib defaults** without styling — always apply a theme.

---

## Chart Type Selection

| Chart Type | Use When | Library |
|------------|----------|---------|
| **Line** | Trends over time | All |
| **Bar** | Comparing categories | All |
| **Histogram** | Distribution of a single variable | seaborn (`histplot`) |
| **KDE / Density** | Smoothed distribution estimate | seaborn (`kdeplot`) |
| **Box / Violin** | Distribution comparison across groups | seaborn (`boxplot`, `violinplot`) |
| **Scatter** | Correlation between two variables | All |
| **Heatmap** | Correlation matrix, pivot table intensity | seaborn (`heatmap`) |
| **Pair plot** | All pairwise relationships in a dataset | seaborn (`pairplot`) — limit to ≤8 columns |
| **Facet grid** | Same chart repeated across subgroups | seaborn (`FacetGrid`) or Plotly (`facet_col`) |

### Anti-Patterns

- **Pie charts**: Avoid — humans compare angles poorly. Use horizontal bar charts instead.
- **3D charts**: Never — they distort proportions without adding information.
- **Dual Y-axes**: Misleading — use two separate subplots.
- **Truncated Y-axis on bar charts**: Start bars at 0 to avoid exaggerating differences.
- **Too many series**: Limit to ≤6 series per chart. Beyond that, use faceting or small multiples.

---

## Styling and Theming

### Global Style Setup

Apply at the top of every notebook or script:

```python
import matplotlib.pyplot as plt
import seaborn as sns

# Use seaborn's clean theme with matplotlib
sns.set_theme(
    style="whitegrid",
    palette="colorblind",  # accessible by default
    font_scale=1.1,
    rc={
        "figure.figsize": (10, 6),
        "axes.titlesize": 14,
        "axes.labelsize": 12,
        "figure.dpi": 150,
        "savefig.dpi": 300,
        "savefig.bbox": "tight",
    },
)
```

### Color Palettes

- **Default**: Use `"colorblind"` palette — accessible to colorblind readers.
- **Sequential**: `"Blues"`, `"Greens"` — for ordered data (low → high).
- **Diverging**: `"RdBu"`, `"coolwarm"` — for data with a meaningful center (e.g., correlation).
- **Categorical**: `"colorblind"`, `"Set2"` — for unordered groups.

```python
# Categorical comparison
sns.barplot(data=df, x="category", y="value", palette="colorblind")

# Sequential heatmap
sns.heatmap(corr_matrix, cmap="Blues", annot=True, fmt=".2f")

# Diverging (centered at 0)
sns.heatmap(corr_matrix, cmap="RdBu", center=0, annot=True, fmt=".2f")
```

**Rule**: Never use `"rainbow"` or `"jet"` colormaps — they are perceptually non-uniform and inaccessible.

---

## Axes, Labels, and Formatting

Every chart must have:

1. **Title** — describes what the chart shows, not how.
2. **Axis labels** with units — `"Revenue (USD)"`, not `"Y"`.
3. **Formatted tick values** — use locale-aware formatting for numbers and dates.

```python
import matplotlib.ticker as mticker

fig, ax = plt.subplots()
sns.barplot(data=df, x="month", y="revenue", ax=ax)

ax.set_title("Monthly Revenue by Region")
ax.set_xlabel("Month")
ax.set_ylabel("Revenue (USD)")

# Format y-axis as currency
ax.yaxis.set_major_formatter(mticker.FuncFormatter(lambda x, _: f"${x:,.0f}"))

# Rotate x-labels if needed (last resort — prefer shorter labels)
ax.tick_params(axis="x", rotation=45)

plt.tight_layout()
```

### Number Formatting

```python
from matplotlib.ticker import FuncFormatter, EngFormatter

# Thousands separator: 1,000,000
fmt_thousands = FuncFormatter(lambda x, _: f"{x:,.0f}")

# Engineering notation: 1M, 2.5k
fmt_engineering = EngFormatter(places=1)

# Percentage: 45.2%
fmt_pct = FuncFormatter(lambda x, _: f"{x:.1%}")
```

---

## Common Chart Recipes

### Time Series

```python
fig, ax = plt.subplots(figsize=(12, 5))
sns.lineplot(data=df, x="date", y="value", hue="category", ax=ax)
ax.set_title("Daily Active Users by Platform")
ax.set_ylabel("Users")
ax.legend(title="Platform", bbox_to_anchor=(1.02, 1), loc="upper left")
plt.tight_layout()
```

### Distribution Comparison

```python
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# Histogram + KDE
sns.histplot(data=df, x="amount", hue="plan", kde=True, ax=axes[0])
axes[0].set_title("Transaction Amount Distribution")

# Box plot comparison
sns.boxplot(data=df, x="plan", y="amount", ax=axes[1])
axes[1].set_title("Amount by Plan")

plt.tight_layout()
```

### Correlation Heatmap

```python
# Compute correlation on numeric columns
corr = df.select(pl.col(pl.NUMERIC_DTYPES)).to_pandas().corr()

fig, ax = plt.subplots(figsize=(8, 8))
mask = [[i < j for j in range(len(corr))] for i in range(len(corr))]
sns.heatmap(corr, mask=mask, annot=True, fmt=".2f", cmap="RdBu", center=0, ax=ax)
ax.set_title("Feature Correlation Matrix")
plt.tight_layout()
```

### Faceted Small Multiples

```python
g = sns.FacetGrid(df.to_pandas(), col="region", col_wrap=3, height=4)
g.map_dataframe(sns.histplot, x="amount", bins=30)
g.set_titles("{col_name}")
g.set_axis_labels("Amount (USD)", "Count")
g.tight_layout()
```

---

## Interactive Charts with Plotly

Use Plotly when interactivity adds value (hover details, zoom, filtering):

```python
import plotly.express as px

fig = px.scatter(
    df.to_pandas(),
    x="spend",
    y="revenue",
    color="category",
    size="users",
    hover_data=["name"],
    title="Spend vs Revenue by Category",
    labels={"spend": "Ad Spend (USD)", "revenue": "Revenue (USD)"},
)
fig.update_layout(template="plotly_white")
fig.show()
```

### Plotly Best Practices

- Use `template="plotly_white"` for clean, readable charts.
- Always set `labels={}` to override column names with human-readable labels.
- Include `hover_data` for context that doesn't fit on axes.
- Export static images for reports: `fig.write_image("chart.png", scale=2)`.

---

## Interactive Charts with Altair

Use Altair for **declarative, composable interactive charts** — especially for exploratory analysis and web integration:

```python
import altair as alt

chart = alt.Chart(df.to_pandas()).mark_circle(size=60).encode(
    x=alt.X("spend:Q", title="Ad Spend (USD)"),
    y=alt.Y("revenue:Q", title="Revenue (USD)"),
    color=alt.Color("category:N", title="Category"),
    tooltip=["name:N", "spend:Q", "revenue:Q"],
).interactive().properties(
    width=600,
    height=400,
    title="Spend vs Revenue by Category"
)
chart.show()
```

### Altair vs Plotly

- **Altair**: Smaller bundle size, pure Vega-Lite specification, excellent for composable/layered charts, native notebook integration.
- **Plotly**: More chart types, easier for imperative updates, good for dashboards.

### Altair Best Practices

- Use **encodings** (`x`, `y`, `color`, `size`) for declarative mapping — more readable than positional arguments.
- Add `.interactive()` to enable zoom and pan.
- Chain charts with `|` (side-by-side) or `&` (stacked) for dashboard layouts.
- Use `alt.condition()` for conditional encoding based on selection.

```python
# Layered chart with conditional color
selection = alt.selection_point(fields=["category"])
base = alt.Chart(df.to_pandas()).encode(
    x="spend:Q",
    y="revenue:Q",
    color=alt.condition(selection, "category:N", alt.value("lightgray")),
).add_params(selection)

base.mark_circle(size=60) | base.mark_line()
```

---

## Multi-Panel Layouts

### matplotlib Subplots

```python
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

sns.lineplot(data=df, x="date", y="revenue", ax=axes[0, 0])
axes[0, 0].set_title("Revenue Over Time")

sns.barplot(data=top_10, x="revenue", y="product", ax=axes[0, 1])
axes[0, 1].set_title("Top 10 Products")

sns.histplot(data=df, x="order_value", bins=50, ax=axes[1, 0])
axes[1, 0].set_title("Order Value Distribution")

sns.heatmap(pivot_table, ax=axes[1, 1], cmap="Blues")
axes[1, 1].set_title("Heatmap")

fig.suptitle("Sales Dashboard — Q1 2026", fontsize=16, y=1.02)
plt.tight_layout()
```

### Plotly Subplots

```python
from plotly.subplots import make_subplots
import plotly.graph_objects as go

fig = make_subplots(rows=1, cols=2, subplot_titles=("Revenue", "Users"))
fig.add_trace(go.Scatter(x=df["date"], y=df["revenue"], name="Revenue"), row=1, col=1)
fig.add_trace(go.Bar(x=df["date"], y=df["users"], name="Users"), row=1, col=2)
fig.update_layout(template="plotly_white", title="Key Metrics")
fig.show()
```

---

## Exporting

### Static Export (Reports, Presentations)

```python
# PNG (raster) — good for slides and documents
fig.savefig("output/chart.png", dpi=300, bbox_inches="tight")

# SVG (vector) — good for web, scales without pixelation
fig.savefig("output/chart.svg", bbox_inches="tight")

# PDF (vector) — good for print
fig.savefig("output/chart.pdf", bbox_inches="tight")
```

### Plotly Static Export

Requires `kaleido`: `uv add kaleido`.

```python
fig.write_image("output/chart.png", scale=2, width=1200, height=600)
fig.write_html("output/chart.html", include_plotlyjs="cdn")
```

### Altair Export

```python
# Export to interactive HTML
chart.save("output/chart.html")

# Export to static PNG (requires `altair_saver` or `selenium + geckodriver`)
chart.save("output/chart.png")

# Export to JSON specification for embedding in web apps
chart.to_json()
```

### Export Rules

- **PNG at 300 DPI** for presentations and documents.
- **SVG** for web embedding or when resolution independence matters.
- **HTML** for interactive Plotly charts shared with stakeholders.
- **Altair HTML** for lightweight interactive charts with minimal dependencies.
- **Never export at screen DPI (72/96)** — always set `dpi=300` or `scale=2`.

---

## Accessibility

- **Use `"colorblind"` palette** as the default — it is distinguishable by people with common color vision deficiencies.
- **Don't rely on color alone** — use markers, line styles, or annotations to differentiate series.
- **Add alt text** when embedding charts in documents or HTML.
- **Use sufficient contrast** — light gray on white is invisible to many readers.
- **Label directly on the chart** when possible instead of requiring a legend lookup.

```python
# Accessible line chart: different markers AND colors
markers = ["o", "s", "D", "^", "v"]
for i, (name, group) in enumerate(df.group_by("category")):
    ax.plot(group["x"], group["y"], marker=markers[i % len(markers)], label=name)
```

---

## Cross-References

- **Data analysis**: See the **data-analysis skill** for the DataFrame workflows that produce the data visualized here.
- **Charts (web)**: See the **charts skill** for React/Recharts visualization in web applications — theming, dark mode, tooltips, and CSS token integration.
- **Data pipelines**: See the **data-pipelines skill** for ingesting the data that feeds analysis and visualization.
- **Python conventions**: See the **python skill** for project setup, type hints, and Ruff configuration.
