---
name: critical-feedback
description: >-
  Rigorous critical analysis of designs, strategies, communications, and proposals.
  Use when: stress-testing a design or architecture, playing devil's advocate on a
  strategy, poking holes in assumptions, evaluating competing arguments, challenging
  a proposal before it ships, reviewing a communication for logical weaknesses,
  pressure-testing a plan before presenting to leadership, or when someone asks
  "what am I missing?" Works on any artifact: technical designs, business strategies,
  product proposals, org changes, communications, or process decisions.
tags:
  - general
  - manager
  - developer
  - executive
---

# Critical Feedback

Rigorous, honest analysis that prioritizes finding the best solution over comfort. The goal is not to tear things down — it's to surface weaknesses early enough to fix them.

## When to Use

- Reviewing a technical design or architecture document
- Stress-testing a strategy before presenting to leadership
- Playing devil's advocate on a product direction
- Evaluating competing approaches when the team is split
- Checking whether a communication will hold up under scrutiny
- Challenging assumptions in a project plan or estimate
- Reviewing a proposal that "feels off" but no one can articulate why
- Pre-mortems: imagining how a plan could fail
- When someone explicitly asks for hard feedback

## When NOT to Use

- Brainstorming sessions where volume of ideas matters more than quality (critique kills ideation)
- When someone needs encouragement, not analysis (read the room)
- Trivial decisions where the cost of being wrong is negligible

---

## Core Principles

1. **Intellectual honesty over diplomacy** — Say what's true, not what's comfortable. A flaw that's unspoken is a flaw that ships.
2. **Attack the idea, not the person** — Critique the design, the argument, the assumption. Never the author's competence or intent.
3. **Justify every objection** — "I don't like it" is not feedback. Every critique must include the reasoning and the risk it creates.
4. **Weigh both sides** — Present the strongest version of the argument before dismantling it (steelman, then critique). If you can't articulate why someone believes the thing, you don't understand it well enough to critique it.
5. **Proportional depth** — Spend critique effort proportional to the decision's blast radius. A reversible config change doesn't need the same rigor as a platform migration.
6. **Offer alternatives** — Criticism without a constructive alternative is noise. For every significant flaw, suggest at least one path forward.
7. **Be willing to say no** — If an idea is fundamentally broken, say so clearly. Hedging ("maybe we could consider...") when the answer is "this won't work" wastes everyone's time.

---

## Process

### Step 1: Understand the Intent

Before critiquing, reconstruct _what the author is trying to achieve_ and _why they chose this approach_. This prevents critiquing a strawman.

```markdown
## Intent Reconstruction

**Goal**: <What is this trying to accomplish?>
**Chosen approach**: <What approach was selected?>
**Author's likely reasoning**: <Why this approach over alternatives?>
**Constraints the author is working within**: <Budget, timeline, team skills, org politics, tech debt>
**Success criteria**: <How would the author judge this successful?>
```

If you can't fill this out, ask clarifying questions before proceeding. Critiquing without understanding intent produces irrelevant feedback.

### Step 2: Steelman the Proposal

State the strongest possible case _for_ the proposal, as the author would make it. This serves two purposes:
- It proves you understand the idea before attacking it
- It establishes which strengths are genuine (and should be preserved)

```markdown
## Strongest Case For

- <Strength 1: why this approach genuinely makes sense>
- <Strength 2: what problem this solves well>
- <Strength 3: what constraints this respects>
```

### Step 3: Identify Assumptions

Every design rests on assumptions. List them explicitly, then classify each:

```markdown
## Assumptions

| # | Assumption | Validated? | Risk if Wrong |
|---|-----------|-----------|---------------|
| 1 | <e.g., "Traffic will stay under 1K rps"> | ❌ Unvalidated | High — architecture breaks at 5K |
| 2 | <e.g., "Users prefer speed over accuracy"> | ⚠️ Partially (one survey) | Medium — wrong bet inverts the value prop |
| 3 | <e.g., "Team can deliver by Q3"> | ✅ Validated (capacity plan) | Low |
```

Focus critique energy on **unvalidated assumptions with high risk**.

### Step 4: Apply Critique Lenses

