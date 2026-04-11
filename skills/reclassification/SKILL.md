---
name: reclassification
description: >-
  Critically evaluate and improve an existing classification system or taxonomy.
  Use when: reviewing how items are grouped (code modules, document categories,
  data labels, skill folders, config buckets, CSV/JSON columns, API enums),
  challenging whether current groupings still serve their purpose, merging
  overlapping categories, splitting overloaded ones, renaming for clarity, or
  redistributing items to better align with downstream use cases. Works on any
  classifiable data: code, markdown, structured data (CSV, JSON), taxonomies,
  hierarchical systems, or free-form lists.
tags:
  - general
  - analyst
  - developer
---

# Reclassification

Critically evaluate an existing classification system and produce an improved version. Applies to any domain where items are grouped into categories: code modules, document folders, data labels, skill libraries, config buckets, or structured datasets.

## When to Use

- A classification system has grown organically and feels inconsistent
- Categories overlap, are too broad, or are too granular
- New items don't fit cleanly into existing groups
- Downstream consumers (users, pipelines, agents) struggle with current groupings
- You need to audit whether a taxonomy still serves its original purpose
- Merging two classification systems after a team or project merge
- Preparing data for a migration where clean categories matter
- Evaluating whether skills, tools, or documents should be consolidated or split

---

## Core Principles

1. **Understand before changing** — Reconstruct the reasoning behind the current classification before proposing changes. Every existing grouping had a reason, even if that reason is no longer valid.
2. **Classify for the consumer** — Optimize groupings for the downstream use case (discoverability, routing, filtering, agent selection), not for the author's mental model.
3. **Mutual exclusivity** — Each item should belong to exactly one category. If an item fits two categories equally well, the boundaries are wrong.
4. **Collectively exhaustive** — Every item must have a home. If orphans exist, the taxonomy has gaps.
5. **Right-sized groups** — Categories that are too large lose signal; categories that are too small add navigation cost. Aim for balance within stated constraints.
6. **Names carry meaning** — A category name should let someone predict its contents without reading a description. Rename when the name misleads.
7. **Stable under growth** — A good taxonomy accommodates new items without restructuring. If every new addition requires a new category, the scheme is too rigid.

---

## Inputs

Gather these before starting. Not all are required — work with what's available.

| Input | Required | Description |
|-------|----------|-------------|
| **Items** | Yes | The things being classified (files, rows, labels, skills, endpoints, etc.) |
| **Current classifications** | Yes | How items are currently grouped (folder structure, labels, column values, tags) |
| **Downstream use cases** | Recommended | Who consumes these classifications and for what purpose |
| **Min category size** | Optional | Minimum number of items per category (default: 2) |
| **Max category size** | Optional | Maximum number of items per category (default: no limit) |
| **Split threshold** | Optional | When a category exceeds this size or has clear sub-clusters, consider splitting |
| **Merge threshold** | Optional | When two categories have fewer than this many items each and overlap, consider merging |
| **Constraints** | Optional | Fixed categories that cannot be renamed/removed, naming conventions, external dependencies |

---

## Process

### Step 1: Inventory the Current State

List every item and its current classification. For large datasets, summarize with counts.

```markdown
## Current Classification Inventory

| Category | Count | Sample Items |
|----------|-------|-------------|
| <category-1> | <n> | <item-a>, <item-b>, <item-c> |
| <category-2> | <n> | <item-d>, <item-e> |
| ... | ... | ... |

**Total items**: <N>
**Total categories**: <N>
**Uncategorized items**: <N or "none">
```

### Step 2: Reconstruct the Classification Logic

Before critiquing, understand the system:

- **What was the likely organizing principle?** (by function, by audience, by lifecycle stage, by domain, alphabetical, chronological)
- **What use case was it designed for?** (navigation, search, routing, reporting, access control)
- **When was it created?** Has the scope of items changed significantly since then?
- **Who classified the items?** One person's mental model or a team convention?
- **Is there a logic at all?** Many classifications are accidental — items added over time with no governing principle. This is a valid finding.

Document this as:

```markdown
## Classification Logic (Reconstructed)

**Organizing principle**: <e.g., "grouped by functional domain" | "no coherent principle — items added ad hoc">
**Original use case**: <e.g., "help users find the right skill by topic">
**Classifier**: <e.g., "single author, added incrementally over 6 months">
**Scope drift**: <e.g., "started with 10 items, now 35 — original structure may not scale">
**Coherence**: <high | medium | low | none — how consistently was the principle applied?>
```

