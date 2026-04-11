---
name: goals
description: >-
  OKRs and contribution goals — writing, cascading, scoring, and individual goal setting.
  Use when: setting team OKRs, writing individual contribution goals, cascading company
  OKRs to team level, coaching someone on goal quality, preparing for OKR reviews,
  scoring OKRs at end of cycle, or linking contribution goals to team OKRs. Covers
  both team-level OKRs and individual contribution goal frameworks.
tags:
  - manager
  - operations
  - executive
---

# Goals — OKRs and Contribution Goals

## When to Use

- Setting team OKRs at the start of a quarter
- Writing individual contribution goals
- Cascading company OKRs to team and individual level
- Coaching a team member on goal quality
- Preparing for mid-cycle or end-of-cycle reviews
- Scoring OKRs or assessing contribution goal attainment

---

## Two Goal Systems, One Framework

| System | Scope | Where It Lives | Audience |
|--------|-------|----------------|----------|
| **Team OKRs** | Team-level outcomes for the quarter | Planning docs, shared trackers | Team, PM, leadership |
| **Contribution Goals** | Individual's commitments for the cycle | Goal-tracking system (e.g., Workday, Lattice, 15Five) | Employee, manager, HR |

Both follow the same quality bar: specific, measurable, outcome-oriented, and time-bound. Contribution goals should cascade from team OKRs — they are an individual's slice of the team's commitments.

---

## Team OKRs

### Objectives

An objective is **what** the team wants to achieve:

- Qualitative and directional — a destination, not a number
- One sentence, no jargon
- Scoped to the quarter
- Within the team's influence

**Good**: "Establish our platform as the most reliable in the industry"
**Bad**: "Improve uptime SLA metrics by optimizing infrastructure monitoring and alerting"

### Key Results

A key result is **how** you measure progress:

- Quantitative — a number, percentage, or binary deliverable
- Has a baseline ("from X") and a target ("to Y")
- Measures outcomes, not activities
- 70% completion is healthy; 100% means you aimed too low

**Good**: "Reduce P1 incident count from 8/quarter to 2/quarter"
**Bad**: "Implement better monitoring" (activity, not outcome)

### Structure

```
Objective: <What we want to achieve>
  KR1: <Metric from X to Y>
  KR2: <Metric from X to Y>
  KR3: <Binary: Ship X by date>
```

Aim for **2–4 objectives** with **2–5 key results each** per team per quarter.

### Cascading

```
Company OKR
  └─ Org OKR (how this org contributes)
       └─ Team OKR (how this team contributes)
            └─ Contribution Goal (how this person contributes)
```

**Rules**:
- Align, don't duplicate — your team OKR contributes to the org OKR, not repeats it
- Own your scope — within the team's control
- Not everything cascades — 1 locally-important OKR per quarter is fine
- Individual goals connect to team OKRs but are personalized

---

## Contribution Goals

Contribution goals are entered into your goal-tracking system by the employee. They should read like individual OKRs — specific to that person's involvement in team priorities.

### The Unambiguity Test

Before submitting a goal, apply this test:

> **If ten people read your goal and independently assessed whether you achieved it, would they all reach the same conclusion?**

If reasonable people could disagree about whether the goal was met, it's too vague. Sharpen it.

### The Differentiation Test

> **If you might have a similar goal last quarter or next quarter, what makes THIS quarter different?**

Goals should reflect the specific context, deliverables, and impact of the current cycle. "Improve code quality" is evergreen and therefore useless. "Reduce escaped defects in the payments service from 12/quarter to 3/quarter" is anchored to now.

### Goal Categories

| Category | % of Goals | Examples |
|----------|-----------|---------|
| **Delivery / Business Impact** | 50–60% | Ship feature X, improve metric Y by Z% |
| **Technical / Craft** | 20–30% | Reduce tech debt in service A, establish testing practices |
| **Growth / Development** | 10–20% | Lead a project end-to-end, mentor a junior engineer |
| **Team / Culture** | 0–10% | Improve onboarding docs, drive hiring for a role |

### Writing Quality: What Good Looks Like

Each goal needs:

- **What**: Specific deliverable or measurable outcome
- **How you'll know**: Quantified success criteria
- **Why it matters**: Connection to team OKR or business priority
- **When**: Target date or quarter

#### Template

