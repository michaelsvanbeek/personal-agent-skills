---
name: statistics
description: >-
  Statistical analysis and hypothesis testing for data-driven decisions.
  Use when: choosing the right statistical test for a question, calculating
  sample sizes, running A/B test analysis, comparing distributions, measuring
  correlation, building confidence intervals, validating assumptions before
  applying a test, interpreting p-values and effect sizes, or selecting the
  right summary statistics for a dataset. Covers descriptive statistics,
  hypothesis testing, regression basics, and practical interpretation.
tags:
  - analyst
  - researcher
  - developer

# Statistics

Select and apply the right statistical methods to answer a specific question. The goal is correct method selection and honest interpretation — not running every test and cherry-picking results.

## When to Use

- Deciding which statistical test applies to your data and question
- Running or interpreting A/B test results
- Comparing two or more groups (treatments, time periods, cohorts)
- Calculating sample sizes for an experiment
- Measuring relationships between variables
- Summarizing a dataset for stakeholders
- Validating whether assumptions hold before applying a method
- Interpreting p-values, confidence intervals, or effect sizes
- Choosing between parametric and non-parametric approaches

---

## Core Principles

1. **Question first, method second** — Define what you're trying to learn before selecting a test. The question determines the method, not the other way around.
2. **Check assumptions before computing** — Every test has assumptions (normality, independence, equal variance). Violating them silently produces misleading results.
3. **Effect size matters more than p-values** — A statistically significant result can be practically meaningless. Always report the magnitude of the effect alongside significance.
4. **Confidence intervals over point estimates** — A mean of 42 is less useful than "42 ± 5 (95% CI)". Always quantify uncertainty.
5. **Don't test what you can describe** — If the question is "what does the data look like?", use descriptive statistics. Hypothesis tests answer "is this difference real?", not "what is the data?"
6. **One question, one test** — Running multiple tests on the same data inflates false positives. If you must run multiple comparisons, correct for it (Bonferroni, Holm, or FDR).
7. **Honest reporting** — Report all results, not just significant ones. Pre-register your hypothesis when possible. State limitations clearly.

---

## Method Selection Guide

### Step 1: Classify Your Question

| Question Type | What You're Asking | Go To |
|--------------|-------------------|-------|
| **Describe** | What does the data look like? | Descriptive Statistics |
| **Compare** | Are these groups different? | Comparison Tests |
| **Relate** | Do these variables move together? | Correlation & Regression |
| **Predict** | What will happen next? | Regression & Modeling (beyond this skill — use ML) |
| **Classify** | Does this differ from expected? | Goodness-of-Fit Tests |

### Step 2: Identify Your Data Structure

| Factor | Options | Why It Matters |
|--------|---------|---------------|
| **Number of groups** | 1, 2, or 3+ | Determines which test family to use |
| **Paired or independent?** | Same subjects measured twice vs. different subjects | Paired tests are more powerful but require matched data |
| **Variable type** | Continuous (interval/ratio) vs. categorical (nominal/ordinal) | Determines parametric vs. non-parametric |
| **Sample size** | Small (< 30) vs. large (≥ 30) | Small samples need non-parametric or exact tests |
| **Distribution shape** | Normal vs. skewed vs. unknown | Parametric tests assume normality |

---

## Descriptive Statistics

Use these to summarize and understand data before testing anything.

### Central Tendency

| Measure | When to Use | Sensitive To |
|---------|------------|-------------|
| **Mean** | Symmetric, roughly normal data | Outliers (a single extreme value shifts it) |
| **Median** | Skewed data or when outliers are present | Nothing — robust by design |
| **Mode** | Categorical data or multimodal distributions | Ties, small samples |

**Rule**: If mean ≠ median by more than 10–15%, the data is skewed — prefer median.

### Spread

| Measure | When to Use | Notes |
|---------|------------|-------|
| **Standard deviation** | Normal-ish data; pair with mean | Same units as data |
| **IQR (Q3 − Q1)** | Skewed data; pair with median | Robust to outliers |
| **Range** | Quick sense of spread | Useless with outliers |
| **Coefficient of variation (CV)** | Comparing spread across datasets with different scales | CV = std / mean |

### Distribution Shape