If no coherent logic exists, note that explicitly. An accidental classification is the strongest signal that reclassification will add value.

### Step 3: Evaluate Each Category

For every category, answer these questions:

| Question | What you're checking |
|----------|---------------------|
| Is the name accurate? | Does the name describe what's actually inside? |
| Is it internally coherent? | Do all items share a clear, common trait? |
| Is it distinct from neighbors? | Could items be confused with another category? |
| Is it right-sized? | Too few items (< min threshold) or too many (> split threshold)? |
| Does it serve the downstream use case? | Would a consumer find what they need here? |

**Quantitative signals** — When working with structured data (CSV, JSON, database rows) or large item sets (50+), compute these before relying on qualitative judgment:

- **Distribution**: Item count per category. Flag categories with < `min_category_size` or > `split_threshold`.
- **Overlap rate**: How many items could plausibly fit in two or more categories? High overlap (>20%) signals boundary problems.
- **Orphan rate**: How many items are uncategorized or in a catch-all? High orphan rate signals taxonomy gaps.
- **Name–content alignment**: For text items, check whether the category name appears in or relates to the item content.

Produce a diagnostic table:

```markdown
## Category Evaluation

| Category | Accurate Name? | Coherent? | Distinct? | Size | Serves Use Case? | Issue |
|----------|---------------|-----------|-----------|------|-------------------|-------|
| <cat-1> | ✅ | ✅ | ⚠️ overlaps with <cat-3> | 12 | ✅ | Boundary issue |
| <cat-2> | ❌ name misleads | ✅ | ✅ | 3 | ⚠️ | Rename candidate |
| <cat-3> | ✅ | ❌ mixed concerns | ⚠️ | 18 | ❌ | Split candidate |
| ... | ... | ... | ... | ... | ... | ... |
```

### Step 4: Identify Reclassification Actions

Based on the evaluation, propose specific actions. Each action must have a rationale tied to a principle or downstream use case.

#### Action Types

| Action | When to Apply | Template |
|--------|--------------|----------|
| **Merge** | Two categories overlap significantly or are both undersized | Merge `<A>` + `<B>` → `<New Name>` because <reason> |
| **Split** | A category contains distinct sub-clusters or exceeds the split threshold | Split `<A>` → `<A1>` + `<A2>` because <reason> |
| **Rename** | The name doesn't match the contents or is ambiguous to consumers | Rename `<Old>` → `<New>` because <reason> |
| **Redistribute** | Individual items are miscategorized | Move `<item>` from `<A>` to `<B>` because <reason> |
| **Create** | Orphaned items share a trait that deserves its own category | Create `<New>` for `<item-1>`, `<item-2>` because <reason> |
| **Dissolve** | A category adds no signal — its items fit better elsewhere | Dissolve `<A>`, redistribute items to `<B>`, `<C>` because <reason> |
| **Keep** | The category is well-formed and serves its purpose | Keep `<A>` as-is |

```markdown
## Proposed Actions

| # | Action | Detail | Rationale |
|---|--------|--------|-----------|
| 1 | Merge | `planning` + `roadmap-planning` → `planning` | 80% content overlap; consumers can't distinguish them |
| 2 | Split | `comms` → `internal-comms` + `external-comms` | Distinct audiences with different voice rules |
| 3 | Rename | `general-coding` → `project-setup` | Contents are scaffolding conventions, not general coding |
| 4 | Keep | `security` | Well-scoped, distinct, right-sized |
```

### Step 5: Propose the New Classification

Present the full reclassified structure side by side with the original.

```markdown
## Proposed Classification

| New Category | Items | Source (Original Category) |
|-------------|-------|---------------------------|
| <new-cat-1> | <item-a>, <item-b> | was <old-cat-1> |
| <new-cat-2> | <item-c>, <item-d>, <item-e> | split from <old-cat-3> |
| ... | ... | ... |

**Total items**: <N> (should match original)
**Total categories**: <N> (was <old N>)
```

### Step 6: Validate the Proposal

Run these checks before finalizing:

