# Personal Agent Skills

A community-maintained catalog of skills for
[personal-agent](https://github.com/michaelsvanbeek/personal-agent). Browse by
persona, install what's relevant, and contribute your own.

## How It Works

Each skill is a `SKILL.md` file in a named directory under `skills/`. Skills
follow the conventions defined in the
[agent-skill](https://github.com/michaelsvanbeek/personal-agent/blob/main/skills/agent-skill/SKILL.md)
skill from the core framework.

```
skills/
├── git-workflow/SKILL.md
├── meeting-agendas/SKILL.md
├── sql-style/SKILL.md
└── ...
```

## Browse by Persona

Not every skill is for everyone. Browse the catalog by role:

| Persona | Guide | Example Skills |
|---------|-------|---------------|
| Developer | [Developer Skills](docs/personas/developer.md) | git-workflow, code-review, testing, api-design |
| Manager | [Manager Skills](docs/personas/manager.md) | meeting-agendas, sprint-planning, stakeholder-comms |
| Analyst | [Analyst Skills](docs/personas/analyst.md) | sql-style, chart-design, data-storytelling |
| Writer | [Writer Skills](docs/personas/writer.md) | editorial-standards, tone-guide, citation-standards |
| Designer | [Designer Skills](docs/personas/designer.md) | design-review, accessibility, component-specs |
| Researcher | [Researcher Skills](docs/personas/researcher.md) | literature-review, citation-standards, academic-writing |
| Operations | [Operations Skills](docs/personas/operations.md) | okr-tracking, process-docs, incident-response |
| Marketing | [Marketing Skills](docs/personas/marketing.md) | brand-voice, campaign-planning, content-calendar |

See the [full skill catalog](docs/catalog.md) for all skills with tags.

## Installing Skills

There are two ways to install skills from this repo:

### Option 1: Copy skills manually

```bash
# Copy a skill
cp -r skills/meeting-agendas ~/code/personal-agent/skills/

# Install into your IDEs
cd ~/code/personal-agent && ./install.sh
```

### Option 2: Symlink the whole repo (Recommended)

This keeps your skills up-to-date when the repo updates:

```bash
cd ~/code/personal-agent
./install.sh ~/code/personal-agent-skills
```

Skills are now symlinked to the source repo. When you pull updates from this repo, your IDE automatically loads the latest versions.

## Contributing

We welcome skill contributions from anyone. Before submitting:

1. **Read the conventions** in the
   [agent-skill](https://github.com/michaelsvanbeek/personal-agent/blob/main/skills/agent-skill/SKILL.md)
   skill
2. **Follow the quality checklist** in [Contributing Guide](docs/contributing.md)
3. **Tag your skill** with relevant personas
4. **Keep it neutral** — public skills should reflect broadly accepted practices,
   not organization-specific standards or polarized opinions

### Pre-commit checks

This repo includes a pre-commit hook that audits all skills for:
- Frontmatter validity (name, description, tags)
- Duplication detection across the collection
- Ambiguity detection (overlapping triggers)
- Quality standards (line count, prescriptive content, examples)

Install the hooks:

```bash
.hooks/install
```

The hook runs the audit before every commit and blocks if errors are found.

## Context Management

You don't need every skill in this catalog. AI assistants have finite context
windows — adding too many skills creates noise and degrades responses.

**Start with 3-5 skills** that directly support your daily work. Add more only
when you notice the agent getting something wrong. Remove skills you haven't
triggered in months.

See the [Design Guide](https://github.com/michaelsvanbeek/personal-agent/blob/main/docs/design.md)
in the core framework for the full context management philosophy.

## Organization-Specific Skills

This catalog is for public, broadly-applicable skills. If you're building skills for your
organization (proprietary workflows, internal tool integrations, company policies), create
a private "internal org skills" repo instead.

See [Internal Org Skills Repositories](docs/internal-org-skills.md) for a complete guide
to building, structuring, and deploying org-specific skills.

## Tags

Skills are tagged with personas to help you discover what's relevant. See
[Tag Registry](docs/tags.md) for the full list of recognized tags.

## License

[MIT](LICENSE)
