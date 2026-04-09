---
name: release-management
description: >-
  Release management with semantic versioning, changelogs, and release workflows.
  Use when: choosing version numbers, creating releases, writing release notes, writing
  changelogs, designing release branches, automating version bumps, tagging releases,
  defining release checklists, or deciding when to cut a release. Covers SemVer 2.0.0,
  pre-release versions, release notes with breaking/functional/non-functional tiers,
  release pipelines, and changelog automation.
tags:
  - developer
  - manager
  - operations
---

# Release Management Standards

## When to Use

- Deciding what version number to assign a change
- Creating a release (tag, branch, changelog entry)
- Writing release notes or changelog entries
- Classifying changes as breaking, functional, or non-functional
- Designing release workflows for a project
- Automating version bumps in CI/CD
- Planning breaking changes or deprecations
- Auditing an existing project for missing version tags, changelogs, or release processes
- Retrofitting semantic versioning and release workflows to an unversioned project

---

## Semantic Versioning (SemVer 2.0.0)

Version format: **`MAJOR.MINOR.PATCH`**

Given a version `1.4.2`:

| Segment | Value | Increment when |
|---------|-------|---------------|
| **MAJOR** | `1` | Breaking change — consumers must update their code |
| **MINOR** | `4` | New feature — backward-compatible addition |
| **PATCH** | `2` | Bug fix — backward-compatible correction |

### When to Bump What

| Change type | Bump | Example |
|-------------|------|---------|
| Fix a bug without changing API | PATCH | `1.4.2` → `1.4.3` |
| Add a new feature, existing behavior unchanged | MINOR | `1.4.3` → `1.5.0` |
| Change or remove existing behavior | MAJOR | `1.5.0` → `2.0.0` |
| Add deprecation warning (feature still works) | MINOR | `1.5.0` → `1.6.0` |
| Remove deprecated feature | MAJOR | `1.6.0` → `2.0.0` |
| Refactor internals without API change | PATCH | `1.4.2` → `1.4.3` |
| Update dependencies (no behavior change) | PATCH | `1.4.2` → `1.4.3` |
| Update dependencies (new features exposed) | MINOR | `1.4.3` → `1.5.0` |
| Documentation-only changes | No bump (or PATCH if versioned docs) | — |

### Reset Rules

- When **MAJOR** increments, MINOR and PATCH reset to `0`: `1.5.3` → `2.0.0`
- When **MINOR** increments, PATCH resets to `0`: `1.4.3` → `1.5.0`

### Pre-Release Versions

Use for testing before a stable release:

```
2.0.0-alpha.1    # early testing, API may change
2.0.0-beta.1     # feature-complete, fixing bugs
2.0.0-rc.1       # release candidate, final validation
```

Pre-release versions have lower precedence than the release: `2.0.0-alpha.1 < 2.0.0`.

Increment the numeric suffix for subsequent pre-releases: `2.0.0-beta.1` → `2.0.0-beta.2`.

### Build Metadata

Append with `+` for informational metadata (ignored in version precedence):

```
1.4.2+build.456
1.4.2+20260320
```

### Version `0.x.y`

Major version zero (`0.y.z`) signals initial development — anything may change at any time. The public API should not be considered stable. Bump to `1.0.0` when the API is stable and used in production.

---

## Version Source of Truth

Every project must have **one** canonical location for its version. Never define the version in multiple places.

| Project type | Version source | How to read |
|--------------|---------------|-------------|
| Python | `pyproject.toml` → `[project] version` | `python -c "import importlib.metadata; print(importlib.metadata.version('pkg'))"` |
| Node.js | `package.json` → `version` | `node -p "require('./package.json').version"` |
| Generic | Git tag | `git describe --tags --abbrev=0` |
| Docker | Image tag | Build-time from git tag or `version` file |

### Git Tags as the Release Record

- Tag every release: `git tag -a v1.4.2 -m "Release v1.4.2"`
- Use **annotated tags** (`-a`), not lightweight tags — annotated tags store author, date, and message.
- Prefix with `v`: `v1.4.2`, not `1.4.2`.
- Tags are immutable — never delete and re-create a tag. If a release is bad, publish a new PATCH.

---

## Release Workflow

### Standard Release (Feature or Fix)

```
main ──────────────────────────────── main
         \                        /
          feat/my-feature ───────    ← development
                                  \
                            PR merge to main
                                  \
                              v1.5.0 tag  ← release
```

1. Develop on a feature branch (see git-workflow skill for branch naming).
2. Open a PR. CI runs lint + test (see cicd skill).
3. Merge to `main`.
4. Update changelog (see Changelog section).
5. Bump version in the version source file.
6. Commit: `chore(release): v1.5.0`
7. Tag: `git tag -a v1.5.0 -m "Release v1.5.0"`
8. Push: `git push origin main --follow-tags`
9. CI detects the tag and runs the release pipeline (build, publish, deploy).

