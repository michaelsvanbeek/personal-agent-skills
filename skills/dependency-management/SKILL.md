---
name: dependency-management
description: 'Dependency evaluation, supply chain security, and maintenance for Python and JavaScript projects. Use when: evaluating whether to add a new dependency, comparing alternatives, pruning unused dependencies, auditing for vulnerabilities, configuring Renovate or Dependabot, managing lockfiles, checking license compliance, resolving version conflicts, remediating CVEs, or auditing an existing project for dependency hygiene. Covers the full lifecycle: evaluate → add → lock → audit → update → prune.'
tags:
  - developer
  - operations
---

# Dependency Management

## When to Use

- Evaluating whether to add a new dependency or write it yourself
- Comparing alternative packages for a given need
- Pruning unused or redundant dependencies from an existing project
- Auditing dependencies for known vulnerabilities and remediating issues
- Configuring Renovate or Dependabot for automated updates
- Managing lockfiles across environments
- Checking dependency licenses for compliance
- Resolving version conflicts or peer dependency issues
- Auditing an existing project for outdated, insecure, or unnecessary dependencies

---

## Core Principle: Every Dependency is a Liability

Dependencies are **attack surface**, **maintenance burden**, and **build-time cost**. The safest, fastest, most maintainable dependency is the one you don't add. Treat every dependency as a decision that requires justification — not a default.

---

## Dependency Evaluation Framework

> **Foundation**: Dependency evaluation is a domain-specific application of the **alternatives-analysis** framework. The generic skill defines decision framing, criteria weighting, summary tables, and recommendation structure. This section adds dependency-specific criteria (supply chain security, maintenance health, bundle size, license compliance).

Before adding any dependency, complete this evaluation. **The default answer is "don't add it."**

### Step 1: Do You Need It at All?

Ask these questions first:

1. **Can the standard library do this?** Python's `pathlib`, `json`, `urllib`, `dataclasses`, `asyncio` — and Node's `crypto`, `fs`, `url`, `util` — cover more than most developers realize.
2. **Is this a small utility?** If the dependency is <100 lines of code, write it yourself. You avoid a supply chain risk for trivial convenience.
3. **Are you using <20% of the library?** If you only need one function from a large package, extract just that logic.
4. **Is this a transient need?** Dev tooling (linters, test frameworks) as dev dependencies is fine. But runtime dependencies for one-off tasks should be scrutinized.

**Rule**: If the answer to any of questions 1–3 is yes, don't add the dependency.

### Step 2: Evaluate Alternatives

When a dependency is justified, evaluate at least **three alternatives** (including "write it ourselves") and produce a decision report.

#### Evaluation Criteria

| Criterion | What to Check | Red Flags |
|-----------|---------------|-----------|
| **Maintenance** | Last commit, release cadence, open issues/PRs | No commits in 12+ months, hundreds of stale issues |
| **Popularity** | Downloads/week, GitHub stars, dependents count | <1,000 weekly downloads for a general-purpose lib |
| **Size** | Install size, transitive dependency count | >5MB install size, >20 transitive deps for simple functionality |
| **Security** | Known CVEs, Snyk/Socket.dev score, maintainer count | Unpatched CVEs, single maintainer, no 2FA on publish |
| **License** | SPDX identifier | GPL/AGPL in proprietary project, no license at all |
| **API quality** | TypeScript types, documentation, breaking change history | No types, poor docs, frequent major version bumps |
| **Alternatives** | Standard library coverage, smaller alternatives | stdlib can do 80% of what this package does |

#### Evaluation Commands

```bash
# Python — check package metadata
uv pip show <package>
pip-audit -r requirements.txt  # or pip-audit on installed env

# Python — check install size and transitive deps
uv pip install --dry-run <package>

# Node.js — check package size and dependencies
npm view <package> dist.unpackedSize
npm view <package> dependencies

# Node.js — bundle size impact
npx bundlephobia-cli <package>

# Both — check maintenance health
# Visit: https://snyk.io/advisor/python/<package>
# Visit: https://snyk.io/advisor/npm-package/<package>
# Visit: https://socket.dev/npm/package/<package>
```

