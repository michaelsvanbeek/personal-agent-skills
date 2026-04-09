---
name: okrs
description: >-
  OKR writing, cascading, scoring, and review cycles. Use when: setting team or
  individual OKRs, cascading company OKRs to team level, writing measurable key
  results, preparing for OKR reviews, scoring OKRs at end of cycle, coaching
  team members on effective OKR writing, or auditing existing OKRs for quality.
tags:
  - manager
  - operations
  - executive
---

# OKRs (Objectives and Key Results)

## When to Use

- Setting team OKRs at the start of a quarter or year
- Cascading company or org-level OKRs to your team
- Writing or refining individual contributor OKRs
- Preparing for mid-cycle or end-of-cycle OKR reviews
- Coaching a team member on how to write better OKRs
- Auditing existing OKRs for clarity and measurability

---

## OKR Fundamentals

### Objectives

An objective is **what** you want to achieve. It is:

- **Qualitative** — a direction, not a number
- **Inspirational** — ambitious enough to drive action
- **Time-bound** — scoped to the planning period (typically quarterly)
- **Actionable** — the team can influence the outcome directly
- **Concise** — one sentence, no jargon

**Good**: "Establish our platform as the most reliable in the industry"
**Bad**: "Improve uptime SLA metrics by optimizing infrastructure monitoring and alerting"

### Key Results

A key result is **how** you measure progress toward the objective. It is:

- **Quantitative** — a number, percentage, or binary (shipped / not shipped)
- **Measurable** — you can check it at any point during the cycle
- **Outcome-oriented** — measures the result, not the activity
- **Ambitious but achievable** — 70% completion is healthy; 100% means you aimed too low

**Good**: "Reduce P1 incident count from 8/quarter to 2/quarter"
**Bad**: "Implement better monitoring" (activity, not outcome)
**Bad**: "Improve reliability" (not measurable)

### Structure

```
Objective: <What we want to achieve>
  KR1: <Metric from X to Y>
  KR2: <Metric from X to Y>
  KR3: <Binary: Ship X by date>
```

Aim for **2–5 key results per objective** and **2–4 objectives per team per quarter**.

---

## Cascading OKRs

Company OKRs flow down through the org, but cascading is **not** copying:

```
Company OKR
  └─ Org OKR (how this org contributes to company goal)
       └─ Team OKR (how this team contributes to org goal)
            └─ Individual OKR (optional — how this person contributes to team goal)
```

### Cascading Rules

1. **Align, don't duplicate** — Your team OKR should contribute to the org OKR, not repeat it verbatim
2. **Own your scope** — Team OKRs must be within the team's control and influence
3. **Not everything cascades** — Some team OKRs are locally important (tech debt, developer experience) and don't map directly to a company OKR. That's fine — cap these at 1 per quarter
4. **Avoid cascading to individuals** unless your org requires it. Team-level OKRs with shared accountability are more effective

### Example Cascade

```
Company: "Accelerate enterprise customer acquisition"
  └─ Org: "Deliver enterprise-ready platform capabilities"
       └─ Team: "Establish SOC 2 compliance for the data pipeline"
            KR1: Pass SOC 2 Type II audit by end of Q2
            KR2: Remediate all critical findings within 2 weeks of discovery
            KR3: Automate 90% of compliance evidence collection
```

---

## Writing Quality Checklist

For each OKR, verify:

| Check | Pass? |
|-------|-------|
| Objective is qualitative and directional | |
| Objective fits in one sentence | |
| Each KR has a specific metric or binary deliverable | |
| Each KR has a baseline ("from X") and target ("to Y") | |
| KRs measure outcomes, not activities or tasks | |
| KRs are within the team's control or strong influence | |
| Achieving all KRs would clearly achieve the objective | |
| 2–5 KRs per objective (not more) | |
| 2–4 objectives per team (not more) | |
| At least one KR is a stretch (not guaranteed at 100%) | |

---

## Common Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Task masquerading as KR | "Deploy monitoring dashboard" is a task | "Reduce mean time to detect from 45min to 5min" |
| Sandbagging | All KRs are easily achievable | At least one KR should be a genuine stretch |
| Too many OKRs | 6 objectives with 5 KRs each = 30 things to track | Max 4 objectives, 5 KRs each |
| Vanity metrics | "Increase test count to 500" | "Reduce escaped defects from 12/quarter to 3/quarter" |
| No baseline | "Improve page load time" — from what? | Always include current state: "from 3.2s to 1.5s" |
| Annual OKRs only | Too long to course-correct | Quarterly OKRs with annual themes |
| Set and forget | OKRs written in January, never reviewed | Check-in monthly at minimum |

---

## Scoring

At the end of each cycle, score each KR:

| Score | Meaning |
|-------|---------|
| **0.0–0.3** | Failed to make meaningful progress |
| **0.4–0.6** | Made progress but fell short of the target |
| **0.7–0.9** | Hit the target or came close — this is the sweet spot |
| **1.0** | Fully achieved (if all KRs score 1.0, you probably aimed too low) |

### Scoring Process

1. **Self-score** each KR with supporting evidence
2. **Team review** — discuss scores together, calibrate
3. **Document** lessons learned: What worked? What would we change?
4. **Feed forward** — carry insights into next cycle's OKR planning

### Quarterly OKR Review Template

```markdown
# Q[N] YYYY OKR Review — [Team Name]

## Summary
Overall score: X.XX / 1.0
Key wins: ...
Key misses: ...

## Objective 1: <Title>
Score: X.X

### KR1: <Description>
- Target: <what we aimed for>
- Actual: <what we achieved>
- Score: X.X
- Evidence: <link to data, dashboard, or artifact>
- Notes: <what helped or blocked>

### KR2: ...

## Lessons Learned
- What we'd do differently next time
- What we'd double down on

## Carry-Forward
Items to continue in the next cycle
```

---

## Cadence

| When | Activity |
|------|----------|
| **Quarter start (Week 1–2)** | Draft OKRs, align with org, finalize and publish |
| **Monthly** | Check-in: update KR progress, flag risks, adjust if needed |
| **Mid-quarter** | Formal mid-cycle review — are we on track? |
| **Quarter end (Last week)** | Score OKRs, write review, feed into next quarter planning |

---

## Tools

- **Spreadsheets / Docs**: Simple OKR tracker for small teams (Google Sheets, Notion, Airtable)
- **Documentation wiki**: OKR review pages linked to your team's space (Confluence, Notion)
- **Issue tracker**: Epics can map to initiatives that support KRs
- **Dedicated goal platforms**: Lattice, Leapsome, 15Five, or your company's goal-tracking system
