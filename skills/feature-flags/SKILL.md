---
name: feature-flags
description: >-
  Feature flag design, lifecycle management, and progressive rollout strategies.
  Use when: adding feature toggles to gate new functionality, implementing
  gradual rollouts, designing A/B experiments, managing flag lifecycle and
  cleanup, auditing an existing codebase for stale flags, or choosing between
  flag implementations. Covers flag types, evaluation strategies, rollout
  patterns, flag hygiene, and testing with flags.
tags:
  - developer
  - manager
  - operations
---

# Feature Flags

## When to Use

- Gating new functionality behind a flag for gradual rollout
- Implementing A/B experiments or percentage-based rollouts
- Decoupling deployment from release (deploy dark, enable later)
- Building kill switches for risky features
- Managing long-running feature branches without merge conflicts
- Designing flag evaluation for server-side and client-side use
- Auditing an existing codebase for stale or orphaned flags
- Cleaning up flags after full rollout

---

## Core Principle: Flags Are Temporary

**Every flag has a planned removal date.** Flags that live forever become tech debt — invisible branching logic that no one understands. Ship the feature, verify it works, remove the flag.

---

## Flag Types

| Type | Lifespan | Example | Owner |
|------|----------|---------|-------|
| **Release** | Days to weeks | Gate new checkout flow for gradual rollout | Engineering |
| **Experiment** | Weeks to months | A/B test pricing page layout | Product |
| **Ops** | Permanent (rare) | Kill switch for expensive feature under load | Engineering |
| **Permission** | Permanent | Feature only for enterprise tier customers | Product |

### Rules

- **Release flags**: Remove within 2 weeks of full rollout.
- **Experiment flags**: Remove when experiment concludes and winner is implemented.
- **Ops flags**: Review quarterly — convert to config if permanent.
- **Permission flags**: These are entitlements, not feature flags. Model in authorization system if they grow complex.

---

## Flag Schema

### Flag Definition

```typescript
interface FeatureFlag {
  key: string;              // Unique identifier: "checkout-v2"
  description: string;      // What this flag controls
  type: "release" | "experiment" | "ops" | "permission";
  owner: string;            // Team or person responsible
  createdAt: string;        // ISO date
  plannedRemovalDate: string; // When to clean up (required for release/experiment)
  defaultValue: boolean;    // Value when flag system is unavailable
  rules: FlagRule[];        // Evaluation rules (ordered, first match wins)
}

interface FlagRule {
  condition: FlagCondition;
  value: boolean | string;  // Boolean for toggles, string for variants
}

type FlagCondition =
  | { type: "percentage"; value: number }          // 0-100 rollout percentage
  | { type: "user_ids"; value: string[] }          // Specific users
  | { type: "attribute"; key: string; value: string } // User attribute match
  | { type: "environment"; value: string }          // "production", "staging"
  | { type: "default" };
```

### Flag Configuration (JSON)

```json
{
  "checkout-v2": {
    "description": "New checkout flow with saved payment methods",
    "type": "release",
    "owner": "payments-team",
    "createdAt": "2025-01-15",
    "plannedRemovalDate": "2025-02-15",
    "defaultValue": false,
    "rules": [
      { "condition": { "type": "environment", "value": "staging" }, "value": true },
      { "condition": { "type": "user_ids", "value": ["internal-tester-1", "internal-tester-2"] }, "value": true },
      { "condition": { "type": "percentage", "value": 10 }, "value": true },
      { "condition": { "type": "default" }, "value": false }
    ]
  }
}
```

---

## Evaluation

### Server-Side (Python)

```python
import hashlib
from typing import Any


def evaluate_flag(flag_key: str, user_id: str, attributes: dict[str, Any]) -> bool:
    """Evaluate a feature flag for a specific user."""
    flag = get_flag_config(flag_key)
    if flag is None:
        return False  # Flag not found — default off

    for rule in flag["rules"]:
        condition = rule["condition"]

        if condition["type"] == "environment":
            if get_environment() == condition["value"]:
                return rule["value"]

        elif condition["type"] == "user_ids":
            if user_id in condition["value"]:
                return rule["value"]

        elif condition["type"] == "attribute":
            if attributes.get(condition["key"]) == condition["value"]:
                return rule["value"]

        elif condition["type"] == "percentage":
            # Deterministic: same user always gets same result for same flag
            hash_input = f"{flag_key}:{user_id}"
            hash_value = int(hashlib.sha256(hash_input.encode()).hexdigest()[:8], 16)
            bucket = hash_value % 100
            if bucket < condition["value"]:
                return rule["value"]

        elif condition["type"] == "default":
            return rule["value"]

    return flag["defaultValue"]
```

