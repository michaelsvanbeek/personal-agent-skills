---
name: cicd
description: >-
  CI/CD pipeline design with Harness Drone. Use when: creating .drone.yml pipelines,
  designing build/test/deploy stages, adding code coverage, publishing artifacts,
  configuring scheduled runs, applying CI/CD best practices, auditing existing pipelines
  for gaps, or improving pipeline efficiency and reliability. Covers linting gates,
  test parallelism, coverage thresholds, Docker image builds, secret management, and
  deployment strategies.
tags:
  - developer
  - operations
---

# CI/CD Standards (Harness Drone)

## When to Use

- Creating or modifying `.drone.yml` pipelines
- Designing build, test, and deploy stages
- Adding code coverage reporting
- Publishing Docker images or other artifacts
- Configuring secrets, triggers, or cron schedules
- Reviewing pipeline design for correctness and efficiency
- Auditing an existing pipeline for missing stages (lint, test, coverage, security scan)
- Improving pipeline speed (parallelism, caching, fail-fast ordering)

## Platform

- **Harness Drone** (OSS / Community Edition) running Docker pipelines.
- Pipeline config lives in `.drone.yml` at the repo root.
- Every pipeline is `kind: pipeline`, `type: docker`.

---

## Core Principles

1. **Pipeline as code** — `.drone.yml` is version-controlled, reviewed, and tested like application code.
2. **Fail fast** — cheapest checks (lint, format) run first. Expensive steps (integration tests, builds) run only if cheap checks pass.
3. **Hermetic builds** — every step starts from a known Docker image. No reliance on host state.
4. **Secrets never in YAML** — use Drone secrets (`from_secret`). No env var values in pipeline definitions.
5. **Idempotent deploys** — running the same pipeline twice produces the same result.

---

## Pipeline Architecture

Separate concerns into distinct pipelines (Drone `---` document separators):

| Pipeline | Trigger | Purpose |
|----------|---------|---------|
| **lint-and-test** | push, pull_request on main | Quality gate — must pass before merge |
| **build-and-publish** | push to main (post-merge) | Build artifacts, push images |
| **deploy** | push to main OR manual promote | Ship to staging/production |
| **scheduled** | cron | Recurring tasks (backups, scans, syncs) |

### Pipeline Ordering

```yaml
# Pipeline 1: Gate — blocks merge if it fails
kind: pipeline
name: lint-and-test
trigger:
  branch: [main]
  event: [push, pull_request]

# Pipeline 2: Build — only on merged code
kind: pipeline
name: build-and-publish
trigger:
  branch: [main]
  event: [push]
depends_on: [lint-and-test]

# Pipeline 3: Deploy — after build succeeds
kind: pipeline
name: deploy
trigger:
  branch: [main]
  event: [push]
depends_on: [build-and-publish]
```

---

## Stage 1: Lint and Format

Linting is the fastest, cheapest check. It runs first and blocks everything else.

```yaml
steps:
  - name: lint
    image: python:3.12-slim
    commands:
      - pip install ruff
      - ruff check .
      - ruff format --check .
```

**Rules**:
- Lint checks are **non-negotiable** — no `|| true`, no `allow_failure`.
- Use the exact same tool versions in CI and local dev.
- For multi-language repos, run language-specific linters in parallel steps.

| Language | Lint step |
|----------|-----------|
| Python | `ruff check . && ruff format --check .` |
| TypeScript | `npx prettier --check . && npx eslint .` |
| YAML / Markdown | `npx prettier --check "**/*.{yml,yaml,md}"` |
| Dockerfile | `hadolint docker/*.Dockerfile` |

---

## Stage 2: Test

Tests run after lint passes. Use parallel steps per test target to maximize throughput.

```yaml
  - name: test-shared
    image: python:3.12-slim
    commands:
      - pip install pytest pytest-cov python-dotenv
      - pytest shared/ -v --tb=short

  - name: test-my-script
    image: python:3.12-slim
    commands:
      - pip install pytest pytest-cov -r scripts/my_script/requirements.txt
      - >-
        pytest scripts/my_script/ -v --tb=short
        --cov=scripts/my_script
        --cov-report=term-missing
        --cov-fail-under=80
```

**Rules**:
- Every test step installs only the dependencies it needs — no global `requirements.txt`.
- Use `--tb=short` for concise failure output in CI.
- Use `--cov-fail-under=80` to enforce minimum coverage (see coverage section).
- If a test step needs system packages, use a custom Docker image (see build stage).
- Never skip tests in CI with `@pytest.mark.skip` unless there's a tracked issue.

### Test Parallelism

Drone runs steps within a pipeline in parallel by default. Steps that depend on each other need explicit `depends_on`:

```yaml
steps:
  - name: lint
    ...

  - name: test-unit
    depends_on: [lint]
    ...

  - name: test-integration
    depends_on: [lint]
    ...
```

Lint → then unit + integration in parallel → then build (depends on both).

---

## Stage 3: Code Coverage

### Configuration

