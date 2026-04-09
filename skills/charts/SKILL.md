---
name: charts
description: 'Data visualization and charting best practices. Use when: choosing a chart type for data, configuring axes, adding tooltips, implementing chart theming for dark mode, connecting chart colors to application design tokens, formatting axis tick values, making charts accessible, auditing existing charts for readability and accessibility, or improving chart consistency across an application.'
tags:
  - developer
  - designer
  - analyst
---

# Chart Design and Implementation Standards

## When to Use

- Choosing the right chart type for a dataset
- Configuring axes with clear labels, appropriate tick density, and "nice" values
- Implementing chart tooltips with locale-formatted values
- Theming charts to match application dark/light mode and color system
- Making charts accessible to screen readers and keyboard users
- Auditing existing charts for readability, accessibility, and theming consistency
- Fixing charts that don't respect dark mode or use inconsistent colors

## Recommended Library

Use **Recharts** for React applications. It is composable, SSR-compatible, and exposes styling through React props and CSS variables, making theme integration straightforward.

```bash
npm install recharts
```

---

## Chart Type Selection

Choose the chart type that matches the structure and purpose of the data:

| Chart Type | Use When |
|------------|----------|
| **Line** | Continuous data over time; trends, progressions |
| **Area** | Like line, but emphasize magnitude or cumulative volume |
| **Bar (vertical)** | Comparing discrete categories; ranking |
| **Bar (horizontal)** | Comparing long category names; rankings where label length matters |
| **Stacked bar** | Part-to-whole comparison across multiple series |
| **Scatter** | Correlation between two continuous variables |
| **Bubble** | Correlation with a third magnitude dimension |
| **Pie / Donut** | Part-to-whole for a small number of categories (≤5); avoid for comparisons |
| **Heatmap** | Two-dimensional distribution, intensity, frequency matrix |
| **Histogram** | Distribution of a single continuous variable |
| **Gauge** | Single metric against a target or range; dashboards |

### Anti-patterns

- **Pie charts with >5 slices**: Humans cannot accurately compare angles. Use a bar chart instead.
- **3D charts**: Distort proportions without adding information. Always use 2D.
- **Dual Y-axes**: Misleading; viewers assume correlation. Use two separate charts.
- **Truncated Y-axis starting above zero for bar charts**: Exaggerates differences. Start bar axes at 0.
- **Line charts for categorical (unordered) data**: Lines imply continuity. Use bars instead.

---

## Axes

### Labels

- Every axis **must** have a label that includes the metric name and unit:
  - Good: `"Disk Usage (GB)"`, `"Response Time (ms)"`, `"Requests / sec"`
  - Bad: `"Value"`, `"Y"`, or no label at all
- Place x-axis label horizontally centered below the axis.
- Place y-axis label vertically on the left, rotated 90°, or as a header above the axis.

### "Nice" Values and Tick Density

- Axis bounds should snap to round numbers, not raw data min/max. This is called "nice" scaling.
  - If data ranges from 0–87, display 0–100 with ticks at 0, 25, 50, 75, 100.
  - If data ranges from 1230–1490, display 1200–1500 with ticks at 1200, 1300, 1400, 1500.
- Target **4–6 ticks** on any axis. Fewer feels sparse; more crowds the axis.
- Recharts exposes `tickCount` and `domain={['auto', 'auto']}` or `domain={[0, 'auto']}`. For explicit control:

```tsx
<YAxis
  tickCount={5}
  domain={[0, 'auto']}
  tickFormatter={(value) => formatMetric(value)}
  label={{ value: 'Disk Usage (GB)', angle: -90, position: 'insideLeft' }}
/>
```

- For time-based x-axes, use human-readable intervals: "Jan", "Feb", not timestamps.
- Avoid diagonal or rotated x-axis labels when possible. If labels are long, use horizontal bar chart or abbreviations.
- Ensure tick labels do not overlap. Reduce `tickCount` or rotate as a last resort.

