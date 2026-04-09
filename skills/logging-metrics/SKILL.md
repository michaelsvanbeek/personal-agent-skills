---
name: logging-metrics
description: 'Structured logging and metrics design for analytics pipelines and dashboards. Use when: implementing application logging, designing log schemas, adding metrics collection, handling exception logging, building observability into services, integrating with log aggregation and dashboarding tools, auditing existing logging for structure and completeness, or improving observability in an existing service.'
tags:
  - developer
  - operations
  - analyst
---

# Logging and Metrics Standards

## When to Use

- Adding logging to any application or service
- Designing log schemas for analytics pipelines
- Implementing exception logging with safe context
- Adding metrics or tracing to services
- Integrating with dashboards (Grafana, CloudWatch, Datadog)
- Auditing existing logging for missing structure, inconsistent fields, or sensitive data leakage
- Upgrading unstructured print/log statements to structured JSON logging
- Reviewing observability coverage (are errors, latency, and key operations instrumented?)

## Core Principle: Logs are Data

Logs are not debug printf statements. They are **structured data events** that feed analytics pipelines, dashboards, alerts, and incident investigation. Design them like you would design a database schema.

---

## Structured Logging

All logs must be **JSON-formatted**, one JSON object per line. Never use unstructured text, multi-line log messages, or custom delimiters.

### Standard Fields

Every log entry must include these base fields:

| Field | Type | Description |
|-------|------|-------------|
| `timestamp` | string | ISO 8601 with timezone: `2026-03-19T23:45:00.123Z` |
| `level` | string | `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |
| `service` | string | Service/application name |
| `environment` | string | `dev`, `staging`, `prod` |
| `message` | string | Human-readable description of the event |

### Contextual Fields (add when applicable)

| Field | Type | When to include |
|-------|------|-----------------|
| `request_id` | string (UUID) | Any event within an HTTP request lifecycle |
| `user_id` | string | Any event tied to a user action (never log PII beyond ID) |
| `trace_id` | string | Distributed tracing across services |
| `span_id` | string | Sub-operation within a trace |
| `route` | string | HTTP endpoint path |
| `method` | string | HTTP method |
| `status` | integer | HTTP response status code |
| `duration_ms` | number | Operation timing |
| `component` | string | Module, class, or function name |
| `action` | string | What the code is doing: `fetch_users`, `send_email`, `sync_cache` |

### Event-Specific Fields

Add fields relevant to the specific event. Use consistent names across the codebase:

```json
{
  "timestamp": "2026-03-19T23:45:00.123Z",
  "level": "INFO",
  "service": "project-api",
  "environment": "prod",
  "request_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "action": "create_project",
  "user_id": "usr_42",
  "project_id": "prj_789",
  "duration_ms": 142.5,
  "message": "Project created successfully"
}
```

### Field Naming Conventions

- Use **snake_case** for all field names.
- Use consistent suffixes: `_id` for identifiers, `_ms` for milliseconds, `_at` for timestamps, `_count` for counts, `_bytes` for sizes.
- Never change the type of a field. If `duration_ms` is a number in one log, it must always be a number.
- Avoid nested objects in log fields. Flatten for easier querying: `user_id` not `user.id`.

---

## Log Levels

Use levels consistently across all services:

| Level | When to use | Example |
|-------|-------------|---------|
| `DEBUG` | Detailed diagnostic info for development. Disabled in production. | `"Resolved user cache key: usr_42"` |
| `INFO` | Normal operations, milestones, state transitions. | `"Project created"`, `"Sync completed: 42 items"` |
| `WARNING` | Recoverable issue that may need attention. | `"Rate limit approaching: 85% of quota"` |
| `ERROR` | Operation failed but service continues. | `"Failed to send notification email"` |
| `CRITICAL` | Service-level failure, requires immediate action. | `"Database connection pool exhausted"` |

Rules:
- **INFO** is the default production level. Every significant operation should produce at least one INFO log.
- A request should produce at most **1 INFO log** for the happy path (the request summary). Use DEBUG for steps within the request.
- **ERROR** means something broke. If it's expected behavior (e.g., user not found), use INFO or WARNING.
- Never log at ERROR for client errors (4xx). Log at WARNING for unexpected 4xx, INFO for expected ones.

---

## Exception Logging

### Include Context

When logging exceptions, include enough context to reproduce and diagnose the issue without accessing the running system:

```python
import logging

logger = logging.getLogger(__name__)

try:
    result = process_order(order_id=order_id, user_id=user_id)
except ExternalServiceError as exc:
    logger.error(
        "Order processing failed",
        extra={
            "action": "process_order",
            "order_id": order_id,
            "user_id": user_id,
            "error_type": type(exc).__name__,
            "error_message": str(exc),
            "retry_count": attempt,
        },
        exc_info=True,
    )
    raise
