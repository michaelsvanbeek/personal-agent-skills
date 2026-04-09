---
name: large-change
description: >-
  Structured workflow for implementing a large or complex change. Use when:
  planning a significant feature, decomposing a multi-day refactor, introducing
  a breaking change, coordinating cross-cutting implementation work, or shipping
  a new subsystem with phased delivery and clear release criteria.
tags:
  - developer
  - manager
  - operations
---

# Large Change Workflow

## When to Use

- A feature or refactor spans many files or multiple days of work
- Multiple valid approaches exist and design needs to be explicit
- Several developers need to coordinate against one plan
- A breaking change is involved (API removal, rename, behavior change)
- A new subsystem or integration is being introduced
- The change crosses layers (data, API, UI, infra)

When not to use: small, self-contained changes that can be completed and tested
in one short session.

---

## Phase 1: Design Document

Before implementation, write a design doc.

Purpose:
- Align on what is being built and why
- Compare alternatives before implementation lock-in
- Surface risks and unknowns early
- Create a durable rationale for future maintainers

### Design Doc Template

Create in `docs/changes/<feature-name>.md` or `docs/adr/<nnnn>-<title>.md`:

```markdown
# Design: <Feature or Change Name>

**Date**: YYYY-MM-DD
**Status**: Draft | Proposed | Accepted
**Author(s)**: <name>

## Problem

What problem does this change solve?

## Goals

- [ ] Goal 1
- [ ] Goal 2

## Non-Goals

- Not changing X
- Not supporting Y in this version

## Proposed Approach

Describe the selected solution, affected components, data flow, API/interface
changes, and dependencies.

## Alternatives Considered

Use a structured alternatives comparison:

| Option | Pros | Cons | Why not chosen |
|--------|------|------|----------------|
| Option A (chosen) | ... | ... | - |
| Option B | ... | ... | ... |
| Option C | ... | ... | ... |

## Risks and Open Questions

- [ ] Risk: <risk and mitigation>
- [ ] Open question: <question>

## Implementation Plan

High-level phases or milestones.

## Testing Strategy

What tests are required?

## Release Plan

- Version bump: PATCH / MINOR / MAJOR
- Breaking migration required: Yes / No
- Feature flag needed: Yes / No
- Rollback strategy: <describe>
```

Before implementation begins, confirm the design answers:
- What is being built and why
- Why this approach was chosen over alternatives
- Key risks and mitigations
- Release strategy

---

## Phase 2: Task Decomposition

Break work into small, independently shippable tasks.

A task is well-sized when:
- It fits one focused session (roughly 4 hours or less)
- It maps to one conventional commit subject
- The codebase remains buildable and testable after completion

### Decomposition Rules

- Prefer vertical slices (thin end-to-end paths) over horizontal layers.
- Split deprecation and removal into separate tasks for breaking changes.
- If task text contains and, consider splitting it.
- Every increment should leave the default branch stable.

### Task List Template

```markdown
## Tasks

### Phase 1: Foundation
- [ ] Add data model changes
- [ ] Implement base service/repository
- [ ] Add unit tests for new core logic

### Phase 2: Integration
- [ ] Add endpoint/adapter layer integration
- [ ] Add input validation and error handling
- [ ] Add integration tests

### Phase 3: UX/Consumer
- [ ] Add UI/client integration
- [ ] Add behavior/state tests
- [ ] Add end-to-end flow test

### Phase 4: Cleanup and Release
- [ ] Remove deprecated paths
- [ ] Update migration docs
- [ ] Update changelog and version
```

---

## Phase 3: Incremental Development

Execute tasks one at a time.

Each completed task should:
- Pass tests
- Pass linting/formatting
- Pass type checks (if applicable)
- Land as one logical commit

### Single-Task Loop

1. Implement one task
2. Run relevant tests
3. Run lint/format checks
4. Run type checks
5. Commit with conventional commit format
6. Mark the task done

Use feature flags for long-running features that cannot ship all at once.

### PR Sizing

- Keep PRs focused and reviewable
- Split very large diffs into phase-based PRs when possible
- Do not merge failing CI

---

## Phase 4: Release

When all planned tasks are complete, run a release checklist.

### Pre-Release Checklist

- [ ] All planned tasks are complete
- [ ] Full test suite passes
- [ ] Linting, formatting, and type checks are clean
- [ ] Docs and migration notes are updated
- [ ] Version and changelog are updated
- [ ] Rollback plan is documented

### Versioning Guidance

| Change type | Bump |
|-------------|------|
| Backward-incompatible behavior/API change | MAJOR |
| Backward-compatible feature | MINOR |
| Bug fix or internal improvements only | PATCH |

### Release Notes Structure

- Breaking changes
- Features and fixes
- Internal/refactor/test/docs updates

---

## Phase 5: Post-Implementation Cleanup

After release:

- [ ] Update design doc status to Accepted
- [ ] Document deferred follow-up work
- [ ] Archive/supersede outdated ADRs if needed
- [ ] Run a short retrospective for large, complex changes

---

## Quick Reference

| Phase | Output |
|-------|--------|
| 1. Design | Design doc or ADR |
| 2. Decompose | Task list with clear increments |
| 3. Implement | Stable incremental commits |
| 4. Release | Versioned release with notes |
| 5. Cleanup | Updated docs and follow-up plan |

---

## Anti-Patterns

| Anti-pattern | Why it hurts | Better approach |
|--------------|--------------|-----------------|
| Design-as-you-go on major changes | High risk of late rework | Write and review design first |
| One giant implementation commit | Hard to review and rollback | One logical change per commit |
| Pure horizontal decomposition | No shippable value until late | Use vertical slices |
| Skipping tests between tasks | Broken state accumulates | Validate after every task |
| Deferring docs until the end | Docs drift from implementation | Update docs within each phase |
| Treating version bump as clerical | Mis-signaled impact to users | Define release plan in design |
