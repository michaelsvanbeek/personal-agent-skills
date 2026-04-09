# Contributing Skills

How to contribute a skill to the community catalog.

---

## Before You Start

1. **Check for duplicates.** Search the [skill catalog](catalog.md) to see if a
   similar skill already exists. If it does, consider improving it rather than
   creating a new one.
2. **Read the conventions.** The
   [agent-skill](https://github.com/YOUR_USERNAME/personal-agent/blob/main/skills/agent-skill/SKILL.md)
   in the core framework defines all the rules for writing skills.

---

## Step by Step

### 1. Create the skill directory

```bash
mkdir -p skills/my-skill-name
```

### 2. Write the SKILL.md

Every skill file needs YAML frontmatter with `name`, `description`, and `tags`:

```yaml
---
name: my-skill-name
description: >-
  One-sentence summary of domain. Use when: trigger 1, trigger 2, trigger 3,
  or trigger 4.
tags:
  - manager
  - communication
---

# Skill Title

## When to Use

- Concrete scenario 1
- Concrete scenario 2

## Core Content

Prescriptive guidance with examples...
```

### 3. Self-check

Before submitting, verify:

- [ ] `name` matches the directory name
- [ ] `description` follows the `<summary>. Use when: <triggers>.` formula
- [ ] At least 3 "Use when" triggers in the description
- [ ] `tags` field includes 1-5 persona tags from the [Tag Registry](tags.md)
- [ ] Content is prescriptive (tells the agent what to do)
- [ ] Includes concrete examples
- [ ] Under 500 lines (200-400 ideal)
- [ ] No organization-specific standards or internal URLs
- [ ] No polarized opinions — focuses on broadly accepted practices
- [ ] No personal data or credentials

### 4. Run the audit

```bash
.hooks/audit-skills
```

This runs the same check as the pre-commit hook. Fix any errors before
submitting.

### 5. Submit a pull request

Open a PR with:
- The new skill directory and SKILL.md
- A brief description of what the skill covers and who it's for
- Confirmation that the audit passes

---

## Content Guidelines

### Broadly accepted practices only

Public skills should reflect practices that most professionals in the domain
would agree with. Avoid:

- Organization-specific standards ("Our team uses X")
- Strongly opinionated stances where the community is divided
- Tool-specific guidance when the principle applies across tools
- Internal jargon, URLs, or credentials

Users can always extend community skills with their own opinionated additions in
their personal `personal-agent` repo.

### Appropriate decomposition

Each skill should cover **one coherent domain** — not one tip, and not an entire
handbook. If your skill tries to do too much, split it. If it's too narrow,
combine it with a related skill.

The pre-commit audit checks for:
- **Overlapping triggers** between your skill and existing ones
- **Ambiguous descriptions** that would match too many conversations
- **Missing tags** or unrecognized tag values

### Examples matter

Every major section should have at least one concrete example. Abstract guidance
without examples forces the agent to guess, which reduces reliability.

---

## Updating Existing Skills

Improvements to existing skills are welcome. Follow the same PR process. The
audit will verify that your changes don't introduce duplication or ambiguity.