Evaluate the proposal through each relevant lens. Not every lens applies to every artifact — select the ones that matter.

#### Lens 1: Logical Soundness

- Does the conclusion follow from the premises?
- Are there logical fallacies? (false dilemma, appeal to authority, survivorship bias, sunk cost reasoning)
- Is correlation being treated as causation?
- Are there circular arguments?

#### Lens 2: Completeness

- What scenarios are not addressed?
- What failure modes are unhandled?
- What stakeholders or user segments are missing?
- Are there edge cases that break the design?

#### Lens 3: Scalability and Durability

- Does this work at 10x the current scale?
- Does this hold up if a key assumption changes?
- What's the maintenance burden over time?
- Will this survive personnel changes?

#### Lens 4: Alternatives Not Considered

- What options were dismissed without evaluation?
- Is there a simpler approach that achieves 80% of the value?
- What would a competitor or external expert do differently?
- Is "do nothing" genuinely worse than this proposal?

#### Lens 5: Second-Order Effects

- What behaviors does this incentivize (intended or not)?
- What precedent does this set?
- How will adjacent teams, systems, or processes be affected?
- What technical debt or organizational debt does this create?

#### Lens 6: Reversibility

- How hard is it to undo this if it's wrong?
- Is there a cheaper way to test the hypothesis first?
- What's the cost of being wrong vs. the cost of not acting?

### Step 5: Produce the Critique

Structure findings by severity, not by the order you found them.

#### Severity Levels

| Level | Meaning | Action Required |
|-------|---------|----------------|
| **Blocker** | Fundamental flaw — the proposal shouldn't proceed as-is | Must be resolved before moving forward |
| **Significant** | Material weakness that degrades the outcome | Should be addressed; explain the cost of ignoring |
| **Minor** | Valid concern with limited blast radius | Note for awareness; author's judgment call |
| **Nitpick** | Style or preference, not correctness | Flag only if asked for exhaustive feedback |

#### Critique Template

For each finding:

```markdown
### [Severity] <Title>

**What**: <What is the issue?>
**Why it matters**: <What risk, cost, or failure does this create?>
**Evidence/reasoning**: <Why you believe this is a real problem, not a hypothetical>
**Steelman counterargument**: <The best argument for leaving it as-is>
**Recommendation**: <What to do instead, or how to mitigate>
```

### Step 6: Weigh Competing Arguments

When the critique surfaces a genuine tension (e.g., speed vs. quality, flexibility vs. simplicity), present both sides explicitly:

```markdown
## Competing Arguments

### Argument A: <position>
- **For**: <strongest reasons>
- **Against**: <strongest objections>
- **Conditions where A wins**: <when is this the right call?>

### Argument B: <position>
- **For**: <strongest reasons>
- **Against**: <strongest objections>
- **Conditions where B wins**: <when is this the right call?>

### Assessment
<Which argument wins given the current context and constraints, and why>
```

### Step 7: Deliver the Verdict

End with a clear, unambiguous assessment. Don't hide the conclusion in paragraphs of nuance.

```markdown
## Verdict

**Overall assessment**: <Proceed | Proceed with changes | Rethink | Stop>

**Blockers** (must resolve):
1. <blocker summary + recommendation>

**Significant concerns** (should resolve):
1. <concern summary + recommendation>

**Strengths to preserve**:
1. <what's working well and should not be changed>

**Suggested next step**: <specific action — e.g., "Spike on assumption #2 before committing to this architecture">
```

---

## Output Format

Every critical feedback deliverable should produce:

1. **Intent Reconstruction** — What the author is trying to achieve
2. **Steelman** — The strongest argument for the proposal
3. **Assumptions Audit** — Explicit list with validation status and risk
4. **Critique Findings** — Severity-ranked issues with reasoning and recommendations
5. **Competing Arguments** — Balanced analysis of genuine tensions (when applicable)
6. **Verdict** — Clear assessment with blockers, concerns, strengths, and next step

---

## Tone Calibration