### Step 3: Produce a Decision Report

For any non-trivial dependency addition, document the decision. This report lives in the PR description or as a comment in code.

#### Decision Report Template

```markdown
## Dependency Decision: <need>

**Need**: <one sentence describing what capability is needed>

### Alternatives Evaluated

| Option | Pros | Cons | Size | Deps | Maintained |
|--------|------|------|------|------|------------|
| Write ourselves | No supply chain risk, exact fit | Dev time, maintenance burden | 0 | 0 | Us |
| <package-a> | <pros> | <cons> | <size> | <count> | <yes/no> |
| <package-b> | <pros> | <cons> | <size> | <count> | <yes/no> |
| stdlib <module> | Zero deps, always available | <limitations> | 0 | 0 | Yes |

### Decision

**Chosen**: <package or "write ourselves">
**Rationale**: <2-3 sentences explaining why>
**Risk**: <what could go wrong, mitigation>
```

**Rule**: No runtime dependency is added without this evaluation. Dev dependencies (test frameworks, linters) need only a brief justification.

---

## Aggressive Dependency Pruning

### When to Prune

- During every major feature completion (part of the PR checklist)
- Quarterly as a scheduled maintenance task
- Immediately when a CVE is found in an unused dependency
- When upgrading major versions of a framework (old compatibility shims may be unnecessary)

### Detection Tools

```bash
# Python — find unused imports (which suggest unused deps)
ruff check --select F401 .

# Python — find unused dependencies
pip-extra-reqs --ignore-module=tests src/

# Node.js — find unused dependencies
npx depcheck

# Node.js — find unused exports (tree-shaking misses)
npx ts-unused-exports tsconfig.json
```

### Pruning Rules

- **Remove, don't comment out.** Commented-out dependencies in manifests are clutter that gets restored accidentally.
- **Remove transitive pins** when the parent dependency is removed.
- **Test after every removal.** Run full test suite + build to confirm nothing breaks.
- **Commit each removal separately** so it can be reverted independently.
- **Check for hidden usage**: config files, scripts, CI pipelines, Dockerfiles — not just source code imports.

### Pruning Checklist

- [ ] Run unused-dependency detection tool
- [ ] Cross-reference with imports across all source files, scripts, and configs
- [ ] Remove each unused dependency individually
- [ ] Run full test suite after each removal
- [ ] Update lockfile after removals
- [ ] Check bundle size / install size before and after

---

## Lockfile Hygiene

### Rules

- **Always commit lockfiles** to version control: `uv.lock`, `package-lock.json`, `pnpm-lock.yaml`.
- **Never edit lockfiles manually.** Regenerate via package manager.
- Install from lockfile in CI and production: `uv sync --frozen`, `npm ci` (not `npm install`).
- Lockfile changes should be in **separate commits** from code changes for clean diffs.

### Python (uv)

```bash
uv add httpx                    # Add dependency (updates pyproject.toml + uv.lock)
uv add --dev pytest             # Add dev dependency
uv sync --frozen                # Install from lockfile (CI/production)
uv lock --upgrade-package httpx # Update single package
uv lock --upgrade               # Update all packages
```

### JavaScript (npm)

```bash
npm ci                          # Install from lockfile (CI/production)
npm install zustand             # Add dependency
npm install -D vitest           # Add dev dependency
npm update zustand              # Update single package
npm audit                       # Audit for vulnerabilities
```

---

## Version Pinning Strategy

| File | Pin Strategy | Why |
|------|-------------|-----|
| `pyproject.toml` | Range (`>=1.2,<2`) | Allow compatible updates; lockfile pins exact |
| `package.json` | Range (`^1.2.3`) | Same; lockfile pins exact |
| `Dockerfile` base image | Exact tag (`python:3.12.8-slim`) | Reproducible builds |
| CI tool versions | Exact (`ruff==0.8.0`) | Reproducible CI |
| `uv.lock` / `package-lock.json` | Exact (auto-generated) | Never edit manually |

