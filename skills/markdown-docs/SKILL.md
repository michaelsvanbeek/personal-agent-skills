---
name: markdown-docs
description: >-
  Markdown documentation standards for developer-facing and project documentation.
  Use when: writing or auditing a README, creating project documentation, structuring
  a docs/ folder, documenting design decisions, writing ADRs, linking documents for
  discoverability, or ensuring an existing project has adequate developer documentation.
tags:
  - developer
  - writer
  - general
---

# Markdown Documentation Standards

## When to Use

- Writing or improving a project README
- Creating a `docs/` folder structure
- Documenting a design decision (ADR)
- Writing a guide for a significant feature
- Auditing an existing project for documentation gaps
- Establishing documentation conventions

> **Scope:** This skill covers developer-facing docs (README, docs/, ADRs).
> For in-product explanatory UX, use the ui-design skill.

---

## Core Principle

Documentation lives with the code. It is version-controlled, reviewed in PRs,
and updated alongside the code it describes. The README is the entry point —
everything else is one or two links away.

- Keep docs in-repo whenever possible.
- Use one canonical source per topic, then link instead of duplicating.
- Treat docs updates as part of the feature/fix definition of done.

---

## README.md

Every project must have a README at the repository root.

### Required Sections

```markdown
# Project Name

One-sentence description of what this project does and why it exists.

## Overview

2-3 paragraphs describing the problem, key design choices, and what
someone should understand before working on this codebase.

## Quick Start

Minimum commands to get the project running locally:

git clone <repo>
cd <project>
npm install && npm run dev

## Usage

How to use the project — commands, endpoints, or screens.

## Local Development

Dependencies, run, test, and lint commands.

## Configuration

Table of environment variables with descriptions and defaults.

## Documentation

Links to docs/ contents: architecture, ADRs, runbooks.
```

### README Rules

- **Keep it short** — the README introduces and orients, it doesn't explain
  everything. Link to `docs/` for depth.
- **Quick Start must be copy-pasteable** — every command must work on a fresh
  checkout.
- **Update with every significant change** — outdated install instructions
  are worse than none.
- No badge farming — only include badges that provide real signal (build
  status, coverage).

---

## docs/ Folder Structure

```
docs/
├── README.md              ← index linking to every document
├── architecture.md        ← system architecture and data flows
├── concepts.md            ← domain objects and terminology
├── runbooks/
│   ├── deploy.md
│   └── rollback.md
├── adr/
│   ├── README.md          ← ADR index with status summaries
│   ├── 0001-use-dynamodb.md
│   └── 0002-oauth2-auth.md
└── changes/
    └── v2.0-migration.md
```

**Every file in `docs/` must be linked from `docs/README.md`.** No orphan
documents.

### docs/README.md Index Template

```markdown
# Documentation

## Architecture
- [Architecture Overview](architecture.md)
- [Key Concepts](concepts.md)

## Runbooks
- [Deployment](runbooks/deploy.md)
- [Rollback](runbooks/rollback.md)

## Decisions
- [ADR Index](adr/README.md)

## Changes
- [Migration Guides](changes/)
```

---

## Architecture Decision Records (ADRs)

Write an ADR when:
- You chose one technology or approach over clear alternatives
- A decision is hard to reverse or expensive to change
- Future developers will wonder "why did they do it this way?"

### ADR Template

```markdown
# ADR 0001: Title of Decision

**Date**: YYYY-MM-DD
**Status**: Proposed | Accepted | Superseded | Deprecated

## Context

What is the problem or situation that requires a decision?

## Decision

What was decided and why.

## Alternatives Considered

| Option | Pros | Cons |
|--------|------|------|
| Option A | ... | ... |
| Option B | ... | ... |
| **Chosen** | ... | ... |

## Consequences

What follows from this decision — both positive and negative.
```

### ADR Status Values

| Status | Meaning |
|--------|---------|
| `Proposed` | Under discussion, not yet accepted |
| `Accepted` | Decision is approved and in effect |
| `Superseded` | Replaced by a later ADR |
| `Deprecated` | No longer recommended for new work |

---

## Linking and Discoverability

Every document must be reachable from the README:

```
README.md
  → docs/README.md (index)
      → docs/architecture.md
      → docs/adr/README.md
          → docs/adr/0001-use-dynamodb.md
      → docs/runbooks/deploy.md
  → CHANGELOG.md
```

### Link rules

- Every file in `docs/` is linked from `docs/README.md`
- `docs/README.md` is linked from the top-level README
- ADRs are catalogued in `docs/adr/README.md`
- No orphan documents — if it exists, it's linked from its parent index
- Reference other docs with links, don't copy content

---

## Markdown Formatting

### Structure

- One `# H1` per document (the title)
- Use `##` for major sections, `###` for subsections
- Don't go deeper than `####`
- Separate headings from preceding content with a blank line

### Lists

- Use `-` for unordered lists
- Use `1.` for ordered steps
- Limit nesting to 2 levels
- Keep items parallel in grammar and length

### Code blocks

Always specify the language:

````markdown
```python
def greet(name: str) -> str:
    return f"Hello, {name}"
```
````

Use inline code for command names (`ruff`), file names (`pyproject.toml`),
environment variables (`APP_DATABASE_URL`), and short expressions (`X | Y`).

### Tables

- Include a header row with `---` separators
- Right-align numeric columns with `---:`
- Use tables for structured comparisons or reference data

### Links

#### Display text vs URL

Every link has two parts with different jobs:

| Part | Role | Rule |
|------|------|------|
| **Display text** | Human-readable label | Friendly, descriptive, never a raw ID or "click here" |
| **URL** | Machine-readable address | Stable, ID-based when possible, relative for repo links |

#### Good links

```markdown
[Architecture Overview](docs/architecture.md)
[GitHub Issues](https://github.com/owner/repo/issues)
```

#### Bad links

```markdown
[click here](docs/architecture.md)
[docs/architecture.md](docs/architecture.md)
https://github.com/owner/repo/issues
```

**Display text rules:**
- Use the canonical human-readable name
- Match detail to context — "architecture doc" inline vs "Architecture
  Overview" in an index
- Never expose raw IDs or URLs as display text

**URL rules:**
- Use relative paths for documents within the repository
- Use stable, ID-based URLs for external links when available
- Validate that links resolve before committing

### Line length and whitespace

- Hard-wrap prose at 100 characters where practical
- Separate paragraphs and sections with a blank line
- No trailing whitespace

### Callouts

Use blockquotes for important notes:

```markdown
> **Note:** This only applies to production environments.

> **Warning:** This operation is destructive and cannot be undone.
```

---

## Change Documents

For major changes, write a focused guide in `docs/changes/`:

```markdown
# Migrating from v1 to v2

## What Changed

Summary of breaking changes.

## Steps

### 1. Update Authentication

**Before (v1):**
GET /api/users
X-API-Key: your-key

**After (v2):**
GET /api/users
Authorization: Bearer your-token
```

Required for every major version release. Recommended for significant changes
that alter development workflows, even if not breaking.