### Client-Side (React)

```typescript
import { createContext, useContext } from "react";
import type { ReactNode } from "react";

interface FlagContextValue {
  flags: Record<string, boolean>;
}

const FlagContext = createContext<FlagContextValue>({ flags: {} });

function FlagProvider({ children, flags }: { children: ReactNode; flags: Record<string, boolean> }) {
  return <FlagContext.Provider value={{ flags }}>{children}</FlagContext.Provider>;
}

function useFlag(key: string): boolean {
  const { flags } = useContext(FlagContext);
  return flags[key] ?? false;
}

// Usage in component
function CheckoutPage() {
  const useNewCheckout = useFlag("checkout-v2");

  if (useNewCheckout) {
    return <NewCheckoutFlow />;
  }
  return <LegacyCheckoutFlow />;
}
```

### Evaluation Rules

- **Deterministic**: Same user + same flag = same result. Use hash-based bucketing, not random.
- **First match wins**: Rules are evaluated in order. Put specific rules (user IDs, environments) before percentage rules.
- **Default off**: If the flag system is unavailable, features default to off (safe fallback).
- **Server-authoritative**: Client receives pre-evaluated flags from the server. Never send flag rules to the client.

---

## Rollout Strategies

### Gradual Percentage Rollout

```
Day 1:  0% → 5%    (internal team + canary users)
Day 2:  5% → 10%   (monitor error rates, latency)
Day 3:  10% → 25%  (monitor business metrics)
Day 5:  25% → 50%  (compare A/B if experiment)
Day 7:  50% → 100% (full rollout)
Day 14: Remove flag (cleanup)
```

### Monitoring During Rollout

At each stage, verify:

- **Error rate**: No increase for flagged users vs control.
- **Latency**: P50/P95/P99 within acceptable bounds.
- **Business metrics**: Conversion rate, revenue per user, engagement.
- **User feedback**: Support tickets, rage clicks, session recordings.

### Rollback

If metrics degrade:

```python
# Instant rollback: set percentage to 0
update_flag("checkout-v2", rules=[
    {"condition": {"type": "default"}, "value": False},
])
```

**Rollback should take seconds, not minutes.** This is the primary advantage of feature flags over code rollback.

---

## Flag Storage

### Simple: Config File (Small Projects)

```json
// flags.json — committed, deployed with application
{
  "checkout-v2": true,
  "new-search": false
}
```

**Pros**: Simple, version-controlled, no external dependency.
**Cons**: Requires deploy to change. No gradual rollout.

### Medium: SSM Parameter Store / DynamoDB

```python
import boto3
import json

ssm = boto3.client("ssm")


def get_flags() -> dict[str, Any]:
    """Load flags from SSM Parameter Store."""
    response = ssm.get_parameter(
        Name="/myapp/prod/feature-flags",
        WithDecryption=False,
    )
    return json.loads(response["Parameter"]["Value"])
```

**Pros**: Change flags without deploying. Cheap. Simple.
**Cons**: No built-in UI, percentage rollout requires custom logic.

### Advanced: Dedicated Flag Service (LaunchDarkly, Unleash, Flagsmith)

Use when:
- Multiple teams manage their own flags.
- You need a UI for non-engineers to toggle flags.
- Sophisticated targeting (geo, device, plan tier) is required.
- Audit log of flag changes is mandatory.

---

## Testing with Feature Flags

### Unit Tests: Test Both Paths

```python
import pytest


@pytest.mark.parametrize("flag_value", [True, False])
def test_checkout_flow(flag_value: bool, monkeypatch: pytest.MonkeyPatch) -> None:
    """Test both checkout flows regardless of current flag state."""
    monkeypatch.setattr("features.evaluate_flag", lambda key, **kw: flag_value)

    result = process_checkout(user_id="test-user", cart=sample_cart)

    if flag_value:
        assert result.flow == "v2"
    else:
        assert result.flow == "legacy"
```

