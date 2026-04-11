# Skills Ecosystem: Cross-Repo Delta

Last updated: 2026-04-10

## Goal

Three clear homes for every skill:

| Repo | Purpose | Who Uses It |
|------|---------|-------------|
| **personal-agent-skills** | Generic, public-grade skills. No org-specific content. | Anyone — public GitHub repo |
| **workday-skills** _(future repo)_ | Workday-specific, HR-system, and employer-specific overlays | mvb only — private repo |
| **mvb-work-skills** | Working repo. Gap skills not yet migrated, private workflows, working notes | mvb only |

**End state**: personal-agent-skills holds everything that's broadly useful. workday-skills holds employer-specific overlays. mvb-work-skills is lean — only what has no other home.

---

## Repo Standards Delta

| Area | personal-agent-skills | mvb-work-skills |
|------|----------------------|-----------------|
| Frontmatter | `name`, `description`, `tags` (from registry) | `name`, `description`, `argument-hint` |
| Org content | None — no internal URLs, tool-specific paths, or employer references | Allowed; expected |
| Line limit | 500 (warning if over) | No limit |
| Persona tags | Required | Optional |
| Storage refs | No Obsidian or leadership-kb paths | Can reference local vault |
| Confluence/Jira | Generic examples only, not tool paths | Org-specific paths allowed |

**When normalizing for personal-agent-skills:**

1. Replace `argument-hint:` with `tags:` (YAML list, pick from registry)
2. Strip Obsidian/leadership-kb storage sections
3. Remove Workday-specific instructions (goal entry screens, review system names)
4. Remove JIRA project keys, queue names, or org-specific field references
5. Check "Related Skills" — remove references to skills that don't exist in PA
6. Verify `description` has "Use when:" with ≥ 3 concrete triggers
7. Verify line count < 500

---

## Migration History

### Batch 1 — Analytical (2026-04-09)

Branch: `feat/add-analytical-skills` (open PR to main)

| Skill | Lines | Org Refs Removed |
|-------|-------|-----------------|
| anomaly-detection | 284 | None |
| critical-feedback | 317 | None |
| reclassification | 302 | Obsidian storage section |
| statistics | 397 | None |

### Batch 2 — Comms + Leadership (2026-04-10)

Branch: `feat/add-comms-leadership-skills` (open PR to main)

| Skill | Lines | Org Refs Removed |
|-------|-------|-----------------|
| comms | 264 | "Confluence note" → "written summary" |
| goals | 259 | "for Workday" in When to Use; "Workday" in Two Systems table; section headers and checklist items |
| presentations | 419 | Obsidian storage section; Workday brand color mapping table; Workday color comments in visual-spec; Workday graduation note |
| stakeholder-comms | 213 | "Confluence note" → "written summary" |
| ticket-writing | 490 | `private/ticket-writing-config.yaml` → generic path |

---

## Current Per-Skill State

### Shared (in both repos — personal-agent canonical)

| Skill | PA Lines | Notes |
|-------|---------|-------|
| anomaly-detection | 284 | Batch 1. PA canonical. |
| comms | 264 | Batch 2. PA canonical. |
| critical-feedback | 317 | Batch 1. PA canonical. |
| goals | 259 | Batch 2. PA canonical. Workday-specific overlay belongs in workday-skills. |
| large-change | — | PA canonical. |
| markdown-docs | — | PA canonical. |
| okrs | — | PA canonical. mvb `okrs` dir = local mirror; verify no divergence. |
| presentations | 419 | Batch 2. PA canonical. Workday brand guide overlay belongs in workday-skills. |
| python | — | PA canonical. |
| reclassification | 302 | Batch 1. PA canonical. |
| retrospectives | — | PA canonical. |
| roadmap-planning | — | PA canonical. mvb `roadmap-planning` = local mirror. |
| stakeholder-comms | 213 | Batch 2. PA canonical. |
| statistics | 397 | Batch 1. PA canonical. |
| status-updates | — | PA canonical. |
| ticket-writing | 490 | Batch 2. PA canonical. |

### mvb-only: Ready to Migrate (0 org refs, under 500 lines)

These can be migrated with only the frontmatter normalization step (`argument-hint` → `tags`).

