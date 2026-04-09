---
name: error-handling
description: 'Error handling and resilience patterns for robust services. Use when: implementing retry logic, adding circuit breakers, configuring timeouts, designing graceful degradation, handling partial failures, building dead letter queues, classifying errors for retry decisions, implementing abort/cancellation patterns, designing cleanup registries, implementing graceful shutdown, auditing an existing service for missing error handling, or improving fault tolerance in distributed systems. Covers retry with backoff, circuit breakers, timeout strategies, error classification, abort signals, cleanup lifecycle, graceful shutdown, fallback patterns, and structured error propagation.'
tags:
  - developer
  - operations
---

# Error Handling and Resilience

## When to Use

- Implementing retry logic for external API calls
- Adding circuit breakers to protect downstream services
- Configuring timeouts for network calls, database queries, or background tasks
- Designing graceful degradation when dependencies fail
- Handling partial failures in distributed systems
- Building dead letter queues for failed async processing
- Auditing an existing service for missing error handling or silent failures
- Improving fault tolerance for Lambda functions or microservices

---

## Core Principle: Fail Predictably

Every failure should be **expected, logged, and recoverable**. Silent failures and unhandled exceptions are bugs, not edge cases.

---

## Error Classification

Categorize errors before choosing a handling strategy:

| Category | Retryable | Example |
|----------|-----------|---------|
| **Transient** | Yes | Network timeout, 503, rate limit (429), connection reset |
| **Permanent** | No | 400 Bad Request, 404 Not Found, validation error, auth failure |
| **Degraded** | Partial | Dependency slow but responding, partial data available |
| **Fatal** | No | Out of memory, disk full, corrupted state |

**Rule**: Only retry transient errors. Retrying permanent errors wastes resources and delays failure reporting.

---

## Retry with Exponential Backoff

### Pattern

```python
import random
import time
from collections.abc import Callable
from typing import TypeVar

T = TypeVar("T")


def retry_with_backoff(
    fn: Callable[[], T],
    *,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 30.0,
    retryable_exceptions: tuple[type[Exception], ...] = (TimeoutError, ConnectionError),
) -> T:
    for attempt in range(max_retries + 1):
        try:
            return fn()
        except retryable_exceptions as exc:
            if attempt == max_retries:
                raise
            delay = min(base_delay * (2**attempt), max_delay)
            jitter = random.uniform(0, delay * 0.1)  # noqa: S311
            time.sleep(delay + jitter)
    raise RuntimeError("Unreachable")
```

### Rules

- **Always add jitter** to prevent thundering herd when multiple clients retry simultaneously.
- **Cap max delay** — unbounded exponential backoff can cause requests to hang for minutes.
- **Set max retries** — typically 3 for API calls, 5 for queue processing. Never unbounded.
- **Only retry on retryable errors** — define the allowlist explicitly.
- **Log every retry** with attempt number and delay, at WARNING level.

### Async Variant (Python)

```python
import asyncio
import random
from collections.abc import Awaitable, Callable
from typing import TypeVar

T = TypeVar("T")


async def retry_with_backoff_async(
    fn: Callable[[], Awaitable[T]],
    *,
    max_retries: int = 3,
    base_delay: float = 1.0,
    max_delay: float = 30.0,
    retryable_exceptions: tuple[type[Exception], ...] = (TimeoutError, ConnectionError),
) -> T:
    for attempt in range(max_retries + 1):
        try:
            return await fn()
        except retryable_exceptions as exc:
            if attempt == max_retries:
                raise
            delay = min(base_delay * (2**attempt), max_delay)
            jitter = random.uniform(0, delay * 0.1)  # noqa: S311
            await asyncio.sleep(delay + jitter)
    raise RuntimeError("Unreachable")
```

---

## Circuit Breaker

Prevent cascading failures by stopping calls to a failing dependency.

### States

```
CLOSED → (failures exceed threshold) → OPEN → (timeout expires) → HALF-OPEN → (probe succeeds) → CLOSED
                                                                                (probe fails)  → OPEN
```

### Implementation

