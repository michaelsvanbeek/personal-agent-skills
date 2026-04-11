---
name: anomaly-detection
description: >-
  Anomaly detection methodologies, strategy selection, and application.
  Use when: identifying outliers in data, detecting unusual patterns in metrics
  or logs, choosing between statistical and ML-based detection methods, building
  alerting thresholds, analyzing time series for anomalies, investigating
  suspicious data points, designing anomaly detection pipelines, or evaluating
  whether a detected "anomaly" is real or an artifact. Works on time series,
  tabular data, event streams, and metric dashboards.
tags:
  - analyst
  - developer
  - operations

# Anomaly Detection

Systematic approach to identifying unusual patterns in data. Covers methodology selection, threshold design, and interpretation — from simple statistical bounds to ML-based detection.

## When to Use

- Investigating unexpected spikes or drops in metrics
- Building alerting rules for operational dashboards
- Detecting fraud, abuse, or security incidents in event data
- Identifying data quality issues in pipelines
- Analyzing time series for drift or regime changes
- Evaluating whether a flagged data point is a true anomaly or noise
- Choosing the right detection method for a dataset
- Designing an anomaly detection pipeline from scratch

---

## Core Principles

1. **Define normal first** — You cannot detect anomalies without a clear model of what "normal" looks like. Start by characterizing the baseline.
2. **Context over thresholds** — A value that's anomalous in one context is expected in another. Day-of-week, seasonality, and deployment events all shift what's normal.
3. **False positives are expensive** — An alerting system that cries wolf gets ignored. Optimize for precision (low false positives) before recall (catching everything).
4. **Anomaly ≠ problem** — An anomaly is a statistical observation, not an automatic incident. Always pair detection with investigation.
5. **Simple methods first** — Z-scores and IQR catch most anomalies. Don't reach for Isolation Forests until simple methods have failed.
6. **Stationarity matters** — Most statistical methods assume your data doesn't change character over time. If it does, you need methods that handle trend, seasonality, or concept drift.

---

## Methodology Reference

### Tier 1: Statistical Methods (Start Here)

Use these when data is well-understood, roughly stationary, and you can define "normal" numerically.

| Method | How It Works | Best For | Assumptions | Limitations |
|--------|-------------|----------|-------------|-------------|
| **Z-Score** | Flag points > _k_ standard deviations from the mean | Univariate, roughly normal data | Normal distribution, no trend | Fails with skewed data, sensitive to outliers in training set |
| **Modified Z-Score (MAD)** | Uses median absolute deviation instead of std dev | Same as Z-score but robust to existing outliers | Symmetric distribution | Less sensitive than Z-score for normally distributed data |
| **IQR (Interquartile Range)** | Flag points outside Q1 − 1.5·IQR or Q3 + 1.5·IQR | Non-normal, skewed distributions | None (non-parametric) | Misses anomalies in heavy-tailed distributions |
| **Grubbs' Test** | Hypothesis test for a single outlier | Confirming a specific point is an outlier | Normal distribution, one outlier at a time | Only tests one point per run |
| **Control Charts (Shewhart)** | Mean ± 3σ boundaries on sequential data | Manufacturing, SPC, stable processes | Stationary process | Doesn't handle seasonality or trend |

**When to use Tier 1**: You have a single metric, the distribution is understandable, and you can eyeball what "normal" looks like. Most operational alerting starts here.

### Tier 2: Time Series Methods

Use when your data has trend, seasonality, or autocorrelation that invalidates simple statistical bounds.

| Method | How It Works | Best For | Assumptions | Limitations |
|--------|-------------|----------|-------------|-------------|
| **Moving Average ± Band** | Rolling mean ± _k_ · rolling std | Slow-moving metrics with noise | Local stationarity | Lags behind sudden shifts |
| **Exponential Smoothing (ETS)** | Weighted average favoring recent observations | Trended or seasonal univariate series | Consistent trend/seasonality pattern | Struggles with abrupt regime changes |
| **STL Decomposition + Residual** | Decompose into trend + seasonal + residual; flag large residuals | Strongly seasonal data (daily/weekly patterns) | Additive or multiplicative seasonality | Needs ≥ 2 full seasonal cycles |
| **ARIMA / SARIMA** | Fit autoregressive model; flag points outside prediction interval | Stationary or differenced series | Linear relationships, stationary residuals | Requires parameter tuning (p, d, q) |
| **Prophet** | Additive model with trend, seasonality, holidays | Business metrics with known calendar effects | Additive components | Can overfit with little data; slow on large series |
| **CUSUM / EWMA Charts** | Accumulate deviations from target; signal when cumulative sum exceeds threshold | Detecting small, persistent shifts | Stationary baseline | Not designed for sudden spikes (better for drift) |

