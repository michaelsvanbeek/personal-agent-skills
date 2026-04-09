# mvb-skills to personal-agent-skills Migration State

Date: 2026-04-09

## Goal

Use personal-agent-skills as the canonical library whenever a skill exists in both places, and keep mvb-skills focused on gaps (skills not yet migrated, or intentionally private/opinionated variants).

## Operating Model

1. Canonical source for shared skills: personal-agent-skills.
2. mvb-skills scope: gap skills, private workflows, and personal overlays.
3. Migration unit: 3-7 skills per batch, each batch normalized to personal-agent standards.
4. Validation per batch: frontmatter schema, portability cleanup, catalog update, persona linkage, commit.

## Repo Standards Delta (What to Normalize)

| Area | personal-agent-skills | mvb-skills |
|---|---|---|
| Frontmatter | `name`, `description`, `tags` | `name`, `description`, `argument-hint` |
| Audience | Public, broadly accepted practices | Personal/internal conventions allowed |
| Internal references | Avoid org/private URLs and machine-local paths | May include local/internal references |
| Persona mapping | Required via tags + persona docs | Optional/less strict |

## Shared Skills: Key Differences

Most shared skills are now metadata-only differences (schema/tagging). Substantive body differences currently remain in:

| Skill | Difference Summary | Recommended Canonical Direction |
|---|---|---|
| alternatives-analysis | personal-agent version is more concise/public; mvb version is stricter and includes personal decision-log workflow | Keep personal-agent body as canonical; optionally add neutralized advanced appendix |
| ide-setup | personal-agent version removes mvb-repo-specific install/config references | Keep personal-agent body as canonical |
| large-change | personal-agent version is shorter and less prescriptive than mvb | Keep personal-agent baseline; optionally layer deeper checklists later |
| logging-metrics | personal-agent removes internal wiki/local-path references | Keep personal-agent body as canonical |
| markdown-docs | personal-agent is streamlined; mvb has deeper templates and examples | Keep personal-agent baseline; selectively port non-opinionated deep examples |

For all other shared skills, differences are primarily frontmatter normalization (`argument-hint` vs `tags`) and minor wording.

## Incremental Migration Plan

### Phase 1: Canonicalize Overlaps (Immediate)

1. Use personal-agent-skills content for all overlapping skills in agent loading order.
2. In mvb-skills, keep overlap files only if they add private overlays; otherwise note they are mirrored from personal-agent.
3. Add a short "Canonical Source" note in mvb README for overlap policy.

### Phase 2: Migrate High-Fit Gap Skills (Batched)

Batch A (platform/dev): `aws-deployment`, `python-web-server`, `ui-design`, `web-ui`, `ui-performance`.
Batch B (delivery/reliability): `performance-testing`, `secrets-management`, `agent-design`, `agent-integrations`, `general-coding` (or split).
Batch C (data/analysis): `data-pipelines`, `financial-dashboards`, `financial-modeling`, `customer-success`, `competitive-analysis`.
Batch D (strategy/product): `business-strategy`, `business-reports`, `marketing-strategy`, `monetization-strategy`, `operations-okrs` (merge path).
Batch E (language/platform): `swift`, `swiftui`, `ios-testing`.

### Phase 3: Resolve Naming and Scope Collisions

1. Merge `operations-okrs` into `okrs` (single canonical skill) or keep one as private overlay.
2. Decide whether `general-coding` should be split into narrower public skills.
3. Keep `knowledge-base` in mvb unless rewritten to remove local path assumptions.

### Phase 4: Keep mvb as Gap/Overlay Layer

1. Retain only: unmigrated skills + intentional private overlays.
2. On each new skill request, check personal-agent first, then mvb for gap-filling.
3. Re-audit quarterly and migrate another batch.

## Per-Skill Repo State

Legend:
- Canonical: where to source by default.
- Fit: suitability for migration to personal-agent standards.
- Next Action: migrate now, keep as gap, or merge/resolve.