```yaml
  - name: test-with-coverage
    image: python:3.12-slim
    commands:
      - pip install pytest pytest-cov
      - >-
        pytest
        --cov=src
        --cov-report=term-missing
        --cov-report=xml:coverage.xml
        --cov-fail-under=80
```

### Coverage Rules

| Rule | Value | Rationale |
|------|-------|-----------|
| **Minimum threshold** | 80% | Floor — not a target. New code should aim higher. |
| **Fail the build** | Yes | `--cov-fail-under=80` makes the step exit non-zero |
| **Report format** | `term-missing` + `xml` | Terminal for quick review, XML for tooling integration |
| **What counts** | Tested lines / total lines | Branch coverage is ideal but line coverage is the practical minimum |

### Coverage Anti-Patterns

- **Don't chase 100%** — diminishing returns above 90%. Focus coverage on business logic and error paths.
- **Don't exclude files just to raise the number** — if code exists, it should be tested.
- **Don't count generated code** — exclude auto-generated files in `.coveragerc` or `pyproject.toml`.

```toml
[tool.coverage.run]
source = ["src", "scripts"]
omit = ["*/test_*", "*/__pycache__/*"]

[tool.coverage.report]
fail_under = 80
show_missing = true
```

---

## Stage 4: Build Artifacts

### Docker Images

Use the Drone Docker plugin for building and pushing images:

```yaml
  - name: build-and-push
    image: plugins/docker
    settings:
      dockerfile: docker/my-service.Dockerfile
      repo: ${DRONE_REPO_OWNER}/my-service
      tags:
        - latest
        - ${DRONE_COMMIT_SHA:0:8}
      username:
        from_secret: docker_username
      password:
        from_secret: docker_password
    when:
      branch: [main]
      event: [push]
```

**Rules**:
- Always tag with both `latest` and a commit-specific tag (short SHA or semver).
- Build images only on main branch, never on PRs (PRs run lint + test only).
- Use multi-stage Dockerfiles to keep published images small (see docker skill).
- Store registry credentials as Drone secrets, never in pipeline YAML.
- For release builds, prefer semver tags over SHA — see the release pipeline section below.

### Python Packages

For internal Python packages:

```yaml
  - name: build-package
    image: python:3.12-slim
    commands:
      - pip install build
      - python -m build
    when:
      branch: [main]
      event: [push]
```

---

## Stage 5: Deploy

### Deployment Strategies

| Strategy | When to use | Risk |
|----------|-------------|------|
| **Direct deploy** | Homelab scripts, internal tools | Low — rollback is a git revert + re-deploy |
| **Blue-green** | Services with zero-downtime requirement | Medium — need two environments |
| **Canary** | User-facing APIs with high traffic | Low — gradual rollout, auto-rollback |
| **Manual promote** | Production deployments requiring approval | Lowest — human gate |

### Homelab Deploy Pattern

For homelab scripts running on a schedule, deployment means the cron pipeline pulls the latest image:

```yaml
---
kind: pipeline
name: deploy-to-homelab
trigger:
  branch: [main]
  event: [push]
depends_on: [build-and-publish]

steps:
  - name: deploy
    image: appleboy/drone-ssh
    settings:
      host:
        from_secret: deploy_host
      username:
        from_secret: deploy_user
      key:
        from_secret: deploy_ssh_key
      script:
        - cd /opt/homelab && docker compose pull && docker compose up -d
```

### Serverless Deploy Pattern

For AWS Lambda services:

```yaml
  - name: deploy-staging
    image: node:20-slim
    environment:
      AWS_ACCESS_KEY_ID:
        from_secret: aws_access_key_id
      AWS_SECRET_ACCESS_KEY:
        from_secret: aws_secret_access_key
    commands:
      - npm ci
      - npx serverless deploy --stage dev
    when:
      branch: [main]
      event: [push]

  - name: deploy-production
    image: node:20-slim
    environment:
      AWS_ACCESS_KEY_ID:
        from_secret: aws_access_key_id
      AWS_SECRET_ACCESS_KEY:
        from_secret: aws_secret_access_key
    commands:
      - npm ci
      - npx serverless deploy --stage prod
    when:
      event: [promote]
      target: [production]
```

Promote to production manually: `drone build promote <repo> <build> production`

---

## Secrets Management

- **All credentials go through Drone secrets** — `from_secret: key_name`.
- Store secrets via `drone secret add` CLI or Drone UI, never in `.drone.yml`.
- Secrets are not exposed in logs, pull request builds, or fork builds by default.
- For multi-repo secrets (Docker Hub creds), use organization-level secrets.

```yaml
environment:
  API_KEY:
    from_secret: api_key        # correct
  # API_KEY: "sk-1234..."       # NEVER — exposed in version control
```

---

## Cron / Scheduled Pipelines

For recurring tasks (backups, health checks, cleanup):

```yaml
---
kind: pipeline
name: backup-daily
trigger:
  event: [cron]
  cron: [backup-daily]

steps:
  - name: run-backup
    image: python:3.12-slim
    environment:
      BACKUP_SOURCE:
        from_secret: backup_source
    commands:
      - pip install -r scripts/backup/requirements.txt
      - python -m scripts.backup.backup
```