```python
import time
from enum import StrEnum


class CircuitState(StrEnum):
    CLOSED = "closed"
    OPEN = "open"
    HALF_OPEN = "half_open"


class CircuitBreaker:
    def __init__(
        self,
        failure_threshold: int = 5,
        reset_timeout: float = 30.0,
        half_open_max_calls: int = 1,
    ):
        self.failure_threshold = failure_threshold
        self.reset_timeout = reset_timeout
        self.half_open_max_calls = half_open_max_calls
        self.state = CircuitState.CLOSED
        self.failure_count = 0
        self.last_failure_time = 0.0
        self.half_open_calls = 0

    def can_execute(self) -> bool:
        if self.state == CircuitState.CLOSED:
            return True
        if self.state == CircuitState.OPEN:
            if time.monotonic() - self.last_failure_time >= self.reset_timeout:
                self.state = CircuitState.HALF_OPEN
                self.half_open_calls = 0
                return True
            return False
        return self.half_open_calls < self.half_open_max_calls

    def record_success(self) -> None:
        if self.state == CircuitState.HALF_OPEN:
            self.state = CircuitState.CLOSED
        self.failure_count = 0

    def record_failure(self) -> None:
        self.failure_count += 1
        self.last_failure_time = time.monotonic()
        if self.state == CircuitState.HALF_OPEN:
            self.state = CircuitState.OPEN
        elif self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
```

### Rules

- **Fail fast when circuit is open** — return a cached/default value or raise immediately.
- **Log state transitions** at WARNING level: `Circuit opened for {service_name}`.
- **Monitor circuit state** — an open circuit is a symptom that needs investigation.
- **Scope per dependency** — one circuit breaker per external service, not one global.
- **Combine with retry** — retry inside the circuit breaker, not outside it.

---

## Timeout Strategy

### Layered Timeouts

```
Client request timeout (e.g. 30s)
  └─ Service-level timeout (e.g. 25s)
       └─ Dependency call timeout (e.g. 5s per call)
            └─ Connection timeout (e.g. 3s)
```

### Rules

- **Every network call must have a timeout.** No exceptions. The default of "wait forever" is never acceptable.
- Inner timeouts must be shorter than outer timeouts, with margin for processing.
- Set **connection timeout** (TCP handshake) separately from **read timeout** (response body).
- For Lambda: set dependency timeouts to `Lambda timeout - 2s` to allow for cleanup and logging.
- Log timeout events with the dependency name, configured timeout, and request context.

### Python Example

```python
import httpx

client = httpx.Client(
    timeout=httpx.Timeout(
        connect=3.0,
        read=10.0,
        write=5.0,
        pool=5.0,
    )
)
```

---

## Graceful Degradation

When a dependency fails, serve a reduced experience instead of a complete failure.

### Patterns

| Pattern | When to Use | Example |
|---------|-------------|---------|
| **Cached fallback** | Data can be stale temporarily | Serve last cached product catalog |
| **Default value** | Missing data has a safe default | Show 0 unread instead of error |
| **Feature toggle** | Non-critical feature failing | Disable recommendations, keep search |
| **Partial response** | Some data sources available | Return products without reviews |
| **Queue for later** | Write can be deferred | Queue email, confirm to user |

### Rules

- **Communicate degradation to the caller.** Return a header or field indicating data staleness or reduced functionality.
- **Log degraded responses** at WARNING level with the reason.
- **Set metrics** on degradation rate — rising degradation rate triggers alerts.
- **Never degrade silently** — the user or consuming service must know.

---

## Dead Letter Queues (DLQ)

For async processing failures:

### Rules

- Every SQS queue must have a dead letter queue configured.
- Set `maxReceiveCount` between 3–5 before DLQ routing.
- DLQ messages must include the original message, error reason, timestamp, and attempt count.
- Monitor DLQ depth — non-zero depth means something needs manual investigation.
- Build reprocessing capability: a script or Lambda that can replay DLQ messages.

---

## Structured Error Propagation

### API Error Shape

Return errors in a consistent, machine-readable format (see api-design skill):

```json
{
  "error": "validation_error",
  "message": "Email address is invalid",
  "status": 422,
  "details": [{"field": "email", "reason": "invalid_format"}],
  "request_id": "req_abc123"
}
```

### Internal Error Context

When propagating errors through service layers, preserve context:

```python
class ServiceError(Exception):
    def __init__(self, message: str, *, code: str, cause: Exception | None = None):
        super().__init__(message)
        self.code = code
        self.cause = cause
```

### Rules

- **Never swallow exceptions silently** — catch, log, and re-raise or return an error.
- **Wrap low-level exceptions** into domain exceptions at service boundaries.
- **Include request_id** in all error responses for traceability.
- **Never expose stack traces** in production API responses.
- **Log the full exception chain** (including `__cause__`) at the error boundary.