```markdown
## Goal: [Title]

### Description
[What will be done and why it matters — 1-2 sentences]

### Success Criteria
- [ ] [Specific, measurable outcome — passes the unambiguity test]
- [ ] [Specific, measurable outcome]

### Alignment
Supports: [Team OKR or business priority]

### Target
[Quarter or specific date]
```

#### Examples by Level

**Mid-level Engineer**:
- "Deliver the payments integration end-to-end, handling 10K transactions/day by end of Q2. Success: integration live in production, processing real transactions, with <0.1% error rate."
- "Reduce flaky test rate in CI from 15% to <2% by mid-quarter. Success: 2-week rolling flaky rate below 2%."

**Senior Engineer**:
- "Design and ship multi-tenant data isolation. Success: architecture doc approved, implementation deployed, zero cross-tenant data leaks in pen testing."
- "Establish observability best practices for the team. Success: every service has dashboards, alerting, and a runbook. MTTR reduced from 45min to 15min."

**Staff Engineer**:
- "Define technical strategy for event-driven architecture migration. Success: strategy doc approved by architecture review, migration plan with milestones published, first service migrated."

### Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| "Improve reliability" | Not measurable — improve how much? | "Reduce P1 incidents from 8 to 2 per quarter" |
| "Work on the migration" | Activity, not outcome | "Complete phase 1 migration: 3 services on new platform by Q2" |
| "Be a better mentor" | Vague, fails unambiguity test | "Mentor [Name] through their first feature lead; they ship independently by end of quarter" |
| "Support the team" | Evergreen, no differentiation | "Drive adoption of contract testing; 80% of inter-service calls covered by Q2" |
| Same goal every quarter | Fails differentiation test | Anchor to THIS quarter's specific deliverables |

---

## OKR Quality Checklist

| Check | ✓ |
|-------|---|
| Objective is qualitative and directional | |
| Objective fits in one sentence | |
| Each KR has a baseline ("from X") and target ("to Y") | |
| KRs measure outcomes, not activities | |
| KRs are within the team's control | |
| 2–5 KRs per objective | |
| 2–4 objectives per team | |
| At least one KR is a stretch | |

## Contribution Goal Checklist

| Check | ✓ |
|-------|---|
| Passes the unambiguity test (10 assessors agree) | |
| Passes the differentiation test (specific to this quarter) | |
| Has quantified success criteria | |
| Aligned to a team OKR or stated business priority | |
| Has a target date | |
| Tracked in goal-tracking system | |

---

## Scoring

### Team OKR Scoring (end of quarter)

| Score | Meaning |
|-------|---------|
| 0.0–0.3 | Failed to make meaningful progress |
| 0.4–0.6 | Made progress but fell short |
| 0.7–0.9 | Hit or came close — the sweet spot |
| 1.0 | Fully achieved (if all KRs score 1.0, you aimed too low) |

### Scoring Process

1. Self-score each KR with evidence
2. Team review — discuss and calibrate
3. Document lessons learned
4. Feed forward into next cycle's planning

### Contribution Goal Assessment

Assessed during performance review cycles:

| Rating | Criteria |
|--------|---------|
| Exceeded | Delivered beyond the stated success criteria |
| Met | Achieved what was committed |
| Partially Met | Made progress but fell short on key criteria |
| Not Met | Significant gaps |

---

## Cadence

| When | Activity |
|------|----------|
| **Quarter start (Week 1–2)** | Draft team OKRs; employees write contribution goals |
| **Monthly** | Check-in: update KR progress, flag risks |
| **Mid-quarter** | Formal mid-cycle review — on track? Goal adjustments? |
| **Quarter end** | Score team OKRs, assess contribution goals, feed into next cycle |

---

## Anti-Patterns

| Anti-Pattern | Fix |
|-------------|-----|
| Task masquerading as KR | Measure the outcome, not the activity |
| Sandbagging | At least one KR should be a genuine stretch |
| No baseline | Always include current state: "from X to Y" |
| Set and forget | Monthly check-ins at minimum |
| Contribution goals copied from OKRs | Personalize: what is THIS person's specific slice? |
| Goals too vague to measure | Apply the unambiguity and differentiation tests |

---

## Related Skills

- **planning** — Quarterly planning process that generates team OKRs
- **performance-cycles** — Review cycles, calibration, where goals get assessed
- **delivery** — Sprint execution against goal-aligned work
- **seasonal-events** — Annual HR calendar and goal-setting timeline