| Context | Tone |
|---------|------|
| Technical design review | Direct, precise, evidence-based. Reference specific components. |
| Strategy or product direction | Analytical, scenario-driven. Focus on assumptions and second-order effects. |
| Communication review | Audience-focused. "How will <stakeholder> interpret this?" |
| Someone explicitly asked for hard feedback | Fully candid. Don't soften. They asked for this. |
| Early-stage idea (not yet a proposal) | Constructive. Focus on "what would need to be true for this to work?" rather than "this won't work." |

---

## Time-Boxing and Diminishing Returns

Critique should be proportional to the decision's blast radius and reversibility:

| Decision Type | Suggested Critique Depth | Time Budget |
|--------------|-------------------------|-------------|
| Reversible config or copy change | Quick scan — skim for blockers only | 5 min |
| Feature design or API contract | Full process — Steps 1–7 | 30–60 min |
| Architecture or platform decision | Full process + competing arguments | 1–3 hours |
| Strategy or org-level direction | Full process + pre-mortem | Half day |

**When to stop**: If you've completed the process and have no Blocker or Significant findings, you're done. Don't manufacture concerns to justify the time spent. "No major issues found" is a valid and useful verdict.

---

## Worked Example: Critiquing an API Design

**Artifact**: A proposal to expose user data via a new public REST API.

### Intent Reconstruction
- **Goal**: Let partners query user profiles for integration purposes.
- **Approach**: New public `/v1/users/{id}` endpoint returning full profile JSON.
- **Author's reasoning**: Simplest approach; reuses internal data model.
- **Constraints**: Ship by end of quarter; one backend engineer available.

### Steelman
- Solves a real partner need — currently handled via CSV exports.
- Reusing the internal model avoids building a new data layer.
- Fast to ship given the timeline.

### Assumptions
| # | Assumption | Validated? | Risk if Wrong |
|---|-----------|-----------|---------------|
| 1 | Partners only need the fields in the internal model | ❌ | Medium — over-exposing PII |
| 2 | Traffic from partners will be low | ❌ | High — no rate limiting in proposal |
| 3 | Internal model is stable | ⚠️ Partially | High — model changes break partners |

### Findings

**[Blocker] No API contract separation from internal model**
- **What**: Exposing the internal data model directly means internal schema changes break partner integrations.
- **Why it matters**: Creates a coupling that makes the internal model unshippable without partner coordination.
- **Steelman counterargument**: We're under time pressure and the model is stable today.
- **Recommendation**: Add a thin response DTO that maps internal fields to a stable public contract. ~2 days of work.

**[Significant] PII exposure**
- **What**: Full profile may include sensitive fields — partners likely only need name and ID.
- **Why it matters**: Regulatory and privacy risk. Violates data minimization principle.
- **Recommendation**: Default to minimal fields; require explicit scope grants for sensitive fields.

**[Minor] No rate limiting mentioned**
- **What**: Public API without rate limiting is an abuse vector.
- **Recommendation**: Add rate limiting (standard middleware, low effort).

### Verdict
**Overall assessment**: Proceed with changes.
- **Blockers**: Separate API contract from internal model.
- **Significant**: Scope down exposed fields; add field-level access control.
- **Strengths**: Solves real need, timeline is achievable with the above changes.
- **Next step**: Author revises with DTO layer and field scoping; re-review in 2 days.

---

## Anti-Patterns

| Anti-Pattern | Why It Fails |
|-------------|-------------|
| **Critique without understanding** | Attacking a strawman wastes everyone's time and destroys credibility |
| **"I don't like it" without reasoning** | Unhelpful. Every objection needs a _because_. |
| **Nitpicking when blockers exist** | Discussing variable names while the architecture is unsound signals broken priorities |
| **Hedging on blockers** | Saying "might be an issue" when you mean "this will fail" delays necessary course corrections |
| **Criticism without alternatives** | Pointing out problems without suggesting solutions is obstruction, not feedback |
| **Critiquing everything equally** | If 15 findings are all "significant," none of them are. Use severity honestly. |
| **Ignoring constraints** | Proposing the ideal solution that ignores the budget, timeline, or team capability is not useful critique |
| **Optimizing for being right** | The goal is the best outcome, not winning the argument. If the author's response resolves your concern, accept it. |