| Skill | Lines | Org Refs | Notes |
|-------|-------|---------|-------|
| delivery | 268 | 0 | Clean. Tags: `manager`, `operations`, `developer` |
| general-coding | 171 | 0 | Clean. Tags: `developer`, `general` |
| inbox-management | 193 | 0 | Clean. Tags: `manager`, `operations`, `communication` |
| meeting-facilitation | 228 | 0 | Clean. Tags: `manager`, `general`, `communication` |
| newsletter-comms | 184 | 0 | Clean. Note: `comms` now in PA covers newsletter format — check for overlap before migrating; may consolidate. Tags: `manager`, `communication` |
| planning | 251 | 0 | Clean. **Naming conflict**: PA has `roadmap-planning`. Rename to `team-planning` or `quarterly-execution` to differentiate. See naming section below. |
| quarterly-planning | 207 | 0 | Clean. **Naming overlap** with both `planning` (mvb) and `roadmap-planning` (PA). Evaluate merge vs rename. |
| slack-messaging | 291 | 0 | Clean. Tags: `manager`, `communication`, `general` |
| sprint-management | 206 | 0 | Clean. Tags: `manager`, `developer`, `operations` |
| team-health | 230 | 0 | Clean. Tags: `manager`, `operations` |

### mvb-only: Near-Ready (minor cleanup, 1–3 org refs)

Strip the listed org refs, then migrate.

| Skill | Lines | Refs to Remove | Notes |
|-------|-------|---------------|-------|
| one-on-ones | 290 | 2 (Jira refs in examples) | Replace Jira example ticket links with generic `[issue tracker]` placeholders. Tags: `manager` |
| project-owner | 310 | 3 (Obsidian `leadership-kb/projects/` storage section) | Remove the "save to Obsidian" section and replace with generic "maintain a project doc" note. Tags: `manager`, `executive`, `operations` |
| project-tracking | 291 | 2 (Jira board config refs) | Replace Jira-specific board setup with generic issue tracker guidance. Tags: `manager`, `operations` |
| alternative-analysis | — | 3 (Obsidian storage section) | Note: covered by `alternatives-analysis` in PA. Determine if mvb version has distinct content; if not, drop it — PA is canonical. If yes, strip Obsidian section and submit as `alternatives-analysis` update. |

### mvb-only: Needs Significant Work (size or deep org coupling)

| Skill | Lines | Issue | Action |
|-------|-------|-------|--------|
| agent-tools | 829 | 6 org refs (JIRA_, org-specific tool names); 329 lines over limit | Split into 2–3 focused skills (e.g., `agent-tool-design`, `agent-cli-tools`) and strip org refs. Not ready this cycle. |
| cross-team-dependencies | 602 | 1 org ref; 102 lines over limit | Trim to < 500 by focusing content, remove 1 ref. Good candidate for next cycle. Tags: `manager`, `operations` |
| problem-decomposition | 605 | 0 org refs; 105 lines over limit | Trim to < 500. Only a size issue — content is fully generic. Good candidate for next cycle. Tags: `developer`, `manager`, `operations` |

### mvb-only: Route to workday-skills Repo

These skills have deep Workday / HR / employer-specific integration. They should not be in a public repo. Create a private `workday-skills` repo (or similar name) to host them.

| Skill | Lines | Org Content | Notes |
|-------|-------|------------|-------|
| internal-research | 299 | 13 refs — structurally tied to Obsidian, Backstage, internal tools | The framework is org-specific by design. Core cannot be separated from the tooling. Move to workday-skills as-is. |
| performance-cycles | 286 | 6 Workday refs — review cycle names, comp planning tied to Workday | Core content is generic but the "how to navigate Workday review screens" is not. Move to workday-skills. Consider splitting: generic `performance-reviews` → PA; Workday UI guidance → workday-skills. |
| seasonal-events | 235 | 10 refs — Workday deadlines, HR calendar, open enrollment | Fundamentally employer-specific. Move to workday-skills as-is. |

**Workday-specific overlays to create** (new files for workday-skills, not migrations):

- `goals-workday` — How to enter and navigate contribution goals in Workday specifically (the parts stripped from the `goals` migration)
- `presentations-brand` — The Workday brand guide specifics (colors, Arquivo, templates) stripped from `presentations`

### personal-agent-skills only (not in mvb — consider backport if useful in mvb-work-skills context)

| Skill | Worth Backporting? | Notes |
|-------|--------------------|-------|
| accessibility-testing | Low | Not core EM workflow |
| api-design | Low | Not core EM workflow |
| async-messaging | Low | Not core EM workflow |
| caching-strategies | Low | Not core EM workflow |
| charts | Medium | Useful for presentations context, but covered tangentially |
| cicd | Low | Not core EM workflow |
| code-review | High | EMs review PRs; useful reference |
| data-analysis | Low | Not core EM workflow |
| data-migrations | Low | Not core EM workflow |
| data-visualization | Low | Not core EM workflow |
| database | Low | Not core EM workflow |
| dependency-management | Low | Not core EM workflow |
| docker | Low | Not core EM workflow |
| error-handling | Low | Not core EM workflow |
| feature-flags | Medium | EMs make flag decisions |
| git-workflow | High | EMs review and model good git hygiene |
| ide-setup | Medium | Personal tooling setup |
| logging-metrics | Low | Not core EM workflow |
| meeting-agendas | High | Core EM skill — mvb already has `meeting-facilitation` which may cover this |
| product-management | High | EMs work alongside PMs; worth having |
| prompt-writing | High | Core for AI-assisted workflows |
| release-management | Medium | EMs coordinate releases |
| security | Medium | EMs make security trade-offs |
| testing | High | EMs make testing strategy decisions |
| typescript | Low | Not core EM workflow |