**Rules**:
- Cron pipeline names match the Drone cron job name for clarity.
- Cron pipelines should not share triggers with CI pipelines — use separate `---` documents.
- Always include error notifications (webhook, email) for scheduled jobs.

---

## Step Design Best Practices

### Base Images

| Language | Image | Notes |
|----------|-------|-------|
| Python | `python:3.12-slim` | Minimal, fast pull |
| Node.js | `node:20-slim` | For Serverless, frontend builds |
| Docker builds | `plugins/docker` | Drone's Docker build plugin |
| SSH deploy | `appleboy/drone-ssh` | Remote execution via SSH |
| General CLI | `alpine:3.19` | When you just need `curl`, `jq`, etc. |

### Caching

Drone Docker pipelines don't have built-in caching. Options:

- **Pre-built images** with dependencies baked in — fastest, most reliable.
- **Drone volume** (`/tmp/cache` mounted) — works for single-node setups.
- **Layer caching** for Docker builds — use `plugins/docker` with `cache_from`.

### Notifications

```yaml
  - name: notify-failure
    image: plugins/slack
    settings:
      webhook:
        from_secret: slack_webhook
      channel: ci-alerts
      template: >
        {{repo.name}} build {{build.number}} failed on {{build.branch}}.
        {{build.link}}
    when:
      status: [failure]
```

---

## Performance Pipeline

Integrate performance testing into CI with a tiered approach (see performance-testing and ui-performance skills for full details):

| Trigger | Test type | Purpose |
|---------|-----------|----------|
| Every PR | **Smoke** (k6, 1 min, 5 VUs) | Catch gross regressions cheaply |
| Release tag | **Load + Stress** (k6, 15 min) | Validate capacity before shipping |
| Release tag | **Lighthouse audit** | Enforce frontend performance score ≥ 80 |
| Weekly cron | **Soak** (k6, 1-4 hours) | Detect memory leaks and connection exhaustion |

```yaml
  - name: perf-smoke
    image: grafana/k6:latest
    commands:
      - k6 run --duration=1m --vus=5 tests/perf/smoke.js
    depends_on: [test]
```

---

## Release Pipeline

### Tag-Triggered Releases

Add a dedicated release pipeline that triggers only on semver tags. This separates release builds from regular CI (see release-management skill for full versioning conventions):

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
          echo "ERROR: Tag '$TAG' does not match semver format"
          exit 1
        fi
        echo "Valid semver tag: $TAG"

  - name: lint
    image: python:3.12-slim
    commands:
      - pip install ruff
      - ruff check .
      - ruff format --check .
    depends_on: [validate-tag]

  - name: test
    image: python:3.12-slim
    commands:
      - pip install pytest pytest-cov
      - pytest -v --tb=short --cov=src --cov-fail-under=80
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
    depends_on: [lint, test]
```

### Release Pipeline Rules

| Rule | Rationale |
|------|-----------|
| **Validate tag format** | Reject accidental or malformed tags before building |
| **Re-run lint + test** | Tag may point to a commit that wasn't the latest CI run |
| **Tag with semver** | `v1.5.0` and `1.5.0` alongside `latest` for flexibility |
| **Skip `latest` for pre-releases** | `v2.0.0-beta.1` should not be tagged `latest` |
| **Version-file validation** | Ensure `pyproject.toml` / `package.json` version matches the tag |

### Version-File Validation Step

If the project stores a version in a file, add a validation step to prevent drift:

```yaml
  - name: validate-version
    image: python:3.12-slim
    commands:
      - |
        FILE_VERSION=$(python -c "import tomllib; print(tomllib.load(open('pyproject.toml','rb'))['project']['version'])")
        TAG_VERSION="${DRONE_TAG#v}"
        if [ "$FILE_VERSION" != "$TAG_VERSION" ]; then
          echo "ERROR: pyproject.toml version ($FILE_VERSION) != tag ($TAG_VERSION)"
          exit 1
        fi
    depends_on: [validate-tag]
```

---

## Pipeline Checklist

Use this checklist when reviewing any `.drone.yml`:

| Check | What to verify |
|-------|---------------|
| **Fail fast** | Lint/format steps run before test steps |
| **No secrets in YAML** | All credentials use `from_secret` |
| **Pinned images** | Step images use specific tags, not `:latest` |
| **Test coverage** | `--cov-fail-under` is set and enforced |
| **PR safety** | Build/deploy steps have `when: event: [push]` (not on PRs) |
| **Cron isolation** | Scheduled pipelines use separate `---` documents |
| **Dependencies** | `depends_on` ensures correct ordering between pipelines |
| **Notifications** | Failure notifications are configured |
| **Minimal installs** | Each step installs only what it needs |
| **Short SHA tags** | Docker images tagged with commit SHA, not just `latest` |
| **Release pipeline** | Tag-triggered pipeline exists for semver releases |
| **Tag validation** | Release pipeline validates semver format before building |