### Zero Baseline

- Bar, area, and histogram charts: Y-axis **must start at 0** unless the range is inherently non-zero (e.g., temperature in Celsius).
- Line and scatter charts: Y-axis may start above 0 when the change relative to the visible range is meaningful.

---

## Tooltips

Every chart must include a tooltip that appears on hover/focus.

### Formatting Rules

- Format all numeric values using **`Intl.NumberFormat`** (locale-aware, not hardcoded separators).
- Format dates using **`Intl.DateTimeFormat`** matching the granularity of the x-axis.
- Include the metric name and unit in the tooltip, not just the number.

```tsx
const formatNumber = (value: number, unit?: string) =>
  new Intl.NumberFormat(undefined, { maximumFractionDigits: 2 }).format(value) +
  (unit ? ` ${unit}` : '');

const formatBytes = (bytes: number) => {
  if (bytes >= 1e9) return `${formatNumber(bytes / 1e9)} GB`;
  if (bytes >= 1e6) return `${formatNumber(bytes / 1e6)} MB`;
  return `${formatNumber(bytes / 1e3)} KB`;
};

const formatDate = (ts: number) =>
  new Intl.DateTimeFormat(undefined, { dateStyle: 'medium', timeStyle: 'short' }).format(ts);
```

### Tooltip Content

- Show the **label (x value)** prominently at the top.
- List each **series name** alongside its **formatted value**.
- For multi-series charts, match each label's color to the series line/bar color.
- Include comparison context when available (e.g., "vs. prior period: +12%").

```tsx
const ChartTooltip = ({ active, payload, label }: TooltipProps<number, string>) => {
  if (!active || !payload?.length) return null;
  return (
    <div className="chart-tooltip">
      <p className="chart-tooltip__label">{formatDate(label)}</p>
      {payload.map((entry) => (
        <p key={entry.name} style={{ color: entry.color }}>
          {entry.name}: {formatBytes(entry.value ?? 0)}
        </p>
      ))}
    </div>
  );
};

<Tooltip content={<ChartTooltip />} />
```

---

## Dark Mode and Theming

Charts must read from the application's design token system — never use hardcoded colors.

### CSS Variables Approach

Define chart-specific tokens alongside your global theme tokens:

```css
:root {
  --chart-axis-color: #6b7280;
  --chart-grid-color: #e5e7eb;
  --chart-tooltip-bg: #ffffff;
  --chart-tooltip-border: #e5e7eb;
  --chart-tooltip-text: #111827;

  /* Series palette — ordered by typical usage priority */
  --chart-color-1: #3b82f6;
  --chart-color-2: #10b981;
  --chart-color-3: #f59e0b;
  --chart-color-4: #ef4444;
  --chart-color-5: #8b5cf6;
  --chart-color-6: #06b6d4;
}

@media (prefers-color-scheme: dark) {
  :root {
    --chart-axis-color: #9ca3af;
    --chart-grid-color: #374151;
    --chart-tooltip-bg: #1f2937;
    --chart-tooltip-border: #374151;
    --chart-tooltip-text: #f9fafb;

    --chart-color-1: #60a5fa;
    --chart-color-2: #34d399;
    --chart-color-3: #fbbf24;
    --chart-color-4: #f87171;
    --chart-color-5: #a78bfa;
    --chart-color-6: #22d3ee;
  }
}
```

### Consuming Tokens in Recharts

Read CSS variable values at render time to pass as props:

```tsx
const useCSSVar = (name: string) =>
  getComputedStyle(document.documentElement).getPropertyValue(name).trim();

function ThemedChart() {
  const axisColor = useCSSVar('--chart-axis-color');
  const gridColor = useCSSVar('--chart-grid-color');
  const color1 = useCSSVar('--chart-color-1');

  return (
    <LineChart data={data}>
      <CartesianGrid stroke={gridColor} strokeDasharray="3 3" />
      <XAxis tick={{ fill: axisColor }} axisLine={{ stroke: axisColor }} />
      <YAxis tick={{ fill: axisColor }} axisLine={{ stroke: axisColor }} />
      <Line type="monotone" dataKey="value" stroke={color1} />
    </LineChart>
  );
}
```

