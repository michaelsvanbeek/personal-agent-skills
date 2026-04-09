---
name: alternatives-analysis
description: >-
  Generic framework for comparing alternatives and making structured decisions.
  Use when: evaluating competing options for architecture, strategy, tools,
  vendors, dependencies, pricing models, or process changes; auditing an
  existing decision for gaps; choosing between build vs buy approaches; or
  presenting a recommendation with explicit trade-offs and evidence.
tags:
  - general
  - developer
  - manager
  - analyst
  - executive
---

# Alternatives Analysis

## When to Use

- Choosing between competing approaches, tools, vendors, architectures, or strategies
- Evaluating a proposed change where the do-nothing option exists
- Any decision where multiple stakeholders need to align on rationale
- Auditing a previous decision for completeness or bias
- Comparing options in technical, business, operational, or product contexts

This is a generic framework for structured comparison. Domain-specific skills
(competitive analysis, dependency management, database selection, and similar)
can apply this process within their own context.

---

## Core Principles

1. Include the requester's options, then expand them.
2. Assess impacts holistically across cost, risk, time, quality, maintainability,
   team capability, reversibility, and downstream effects.
3. Surface blind spots explicitly by stating assumptions and unknowns.
4. End with a clear, justified recommendation.

---

## Analysis Process

### Step 1: Frame the Decision

Before comparing options, define what is being decided and why.

```markdown
## Decision

**What**: [One sentence: what choice must be made]
**Why now**: [Trigger or deadline forcing this decision]
**Constraints**: [Budget, timeline, team size, compatibility, regulation, etc.]
**Success criteria**: [What a good outcome looks like - measurable when possible]
```

### Step 2: Enumerate Options

- Always include all user-provided options.
- Include a status quo/do-nothing option when change is not mandatory.
- Propose at least one additional option the requester may have missed.
- Cap the list at 5-7 viable options.

For each option, add a one-sentence concrete description.

### Step 3: Define Evaluation Criteria

Select criteria relevant to this decision and weight them.

| Criterion | What it measures | When it matters |
|-----------|------------------|-----------------|
| Cost | Upfront and ongoing financial impact | Budget-constrained decisions |
| Time to value | How fast value appears | Time-sensitive situations |
| Risk | Probability and severity of downside | High-stakes decisions |
| Reversibility | How easy it is to undo | One-way-door decisions |
| Quality/fit | How well it meets requirements | Specific requirement sets |
| Scalability | Performance under growth | Growth-stage decisions |
| Maintainability | Long-term operation effort | Multi-year impact |
| Team capability | Ability to execute with current team | Skill-constrained situations |
| Ecosystem compatibility | Integration with current systems | Platform/toolchain decisions |
| Vendor/dependency risk | Exposure to external control | Build vs buy/vendor choice |
| Opportunity cost | What is not done if this is chosen | Resource-constrained planning |
| Stakeholder alignment | How well it balances competing interests | Cross-functional decisions |
| Regulatory/compliance | Legal and policy obligations | Regulated contexts |

Use High/Medium/Low weights or numeric weights summing to 1.0.

### Step 4: Analyze Each Option

Evaluate each option against each selected criterion with evidence.

```markdown
### Option: [Name]

**Description**: [One sentence]

| Criterion | Assessment | Notes |
|-----------|------------|-------|
| Cost | $X upfront, $Y/mo ongoing | [Source or assumption] |
| Time to value | ~N weeks | [Prerequisites] |
| Risk | Medium - [specific risk] | [Mitigation] |
| ... | ... | ... |

**Strengths**: [Where this option is strongest]
**Weaknesses**: [Where it falls short]
**Assumptions**: [What must be true]
**Unknowns**: [What is uncertain]
```

### Step 5: Build the Summary Table

Put criteria on rows and options on columns for quick comparison.

```markdown
## Summary Comparison

| Criterion | Weight | Option A | Option B | Option C | Status Quo |
|-----------|--------|----------|----------|----------|------------|
| Cost | High | $$ | $ | $$$ | $0 |
| Time to value | High | 2 weeks | 6 weeks | 1 week | - |
| Risk | Med | Low | Med | High | Med (drift) |
| Reversibility | Med | Easy | Hard | Easy | - |
| Team capability | Low | Ready | Needs hire | Ready | - |
```

Guidelines:
- Include a weight column.
- Keep cell values concise.
- Fill every cell. If unknown, write Unknown.

---

## Output Format

Deliver the analysis in this order:

1. Decision frame
2. Summary comparison table
3. Detailed per-option analysis
4. Blind spots, assumptions, and unknowns
5. Final recommendation

Recommendation template:

```markdown
## Recommendation

**Choose**: [Option name]

**Why**: [2-4 sentences tied to weighted criteria and success criteria]

**What you give up**: [Best argument for the runner-up]

**Conditions**: [What must remain true; fallback if assumptions fail]
```

The recommendation should be falsifiable: it should be clear what evidence would
change the decision.

---

## Anti-Patterns

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| Straw-man options | Weak options make one option look best | Compare strongest realistic versions |
| Anchoring on first option | Relative comparison instead of criteria-based evaluation | Score independently against criteria |
| Missing status quo | Implies change is always required | Include do-nothing explicitly |
| Single-dimension comparison | Over-optimizes one factor | Use multiple weighted criteria |
| Analysis paralysis | Analysis cost exceeds decision value | Match depth to stakes |
| Hidden recommendation | Decision-maker cannot act | State clear recommendation early |
| Ignoring unknowns | Assumptions treated as facts | Label assumptions and unknowns |

---

## Scale Depth to Stakes

| Decision type | Depth | What to include |
|---------------|-------|-----------------|
| Low stakes, reversible | Light (10 min) | 2-3 options, 2-3 criteria, concise recommendation |
| Medium stakes | Standard (30-60 min) | 3-5 options, 4-6 weighted criteria, full table and rationale |
| High stakes, irreversible | Deep (hours-days) | 5-7 options, 6+ criteria, scenario analysis, risk register |

A fast structured comparison is usually better than an unstructured gut call.

---

## Decision Logging (Recommended)

Record significant decisions so teams can trace why a path was chosen.

Use a lightweight template in your repo docs (for example,
`docs/decisions/YYYY-MM-DD-<slug>.md`):

```markdown
# Decision: <title>

- Date: YYYY-MM-DD
- Context: [what triggered this decision]
- Options considered: [A, B, C, status quo]
- Recommendation: [chosen option]
- Why: [short rationale]
- Revisit when: [condition or date]
```

For repeated decision-making in the same project, maintain an index file that
links all decision records.

---

## Audit Checklist

- [ ] Decision is clearly framed with trigger, constraints, and success criteria
- [ ] All requester-provided options are included
- [ ] At least one additional option is considered
- [ ] Status quo is included or explicitly excluded with reason
- [ ] Criteria are relevant and weighted
- [ ] Every option is assessed against every criterion
- [ ] Summary comparison table is present near the top
- [ ] Blind spots, assumptions, and unknowns are explicit
- [ ] Recommendation is clear and justified
- [ ] Trade-offs are acknowledged
- [ ] Recommendation is falsifiable
- [ ] Analysis depth matches decision stakes
