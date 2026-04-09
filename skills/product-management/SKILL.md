---
name: product-management
description: >-
  Product roadmapping, prioritization, and specification writing. Use when:
  building a product roadmap, prioritizing features using RICE or other frameworks,
  writing product requirements documents (PRDs), creating user stories, defining
  acceptance criteria, planning sprints or milestones, evaluating feature requests,
  making build vs buy decisions for product features, auditing an existing roadmap
  for focus, or writing product specs for engineering handoff.
tags:
     - manager
     - operations
     - executive
---

# Product Management

## When to Use

- Building or updating a product roadmap
- Prioritizing a backlog of features or initiatives
- Writing a product requirements document (PRD) or product spec
- Creating user stories with clear acceptance criteria
- Planning milestones, sprints, or quarterly product goals
- Evaluating inbound feature requests from customers or stakeholders
- Making build vs buy decisions for product components
- Auditing an existing roadmap for focus, alignment, or missing rationale
- Communicating product plans to engineering, design, or leadership

---

## Prioritization Frameworks

### RICE Scoring

Score each initiative on four dimensions:

| Dimension | Definition | Scale |
|-----------|-----------|-------|
| **Reach** | How many users/customers will this impact per quarter? | Absolute number (e.g., 500 users) |
| **Impact** | How much will this move the target metric per user? | 3 = massive, 2 = high, 1 = medium, 0.5 = low, 0.25 = minimal |
| **Confidence** | How sure are you about reach and impact estimates? | 100% = high, 80% = medium, 50% = low |
| **Effort** | How many person-months will this take? | Absolute number (e.g., 2 person-months) |

```
RICE Score = (Reach × Impact × Confidence) / Effort
```

**Example**:

| Initiative | Reach | Impact | Confidence | Effort | RICE Score |
|-----------|-------|--------|-----------|--------|------------|
| API rate limiting | 2,000 | 2 | 80% | 1 | 3,200 |
| Dark mode | 5,000 | 0.5 | 100% | 2 | 1,250 |
| SSO support | 200 | 3 | 90% | 3 | 180 |
| Mobile app | 3,000 | 2 | 50% | 6 | 500 |

### ICE Scoring (Lightweight Alternative)

| Dimension | Definition | Scale |
|-----------|-----------|-------|
| **Impact** | How much will this improve the target metric? | 1-10 |
| **Confidence** | How sure are we about the impact estimate? | 1-10 |
| **Ease** | How easy is this to implement? | 1-10 |

```
ICE Score = Impact × Confidence × Ease
```

Best for quick triage when you don't have detailed reach data.

### MoSCoW Method

Categorize rather than score:

| Category | Definition | Rule |
|----------|-----------|------|
| **Must have** | Non-negotiable for this release; product fails without it | If you cut it and the release is useless, it's a Must |
| **Should have** | Important but not critical; workarounds exist | Significant value, but launch is viable without it |
| **Could have** | Nice-to-have; include if time allows | Incremental improvement, not essential |
| **Won't have** | Explicitly out of scope for this cycle | Prevents scope creep; document for future |

### Value vs Effort Matrix

For visual prioritization:

```
         High Value
              │
    Quick     │    Big Bets
    Wins ★    │    (plan carefully)
              │
──────────────┼──────────────
              │
    Fill-ins  │    Money Pit
    (maybe)   │    (avoid)
              │
         Low Value

    Low Effort ←──────→ High Effort
```

- **Quick wins** (high value, low effort): Do these first
- **Big bets** (high value, high effort): Plan, resource, and schedule these
- **Fill-ins** (low value, low effort): Only when nothing else is available
- **Money pit** (low value, high effort): Decline or defer indefinitely

---

## Roadmapping

### Roadmap Types

| Type | Time horizon | Audience | Detail level |
|------|-------------|----------|-------------|
| **Vision roadmap** | 12-24 months | Board, investors, company all-hands | Themes and strategic bets |
| **Strategic roadmap** | 6-12 months | Leadership, cross-functional teams | Initiatives with goals and rough timing |
| **Delivery roadmap** | 1-3 months | Engineering, design, QA | Epics, milestones, dependencies |
| **Now/Next/Later** | Rolling | Anyone | Simple three-column prioritization |

### Now / Next / Later

The simplest and most honest roadmap format:

| Column | Definition | Commitment level |
|--------|-----------|-----------------|
| **Now** | Currently in progress or about to start | High — committed, resourced |
| **Next** | Planned for the coming 1-2 months | Medium — sequenced, but may shift |
| **Later** | On the radar but not yet prioritized | Low — under consideration, no timeline |

### Roadmap Principles

- **Outcomes over outputs**: Frame roadmap items as problems to solve, not features to build
- **No fixed dates without scope trade-offs**: If the date is fixed, scope must flex (and vice versa)
- **Theme by strategic goal**: Group roadmap items under 3-5 strategic themes (growth, retention, platform, etc.)
- **Exclude the backlog**: A roadmap is not a project list; only include items with intentional prioritization
- **Review monthly**: Remove completed items, re-evaluate priorities, add new learnings

### Roadmap Anti-patterns

- **Feature factory**: Shipping features without connecting them to business outcomes
- **Date-driven**: Every item has a deadline with no scope flexibility
- **Stakeholder-driven**: Roadmap determined by who yells loudest, not by data
- **No pruning**: Items stay on the roadmap for years without review
- **Everything is P0**: No real prioritization — everything is "critical"

