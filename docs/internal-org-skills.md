# Internal Org Skills Repositories

If you're building a software team, marketing department, or any organizational unit with shared practices, you can create a private, organization-specific skills repository following the same pattern as `personal-agent-skills`.

## Why Create an Internal Repo

**personal-agent-skills** is for public, broadly-applicable skills:
- Code review best practices (applies to any team)
- Git workflow conventions (broadly accepted standards)
- Testing strategies (universally useful)

**Internal org repos** are for organization-specific knowledge:
- Your company's approval workflows
- Internal tool integrations (proprietary systems, custom databases)
- Company-specific policies and standards
- Lessons learned from your specific business domain
- Org-specific terminology and acronyms

The separation allows you to:
1. Use community skills without modification
2. Layer org-specific knowledge on top
3. Maintain privacy (internal repo stays private)
4. Version and audit org knowledge independently

## Repository Structure

Create `my-org-agent-skills/` (or `my-company-skills/`) with this layout:

```
my-org-agent-skills/
├── README.md                          ← installation guide
├── LICENSE                            ← MIT or your org's license
├── skills/
│   ├── internal-processes/
│   │   └── SKILL.md                   ← e.g., expense approval, onboarding
│   ├── company-standards/
│   │   └── SKILL.md                   ← e.g., code standards, naming conventions
│   └── tool-integrations/
│       └── SKILL.md                   ← e.g., Jira setup, Slack best practices
├── instructions/
│   └── internal-policies.instructions.md  ← always-on rules (compliance, etc.)
├── docs/
│   ├── README.md                      ← docs index
│   └── contributing.md                ← contribution guide
├── .hooks/
│   └── audit-skills                   ← pre-commit audit (same as base repo)
└── .gitignore
```

### Naming Conventions

Use **clear, specific names** that signal org content:

| Example | Good | Bad |
|---------|------|-----|
| Approver workflows | `expense-approval`, `hiring-workflow` | `processes`, `approvals` |
| Tool setup | `jira-setup-guide`, `slack-integration` | `tools`, `integrations` |
| Company policies | `data-retention-policy`, `security-compliance` | `policies`, `rules` |

### Key Differences from public-agent-skills

1. **Organization-specific content is expected** — Internal skills *should* reference company tools, workflows, and standards.

2. **No persona filtering needed** — If it's your company, everyone in a role gets the same skills. You don't need the public library's persona tags. (Optional: use simpler tags like `engineering`, `operations`, `sales`.)

3. **Stricter governance** — Audit internal skills for:
   - **Compliance violations** — No hardcoded credentials, API keys, database schemas, or customer PII
   - **Outdated information** — Skills referencing deprecated internal systems
   - **Security gaps** — No sensitive URLs, hostnames, or authentication details

4. **Optional: deeper integration** — Unlike the public library (which stays lean), org repos can be more detailed:
   - 800+ line skills covering complex internal workflows
   - Step-by-step guides for proprietary systems
   - Links to internal wikis, Confluence docs, or runbooks

## Installation & Deployment

### For a Single Developer

Clone the org-specific repo and install:

```bash
git clone https://github.com/my-org/my-org-agent-skills.git
cd personal-agent
./install.sh ../my-org-agent-skills
```

This symlinks both base skills and org skills into your IDE.

### For a Team (CI/CD Automation)

Set up automated deployment so developers get skillsupdates without manual installation:

1. **Create a deploy script** in your org repo:

```bash
#!/bin/bash
# .github/workflows/deploy-skills.yml

name: Deploy Skills
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Deploy to team machines
        run: |
          for dev_machine in $(cat .github/team-machines.txt); do
            ssh "$dev_machine" "cd personal-agent && \
              git pull && \
              ./install.sh ../my-org-agent-skills"
          done
```

2. **Team members pull changes:**

```bash
cd my-org-agent-skills
git pull

# Their install.sh automatically picks up the updated skills
cd ../personal-agent
./install.sh ../my-org-agent-skills
```

### For Multiple Org Repos

Layer public + private skills:

```bash
cd personal-agent

# Install base framework
./install.sh

# Install public community skills
./install.sh /path/to/personal-agent-skills

# Install org-specific skills
./install.sh /path/to/my-org-agent-skills
```

Each call to `install.sh` adds more skills without removing previous ones.

## README.md Template

