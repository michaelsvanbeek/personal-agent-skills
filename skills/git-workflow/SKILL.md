---
name: git-workflow
description: >-
  Git commit, branching, and documentation conventions. Use when: writing commit
  messages, creating branches, preparing pull requests, reviewing changelogs,
  squashing commits, establishing git workflow standards, auditing commit history
  for convention compliance, or improving an existing project's git workflow.
tags:
  - developer
  - manager
---

# Git Workflow Standards

## When to Use

- Writing or reviewing commit messages
- Creating branches for features, fixes, or releases
- Preparing pull request descriptions
- Squashing or cleaning up commit history
- Establishing git conventions for a project
- Auditing an existing project's commit history and branch strategy

---

## Commit Messages

Follow the **Conventional Commits** specification. This enables automated
changelog generation, semantic versioning, and clear history.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Rules

- **Subject line**: Imperative mood, lowercase, no period, max 72 characters.
- **Body**: Wrap at 80 characters. Explain *what* and *why*, not *how*. Separate
  from subject with a blank line.
- **Footer**: Reference issues, breaking changes, or co-authors.

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace (no logic change) |
| `refactor` | Code restructuring without behavior change |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system, dependencies |
| `chore` | Maintenance tasks, tooling, config |
| `ci` | CI pipeline changes |
| `revert` | Reverting a previous commit |

### Scope

Optional, but encouraged. Use the module, component, or area affected:

```
feat(api): add user search endpoint
fix(auth): handle expired refresh tokens
build(deps): update fastapi to 0.115
docs(readme): add deployment instructions
```

### Examples

Single-line (small, self-explanatory changes):

```
fix(cors): allow credentials in preflight responses
```

With body (changes that need context):

```
feat(cache): add IndexedDB caching for dashboard data

Store API responses in IndexedDB keyed by endpoint and query params.
Serve cached data immediately on page load, then refresh in the
background. Reduces perceived load time for repeat visits.
```

With breaking change:

```
feat(api)!: change auth from API key to OAuth2

Migrate all endpoints to require Bearer token authentication.
API key header is no longer accepted.

BREAKING CHANGE: Clients must update to use OAuth2 tokens.
```

### Anti-patterns

| Bad commit message | Problem |
|-------------------|---------|
| `update stuff` | No type, no scope, no useful information |
| `fix bug` | Which bug? What was the symptom? |
| `WIP` | Use `git stash` or draft branches instead |
| `address review comments` | Describe what actually changed |
| Giant commit touching many files | Split into focused commits |

---

## Commit Granularity

Each commit should represent **one logical change**.

- If you can describe it with "and", split it into two commits.
- The test suite should pass on every commit (enables `git bisect`).
- Prefer many small commits during development, squash into logical units
  before merging.

### Signs of good granularity

- A single commit can be reverted without side effects
- The diff is reviewable in under 5 minutes
- The commit message fully describes the change without "also"

---

## Branch Naming

```
<type>/<short-description>
```

| Pattern | Example |
|---------|---------|
| `feat/user-search` | Feature work |
| `fix/token-expiry` | Bug fix |
| `docs/api-reference` | Documentation |
| `refactor/auth-module` | Refactoring |
| `release/v1.2.0` | Release branch |
| `hotfix/null-pointer` | Production hotfix |

**Rules:**
- Lowercase, kebab-case
- Short but descriptive (3-5 words max)
- Include ticket number if applicable: `feat/PROJ-123-user-search`

---

## Pull Requests

### Title

Follow the same conventional commit format as the squash/merge commit:

```
feat(api): add user search endpoint
```

### Description Template

```markdown
## What

Brief description of what this PR does.

## Why

Context on why this change is needed. Link to issue or ticket.

## How

High-level description of the approach taken.

## Testing

How this was tested. Include commands, test output, or screenshots.

## Checklist

- [ ] Linter passes
- [ ] Formatter applied
- [ ] Type checker passes
- [ ] Tests pass
- [ ] Documentation updated
- [ ] No secrets or credentials committed
```

---

## Changelog

When maintaining a CHANGELOG.md, follow **Keep a Changelog**:

```markdown
# Changelog

## [Unreleased]

### Added
- User search endpoint with fuzzy matching (#142)

### Fixed
- Token refresh handling for expired sessions (#138)

## [1.1.0] - 2026-03-15

### Added
- Dashboard caching with IndexedDB
```

**Categories:** Added, Changed, Deprecated, Removed, Fixed, Security.

---

## Git Hygiene

- Write meaningful messages on every commit, even during early development.
- Use `.gitignore` before the first commit.
- Never commit secrets, credentials, or environment files.
- Never commit code that fails linting or type checking.
- Use `git stash` for work-in-progress rather than WIP commits.
- Rebase/squash feature branches before merging to keep main history clean.

## Release Tagging

Use **annotated tags** for all releases:

```bash
git tag -a v1.5.0 -m "Release v1.5.0"
git push origin main --follow-tags
```

**Tag rules:**
- Prefix with `v`: `v1.5.0`, not `1.5.0`
- Tags are immutable — never delete and re-create
- Tag only commits where all CI checks pass
- Use `chore(release): v1.5.0` for the version-bump commit
