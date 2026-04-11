---
name: stakeholder-comms
description: >-
  Communicating effectively up, down, and laterally within an org chart. Use when:
  preparing an executive briefing, presenting to leadership, managing stakeholder
  expectations, negotiating priorities with product or peer teams, escalating issues,
  delivering difficult messages, or building influence without authority.
tags:
  - manager
  - executive
  - communication
  - general
---

# Stakeholder Communication

## When to Use

- Preparing an executive briefing or leadership update
- Managing expectations when timelines slip or scope changes
- Negotiating priorities with product managers or peer engineering leaders
- Escalating a blocking issue through the org
- Delivering difficult messages (missed targets, team changes, resource constraints)
- Building influence and visibility for your team across the org
- Presenting technical decisions to non-technical stakeholders

---

## Communication Principles

1. **Know your audience** — Tailor depth, language, and framing for the recipient
2. **Lead with "so what"** — Why should this person care? Start there.
3. **Be direct** — State your point in the first 2 sentences. Context comes after.
4. **Propose, don't just present** — Come with a recommendation, not just options
5. **Quantify** — Numbers build credibility. "3 customers impacted" > "some customers affected"
6. **Close the loop** — Every communication should end with a clear next step or ask

---

## Communicating Upward (Manager, VP, Exec)

### What They Care About

- **Business impact**: Revenue, customers, retention, time-to-market
- **Risks**: What might go wrong and what's the plan?
- **Decisions needed**: What do you need from them?
- **Confidence level**: How sure are you about your plan?

### Format: Executive Briefing

```markdown
## [Topic] — [Date]

### Bottom Line
[1–2 sentences: the key message. Put the most important thing first.]

### Context
[2–3 sentences: enough background for the reader to understand the situation]

### Options
| Option | Pros | Cons | Effort | My Rec |
|--------|------|------|--------|--------|
| A | ... | ... | ... | ← |
| B | ... | ... | ... | |
| C (do nothing) | ... | ... | ... | |

### Recommendation
[Which option and why, in 2 sentences]

### Ask
[Specific: "I need a decision by Friday" or "Approve option A so we can proceed"]
```

### Tips for Upward Communication

- **Don't bury the lede** — If the project is off track, say so in sentence one
- **Frame problems with solutions** — "X is at risk. Here's my plan to address it: Y"
- **Respect their time** — Executives get hundreds of messages. Be ruthlessly concise.
- **Use their language** — Business metrics, not story points. Customer impact, not code complexity.
- **Pre-wire important decisions** — Socialize with your manager before presenting to the VP

---

## Communicating Laterally (Peer EMs, PMs, Designers)

### What They Care About

- **Dependencies**: What do they need from your team? What do you need from theirs?
- **Shared priorities**: Are you aligned on what matters this quarter?
- **Capacity**: Can you take on their request? Can they take on yours?
- **Coordination**: Release timelines, migrations, shared component changes

### Format: Cross-Team Alignment

```markdown
## [Topic] — Cross-Team Update

### Context
[What's happening and why does it affect both teams?]

### Our Plan
[What we're doing and our timeline]

### What We Need From You
[Specific, timeboxed asks]

### What You Can Expect From Us
[Deliverables and dates]

### Open Questions
[Things we need to resolve together]
```

### Tips for Lateral Communication

- **Be transparent about constraints** — "We can do X but not until Sprint 7" is more useful than "maybe"
- **Negotiate, don't mandate** — You don't have authority over peer teams, only influence
- **Document agreements** — Follow up Slack conversations with a written summary or email
- **Proactive > reactive** — Tell peers about changes before they discover them

---

## Communicating Downward (Your Team)

### What They Care About

- **Context**: Why are we doing this? How does it connect to the bigger picture?
- **Impact on their work**: What changes for them?
- **Recognition**: Is their work valued?
- **Honesty**: Are you being straight with them?

### Tips for Downward Communication

- **Share context generously** — The "why" matters more than the "what"
- **Be transparent about uncertainty** — "I don't know yet, but here's what I do know"
- **Celebrate publicly, coach privately** — Wins in Slack, feedback in 1:1s
- **Don't sugarcoat** — Teams respect honesty about challenges
- **Repeat important messages** — Say it in Slack, say it in standup, say it in the 1:1

---

## Escalation Framework

When you need to escalate an issue:

### Before Escalating

1. **Attempt to resolve** at the current level first
2. **Document** the issue, its impact, and what you've tried
3. **Notify** the other party that you plan to escalate

### Escalation Message

```markdown
## Escalation: [Issue Title]

### Situation
[What is happening — factual, no editorializing]

### Impact
[Business impact: customers affected, revenue at risk, timeline delay]

### What We've Tried
[Steps already taken to resolve]

### What We Need
[Specific decision or action from the escalation recipient]

### Timeline
[How urgent: "Need a decision by EOD" or "Blocking until resolved"]
```

### Rules

- **Stick to facts** — No blame, no emotion, just the situation
- **Quantify impact** — "3-day delay affecting 200 customers" not "this is really bad"
- **Propose a path forward** — Don't just throw the problem over the wall
- **Follow up** — After the escalation resolves, close the loop with all parties

---

## Difficult Conversations

### Delivering Bad News

1. **Be direct** — "We're going to miss the deadline by two weeks" not "we might have a slight delay"
2. **Own it** — Don't blame external factors reflexively. If your team could have done something differently, acknowledge it
3. **Bring a plan** — Bad news + recovery plan is 10× better than bad news alone
4. **Give advance notice** — Get ahead of it. Finding out through a missed deadline is worse than hearing about it a week early

### Pushing Back on a Request

```markdown
"I understand the importance of [their request]. Here's the trade-off:
taking this on would mean [what we'd have to deprioritize].

My recommendation is [alternative approach]. If [their request] is the
higher priority, I'm open to adjusting — I just want to make sure we're
making that trade-off intentionally."
```

---

## Anti-Patterns

| Anti-Pattern | Problem | Better Approach |
|-------------|---------|----------------|
| Burying bad news | Erodes trust when discovered | Lead with the most important/difficult item |
| Presenting problems without solutions | Makes you a messenger, not a leader | Always propose a path forward |
| Using team jargon with execs | They don't understand and won't ask | Translate to business language |
| Escalating without trying first | Burns political capital | Document what you've tried before escalating |
| Vague asks | "Provide more support" vs. "Approve 1 additional headcount" | Make every ask specific and actionable |
| Over-communicating | People tune out | Match detail to audience; less is more for execs |