---

## Product Specification (PRD)

### PRD Template

```
# [Feature Name]

## Meta
- **Author**: [Name]
- **Status**: Draft / In Review / Approved
- **Target release**: [Quarter or milestone]
- **Last updated**: [Date]

## Problem Statement
[What problem does this solve? Who has this problem? How painful is it?
Include data: support tickets, user research, churn analysis.]

## Goal
[What outcome will this feature drive? Be specific and measurable.]

**Success metric**: [Primary metric this feature moves]
**Target**: [Quantified goal, e.g., "Reduce onboarding drop-off from 40% to 25%"]

## Non-Goals
[Explicitly state what this feature will NOT do. Prevents scope creep.]

- Not building [X]
- Not supporting [Y] in this iteration
- Not optimizing for [Z] persona

## User Stories

### Story 1: [Persona] [action]
**As a** [persona],
**I want to** [action],
**So that** [outcome].

**Acceptance criteria**:
- [ ] [Specific, testable criterion]
- [ ] [Specific, testable criterion]
- [ ] [Specific, testable criterion]

### Story 2: [Persona] [action]
[Same format]

## Design
[Link to designs, wireframes, or mockups.
Include key interaction flows and edge cases.]

## Technical Approach
[High-level technical direction agreed with engineering.
Not a full design doc — enough to unblock estimation.]

- Approach: [Brief description]
- Key dependencies: [APIs, services, libraries]
- Data model changes: [New tables, schema changes]
- Migration needed: [Yes/No — if yes, link to migration plan]

## Edge Cases
| Scenario | Expected behavior |
|----------|------------------|
| [Edge case 1] | [What happens] |
| [Edge case 2] | [What happens] |
| [Error condition] | [How the system responds] |

## Open Questions
- [ ] [Unresolved question 1]
- [ ] [Unresolved question 2]

## Rollout Plan
| Phase | Audience | Duration | Success criteria |
|-------|----------|----------|-----------------|
| Internal dogfood | Team | 1 week | No critical bugs |
| Beta | 10% of users | 2 weeks | [Metric threshold] |
| GA | All users | — | [Metric threshold] |

## Dependencies
| Dependency | Owner | Status | Blocking? |
|-----------|-------|--------|-----------|
| [API change] | [Team] | [Status] | Yes/No |
```

### Writing Good User Stories

| Element | Guideline |
|---------|---------|
| **Persona** | Specific role, not "the user" — "a team admin", "a free-tier developer" |
| **Action** | What they want to do, in their language |
| **Outcome** | The value they get, not the feature they use |
| **Acceptance criteria** | Testable, binary (pass/fail), and complete |
| **Size** | One story = one sprint or less; split larger stories |

### Acceptance Criteria Rules

- Write as **Given / When / Then** or **checklist format**
- Each criterion must be **independently verifiable**
- Include **happy path AND edge cases**
- Specify **error states and messages**
- Define **performance expectations** if relevant (e.g., "loads in <2s")
- NO implementation details — describe *what*, not *how*

---

## Feature Request Evaluation

When a feature request comes in (from customers, sales, or stakeholders):

| Question | Why it matters |
|----------|---------------|
| How many customers have asked for this? | Demand signal vs single-customer noise |
| What segment do they represent? | Enterprise ask vs SMB ask has different weight |
| What's the underlying problem? | The requested feature may not be the right solution |
| What's the cost of NOT doing this? | Churn risk, competitive disadvantage, or nothing |
| Does it align with current strategic themes? | Even good ideas can be wrong timing |
| What's the estimated effort? | Quick win or multi-quarter investment? |

### Decision: Build / Defer / Decline

| Decision | When | Communication |
|----------|------|---------------|
| **Build** | Aligns with strategy, clear demand, achievable effort | Add to roadmap with timeline |
| **Defer** | Good idea, wrong timing or dependencies needed first | Acknowledge, add to "Later", explain why |
| **Decline** | Doesn't align with strategy or serves too narrow an audience | Thank, explain positioning, suggest workaround |

---

## Product Audit Checklist

When reviewing an existing product roadmap or spec:

- [ ] Is every roadmap item tied to a measurable business outcome?
- [ ] Is there a prioritization framework applied (not just gut feeling)?
- [ ] Are non-goals explicitly stated in specs?
- [ ] Do user stories have acceptance criteria that QA can test independently?
- [ ] Is the roadmap reviewed and pruned at least monthly?
- [ ] Are feature requests evaluated against strategic themes before committing?
- [ ] Is scope separated from timeline (fixed date = flexible scope)?
- [ ] Are edge cases and error states documented in specs?
- [ ] Is there a rollout plan with phased rollout, not big-bang release?
- [ ] Is the roadmap communicated in a format appropriate for each audience?

---

## Output Format

When delivering product management artifacts, structure the output as:

1. **Context**: Product area, strategic theme, and current state
2. **Prioritized list**: Initiatives scored and ranked with chosen framework
3. **Roadmap**: Now/Next/Later or timeline view with rationale
4. **Spec** (if applicable): Full PRD following the template
5. **Risks and dependencies**: What could block delivery
6. **Success metrics**: How to measure whether this worked