---

## Error Handling in Frontend

### React Error Boundaries

- Place error boundaries at route level and major section level.
- Show user-friendly fallback UI with a retry action.
- Log the error to your monitoring service from `componentDidCatch`.

### Network Error Handling

```typescript
async function fetchWithFallback<T>(
  url: string,
  fallback: T,
  options?: RequestInit,
): Promise<T> {
  try {
    const response = await fetch(url, options);
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    return (await response.json()) as T;
  } catch (error) {
    console.warn(`Fetch failed for ${url}, using fallback`, error);
    return fallback;
  }
}
```

### Rules

- Show loading → data | error states; never show stale UI without indication.
- Retry buttons for recoverable errors; clear error messages for permanent ones.
- Never show raw error messages or status codes to users.

---

## Error Classification Before Handling

Classify errors into a structured result before choosing a handling strategy. This pattern separates classification logic from retry/fallback logic:

### TypeScript Pattern

```typescript
interface ClassifiedError {
  kind: "auth" | "timeout" | "network" | "rate_limit" | "validation" | "server" | "unknown";
  status?: number;
  message: string;
  retryable: boolean;
}

function classifyError(error: unknown): ClassifiedError {
  if (error instanceof Response || (error instanceof Error && "status" in error)) {
    const status = (error as { status: number }).status;
    if (status === 401 || status === 403) return { kind: "auth", status, message: "Authentication failed", retryable: false };
    if (status === 429) return { kind: "rate_limit", status, message: "Rate limited", retryable: true };
    if (status === 400 || status === 422) return { kind: "validation", status, message: String(error), retryable: false };
    if (status >= 500) return { kind: "server", status, message: "Server error", retryable: true };
  }
  if (error instanceof TypeError && error.message.includes("fetch")) {
    return { kind: "network", message: "Network unavailable", retryable: true };
  }
  if (error instanceof DOMException && error.name === "TimeoutError") {
    return { kind: "timeout", message: "Request timed out", retryable: true };
  }
  return { kind: "unknown", message: String(error), retryable: false };
}

// Usage: classify first, then decide
const classified = classifyError(error);
if (classified.retryable) {
  await retryWithBackoff(() => fetchData(), { maxRetries: 3 });
} else {
  throw error;
}
```

### Python Pattern

```python
from dataclasses import dataclass


@dataclass
class ClassifiedError:
    kind: str  # "auth" | "timeout" | "network" | "rate_limit" | "validation" | "server"
    status: int | None
    message: str
    retryable: bool


def classify_error(exc: Exception) -> ClassifiedError:
    import httpx

    if isinstance(exc, httpx.HTTPStatusError):
        status = exc.response.status_code
        if status in (401, 403):
            return ClassifiedError("auth", status, "Authentication failed", retryable=False)
        if status == 429:
            return ClassifiedError("rate_limit", status, "Rate limited", retryable=True)
        if 400 <= status < 500:
            return ClassifiedError("validation", status, str(exc), retryable=False)
        if status >= 500:
            return ClassifiedError("server", status, "Server error", retryable=True)
    if isinstance(exc, (TimeoutError, httpx.TimeoutException)):
        return ClassifiedError("timeout", None, "Request timed out", retryable=True)
    if isinstance(exc, (ConnectionError, httpx.ConnectError)):
        return ClassifiedError("network", None, "Network unavailable", retryable=True)
    return ClassifiedError("unknown", None, str(exc), retryable=False)
```

### Rules

- **Classify before deciding** — don't scatter `if status == 429` checks throughout retry logic.
- **Return a typed result** — not a string or boolean. Include enough info for logging and metrics.
- **Centralize classification** — one function per external dependency or HTTP client.
- **Map classification to action** — retryable → retry with backoff; auth → re-authenticate; validation → fail immediately.

---

## Abort Signal / Cancellation

For long-running or user-cancelable operations, propagate `AbortSignal` to enable clean cancellation:

### TypeScript

```typescript
async function processItems(items: string[], signal: AbortSignal): Promise<void> {
  for (const item of items) {
    if (signal.aborted) return; // check before expensive work
    await processOne(item, signal);
  }
}

// Guard against abort errors in catch blocks
function isAbortError(error: unknown): boolean {
  return (
    (error instanceof DOMException && error.name === "AbortError") ||
    (error instanceof Error && error.name === "AbortError")
  );
}

// Usage
const controller = new AbortController();
try {
  await processItems(items, controller.signal);
} catch (error) {
  if (isAbortError(error)) return; // not a real error — user cancelled
  throw error;
}
```