**When to use Tier 2**: Your metric has daily, weekly, or seasonal patterns. A flat threshold would fire every Monday morning or every holiday.

### Tier 3: Machine Learning Methods

Use when the data is multivariate, the patterns are complex, or statistical methods can't capture what "normal" means.

| Method | How It Works | Best For | Pros | Cons |
|--------|-------------|----------|------|------|
| **Isolation Forest** | Randomly partitions feature space; anomalies are isolated in fewer splits | Tabular multivariate data | No distribution assumptions, handles high dimensionality | Requires training data, not inherently temporal |
| **Local Outlier Factor (LOF)** | Compares local density of a point to its neighbors | Clusters of varying density | Detects local anomalies that global methods miss | Expensive on large datasets; sensitive to _k_ |
| **One-Class SVM** | Learns a boundary around "normal" data | Small datasets with clean normal examples | Works in high dimensions | Sensitive to kernel choice, slow on large data |
| **Autoencoder (Neural)** | Learns to reconstruct normal data; high reconstruction error = anomaly | Complex, high-dimensional data (images, logs, embeddings) | Captures non-linear patterns | Needs significant normal data; opaque |
| **DBSCAN** | Density-based clustering; points not in any cluster are anomalies | Spatial or clustered data | No need to specify cluster count | Sensitive to epsilon and min_samples |

**When to use Tier 3**: Simple methods produce too many false positives, the data is multivariate, or "normal" is a complex region in feature space — not a simple range.

---

## Selection Strategy

Follow this decision tree to choose a method:

```
1. Is the data univariate (single metric)?
   ├── Yes → Is there seasonality or trend?
   │         ├── No  → Tier 1: Z-Score, IQR, or Control Chart
   │         └── Yes → Tier 2: STL + Residual, SARIMA, or Prophet
   └── No (multivariate) → 
       Is labeled anomaly data available?
       ├── Yes → Supervised classifier (beyond this skill's scope)
       └── No  → Tier 3: Isolation Forest, LOF, or Autoencoder

2. How much data is available?
   ├── < 100 points   → Tier 1 only (not enough for ML)
   ├── 100–10,000     → Tier 1 or Tier 2
   └── > 10,000       → Any tier is viable

3. What's the cost of a false positive vs. false negative?
   ├── FP is expensive (alert fatigue, manual investigation)
       → Favor higher thresholds, conservative methods (IQR, Grubbs')
   └── FN is expensive (missed fraud, missed outage)
       → Favor lower thresholds, ensemble methods
```

---

## Process

### Step 1: Characterize the Data

Before selecting a method, understand what you're working with:

```markdown
## Data Profile

**Source**: <e.g., "Prometheus metric: api_request_latency_p99">
**Type**: <univariate | multivariate> — <time series | tabular | event stream>
**Volume**: <row/point count, time range>
**Frequency**: <per-second | per-minute | daily | irregular>
**Distribution**: <roughly normal | skewed | heavy-tailed | multimodal | unknown>
**Stationarity**: <stationary | trend | seasonal | both | unknown>
**Known patterns**: <e.g., "higher on weekdays, spikes during deploys">
**Labeled anomalies available?**: <yes (n examples) | no>
```

### Step 2: Define What "Anomalous" Means

An anomaly is domain-specific. Define it before applying any method:

```markdown
## Anomaly Definition

**Point anomaly**: A single observation that deviates significantly
  → Example: "latency > 500ms when baseline is 50ms"

**Contextual anomaly**: Normal in one context, abnormal in another
  → Example: "1000 rps is normal at 2pm, anomalous at 3am"

**Collective anomaly**: A sequence of observations that together are unusual
  → Example: "5 consecutive requests from the same IP to /admin"

**Detection goal**: <Which type(s) are we looking for?>
**Action on detection**: <Alert | Log | Auto-remediate | Queue for investigation>
```

### Step 3: Establish the Baseline

Build a model of "normal" before looking for deviations:

- **For statistical methods**: Compute mean, median, std dev, percentiles on a _clean_ reference period (no known incidents)
- **For time series methods**: Decompose into trend + seasonality + residual on historical data
- **For ML methods**: Train on a dataset of known-good observations

```markdown
## Baseline

**Reference period**: <date range used for baseline>
**Exclusions**: <removed known incidents, maintenance windows, deploys>
**Statistics**: mean=<X>, median=<X>, std=<X>, p50=<X>, p95=<X>, p99=<X>
**Seasonality**: <daily cycle? weekly? none?>
```

### Step 4: Select and Apply the Method

Choose a method from the tiers above using the selection strategy. Document the choice:

```markdown
## Method Selection

**Method**: <name>
**Tier**: <1 | 2 | 3>
**Why this method**: <reasoning tied to data profile>
**Parameters**: <e.g., "Z-score threshold = 3.0", "IQR multiplier = 1.5", "Isolation Forest contamination = 0.01">
**Alternatives considered**: <what else was viable and why it wasn't chosen>
```

### Step 5: Evaluate Results

Every detection method needs calibration. Check these before trusting results:

| Check | What to Look For |
|-------|-----------------|
| **False positive rate** | Are flagged points actually anomalous? Manually inspect a sample. |
| **False negative rate** | Are known anomalies being caught? Test against historical incidents. |
| **Threshold sensitivity** | How much do results change with ±10% threshold adjustment? |
| **Temporal stability** | Does the method degrade over time as data distribution shifts? |
| **Edge behavior** | Does it flag every Monday? Every deploy? Every restart? (Contextual blindness) |

```markdown
## Evaluation

**Flagged anomalies**: <count>
**Manually verified (sample of N)**: <true positive rate>
**Known incidents caught**: <X of Y>
**False positive pattern**: <e.g., "flags every deploy; needs deploy-window exclusion">
**Recommended threshold adjustment**: <raise / lower / add context filter>
```

### Step 6: Operationalize (If Building a Pipeline)

For ongoing detection (not one-time analysis):

```markdown
## Operational Design

**Detection frequency**: <real-time | every 5 min | hourly | daily>
**Baseline refresh**: <static | rolling window of N days | retrain weekly>
**Alert routing**: <PagerDuty | Slack channel | log only>
**Suppression rules**: <e.g., "suppress during deploy windows", "deduplicate within 30 min">
**Escalation**: <when does a detected anomaly become an incident?>
**Dashboard**: <link to Grafana / CloudWatch / custom dashboard>
```

---

## Output Format

Every anomaly detection analysis should produce:

1. **Data Profile** — What the data looks like
2. **Anomaly Definition** — What counts as anomalous in this domain
3. **Baseline** — What "normal" looks like, with statistics
4. **Method Selection** — Which method, why, and with what parameters
5. **Results** — What was flagged, with evaluation of accuracy
6. **Operational Design** — How to run this ongoing (if applicable)

---

## Combining Methods

Real-world pipelines often layer methods for better results:

| Pattern | How It Works | When to Use |
|---------|-------------|-------------|
| **Decompose + Threshold** | STL/seasonal decomposition → Z-score or IQR on residuals | Seasonal data where raw Z-scores fire on every peak |
| **Ensemble voting** | Run 2–3 methods; flag only when ≥ 2 agree | Reducing false positives when no single method is reliable |
| **Tiered detection** | Tier 1 for fast screening → Tier 3 on flagged windows | High-volume data where ML on every point is too expensive |
| **Context layering** | Statistical method + deploy/event calendar → suppress known causes | Operational alerting where deploys and maintenance cause expected spikes |

**Rule of thumb**: Don't combine methods to increase sensitivity — combine them to increase precision (fewer false positives).

---

## Tooling Reference

| Library | Language | Best For |
|---------|----------|----------|
| `scipy.stats` | Python | Z-scores, Grubbs', distribution tests |
| `statsmodels` | Python | STL decomposition, ARIMA, ETS, control charts |
| `scikit-learn` | Python | Isolation Forest, LOF, One-Class SVM, DBSCAN |
| `prophet` | Python | Business metrics with calendar seasonality |
| `pyod` | Python | Unified API for 30+ anomaly detection algorithms |
| `adtk` | Python | Time series anomaly detection toolkit |
| `ruptures` | Python | Change point detection |
| Grafana ML | Dashboard | Built-in anomaly detection on Prometheus/InfluxDB metrics |

Use the **statistics** skill for assumption validation (normality tests, distribution checks) before applying Tier 1 methods.

---

## Common Pitfalls

| Pitfall | Why It Fails |
|---------|-------------|
| **Training on dirty data** | If the baseline includes anomalies, the model learns them as normal |
| **Flat thresholds on seasonal data** | A fixed "alert at > 100ms" fires every peak hour and gets ignored |
| **Ignoring concept drift** | A model trained 6 months ago may flag today's normal traffic as anomalous |
| **Over-alerting** | 50 alerts per day = zero alerts acted on. Fewer, high-confidence alerts win. |
| **Skipping manual inspection** | Auto-detected anomalies must be spot-checked. Precision matters. |
| **Using ML when Z-scores work** | Complexity for its own sake adds maintenance burden and opacity |
| **Confusing anomaly with root cause** | Detection tells you _something is unusual_, not _why_. Investigation is a separate step. |
| **No baseline exclusions** | Including Black Friday in baseline data makes next week look anomalous |
