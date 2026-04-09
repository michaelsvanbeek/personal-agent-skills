---
name: roadmap-planning
description: >-
  Annual and quarterly roadmap planning for engineering teams. Use when: building
  or updating a team roadmap, prioritizing initiatives, mapping dependencies across
  teams, preparing roadmap presentations for leadership, evaluating trade-offs between
  competing priorities, or aligning roadmap items to company strategy and OKRs.
tags:
  - manager
  - executive
  - operations
---

# Roadmap Planning

## When to Use

- Building an annual or quarterly roadmap for your team
- Prioritizing competing initiatives when capacity is constrained
- Mapping cross-team dependencies for a planning cycle
- Preparing a roadmap presentation for leadership, stakeholders, or the team
- Re-evaluating priorities mid-cycle due to shifting business needs
- Translating company strategy into team-level initiatives

---

## Roadmap Anatomy

A roadmap is not a feature list. It communicates **what the team intends to accomplish, why, and in what sequence** — at a level of abstraction appropriate for the audience.

### Required Elements

| Element | Description |
|---------|-------------|
| **Theme** | High-level area of investment (e.g., "Platform reliability", "Self-serve onboarding") |
| **Initiative** | A scoped body of work within a theme, deliverable within a quarter |
| **Outcome** | The measurable result the initiative produces (tied to OKR or business metric) |
| **Owner** | The person or squad accountable for delivery |
| **Dependencies** | Teams, services, or decisions this initiative depends on |
| **Confidence** | High / Medium / Low — how certain is this commitment? |
| **Time horizon** | Now (this sprint), Next (this quarter), Later (future quarter) |

### What a Roadmap is NOT

- A Gantt chart with exact dates for every task
- A backlog sorted by priority
- A promise to stakeholders — it's a plan that evolves
- A list of features without outcomes

---

## Planning Process

### Step 1: Gather Inputs

Before planning, collect:

1. **Company/org OKRs** — What is the company betting on this quarter/year?
2. **Customer feedback** — Top pain points, feature requests, churn reasons
3. **Tech debt inventory** — What slows the team down? What's fragile?
4. **Stakeholder requests** — What do product, sales, support, and leadership need?
5. **Team retro themes** — Recurring process or tooling frustrations
6. **Carry-over** — Unfinished work from last cycle

### Step 2: Generate Candidates

List all potential initiatives. For each, capture:

```markdown
### Initiative: <Name>

- **Theme**: <which strategic area>
- **Problem**: <what problem does this solve>
- **Outcome**: <what measurable result>
- **Effort**: S / M / L / XL
- **Impact**: High / Medium / Low
- **Confidence**: High / Medium / Low
- **Dependencies**: <list any cross-team dependencies>
- **Requestor**: <who is asking for this>
```

### Step 3: Prioritize

Use a structured framework. Do not priority-sort by gut feeling alone.

#### RICE Framework (adapted)

| Factor | Definition | Scale |
|--------|-----------|-------|
| **Reach** | How many users/teams/customers are affected? | Number or % |
| **Impact** | How much does it improve the outcome? | 3 = massive, 2 = high, 1 = medium, 0.5 = low |
| **Confidence** | How sure are we about reach and impact? | 100% / 80% / 50% |
| **Effort** | Person-weeks to deliver | Number |

**RICE Score** = (Reach × Impact × Confidence) / Effort

#### Alternative: Value vs. Effort Matrix

For simpler prioritization, plot initiatives on a 2×2:

```
        High Value
            │
    DO FIRST│  PLAN CAREFULLY
   (quick   │  (high effort,
    wins)   │   high value)
────────────┼──────────────── Effort →
    FILL    │  AVOID / DEFER
   (low     │  (high effort,
    value,  │   low value)
    easy)   │
        Low Value
```

### Step 4: Allocate Capacity

Split team capacity deliberately:

| Category | Target % | Examples |
|----------|----------|---------|
| **Roadmap features** | 60–70% | New capabilities, customer-facing work |
| **Tech debt / platform** | 15–20% | Reliability, performance, developer experience |
| **Support / unplanned** | 10–15% | Bugs, incidents, ad hoc requests |
| **Growth / learning** | 5% | Spikes, prototypes, skill development |

Document the allocation in your quarterly plan so the team and stakeholders share expectations.

### Step 5: Sequence and Commit

Arrange initiatives into time horizons:

- **Now** (this sprint/month): Actively in progress, fully committed
- **Next** (this quarter): High confidence, planned but not yet started
- **Later** (future quarter): Lower confidence, directional

Only items in **Now** and **Next** should be broken into epics in your issue tracker. **Later** stays in the roadmap doc to signal direction.

---

## Roadmap Artifacts

### Quarterly Roadmap Brief

Create in your team's shared documentation tool (Confluence, Notion, Google Docs, etc.):

```markdown
# Q[N] YYYY Roadmap — [Team Name]

## Strategy Alignment
How this quarter's work connects to company OKRs.

## Themes
1. Theme A — why this matters
2. Theme B — why this matters

## Committed Initiatives
| Initiative | Theme | Owner | Outcome | Effort | Dependencies |
|-----------|-------|-------|---------|--------|-------------|
| ... | ... | ... | ... | ... | ... |

## Stretch / Aspirational
Initiatives we'd take on if capacity allows.

## What We're NOT Doing (and Why)
Explicitly list deprioritized requests and the reasoning.

## Risks and Dependencies
| Risk | Mitigation | Owner |
|------|-----------|-------|

## Capacity Allocation
Pie chart or table showing planned split.
```

### Roadmap Presentation (Leadership)

When presenting to leadership:

1. **Lead with outcomes**, not features — "Reduce onboarding time from 3 days to 3 hours" not "Build self-serve provisioning"
2. **Show trade-offs** — What you chose NOT to do and why
3. **Quantify confidence** — Don't present everything as a guarantee
4. **Highlight dependencies and risks** — Especially cross-team blockers
5. **Keep it to one page** — Executives skim; make it scannable

---

## Anti-Patterns

| Anti-Pattern | Why It's Bad | Better Approach |
|-------------|-------------|----------------|
| Overcommitting | Everything is "committed" with no slack | Leave 15–20% unplanned capacity |
| Feature factory | Roadmap is a list of features with no outcomes | Every item must have a measurable outcome |
| Planning in isolation | No input from team members on effort or feasibility | Include ICs in estimation and trade-off discussions |
| All or nothing | Only big bets, no quick wins | Mix initiative sizes for momentum and morale |
| Ignoring tech debt | Debt compounds silently until it explodes | Allocate 15–20% deliberately per cycle |
| Roadmap as contract | Treating roadmap as immutable promise | Review and adjust monthly; communicate changes |

---

## Review Cadence

| Frequency | Activity |
|-----------|----------|
| **Weekly** | Check initiative progress against plan in standup or status update |
| **Monthly** | Review roadmap health — are we on track? Any re-prioritization needed? |
| **Quarterly** | Full roadmap refresh — score last quarter, plan next quarter |
| **Annually** | Strategic planning — set themes and direction for the year |