### Rules

- Ranges in manifest files; exact pins in lockfiles.
- **Never use `latest` or `*`** as a version specifier.
- Pin Dockerfile base images to specific tags, not `latest` or just major version.
- When vendoring, document the exact version and source.

---

## Supply Chain Security

### Artifact Verification

- **Enable hash verification** in lockfiles. `uv.lock` and `package-lock.json` include integrity hashes by default — never disable this.
- **Verify signatures** when available. Use `--require-hashes` with pip-audit.
- For critical dependencies, pin to a specific hash in addition to version.

### Dependency Confusion Prevention

- When mixing private and public packages, configure scoped registries:
  ```bash
  # .npmrc — scope internal packages to private registry
  @myorg:registry=https://npm.pkg.github.com
  ```
- Never publish internal package names to public registries.
- Use distinct naming conventions for internal packages (e.g., `@myorg/` scope).

### Typosquatting Awareness

- Double-check package names before installing. Common attack: `reqeusts` vs `requests`.
- Use Snyk or Socket.dev to flag suspicious packages.
- In CI, fail on packages with no published source or provenance.

### Maintainer Risk

- Prefer packages with **multiple maintainers** and organizational ownership.
- Single-maintainer packages with high download counts are high-value attack targets.
- Monitor for **maintainer transfers** — a new maintainer on a popular package warrants review.

---

## Vulnerability Auditing and Remediation

### Audit Commands

| Ecosystem | Command | Output |
|-----------|---------|--------|
| Python | `pip-audit --strict` | Known CVEs with fix versions |
| Python | `pip-audit --require-hashes --strict` | CVEs with hash verification |
| Node.js | `npm audit` | Known advisories in dependency tree |
| Docker | `docker scout cves <image>` | CVEs in container image layers |
| Multi | `trivy fs .` | Filesystem scan across ecosystems |

### Remediation Workflow

When a vulnerability is found:

1. **Classify severity**: Critical/High → fix immediately. Medium → fix this sprint. Low → fix this quarter.
2. **Check if a fix exists**: `pip-audit --fix` or `npm audit fix` will auto-update if a patched version exists.
3. **If no fix exists**:
   - Check if the vulnerable code path is actually reachable in your usage.
   - If reachable: find an alternative package, or implement the functionality yourself.
   - If not reachable: document with a `# safety:` comment, link to the CVE, and set a calendar reminder to re-check.
4. **Test the fix**: Run full test suite. Dependency updates can introduce breaking changes.
5. **Document**: Add CVE number to the commit message. If the fix required a workaround, document it.

### CI Integration

```yaml
# Drone CI — Python
- name: audit-dependencies
  image: python:3.12-slim
  commands:
    - pip install pip-audit
    - pip-audit --require-hashes --strict

# Drone CI — Node.js
- name: audit-dependencies
  image: node:22-alpine
  commands:
    - npm ci
    - npm audit --audit-level=high
```

### Rules

- Run audits in **every CI pipeline**, not just on a schedule.
- **Critical/High severity** vulnerabilities block the build.
- Schedule a **weekly audit job** to catch newly disclosed vulnerabilities between code changes.
- Subscribe to security advisories for your critical dependencies (GitHub Advisory Database, Snyk).

---

## Automated Update Tools

### Renovate (Recommended)

```json
{
  "$schema": "https://docs.renovatebot.com/renovate-schema.json",
  "extends": ["config:recommended"],
  "labels": ["dependencies"],
  "rangeStrategy": "pin",
  "lockFileMaintenance": {
    "enabled": true,
    "schedule": ["before 6am on monday"]
  },
  "packageRules": [
    {
      "matchUpdateTypes": ["patch"],
      "automerge": true
    },
    {
      "matchUpdateTypes": ["minor"],
      "groupName": "minor updates"
    },
    {
      "matchUpdateTypes": ["major"],
      "labels": ["dependencies", "breaking"]
    }
  ]
}
```