### Hotfix Release

For urgent fixes to production when `main` has unreleased changes:

```
v1.4.0 ──── hotfix/fix-crash ──── v1.4.1
```

1. Branch from the release tag: `git checkout -b hotfix/fix-crash v1.4.0`
2. Fix the issue. Add a regression test.
3. Bump PATCH version, update changelog.
4. Tag: `git tag -a v1.4.1 -m "Release v1.4.1"`
5. Push the hotfix branch and tag.
6. Cherry-pick the fix into `main` to keep it up to date.

### Breaking Change Release

Breaking changes require extra care:

1. **Deprecate first** — add deprecation warnings in a MINOR release. Document the migration path.
2. **Communicate** — changelog, README, and (if applicable) migration guide.
3. **Remove** — in the next MAJOR release, remove the deprecated code.
4. **Bump MAJOR** — `1.x.y` → `2.0.0`.

---

## Release Notes

Release notes are the primary communication artifact for every release. They must be clear enough for any consumer — developer, operator, or product stakeholder — to understand what changed, what might break, and what they need to do.

### Change Classification

Every change in a release falls into one of three tiers. Group and label changes by tier in the release notes:

| Tier | Label | What it covers | Reader action |
|------|-------|---------------|---------------|
| **Breaking** | `⚠ BREAKING` | Removed APIs, changed behavior, renamed fields, dropped support | Must update code or config before upgrading |
| **Functional** | Features & Fixes | New features, bug fixes, behavior changes that are backward-compatible | Review and adopt at their discretion |
| **Non-Functional** | Internal | Refactors, dependency updates, performance, docs, CI, test changes | No action required — informational only |

### Release Notes Template

```markdown
# v2.0.0 — 2026-03-20

## ⚠ Breaking Changes

Changes that require action from consumers before upgrading.

- **Auth: API key authentication removed** — All endpoints now require OAuth2
  Bearer tokens. API keys are no longer accepted. See [Migration Guide](./docs/migrate-v2.md).
- **Config: `DATABASE_URL` renamed to `APP_DATABASE_URL`** — Update environment
  variables in all deployment environments.

### Migration Guide

Step-by-step instructions for upgrading from v1.x:

1. Replace `X-API-Key` header with `Authorization: Bearer <token>` in all clients.
2. Rename `DATABASE_URL` → `APP_DATABASE_URL` in `.env` and deployment config.
3. Run `python -m myapp.migrate_v2` to update stored data formats.

## Features & Fixes

Backward-compatible additions and corrections.

### Added
- User search endpoint with fuzzy matching (`GET /api/users/search?q=`) (#142)
- Dashboard export to CSV (#156)

### Fixed
- Token refresh no longer fails silently when the session cookie is expired (#138)
- Chart tooltips render correctly in dark mode (#161)

### Deprecated
- `GET /api/users?name=` query parameter — use `/api/users/search?q=` instead.
  Will be removed in v3.0.0.

## Internal

Changes with no user-facing impact. No action required.

- Upgraded FastAPI 0.109 → 0.115
- Added integration tests for payment webhook handler
- Reduced Docker image size from 340MB → 120MB via multi-stage build
- Migrated CI from GitHub Actions to Harness Drone
```

### Deriving Release Notes from Conventional Commits

When using conventional commits (see git-workflow skill), map commit types to release note tiers:

| Commit type | Release note tier | Section |
|-------------|-------------------|---------|
| `feat` | Functional | **Added** |
| `fix` | Functional | **Fixed** |
| `BREAKING CHANGE` footer or `!` suffix | Breaking | **⚠ Breaking Changes** |
| `deprecated` in body | Functional | **Deprecated** |
| `perf` | Non-Functional (or Functional if user-visible) | **Internal** or **Changed** |
| `refactor`, `build`, `ci`, `test`, `chore` | Non-Functional | **Internal** |
| `docs` | Non-Functional (or Functional if user-facing docs) | **Internal** |

### Writing Rules

- Write from the **consumer's perspective** — describe what changed for them, not what code changed.
- **Breaking changes always come first** — they demand immediate attention.
- Every breaking change must include a **migration path** — what to do, not just what broke.
- Deprecations must state **when removal will happen** (version or date).
- Link to issues or PRs for traceability: `(#142)`.
- Keep entries to **one line** unless a migration guide is needed.
- Use imperative mood, present tense: "Add user search" not "Added user search endpoint".

### Quality Gate

Before publishing release notes, verify:

| Check | Question |
|-------|----------|
| **Complete** | Does every merged PR appear in a tier? |
| **Categorized** | Is each change in the correct tier (breaking / functional / non-functional)? |
| **Actionable** | Does every breaking change have a migration path? |
| **Traceable** | Does every entry link to an issue or PR? |
| **Consumer-readable** | Would someone outside the team understand each entry? |