### Rules

- Pass `AbortSignal` through all async function chains that support cancellation.
- Check `signal.aborted` before expensive operations inside loops.
- Never log abort errors at ERROR level — they are expected cancellations.
- Combine abort signals when child operations have different cancellation scopes.

---

## Cleanup Registry

Register cleanup callbacks centrally so resources are released during shutdown regardless of which code path triggered it:

### TypeScript

```typescript
type CleanupFn = () => void | Promise<void>;

const cleanupRegistry: CleanupFn[] = [];

function registerCleanup(fn: CleanupFn): () => void {
  cleanupRegistry.push(fn);
  return () => {
    const index = cleanupRegistry.indexOf(fn);
    if (index >= 0) cleanupRegistry.splice(index, 1);
  };
}

async function runCleanup(): Promise<void> {
  for (const fn of cleanupRegistry.reverse()) {
    try {
      await fn();
    } catch {
      // log but don't throw — other cleanup must still run
    }
  }
}
```

### Python

```python
import atexit
from collections.abc import Callable

_cleanup_fns: list[Callable[[], None]] = []


def register_cleanup(fn: Callable[[], None]) -> Callable[[], None]:
    _cleanup_fns.append(fn)
    return lambda: _cleanup_fns.remove(fn) if fn in _cleanup_fns else None


def run_cleanup() -> None:
    for fn in reversed(_cleanup_fns):
        try:
            fn()
        except Exception:
            pass  # log but continue — other cleanup must still run


atexit.register(run_cleanup)
```

### Rules

- Cleanup runs in **reverse registration order** (LIFO) — most recently acquired resources release first.
- Individual cleanup failures must not prevent other cleanup from running.
- Set a **failsafe timeout** on the cleanup process to prevent hangs (e.g., 5 seconds).
- Register cleanup at acquisition time, unregister when the resource is explicitly released.

---

## Graceful Shutdown

Handle process signals to clean up before exit:

```typescript
function gracefulShutdown(code: number, reason?: string): void {
  if (reason) console.warn(`Shutting down: ${reason}`);
  const timeout = setTimeout(() => process.exit(1), 5000); // failsafe
  runCleanup().finally(() => {
    clearTimeout(timeout);
    process.exit(code);
  });
}

process.on("SIGINT", () => gracefulShutdown(0, "SIGINT"));
process.on("SIGTERM", () => gracefulShutdown(0, "SIGTERM"));
```

### Shutdown Order

1. Stop accepting new work (close servers, drain queues)
2. Wait for in-flight operations to complete (with timeout)
3. Flush buffers (analytics, logs, caches)
4. Close connections (database, external services)
5. Exit

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Catch-all `except Exception` | Masks bugs, swallows keyboard interrupts | Catch specific exceptions |
| Retry permanent errors | Wastes resources, delays failure | Classify before retrying |
| No timeout on HTTP calls | Threads/connections leak | Always set explicit timeout |
| Silent failure in background jobs | Data loss, missing processing | Log and DLQ failed items |
| Infinite retry loops | Service never recovers | Set max retries and circuit break |
| Retrying without backoff | Thundering herd on recovery | Exponential backoff + jitter |
| Catching and logging only | Error propagation lost | Catch, log, and re-raise |

---

## Audit Checklist

When auditing an existing service for error handling:

- [ ] Every external HTTP call has an explicit timeout configured
- [ ] Retry logic uses exponential backoff with jitter
- [ ] Only transient errors are retried; permanent errors fail immediately
- [ ] Errors are classified into a typed result before retry/fallback decisions
- [ ] Circuit breakers protect calls to external dependencies
- [ ] SQS queues have dead letter queues with `maxReceiveCount` configured
- [ ] API errors return consistent, structured format with `request_id`
- [ ] Exceptions are logged with full context (not swallowed silently)
- [ ] Frontend has error boundaries at route and section levels
- [ ] Graceful degradation is in place for non-critical dependencies
- [ ] Timeout values are layered (inner < outer) and documented
- [ ] Abort signals propagated for cancelable async operations
- [ ] Cleanup callbacks registered for acquired resources
- [ ] Graceful shutdown handles SIGINT/SIGTERM with failsafe timeout
- [ ] Abort errors distinguished from real failures in catch blocks