```

### What to Include in Exception Logs

- **action**: What operation was being attempted
- **error_type**: Exception class name
- **error_message**: Exception message text
- **identifiers**: IDs of the entities involved (order_id, user_id, etc.)
- **state**: Relevant state at time of failure (retry_count, batch_index, etc.)
- **stack trace**: Via `exc_info=True` (Python) or `error.stack` (JS). Include in dev/staging; optionally truncate in production.

### What to NEVER Include

- **Passwords, tokens, API keys, secrets** in any form
- **Credit card numbers, SSNs**, or other PII
- **Full request/response bodies** that may contain user data
- **Database connection strings** with credentials
- **Environment variables** that contain secrets
- **Email addresses, phone numbers** unless essential and privacy-compliant

### Sanitization

Build a sanitization layer that runs before logging:

```python
SENSITIVE_KEYS = {"password", "token", "secret", "api_key", "authorization", "credit_card", "ssn"}

def sanitize_context(context: dict) -> dict:
    """Remove or mask sensitive fields before logging."""
    return {
        k: "***REDACTED***" if any(s in k.lower() for s in SENSITIVE_KEYS) else v
        for k, v in context.items()
    }
```

For JavaScript/TypeScript:

```typescript
const SENSITIVE_PATTERNS = /password|token|secret|api_key|authorization|credit_card|ssn/i;

function sanitize(context: Record<string, unknown>): Record<string, unknown> {
  return Object.fromEntries(
    Object.entries(context).map(([k, v]) =>
      SENSITIVE_PATTERNS.test(k) ? [k, "***REDACTED***"] : [k, v],
    ),
  );
}
```

---

## Python Logging Setup

Use the stdlib `logging` module with a JSON formatter. Never use `print()`.

```python
import json
import logging
import os
import sys
from datetime import datetime, timezone


class JSONFormatter(logging.Formatter):
    def format(self, record: logging.LogRecord) -> str:
        entry = {
            "timestamp": datetime.now(timezone.utc).isoformat(),
            "level": record.levelname,
            "service": os.environ.get("SERVICE_NAME", "unknown"),
            "environment": os.environ.get("ENVIRONMENT", "dev"),
            "component": record.name,
            "message": record.getMessage(),
        }
        # Merge extra fields
        if hasattr(record, "__dict__"):
            for key, value in record.__dict__.items():
                if key not in logging.LogRecord.__dict__ and key not in entry:
                    entry[key] = value
        # Include exception info
        if record.exc_info and record.exc_info[1]:
            entry["error_type"] = type(record.exc_info[1]).__name__
            entry["error_message"] = str(record.exc_info[1])
            entry["traceback"] = self.formatException(record.exc_info)
        return json.dumps(entry, default=str)


def setup_logging(level: str = "INFO") -> None:
    handler = logging.StreamHandler(sys.stdout)
    handler.setFormatter(JSONFormatter())
    logging.root.handlers = [handler]
    logging.root.setLevel(getattr(logging, level.upper()))
```

---

## JavaScript/TypeScript Logging

Use a structured logger like **pino** for Node.js services:

```typescript
import pino from "pino";