### Dependabot

```yaml
version: 2
updates:
  - package-ecosystem: pip
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
    labels:
      - dependencies
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 10
    labels:
      - dependencies
```

### Update Cadence

| Priority | Cadence | Action |
|----------|---------|--------|
| **Critical security fix** | Immediately | Patch and deploy same day |
| **High severity CVE** | Within 48 hours | Update, test, deploy |
| **Patch versions** | Weekly (automated) | Automerge via Renovate/Dependabot |
| **Minor versions** | Bi-weekly | Review grouped PR, check changelogs |
| **Major versions** | Monthly review | Evaluate breaking changes, plan migration |

### Rules

- **Patch updates**: Automerge if CI passes.
- **Minor updates**: Group into a single PR per week; review changelog.
- **Major updates**: Individual PRs with changelog review, alternative evaluation, and manual testing.
- **Schedule updates** for low-traffic times (Monday morning).

---

## License Compliance

### Approved Licenses

`MIT`, `Apache-2.0`, `BSD-2-Clause`, `BSD-3-Clause`, `ISC`, `0BSD`.

### Requires Legal Review

`GPL-2.0`, `GPL-3.0`, `AGPL-3.0`, `SSPL`, `EUPL` — flag before use in proprietary projects.

### Blocked

Unlicensed packages must not be used in production.

### Tools

```bash
# Python
pip-licenses --format=table --with-urls

# Node.js
npx license-checker --summary
npx license-checker --onlyAllow "MIT;Apache-2.0;BSD-2-Clause;BSD-3-Clause;ISC;0BSD"
```

---

## Resolving Version Conflicts

### Diagnosis

```bash
uv pip tree                     # Python dependency tree
npm ls                          # Node.js dependency tree
npm explain <package>           # Why is this installed?
```

### Resolution Strategy

1. Identify which top-level dependencies pull in conflicting transitive versions.
2. Check if updating the top-level dependency resolves the conflict.
3. If not, check if the libraries are compatible with a shared range.
4. As a last resort, use overrides (`resolutions` in package.json, `tool.uv.override` in pyproject.toml).
5. **Document any overrides** with a comment explaining why and a link to the upstream issue.

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Adding deps without evaluation | Bloated supply chain, hidden risk | Complete the evaluation framework |
| No lockfile committed | Non-reproducible builds | Commit lockfile, use `--frozen` in CI |
| `npm install` in CI | Ignores lockfile | Use `npm ci` |
| Ignoring audit warnings | Vulnerabilities accumulate | Enforce in CI, triage immediately |
| Never pruning | Dead deps increase attack surface | Quarterly prune with detection tools |
| Pinning everything in manifest | Can't receive compatible patches | Use ranges in manifest, pins in lockfile |
| `latest` base images | Builds break unpredictably | Pin to specific tag |
| Single-maintainer runtime deps | High supply chain risk | Prefer org-owned, multi-maintainer packages |
| Copy-pasting `npm install <x>` from tutorials | Untriaged deps enter the project | Every addition goes through evaluation |

---

## Audit Checklist

When auditing an existing project for dependency hygiene:

- [ ] Every runtime dependency has a documented justification
- [ ] Unused dependencies have been removed (run detection tools)
- [ ] Lockfile exists and is committed to version control
- [ ] CI installs from lockfile (`npm ci`, `uv sync --frozen`)
- [ ] `pip-audit` or `npm audit` runs in every CI pipeline
- [ ] Critical/High CVEs block the build
- [ ] Automated update tool (Renovate or Dependabot) is configured
- [ ] Patch updates automerge; minor/major require human review
- [ ] No `latest` tags in Dockerfile base images
- [ ] License compliance check runs
- [ ] Dependency overrides are documented with comments
- [ ] A scheduled weekly CI job audits for newly disclosed vulnerabilities
- [ ] Supply chain protections are in place (hash verification, scoped registries)
- [ ] Transitive dependency count has been reviewed for outliers