```markdown
# My Organization Agent Skills

Agent skills for [Organization Name] — internal workflows, standards, and tool integrations.

## Installing

```bash
git clone https://github.com/my-org/personal-agent.git
cd personal-agent
./install.sh /path/to/my-org-agent-skills
\`\`\`

This symlinks all skills into your IDE context.

## Skills Overview

| Skill | Purpose | Tags |
|-------|---------|------|
| [expense-approval](skills/expense-approval/SKILL.md) | End-to-end expense approval workflow | operations, finance |
| [jira-setup](skills/jira-setup/SKILL.md) | How to configure Jira for our team | operations, engineering |
| [data-retention](skills/data-retention/SKILL.md) | Data retention and compliance policy | operations, compliance |

## Contributing

Before submitting a skill:

1. **Test it** — Run the skill and verify it works as expected
2. **Check for security** — No hardcoded credentials, API keys, or PII
3. **Keep it current** — Update if it references deprecated systems
4. **Write clear "Use when:" triggers** — So the IDE can match and load it
5. **Document assumptions** — What systems/tools does it assume are set up?

See [Contributing](docs/contributing.md) for the full checklist.

## Linking to Public Resources

- [personal-agent](https://github.com/michaelsvanbeek/personal-agent) — Base framework
- [personal-agent-skills](https://github.com/michaelsvanbeek/personal-agent-skills) — Public skills library

## License

[MIT](LICENSE)
\`\`\`

## Contributing Guide

Create `docs/contributing.md` with your org-specific checklist:

```markdown
# Contributing

Thanks for sharing skills with the team!

## Self-Check Before Submitting

- [ ] I've tested this skill end-to-end
- [ ] No hardcoded credentials, API keys, or internal APIs
- [ ] No customer PII or confidential data
- [ ] The "Use when:" describes when someone would actually need this
- [ ] It references internal systems by name (e.g., "our Jira", "our PostgreSQL cluster")
- [ ] The skill works without modification for the target audience
- [ ] It links to relevant internal wikis/docs where applicable

## Quality Bar

Skills should:
- Solve a real problem your org faces
- Follow the same structure as [agent-skill](https://github.com/michaelsvanbeek/personal-agent-skills/blob/main/skills/agent-skill/SKILL.md)
- Be <500 lines (shorter is better)
- Avoid duplication (check [catalog.md](catalog.md) first)
- Maintain privacy (no hardcoded secrets, URLs, or customer data)

## Review Process

1. Open a PR with your new skill
2. Maintainers review for quality, security, and accuracy
3. Merge and deploy via CI/CD

## Questions?

Ask in #agent-skills on Slack or open an issue.
\`\`\`

## Example Internal Skills

### Skill: Expense Approval Workflow

```markdown
# Expense Approval Workflow

**Description:** Complete guide to submitting and approving expenses at ACME Corp. Covers what's reimbursable, approval tiers, documentation requirements, and timeline.

**Use when:**
- You need to submit an expense for reimbursement
- You're reviewing an expense report from your team
- You're unsure what's eligible for reimbursement
```

### Skill: Jira Setup for Engineering

```markdown
# Jira Setup for Our Engineering Team

**Description:** How to configure Jira projects, custom fields, workflows, and issue templates for ACME's engineering team. Includes our naming conventions, issue types, and board setup.

**Use when:**
- Setting up a new Jira project
- Onboarding a new engineer who hasn't used our Jira before
- You're unsure about our issue naming or workflow
```

### Skill: Data Retention Policy

```markdown
# Data Retention and Compliance

**Description:** ACME Corp's data retention requirements, customer privacy obligations, and what data we can safely delete. Required reading for engineers, ops, and data teams.

**Use when:**
- Writing data deletion or cleanup code
- Designing a new system that stores customer data
- Preparing for an audit or compliance review
```

## Security Best Practices

### ✗ Do NOT include in skills:

- API keys, tokens, or credentials
- Customer names, emails, or personal information
- Internal IP addresses or hostnames
- Database credentials or connection strings
- Slack channel IDs or team membership lists
- Sensitive pricing or financial data

### ✓ DO include:

- Process and workflow descriptions
- System names (e.g., "our PostgreSQL cluster", "our Jira instance")
- Generic examples (make-up fake data)
- Links to internal wikis where sensitive details live
- Public documentation and standards
- General organizational practices

### Example (Secure):

```markdown
# Database Setup

To connect to our data warehouse, follow these steps:

1. Install the PostgreSQL client: `brew install postgresql@15`
2. Retrieve credentials from [1Password](https://1password.com) (ask #ops for access)
3. Connect: `psql -h <hostname> -U <username> -d warehouse`

See [Internal Wiki: Database Access](https://wiki.internal/database-access) for detailed setup instructions.
```

The wiki link has the actual hostname and credentials; the skill just describes the process.

## Syncing with Public Skills

Since personal-agent-skills may update with new public skills, periodically:

1. Check the [personal-agent-skills changelog](https://github.com/michaelsvanbeek/personal-agent-skills/blob/main/CHANGELOG.md)
2. Merge updates: `git pull upstream main` (if you forked it)
3. Layer your org skills on top without modifying public ones

You can use both simultaneously — they don't conflict.

## Next Steps

- [Design your first skill](https://github.com/michaelsvanbeek/personal-agent-skills/blob/main/docs/contributing.md)
- [Personal Agent documentation](https://github.com/michaelsvanbeek/personal-agent/blob/main/docs/README.md)
- [Contact us](https://github.com/michaelsvanbeek/personal-agent/discussions) with questions