---

## Changelog

Maintain a `CHANGELOG.md` following the **Keep a Changelog** format (see git-workflow skill for full template). The changelog is the cumulative history of all release notes.

### Changelog Rules

- Group by release version with a date: `## [1.5.0] - 2026-03-20`.
- Keep an `[Unreleased]` section at the top for in-progress changes.
- Use the same three-tier classification (Breaking / Functional / Non-Functional) as release notes.
- Never edit entries for past releases — corrections go in a new release.

---

## Release Checklist

Run through before every release:

| Step | Check |
|------|-------|
| **Tests pass** | All tests green on the release commit (see testing skill) |
| **Lint clean** | No lint or format violations |
| **Version bumped** | Version source file updated to the new version |
| **Changelog updated** | `[Unreleased]` entries moved under the new version heading |
| **No secrets** | No credentials, keys, or tokens in the release |
| **Dependencies audited** | `pip audit` / `npm audit` clean (see security skill) |
| **Breaking changes documented** | Migration guide written if MAJOR bump |
| **Tag created** | Annotated tag matches the version: `v1.5.0` |
| **CI green on tag** | Release pipeline triggered and completed successfully |

---

## CI/CD Integration

### Tag-Triggered Release Pipeline

Configure CI to build and publish only when a semver tag is pushed (see cicd skill for full pipeline patterns):

```yaml
---
kind: pipeline
name: release
trigger:
  event: [tag]
  ref: ["refs/tags/v*"]

steps:
  - name: validate-tag
    image: alpine:3.19
    commands:
      - |
        TAG="${DRONE_TAG}"
        if ! echo "$TAG" | grep -qE '^v[0-9]+\.[0-9]+\.[0-9]+(-[a-zA-Z0-9.]+)?$'; then
          echo "ERROR: Tag '$TAG' does not match semver format (vMAJOR.MINOR.PATCH)"
          exit 1
        fi
        echo "Valid semver tag: $TAG"

  - name: test
    image: python:3.12-slim
    commands:
      - pip install pytest
      - pytest -v --tb=short
    depends_on: [validate-tag]

  - name: build-and-publish
    image: plugins/docker
    settings:
      repo: ${DRONE_REPO_OWNER}/my-service
      tags:
        - latest
        - ${DRONE_TAG}
        - ${DRONE_TAG##v}
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
    depends_on: [test]
```

### Semver-Aware Docker Tags

Tag images with both the `v`-prefixed tag and the bare version:

| Git tag | Docker tags |
|---------|-------------|
| `v1.5.0` | `latest`, `v1.5.0`, `1.5.0` |
| `v2.0.0-beta.1` | `v2.0.0-beta.1`, `2.0.0-beta.1` (not `latest`) |

**Never tag pre-releases as `latest`** — only stable releases.

### Automated Version Bumping

For projects that derive version from git tags at build time:

```bash
# Get the latest version from git tags
VERSION=$(git describe --tags --abbrev=0 | sed 's/^v//')
echo "Building version: $VERSION"
```

For projects with a version file, use CI to validate that the tag matches the file:

```bash
FILE_VERSION=$(python -c "import tomllib; print(tomllib.load(open('pyproject.toml','rb'))['project']['version'])")
TAG_VERSION="${DRONE_TAG#v}"
if [ "$FILE_VERSION" != "$TAG_VERSION" ]; then
  echo "ERROR: pyproject.toml version ($FILE_VERSION) != tag ($TAG_VERSION)"
  exit 1
fi
```

---

## Release Cadence

| Model | When to use | Trade-offs |
|-------|-------------|------------|
| **Release when ready** | Most projects, especially small teams | Simple, flexible, no artificial deadlines |
| **Scheduled releases** | Products with external consumers or SLAs | Predictable for consumers, may delay features |
| **Continuous delivery** | Mature CI/CD, high test confidence | Fastest feedback, requires strong automation |

For most projects, **release when ready** is the right default. Ship when the feature set is coherent and tests pass.

---

## Anti-Patterns

| Anti-pattern | Why it's bad | Do this instead |
|--------------|-------------|-----------------|
| Bumping MAJOR for every change | Consumers can't trust version stability | Reserve MAJOR for actual breaking changes |
| Skipping versions (`1.0.0` → `1.5.0`) | Confusing, implies missing releases | Increment sequentially |
| Tagging without tests passing | Broken releases erode trust | CI must validate before publish |
| Manual changelog from memory | Incomplete, error-prone | Derive from conventional commits |
| Version in multiple files | Drift between sources | Single source of truth + build-time injection |
| Deleting and re-creating tags | Breaks downstream caches and references | Bad release → new PATCH release |
| No changelog | Consumers can't assess upgrade risk | Maintain CHANGELOG.md for every release |
| `v0.x.y` forever | Signals instability, discourages adoption | Promote to `1.0.0` when API is stable |
