---
name: git-workflow
description: 'Git commit, branching, and documentation conventions. Use when: writing commit messages, creating branches, preparing pull requests, reviewing changelogs, squashing commits, establishing git workflow standards, auditing commit history for convention compliance, or improving an existing project git workflow.'
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
- Auditing an existing project's commit history and branch strategy for convention compliance
- Retrofitting conventional commits, PR templates, or changelog practices to an existing repo

## Commit Messages

Follow the **Conventional Commits** specification. This enables automated changelog generation, semantic versioning, and clear history.

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Rules

- **Subject line**: Imperative mood, lowercase, no period, max 72 characters.
- **Body**: Wrap at 80 characters. Explain *what* and *why*, not *how*. Separate from subject with a blank line.
- **Footer**: Reference issues, breaking changes, or co-authors.

### Types

| Type | When to use |
|------|-------------|
| `feat` | New feature or capability |
| `fix` | Bug fix |
| `docs` | Documentation only |
| `style` | Formatting, whitespace, semicolons (no logic change) |
| `refactor` | Code restructuring without behavior change |
| `perf` | Performance improvement |
| `test` | Adding or updating tests |
| `build` | Build system, dependencies, CI config |
| `chore` | Maintenance tasks, tooling, config |
| `ci` | CI pipeline changes |
| `revert` | Reverting a previous commit |

### Scope

Optional, but encouraged. Use the module, component, or area affected:

- `feat(api): add user search endpoint`
- `fix(auth): handle expired refresh tokens`
- `build(deps): update fastapi to 0.115`
- `docs(readme): add deployment instructions`

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
All existing API keys are invalidated.

Closes #142
```

### Anti-patterns

- `update stuff` - Vague, no type, no scope, no useful information
- `fix bug` - Which bug? What was the symptom?
- `WIP` - Never commit with WIP; use `git stash` or draft branches
- `address review comments` - Describe what actually changed
- Giant commits touching unrelated files - Split into focused commits

## Commit Granularity

- Each commit should represent **one logical change**.
- If you can describe the commit with "and" (e.g., "fix login and update styles"), split it into two commits.
- Test suite should pass on every commit (enables `git bisect`).
- Prefer many small commits during development, squash into logical units before merging.

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

Rules:
- Lowercase, kebab-case.
- Short but descriptive (3-5 words max).
- Include ticket number if applicable: `feat/PROJ-123-user-search`.

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

How this was tested. Include commands to verify, test output, or screenshots.

## Checklist

- [ ] Linter passes with no errors (`ruff check .` / `eslint .`)
- [ ] Formatter applied (`ruff format .` / `prettier --write .`)
- [ ] Type checker passes (`mypy .` for Python, `tsc --noEmit` for TypeScript)
- [ ] Tests pass
- [ ] Documentation updated
- [ ] No secrets or credentials committed
```

## Changelog

When maintaining a CHANGELOG.md, follow the **Keep a Changelog** format. See the **release-management skill** for changelog automation from conventional commits and release cadence guidance.

```markdown
# Changelog

## [Unreleased]

### Added
- User search endpoint with fuzzy matching (#142)

### Fixed
- Token refresh handling for expired sessions (#138)

### Changed
- Auth migrated from API key to OAuth2 (#140)

## [1.1.0] - 2026-03-15

### Added
- Dashboard caching with IndexedDB
```

Categories: Added, Changed, Deprecated, Removed, Fixed, Security.

## Pre-Commit Checklist

Run these before every commit:

**Python projects:**
```bash
ruff format .        # format
ruff check --fix .   # lint with auto-fix
mypy .               # type check
pytest               # tests
```

**Web projects:**
```bash
prettier --write .   # format
eslint . --fix       # lint with auto-fix
tsc --noEmit         # type check
npm test             # tests
```

Consider automating this with a pre-commit hook or `lint-staged`.

## Git Hygiene

- Write meaningful messages on every commit, even during early development.
- Use `.gitignore` before the first commit to avoid committing generated files.
- Never commit secrets, credentials, or environment files.
- Never commit code that fails linting or type checking.
- Use `git stash` for work-in-progress rather than WIP commits.
- Rebase/squash feature branches before merging to keep main history clean.
- Tag releases using semantic versioning: `v1.2.0` (see release-management skill for full SemVer conventions).

## Release Tagging

Use **annotated tags** for all releases. Annotated tags store author, date, and message — essential for audit and tooling:

```bash
git tag -a v1.5.0 -m "Release v1.5.0"
git push origin main --follow-tags
```

### Tag Rules

- Prefix with `v`: `v1.5.0`, not `1.5.0`.
- Tags are immutable — never delete and re-create. Fix with a new PATCH release.
- Tag only commits where all CI checks pass.
- Use the conventional commit type `chore(release)` for the version-bump commit:

```
chore(release): v1.5.0
```

For full semantic versioning rules (when to bump MAJOR vs MINOR vs PATCH), release workflows, hotfix procedures, and changelog automation, see the **release-management skill**.