### Tooltip Theming

Style the tooltip container via CSS classes that reference design tokens:

```css
.chart-tooltip {
  background: var(--chart-tooltip-bg);
  border: 1px solid var(--chart-tooltip-border);
  color: var(--chart-tooltip-text);
  border-radius: 6px;
  padding: 8px 12px;
  font-size: 13px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}
```

---

## Color Consistency

- Chart series colors come from the application's palette, not arbitrary choices.
- Use the same color for the same metric everywhere it appears across the application.
- Ensure meaningful category colors match usage elsewhere (e.g., "danger" series uses `--color-danger`).
- Use **distinct, perceptually separated colors** for multi-series charts. Avoid red/green together for accessibility.
- Provide visual differentiation beyond color: use different line dash patterns or marker shapes for line charts.
- Colorblind-safe palette starting point: blue, orange, teal, red, purple, brown (avoid red/green pairs).

---

## Accessibility

- Wrap each chart in a `<figure>` element with a `<figcaption>` describing what the chart shows.
- Add `role="img"` and `aria-label` to the chart's root SVG element with a summary of the data.
- For keyboard users, ensure tooltips are reachable via Tab and triggered on focus, not just mouse hover.
- Provide a **data table alternative** alongside complex charts, toggled by a "View as table" button.
- Do not rely solely on color to distinguish series — use labels, patterns, or direct annotations.

```tsx
<figure>
  <ResponsiveContainer width="100%" height={300}>
    <LineChart data={data} role="img" aria-label="Disk usage over the past 30 days, peaking at 87 GB on March 15.">
      {/* ... */}
    </LineChart>
  </ResponsiveContainer>
  <figcaption className="sr-only">
    Line chart showing disk usage from March 1 to March 30. Values range from 42 GB to 87 GB.
  </figcaption>
</figure>
```

---

## Responsiveness

- Always wrap Recharts in `<ResponsiveContainer width="100%" height={height}>` — never hardcode pixel widths.
- Choose `height` based on the chart type:
  - Line / Area / Bar: 200–350px
  - Scatter / Heatmap: 300–450px
  - Gauge / Pie (small): 150–200px
- On narrow viewports, reduce `tickCount`, hide secondary gridlines, or abbreviate labels.
- Legends should wrap below the chart on mobile rather than beside it.

---

## Legend

- Use a legend whenever there are 2 or more series.
- Place the legend **above or below** the chart, not to the right (wastes horizontal space and breaks responsiveness).
- Each legend entry should be clickable to toggle the corresponding series visibility.
- Match legend item color swatch to the exact series stroke/fill color.

---

## Empty and Loading States

- **Loading**: Show a pulsing skeleton in the chart area that matches the approximate shape of the expected chart.
- **Empty data**: Show a centered message inside the chart area explaining why there is no data and what the user can do (e.g., "No data for this period. Try expanding the date range.").
- **Partial data**: Render available data; annotate gaps with a visual break in the line or a note.

---

## Performance

- For large datasets (>500 points on a line chart), downsample before rendering. The user cannot distinguish individual points at that density.
- Use `isAnimationActive={false}` for charts that update frequently (dashboards, live data) to avoid re-animation on every data refresh.
- Memoize data transformation functions with `useMemo` to avoid unnecessary recalculations on unrelated re-renders.

---

## Cross-References

- **Python visualization**: For matplotlib, seaborn, and Plotly charts in Jupyter notebooks and Python scripts, see the **data-visualization skill**. This skill covers React/Recharts for web applications.
- **Data analysis**: See the **data-analysis skill** for the DataFrame workflows that produce the data visualized in charts.
