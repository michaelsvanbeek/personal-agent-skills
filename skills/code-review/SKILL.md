---
name: code-review
description: >-
  Code review standards and pull request quality checklists. Use when: reviewing
  a pull request, preparing code for review, writing review comments, establishing
  review guidelines for a team, or auditing an existing project's review practices.
tags:
  - developer
---

# Code Review Standards

## When to Use

- Reviewing a pull request
- Preparing code for review
- Writing constructive review comments
- Establishing team review guidelines
- Auditing an existing project's PR quality

---

## Review Checklist

Before approving any pull request, verify:

### Correctness

- [ ] The code does what the PR description says it does
- [ ] Edge cases are handled
- [ ] Error paths don't swallow failures silently
- [ ] No regressions in existing behavior

### Quality

- [ ] Linter and formatter pass with no warnings
- [ ] Type checker passes (no `any`, no `@ts-ignore`, no `# type: ignore`)
- [ ] Tests cover the changed behavior
- [ ] No hardcoded secrets, credentials, or internal URLs
- [ ] Documentation updated if behavior changed

### Design

- [ ] Change is appropriately scoped (not too large)
- [ ] Abstractions are warranted (not premature)
- [ ] Naming is clear and consistent with the codebase
- [ ] No dead code or commented-out blocks

---

## Writing Good Review Comments

### Be specific

```
# Good
This function allocates a new list on every call. Consider reusing the buffer
passed in from the caller to avoid the allocation.

# Bad
This could be more efficient.
```

### Distinguish severity

| Prefix | Meaning |
|--------|---------|
| `nit:` | Style preference, non-blocking |
| `suggestion:` | Improvement idea, non-blocking |
| `question:` | Clarification needed before approval |
| `blocker:` | Must fix before merging |

### Praise good work

Review comments shouldn't only point out problems. Call out clean abstractions,
thorough tests, and clear documentation.

---

## PR Size Guidelines

| Lines changed | Assessment |
|---------------|-----------|
| < 50 | Easy to review — ideal for focused fixes |
| 50-200 | Comfortable — most feature PRs should land here |
| 200-500 | Large — consider splitting by layer or concern |
| > 500 | Too large — split into stacked PRs |

Large PRs get slower, lower-quality reviews. Prefer many small PRs over one
large one.