### Integration Tests: Pin Flag Values

```python
@pytest.fixture
def flags_all_on() -> dict[str, bool]:
    """Enable all feature flags for integration testing."""
    return {"checkout-v2": True, "new-search": True}


@pytest.fixture
def flags_all_off() -> dict[str, bool]:
    """Disable all feature flags for integration testing."""
    return {"checkout-v2": False, "new-search": False}
```

### E2E Tests: Test Current State

E2E tests should test the **current production state** of flags. Don't override flags in E2E — if a flag is on in production, test with it on.

---

## Flag Cleanup

### When to Remove

Remove a flag when:
- The feature is fully rolled out to 100% for at least 1 week.
- The experiment has concluded and the winning variant is hardcoded.
- The ops flag has been on for 6+ months and no one has toggled it.

### Cleanup Process

1. **Verify**: Confirm flag is at 100% (or permanently on) with no recent toggles.
2. **Remove evaluation calls**: Replace `if flag("checkout-v2")` with the winning branch.
3. **Remove losing code path**: Delete the old implementation.
4. **Remove flag definition**: Delete from config/flag service.
5. **Commit**: One commit per flag cleanup.

### Detecting Stale Flags

```bash
# Find flag references in code
grep -r "checkout-v2" --include="*.py" --include="*.ts" --include="*.tsx" src/

# List flags with past planned removal dates
jq 'to_entries[] | select(.value.plannedRemovalDate < "2025-01-15") | .key' flags.json
```

### Cleanup Cadence

- **Weekly**: Check for flags past their planned removal date.
- **Monthly**: Review all active flags — any that can be removed?
- **Quarterly**: Audit permanent ops/permission flags — still needed?

---

## Flag Naming Convention

```
<domain>-<feature>[-<variant>]
```

| Example | Type | Description |
|---------|------|-------------|
| `checkout-v2` | Release | New checkout flow |
| `search-fuzzy-matching` | Release | Enable fuzzy search |
| `pricing-annual-discount` | Experiment | Test annual plan discount |
| `api-rate-limit-strict` | Ops | Tighter rate limits under load |
| `plan-enterprise-analytics` | Permission | Analytics for enterprise tier |

**Rules**:
- Lowercase, kebab-case.
- Domain prefix for grouping.
- Descriptive — someone unfamiliar should understand the purpose.
- No generic names (`flag-1`, `test`, `new-feature`).

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Flag with no planned removal date | Lives forever, becomes invisible tech debt | Require removal date on creation for release/experiment flags |
| Nested flags (`if flag_a and flag_b`) | Combinatorial explosion of states to test | Keep flags independent; one flag per feature |
| Flag evaluated differently in two places | Inconsistent behavior for the same user | Centralize evaluation; pass result as parameter |
| Client-side flag evaluation with rules | Exposes targeting logic and user segments | Server evaluates; client receives boolean result |
| Using flags instead of authorization | Flag system becomes a permissions system | Use proper RBAC/ABAC for entitlements |
| No monitoring during rollout | Degradation goes unnoticed until 100% | Define monitoring criteria before enabling |
| Random-based percentage rollout | Same user gets different experience on each request | Use hash-based deterministic bucketing |
| Flag config not version-controlled | No history of who changed what, when | Store in git or use a flag service with audit log |
| Monster flag guarding 500 lines | Too much code behind one toggle | Break into smaller flags or defer merge until feature is complete |

---

## Audit Checklist

When auditing an existing codebase for feature flag practices:

- [ ] Every flag has a documented owner and description
- [ ] Release and experiment flags have a planned removal date
- [ ] No flags past their planned removal date exist in the codebase
- [ ] Flag evaluation is centralized (single function/hook, not ad-hoc checks)
- [ ] Percentage-based rollout uses deterministic hashing, not random
- [ ] Both code paths (flag on and off) are tested
- [ ] Client-side flags are pre-evaluated by the server
- [ ] Monitoring criteria defined before enabling a flag in production
- [ ] Gradual rollout follows staged percentage increases
- [ ] Flag changes are logged with who, when, and why
- [ ] No nested or dependent flag conditions
- [ ] Cleanup process exists and runs at least monthly
- [ ] Flag naming follows a consistent convention
