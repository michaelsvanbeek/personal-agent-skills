---
name: code-review
description: >-
  Code review practices and pull request quality standards. Use when: reviewing a pull
  request, preparing code for review, writing review comments, checking for common issues,
  establishing review guidelines, auditing an existing project's review practices, or
  improving PR quality standards. Integrates linting, testing, security, and CI/CD
  checks into a cohesive review workflow.
tags:
  - developer
---

# Code Review Standards

## When to Use

- Reviewing a pull request or diff
- Preparing your own code for review
- Writing constructive review comments
- Establishing team review conventions
- Checking code against project standards before merge

---

## Review Priorities

Review in this order — stop at the first category with issues before nitpicking lower ones:

| Priority | Category | Questions to ask |
|----------|----------|-----------------|
| **1** | **Correctness** | Does it do what it claims? Are edge cases handled? |
| **2** | **Security** | Input validation? Secret exposure? Injection? Access control? |
| **3** | **Design** | Right abstraction level? Follows existing patterns? Over-engineered? |
| **4** | **Reliability** | Error handling? Logging? Graceful degradation? |
| **5** | **Performance** | Obvious N+1 queries, unbounded loops, or missing indexes? |
| **6** | **Readability** | Clear names? Minimal complexity? Would a new team member understand? |
| **7** | **Style** | Formatting, naming conventions, import order (should be caught by linters) |

**Don't review style manually** — that's what linters and formatters enforce in CI. If CI passes lint, style is not a review concern.

---

## What CI Should Catch (Not Reviewers)

These should fail the build automatically — don't waste review time on them:

| Check | CI tool | Reference |
|-------|---------|-----------|
| Code formatting | Ruff format / Prettier | linting skills |
| Lint violations | Ruff / ESLint | linting skills |
| Type errors | mypy / TypeScript strict | python / typescript skills |
| Test failures | pytest / vitest | testing skill |
| Coverage regression | pytest-cov `--cov-fail-under` | cicd skill |
| Known vulnerabilities | `pip audit` / `npm audit` | security skill |

---

## Reviewer Checklist

Use as a mental model during review. Not every point applies to every PR.

### Logic & Correctness

- [ ] Changes match the stated intent (PR description / issue).
- [ ] Edge cases are handled (empty inputs, nulls, boundary values, concurrent access).
- [ ] Error paths log meaningful context and return appropriate codes.
- [ ] No silent failures — errors are either handled or propagated, never swallowed.

### Security

- [ ] No hardcoded secrets, API keys, or credentials.
- [ ] External input is validated and sanitized.
- [ ] SQL queries use parameterized statements (no string interpolation).
- [ ] File paths are not constructed from user input without validation.
- [ ] New dependencies don't introduce known vulnerabilities.

### Design

- [ ] Changes follow existing patterns in the codebase — not a new pattern for one use.
- [ ] No unnecessary abstractions (no new helper class for one call site).
- [ ] Public API surface is minimal — don't expose internals that could change.
- [ ] Side effects are explicit and contained (not hidden in utility functions).

### Testing

- [ ] New behavior has tests. Bug fixes include a regression test.
- [ ] Tests verify behavior, not implementation details.
- [ ] Mocks are used for external dependencies only — not for the code under test.
- [ ] Test names describe the expected behavior as a specification.

### Operations

- [ ] Logging is present for significant operations and error paths.
- [ ] Configuration uses environment variables with sensible defaults.
- [ ] Database migrations have a down/rollback path.
- [ ] Deploy is backward-compatible (no breaking change without migration plan).

---

## Writing Review Comments

### Be Specific

```
# Bad
"This doesn't look right."

# Good
"This will throw a KeyError if `user_data` doesn't contain 'email'.
Consider using `user_data.get('email')` or validating the field upstream."
```

### Categorize Comments

Prefix comments to signal intent:

| Prefix | Meaning |
|--------|---------|
| **blocker:** | Must fix before merge. Correctness, security, or data loss risk. |
| **suggestion:** | Improvement idea. Take it or leave it. |
| **question:** | Genuine question — I don't understand the intent. |
| **nit:** | Minor style preference. Not worth blocking on. |
| **praise:** | Something done well. Positive reinforcement matters. |

### Rules for Reviewers

- **Approve when it's good enough**, not when it's perfect. Perfect is the enemy of shipped.
- **Limit to one round of blocker feedback** — if you keep finding new blockers, the PR is too large.
- **Don't rewrite their code** — describe the problem and trust the author to solve it.
- **Respond within one business day** — stale PRs are costly.
- **Prefer questions over commands** — "Have you considered X?" is better than "Do X."

### Rules for Authors

- **Keep PRs small** — under 400 lines of meaningful diff. Split large changes into stacked PRs.
- **Write a clear PR description** — what changed, why, and how to test it.
- **Self-review before requesting** — read your own diff. Catch the obvious stuff yourself.
- **Don't force-push after review** — reviewers lose context. Use fixup commits, squash at merge.

---

## PR Size Guidelines

| Lines changed | Assessment | Action |
|--------------|------------|--------|
| **< 100** | Ideal | Fast to review, easy to understand |
| **100 – 400** | Acceptable | Should have a clear description |
| **400 – 800** | Too large | Split if possible |
| **> 800** | Unacceptable | Must be split. No reviewer can effectively review this. |

Exceptions: auto-generated code, migrations, dependency lockfile updates.

---

## PR Description Template

```markdown
## What

Brief description of the change.

## Why

Link to issue or explain the motivation.

## How

Key design decisions and approach taken.

## Testing

How to verify the change works:
- [ ] Unit tests pass
- [ ] Manual testing steps (if applicable)

## Checklist

- [ ] Lint / format passes
- [ ] Tests added for new behavior
- [ ] No secrets in code
- [ ] Documentation updated (if user-facing)
```