| Check | Pass? |
|-------|-------|
| Every item from the original inventory appears exactly once | |
| No category is below the minimum size threshold | |
| No category is above the maximum size threshold (if set) | |
| Every category name is unambiguous to the downstream consumer | |
| The taxonomy is stable — a plausible new item would fit without restructuring | |
| The number of actions is proportional to the problems found (don't over-reorganize) | |

### Step 7: Iterate if Needed

Reclassification is rarely one-pass. Before finalizing:

- **Present the proposal to stakeholders** (or re-read it yourself after a break). Fresh eyes catch category names that feel obvious to the author but confuse consumers.
- **Test with new items**: Take 3–5 items that don't exist yet (or were recently added) and try to classify them under the proposed scheme. If placement is ambiguous, the boundaries need work.
- **Adjust and re-validate**: Loop back to Step 6 after changes. Don't skip validation on the revised version.

### Step 8: Present the Migration Path

If the reclassification will be implemented (not just theoretical), outline the steps:

```markdown
## Migration Path

1. <action — e.g., "Rename `general-coding/` to `project-setup/`">
2. <action — e.g., "Move items X, Y from `comms/` to new `external-comms/`">
3. <action — e.g., "Update references in README.md and any index files">
4. <action — e.g., "Delete empty `old-category/` directory">

**Breaking changes**: <list any downstream references, configs, or integrations that will break>
**Rollback**: <how to revert if the new classification causes problems>
```

---

## Output Format

Every reclassification analysis should produce these sections:

1. **Current Classification Inventory** — What exists today, with counts
2. **Classification Logic** — Reconstructed rationale for the original scheme
3. **Category Evaluation** — Per-category diagnostic table (with quantitative signals for large datasets)
4. **Proposed Actions** — Specific merge/split/rename/redistribute/create/dissolve actions with rationale
5. **Proposed Classification** — The full new structure with item mapping
6. **Validation Checklist** — Confirm the proposal is sound
7. **Iteration** — Test with new items, gather feedback, adjust
8. **Migration Path** — Steps to implement (if applicable)

---

## Tuning Parameters

These defaults can be overridden by the user:

| Parameter | Default | Effect |
|-----------|---------|--------|
| `min_category_size` | 2 | Categories below this trigger a merge review |
| `max_category_size` | none | Categories above this trigger a split review |
| `split_threshold` | 15 | Suggest splitting when a category exceeds this count |
| `merge_threshold` | 3 | Suggest merging when two related categories are each below this count |
| `rename_confidence` | high | Only rename when the mismatch between name and contents is clear |
| `preserve_list` | [] | Category names that must not be changed (external dependencies) |

---

## Flat vs. Hierarchical

Not every classification needs to be flat. Consider hierarchy when:

| Signal | Recommendation |
|--------|---------------|
| < 20 categories, each well-sized | **Stay flat** — hierarchy adds navigation cost for little benefit |
| > 20 categories with natural parent groups | **Add one level** — group related categories under parents |
| Items have a clear primary axis + secondary axis | **Use hierarchy for primary, tags for secondary** |
| Consumers navigate top-down (browse) | Hierarchy helps |
| Consumers search or filter (query) | Flat + tags is often better |

Never go deeper than two levels. If you need three, the top level is too granular.

---

## Applying to Structured Data (CSV, JSON, Database)

When reclassifying a column in a dataset rather than a folder structure:

1. **Inventory** = `SELECT category, COUNT(*) FROM data GROUP BY category ORDER BY COUNT(*) DESC`
2. **Evaluate** = Look at value distribution, null rates, and sample rows per category
3. **Actions** = Express as a mapping table: `old_value → new_value`
4. **Validate** = Run the mapping and verify counts match, no NULLs introduced, downstream queries still work
5. **Migrate** = Provide the SQL `UPDATE` or Python/pandas transformation, not just the conceptual mapping

```markdown
## Value Mapping

| Old Value | New Value | Count | Rationale |
|-----------|-----------|-------|-----------|
| "bug" | "defect" | 142 | Align with established terminology |
| "enhancement" | "feature" | 89 | Merge: distinction wasn't meaningful for reporting |
| "task" | "task" | 204 | Keep as-is |
| "question" | → dissolve | 12 | Redistribute: 8 are actually bugs, 4 are feature requests |
```

---

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|-------------|-------------|
| **Reclassify everything** | Massive reorganizations break references and confuse users. Change only what's broken. |
| **Classify by implementation** | Grouping by technology instead of user need creates categories that don't help consumers. |
| **One item per category** | Hyper-granular taxonomies add navigation cost without adding signal. |
| **Catch-all categories** | "Other" or "Misc" categories are where items go to die. If items don't fit, the taxonomy has gaps. |
| **Rename without redirects** | Renaming breaks links, configs, and muscle memory. Always provide a migration path. |
| **Optimize for the author** | The author's mental model is not the consumer's. Classify for retrieval, not for filing. |
| **Skip the inventory** | Proposing changes without counting what exists leads to misjudged merges and splits. |