| Check | Method | Interpretation |
|-------|--------|---------------|
| **Normality** | Shapiro-Wilk (n < 5000), Anderson-Darling, or Q-Q plot | p < 0.05 → not normal |
| **Skewness** | `scipy.stats.skew` | \|skew\| > 1 = substantially skewed |
| **Kurtosis** | `scipy.stats.kurtosis` | > 0 = heavy tails, < 0 = light tails |

---

## Comparison Tests

### Choosing the Right Test

```
How many groups?
├── 1 group vs. known value
│   ├── Normal → One-sample t-test
│   └── Not normal → Wilcoxon signed-rank
├── 2 groups
│   ├── Paired (same subjects)?
│   │   ├── Normal → Paired t-test
│   │   └── Not normal → Wilcoxon signed-rank
│   └── Independent?
│       ├── Normal, equal variance → Independent t-test
│       ├── Normal, unequal variance → Welch's t-test
│       └── Not normal → Mann-Whitney U
└── 3+ groups
    ├── Normal, equal variance → One-way ANOVA
    ├── Not normal → Kruskal-Wallis
    └── Repeated measures → Repeated measures ANOVA or Friedman test
```

### Test Reference

| Test | Compares | Assumptions | Non-Parametric Alternative |
|------|---------|-------------|--------------------------|
| **One-sample t-test** | Sample mean vs. known value | Normal, continuous | Wilcoxon signed-rank |
| **Independent t-test** | Means of 2 independent groups | Normal, equal variance, independent | Mann-Whitney U |
| **Welch's t-test** | Means of 2 independent groups | Normal, independent (no equal variance needed) | Mann-Whitney U |
| **Paired t-test** | Means of 2 paired measurements | Normal differences, continuous | Wilcoxon signed-rank |
| **One-way ANOVA** | Means of 3+ independent groups | Normal, equal variance, independent | Kruskal-Wallis |
| **Chi-squared test** | Observed vs. expected frequencies | Expected count ≥ 5 in each cell | Fisher's exact test |

### Post-Hoc Tests (After ANOVA / Kruskal-Wallis)

If the omnibus test is significant (groups differ), use post-hoc tests to find _which_ groups differ:

| Test | When to Use |
|------|------------|
| **Tukey HSD** | All pairwise comparisons, equal sample sizes |
| **Bonferroni correction** | Few planned comparisons |
| **Dunn's test** | Post-hoc for Kruskal-Wallis |

---

## Correlation and Association

| Method | Variable Types | Measures | Assumptions |
|--------|---------------|----------|-------------|
| **Pearson's r** | Both continuous | Linear relationship strength | Normal, linear, no outliers |
| **Spearman's ρ** | Ordinal or continuous | Monotonic relationship strength | None (rank-based) |
| **Kendall's τ** | Ordinal, small samples | Monotonic relationship strength | None (rank-based, more robust than Spearman for small n) |
| **Chi-squared test of independence** | Both categorical | Whether variables are associated | Expected counts ≥ 5 |
| **Point-biserial** | One binary, one continuous | Correlation | Normal continuous variable |

**Correlation ≠ causation.** Always state this explicitly when reporting. A correlation is a starting point for investigation, not a conclusion.

---

## Effect Size

Always report effect size alongside significance. A tiny p-value with a tiny effect is not actionable.

| Context | Measure | Small | Medium | Large |
|---------|---------|-------|--------|-------|
| 2-group comparison | **Cohen's d** | 0.2 | 0.5 | 0.8 |
| ANOVA | **Eta-squared (η²)** | 0.01 | 0.06 | 0.14 |
| Correlation | **r** or **R²** | 0.1 | 0.3 | 0.5 |
| Chi-squared | **Cramér's V** | 0.1 | 0.3 | 0.5 |
| Binary outcome | **Odds ratio** | 1.5 | 2.5 | 4.0 |

---

## Sample Size and Power

Before running an experiment, calculate the required sample size:

```markdown
## Power Analysis

**Test**: <e.g., "Independent t-test">
**Desired power**: 0.80 (standard) or 0.90 (high confidence)
**Significance level (α)**: 0.05
**Minimum detectable effect (MDE)**: <e.g., "Cohen's d = 0.3" or "5% conversion lift">
**Required sample size per group**: <computed value>
**Expected duration to collect**: <e.g., "2 weeks at current traffic">
```