const logger = pino({
  level: process.env.LOG_LEVEL ?? "info",
  formatters: {
    level: (label) => ({ level: label.toUpperCase() }),
  },
  base: {
    service: process.env.SERVICE_NAME ?? "unknown",
    environment: process.env.ENVIRONMENT ?? "dev",
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});

// Usage
logger.info({ action: "create_project", project_id: "prj_789", duration_ms: 142 }, "Project created");
```

---

## Metrics

### What to Measure

Instrument the **Four Golden Signals** for every service:

| Signal | Metric | Example |
|--------|--------|---------|
| **Latency** | Request duration | p50, p95, p99 of response time in ms |
| **Traffic** | Request throughput | Requests per second by endpoint |
| **Errors** | Failure rate | Errors per second, error rate percentage |
| **Saturation** | Resource utilization | CPU, memory, connection pool usage, queue depth |

### Metric Naming

Use a consistent naming scheme:

```
{service}_{component}_{metric}_{unit}
```

Examples:
- `api_http_request_duration_ms`
- `api_http_request_total`
- `api_http_error_total`
- `worker_queue_depth_count`
- `cache_hit_total`, `cache_miss_total`

### Metric Types

| Type | Use when | Example |
|------|----------|---------|
| **Counter** | Value only increases | Total requests, total errors |
| **Gauge** | Value goes up and down | Queue depth, active connections, memory usage |
| **Histogram** | Distribution of values | Request latency, response size |

### Dashboard Design

- Every service needs a dashboard with at minimum: request rate, error rate, latency (p50/p95/p99), and saturation metrics.
- Align log fields with dashboard dimensions so you can drill from dashboard → filtered log view.
- Use the same `service`, `environment`, `route`, and `status` field names in both logs and metrics for join-ability.

---

## Log Pipeline Integration

Design logs to flow through analytics pipelines:

```
Application → stdout (JSON) → Log collector → Storage → Query/Dashboard
```

- Write logs to **stdout**, never to files. Let the runtime (Docker, Lambda, systemd) handle routing.
- One JSON object per line. No multi-line log entries.
- Keep log entries under **4KB** for compatibility with most log aggregation services.
- Include all dimensions needed for filtering in the log entry itself. Don't rely on external metadata enrichment for core fields.
- Use consistent field names across all services so dashboards and alerts work across the fleet.

---

## Correlation

- Generate a `request_id` (UUID) at the entry point of every request. Pass it through all downstream calls and include it in every log entry for that request.
- For distributed systems, propagate `trace_id` and `span_id` in headers (W3C Trace Context or similar).
- Log the `request_id` in API error responses so users can report it for debugging: `{"error": "internal_error", "request_id": "abc-123"}`.

---

## Alerting

### What to Alert On

Alert on **symptoms**, not causes. Page when user-facing behavior is degraded, not on every noisy internal signal.

| Signal | Condition | Severity |
|--------|-----------|----------|
| **Error rate** | >1% of requests return 5xx over a 5-minute window | PAGE |
| **Error rate** | >0.1% sustained over 30 minutes | WARN |
| **p95 latency** | >2× baseline over a 10-minute window | PAGE |
| **p95 latency** | >1.5× baseline over 30 minutes | WARN |
| **Queue depth** | >80% of max capacity | WARN |
| **Queue depth** | >95% of max capacity (unbounded growth) | PAGE |
| **Availability** | Health check fails for 2 consecutive minutes | PAGE |
| **Error budget** | SLO error budget <20% remaining in billing period | WARN |

### Alert Fatigue Prevention

An alert that fires constantly becomes noise — it trains engineers to ignore it.

- **PAGE only on actionable, user-impacting conditions.** If the on-call engineer can't do anything about it at 2am, it must not be a page.
- **Every alert must link to a runbook** in the alert description. No runbook = no alert.
- **Use warning notifications** (Slack, email) for elevated-but-not-critical conditions. Reserve PagerDuty/phone for true incidents.
- **Set thresholds above the noise floor** — measure 2 weeks of baseline before configuring thresholds. An alert firing 10 times a day from normal traffic is useless.
- **Use multi-condition alerts** — require traffic above a minimum AND elevated error rate. One error in 1 request/hour is not an incident.
- **Use time windows** — 5-minute windows for page-severity, 30-minute windows for warn-severity. Short windows are noisy.
- **Review and prune quarterly** — remove any alert that fired without resulting in a real incident.

### AWS CloudWatch Alarm Example

```yaml
resources:
  Resources:
    LambdaErrorRateAlarm:
      Type: AWS::CloudWatch::Alarm
      Properties:
        AlarmName: ${self:service}-${self:custom.stage}-error-rate
        AlarmDescription: "Lambda errors >5 in 5 min. Runbook: docs/runbooks/lambda-errors.md"
        MetricName: Errors
        Namespace: AWS/Lambda
        Dimensions:
          - Name: FunctionName
            Value: !Ref ApiLambdaFunction
        Statistic: Sum
        Period: 300
        EvaluationPeriods: 1
        Threshold: 5
        ComparisonOperator: GreaterThanOrEqualToThreshold
        TreatMissingData: notBreaching
        AlarmActions: [!Ref OpsAlertTopic]
        OKActions: [!Ref OpsAlertTopic]

    LambdaLatencyAlarm:
      Type: AWS::CloudWatch::Alarm
      Properties:
        AlarmName: ${self:service}-${self:custom.stage}-latency-p95
        AlarmDescription: "Lambda p95 latency >2000ms. Runbook: docs/runbooks/latency.md"
        MetricName: Duration
        Namespace: AWS/Lambda
        Dimensions:
          - Name: FunctionName
            Value: !Ref ApiLambdaFunction
        ExtendedStatistic: p95
        Period: 600
        EvaluationPeriods: 1
        Threshold: 2000
        ComparisonOperator: GreaterThanOrEqualToThreshold
        TreatMissingData: notBreaching
        AlarmActions: [!Ref OpsAlertTopic]

    OpsAlertTopic:
      Type: AWS::SNS::Topic
      Properties:
        TopicName: ${self:service}-${self:custom.stage}-ops-alerts
        # Subscribe Slack webhook via Lambda or PagerDuty endpoint via HTTPS subscription
```

### Grafana Alerting

For services queried via Grafana (Athena, InfluxDB, Prometheus):

- Create alert rules directly on dashboard panels — keep alerts and visualization co-located.
- Use **contact points** to route by severity: `severity=page` → PagerDuty, `severity=warn` → Slack.
- Use **notification policies** with label matchers to enforce routing without duplicating contact config.
- Set a **pending duration** (5 minutes minimum) before firing — eliminates single-spike false positives.
- Use **silence rules** during planned maintenance windows rather than disabling alert rules.

---

## Further Reading

- Compare observability stacks using explicit criteria: ingestion cost, query latency,
  retention, alerting quality, and operational overhead.
- Record tool choices in ADRs so future maintainers understand trade-offs and revisit
  assumptions when scale or budget changes.