| Skill | In mvb | In personal | Canonical | Fit to Personal | Next Action |
|---|---|---|---|---|---|
| accessibility-testing | yes | yes | personal | n/a | shared; keep personal canonical |
| agent-design | yes | no | mvb (gap) | high | queue for migration batch |
| agent-integrations | yes | no | mvb (gap) | medium | migrate after scope refinement |
| alternatives-analysis | yes | yes | personal | n/a | shared; keep personal canonical |
| api-design | yes | yes | personal | n/a | shared; keep personal canonical |
| async-messaging | yes | yes | personal | n/a | shared; keep personal canonical |
| aws-deployment | yes | no | mvb (gap) | high | queue for migration batch |
| branding-strategy | yes | no | mvb (gap) | high | queue for migration batch |
| business-reports | yes | no | mvb (gap) | high | queue for migration batch |
| business-strategy | yes | no | mvb (gap) | high | queue for migration batch |
| caching-strategies | yes | yes | personal | n/a | shared; keep personal canonical |
| charts | yes | yes | personal | n/a | shared; keep personal canonical |
| cicd | yes | yes | personal | n/a | shared; keep personal canonical |
| code-review | yes | yes | personal | n/a | shared; keep personal canonical |
| competitive-analysis | yes | no | mvb (gap) | high | queue for migration batch |
| customer-success | yes | no | mvb (gap) | high | queue for migration batch |
| data-analysis | yes | yes | personal | n/a | shared; keep personal canonical |
| data-migrations | yes | yes | personal | n/a | shared; keep personal canonical |
| data-pipelines | yes | no | mvb (gap) | high | queue for migration batch |
| data-visualization | yes | yes | personal | n/a | shared; keep personal canonical |
| database | yes | yes | personal | n/a | shared; keep personal canonical |
| dependency-management | yes | yes | personal | n/a | shared; keep personal canonical |
| docker | yes | yes | personal | n/a | shared; keep personal canonical |
| error-handling | yes | yes | personal | n/a | shared; keep personal canonical |
| feature-flags | yes | yes | personal | n/a | shared; keep personal canonical |
| financial-dashboards | yes | no | mvb (gap) | high | queue for migration batch |
| financial-modeling | yes | no | mvb (gap) | high | queue for migration batch |
| general-coding | yes | no | mvb (gap) | medium | split/refine before migration |
| git-workflow | yes | yes | personal | n/a | shared; keep personal canonical |
| ide-setup | yes | yes | personal | n/a | shared; keep personal canonical |
| ios-testing | yes | no | mvb (gap) | high | queue for migration batch |
| knowledge-base | yes | no | mvb (gap) | low-medium | keep in mvb until neutralized |
| large-change | yes | yes | personal | n/a | shared; keep personal canonical |
| logging-metrics | yes | yes | personal | n/a | shared; keep personal canonical |
| markdown-docs | yes | yes | personal | n/a | shared; keep personal canonical |
| marketing-strategy | yes | no | mvb (gap) | high | queue for migration batch |
| meeting-agendas | no | yes | personal-only | n/a | consider backport to mvb if needed |
| monetization-strategy | yes | no | mvb (gap) | high | queue for migration batch |
| okrs | no | yes | personal-only | n/a | map against operations-okrs merge plan |
| operations-okrs | yes | no | mvb (gap) | medium-high | merge with okrs strategy |
| performance-testing | yes | no | mvb (gap) | high | queue for migration batch |
| product-management | yes | yes | personal | n/a | shared; keep personal canonical |
| prompt-writing | yes | yes | personal | n/a | shared; keep personal canonical |
| python | yes | yes | personal | n/a | shared; keep personal canonical |
| python-web-server | yes | no | mvb (gap) | high | queue for migration batch |
| release-management | yes | yes | personal | n/a | shared; keep personal canonical |
| retrospectives | no | yes | personal-only | n/a | consider backport to mvb if needed |
| roadmap-planning | no | yes | personal-only | n/a | consider backport to mvb if needed |
| secrets-management | yes | no | mvb (gap) | high | queue for migration batch |
| security | yes | yes | personal | n/a | shared; keep personal canonical |
| status-updates | no | yes | personal-only | n/a | consider backport to mvb if needed |
| swift | yes | no | mvb (gap) | high | queue for migration batch |
| swiftui | yes | no | mvb (gap) | high | queue for migration batch |
| testing | yes | yes | personal | n/a | shared; keep personal canonical |
| typescript | yes | yes | personal | n/a | shared; keep personal canonical |
| ui-design | yes | no | mvb (gap) | high | queue for migration batch |
| ui-performance | yes | no | mvb (gap) | high | queue for migration batch |
| web-ui | yes | no | mvb (gap) | high | queue for migration batch |

## Batch Execution Checklist (Use per PR)

- [ ] Copy skill folder from mvb-skills to personal-agent-skills
- [ ] Normalize frontmatter to `name`, `description`, `tags`
- [ ] Remove private/internal URLs or machine-local paths
- [ ] Confirm 3+ concrete "Use when" triggers in description
- [ ] Add to `docs/catalog.md` in alphabetical order
- [ ] Link to relevant persona pages
- [ ] Run repository audit/lint checks
- [ ] Commit with conventional message
- [ ] Update this migration state document