**Rules of thumb**:
- Power < 0.80 = underpowered; you'll likely miss real effects
- Larger effects need smaller samples; smaller effects need larger samples
- When in doubt, aim for more data — underpowered studies waste everyone's time

---

## A/B Testing

A/B tests are the most common applied statistics scenario. Follow this workflow:

### Before the Test

1. **State the hypothesis**: "Variant B will increase conversion rate by ≥ 5% relative to control"
2. **Choose the metric**: Primary metric (what you're optimizing) + guardrail metrics (what must not degrade)
3. **Run power analysis**: Calculate required sample size per variant
4. **Set the duration**: Don't peek at results before the required sample size is reached
5. **Pre-register**: Document hypothesis, metric, sample size, and success criteria before launching

### During the Test

- **Don't peek**: Checking results daily and stopping early inflates false positives
- **Don't change the test mid-flight**: Adding variants, changing allocation, or modifying the feature during the test invalidates results
- **Monitor guardrails**: If error rates or latency spike, you have a quality issue, not a test result

### After the Test

```markdown
## A/B Test Results

**Test**: <name>
**Duration**: <start – end>
**Sample size**: Control: <n>, Variant: <n>

**Primary metric**: <metric name>
| Group | Value | 95% CI |
|-------|-------|--------|
| Control | <X> | [<lo>, <hi>] |
| Variant | <X> | [<lo>, <hi>] |

**Relative lift**: <X%> [<CI lo>, <CI hi>]
**p-value**: <X>
**Effect size**: <measure and value>

**Guardrail metrics**: <all stable / degradation in X>

**Decision**: <Ship variant | Keep control | Inconclusive — extend or redesign>
**Reasoning**: <why this decision, referencing effect size and practical significance>
```

---

## Assumption Validation Checklist

Before applying any parametric test, verify:

| Assumption | How to Check | If Violated |
|-----------|-------------|-------------|
| **Normality** | Shapiro-Wilk test, Q-Q plot, histogram | Use non-parametric alternative |
| **Equal variance** | Levene's test, F-test | Use Welch's t-test or rank-based test |
| **Independence** | Study design (not a statistical test) | Use paired/repeated measures test |
| **Linearity** (regression) | Scatter plot of residuals vs. fitted | Transform variables or use non-linear model |
| **No multicollinearity** (regression) | VIF < 5 for each predictor | Remove or combine correlated predictors |

---

## Process

### Step 1: State the Question

```markdown
## Statistical Question

**Question**: <What are you trying to learn? Be specific.>
**Population**: <Who or what does this apply to?>
**Hypothesis** (if testing): 
  - H₀: <null hypothesis — no effect, no difference>
  - H₁: <alternative hypothesis — the effect you expect>
**Practical significance threshold**: <What size effect would actually matter?>
```

### Step 2: Profile the Data

```markdown
## Data Profile

**Source**: <where the data comes from>
**Sample size**: <n>
**Variables**: 
  - <var1>: <type> — <description>
  - <var2>: <type> — <description>
**Missing data**: <count, percentage, pattern>
**Distribution**: <per variable — normal, skewed, categorical frequencies>
**Assumption checks**: <normality, variance, independence — pass/fail>
```

### Step 3: Select the Method

Use the selection guide above. Document:

```markdown
## Method Selection

**Test**: <name>
**Why**: <reasoning tied to question type, data structure, and assumption checks>
**Alternative considered**: <what else could work and why it wasn't chosen>
**Correction applied**: <e.g., "Bonferroni for 3 pairwise comparisons" or "none — single test">
```

### Step 4: Compute and Interpret

```markdown
## Results

**Test statistic**: <name> = <value>
**p-value**: <value>
**Effect size**: <measure> = <value> (<small | medium | large>)
**Confidence interval**: [<lower>, <upper>] at <confidence level>

**Interpretation**: <Plain-language statement of what this means. Not just "p < 0.05 so we reject H₀" — state the practical implication.>

**Limitations**: <What this result does NOT tell us. Confounders, generalizability, assumptions that were borderline.>
```

---

## Output Format

Every statistical analysis should produce:

1. **Question** — What you're trying to learn, with hypothesis if applicable
2. **Data Profile** — Summary of the data, assumption checks
3. **Method Selection** — Which test, why, and what alternatives were considered
4. **Results** — Test statistic, p-value, effect size, confidence interval
5. **Interpretation** — Plain-language meaning and practical significance
6. **Limitations** — What the result doesn't tell you

---

## Regression Basics

Regression is the most common applied statistics method. Use this decision guide:

| Outcome Variable | Predictor(s) | Method | Use When |
|-----------------|-------------|--------|----------|
| Continuous | 1 continuous | **Simple linear regression** | Predicting one variable from another (e.g., revenue from ad spend) |
| Continuous | Multiple | **Multiple linear regression** | Predicting with several factors; controlling for confounders |
| Binary (0/1) | Any | **Logistic regression** | Predicting yes/no outcomes (e.g., churn, conversion) |
| Count (0, 1, 2...) | Any | **Poisson regression** | Predicting event counts (e.g., support tickets per day) |

**Before running regression, check**:
- Linearity (scatter plot of residuals vs. fitted)
- Independence of residuals
- Homoscedasticity (constant variance of residuals)
- No multicollinearity (VIF < 5 for each predictor)
- Normality of residuals (for inference, not prediction)

**Key outputs to report**: R², adjusted R², coefficients with CIs, residual plots, and F-test p-value.

---

## When to Consider Bayesian Methods

Frequentist methods (everything above) are the default. Consider Bayesian approaches when:

| Signal | Why Bayesian Helps |
|--------|-------------------|
| Small sample size (n < 30) | Priors regularize unstable estimates |
| You have strong prior knowledge | Incorporating domain expertise improves estimates |
| You need P(hypothesis \| data), not P(data \| hypothesis) | Credible intervals answer "what's the probability the effect is > X?" directly |
| Sequential testing / continuous monitoring | Bayesian A/B tests allow peeking without inflating error rates |
| Stakeholders struggle with p-values | "95% probability the effect is between 2% and 8%" is more intuitive |

**Practical tools**: `pymc`, `arviz` for general Bayesian analysis; Bayesian A/B testing via `bayesian-testing` or built-in platform features (Optimizely, LaunchDarkly).

**Default stance**: Use frequentist methods unless one of the above signals is present. Don't switch to Bayesian for complexity's sake.

---

## Tooling Reference

| Library | Language | Best For |
|---------|----------|----------|
| `scipy.stats` | Python | Hypothesis tests, distributions, descriptive stats |
| `statsmodels` | Python | Regression, ANOVA, time series, assumption diagnostics |
| `pingouin` | Python | Clean API for t-tests, ANOVA, correlation, effect sizes |
| `scikit-learn` | Python | Train/test splits, cross-validation, preprocessing |
| `pymc` | Python | Bayesian modeling and inference |
| `power_analysis` / `statsmodels.stats.power` | Python | Sample size and power calculations |

---

## Common Pitfalls

| Pitfall | Why It Fails |
|---------|-------------|
| **p-hacking** | Running multiple tests and reporting only significant results inflates false positives. Pre-register your hypothesis. |
| **Ignoring effect size** | p = 0.001 with Cohen's d = 0.05 means "we're very sure about a trivially small difference." Not actionable. |
| **Small sample, big claims** | A study with n = 12 that finds p = 0.04 is fragile. One outlier changes the conclusion. |
| **Violating independence** | Using the same users in both groups, or multiple measurements without paired tests. Results are invalid. |
| **Peeking at A/B tests** | Checking daily and stopping when p < 0.05 dramatically inflates false positive rate. Use sequential testing if you must peek. |
| **Treating non-significant as "no effect"** | Absence of evidence ≠ evidence of absence. You may be underpowered. Report power. |
| **Applying parametric tests to ordinal data** | A Likert scale (1–5) is not continuous. Use non-parametric methods. |
| **Confusing correlation with causation** | Pearson r = 0.8 does not mean X causes Y. It means they move together. |
| **Cherry-picking subgroups** | "It wasn't significant overall, but it was significant for users aged 25–30 on Tuesdays." This is noise. |
| **Reporting without uncertainty** | "Conversion rate is 4.2%" is less useful than "4.2% ± 0.8% (95% CI)." Always show the interval. |