**Recommendation**: install PA symlinks for `code-review`, `git-workflow`, `meeting-agendas`, `prompt-writing`, `testing` into mvb-work-skills via `install.sh`. No file copies needed.

---

## Naming Collisions to Resolve

| mvb skill | PA skill | Issue | Recommendation |
|-----------|----------|-------|---------------|
| `planning` | `roadmap-planning` | Different scope: mvb `planning` covers quarterly sprint/capacity planning; PA `roadmap-planning` covers multi-quarter roadmaps | Rename mvb `planning` to `team-planning` before migrating. Keep PA `roadmap-planning` as-is. |
| `quarterly-planning` | `roadmap-planning` | Partial overlap with both `planning` (mvb) and `roadmap-planning` (PA) | Evaluate if `quarterly-planning` content is distinct from `team-planning`; merge if not, otherwise migrate as `quarterly-planning`. |
| `alternative-analysis` | `alternatives-analysis` | Spelling difference ("alternative" vs "alternatives"); content is the same skill | PA version is canonical. Drop mvb file; remove the Obsidian storage section from PA if needed for completeness. |
| `okrs` (mvb) | `okrs` (PA) | Both exist; unclear if they diverge | Check content. If mvb version adds nothing over PA, rm mvb/okrs and symlink to PA. |

---

## Priority Queue: Next Iteration

### Recommended Batch 3 — Quick Wins (all ready now)

10 skills, 0 org refs, all under 500 lines. Frontmatter normalization only.

1. `delivery`
2. `inbox-management`
3. `sprint-management`
4. `team-health`
5. `meeting-facilitation`
6. `slack-messaging`
7. `general-coding`
8. `newsletter-comms` _(verify no content overlap with `comms`)_
9. `planning` _(rename → `team-planning` first)_
10. `quarterly-planning` _(evaluate against `team-planning` for merge/keep)_

### Recommended Batch 4 — Near-Ready (minor cleanup)

1. `one-on-ones` — strip 2 Jira example refs
2. `project-tracking` — strip 2 Jira board-config refs
3. `project-owner` — strip 3 Obsidian storage-section refs
4. `problem-decomposition` — trim ~105 lines (0 org refs; content only)

### Recommended Batch 5 — Heavy Lift

1. `cross-team-dependencies` — trim + remove 1 ref
2. `agent-tools` — split into 2–3 focused skills + full org ref cleanup

### workday-skills Repo Setup (whenever ready)

1. Create private `workday-skills` repo
2. Move: `internal-research`, `performance-cycles`, `seasonal-events`
3. Create: `goals-workday`, `presentations-brand`
4. Wire into `install.sh` alongside personal-agent-skills symlinks

---

## Audit Snapshot (2026-04-10)

| Repo | Skills | Status |
|------|--------|--------|
| personal-agent-skills (`feat/add-comms-leadership-skills` branch) | 42 | 0 errors, 3 pre-existing warnings (cicd, error-handling, prompt-writing each over 500 lines) |
| mvb-work-skills | 36 | Not audited against PA standards |

**PA skills count by batch:**
- Before Batch 1: 33 skills
- After Batch 1: 37 skills (+ anomaly-detection, critical-feedback, reclassification, statistics)
- After Batch 2: 42 skills (+ comms, goals, presentations, stakeholder-comms, ticket-writing)
- After Batch 3 (projected): 52 skills
- After Batch 4 (projected): 55–56 skills

---

## Open PRs

| Branch | Skills | Status |
|--------|--------|--------|
| `feat/add-analytical-skills` | anomaly-detection, critical-feedback, reclassification, statistics | Pushed. Create PR at `https://github.com/michaelsvanbeek/personal-agent-skills/compare/main...feat/add-analytical-skills` |
| `feat/add-comms-leadership-skills` | comms, goals, presentations, stakeholder-comms, ticket-writing | Pushed. Create PR at `https://github.com/michaelsvanbeek/personal-agent-skills/compare/main...feat/add-comms-leadership-skills` |
