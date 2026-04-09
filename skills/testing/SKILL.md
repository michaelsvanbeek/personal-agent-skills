---
name: testing
description: >-
  Test strategy, patterns, coverage, and evaluation for software projects. Use
  when: writing tests, choosing test boundaries, structuring a test suite,
  setting coverage thresholds, designing test data, auditing test coverage gaps,
  or improving test quality and structure.
tags:
  - developer
---

# Testing Strategy

## When to Use

- Writing unit, integration, or end-to-end tests
- Choosing what to test and at what boundary
- Structuring a test suite for a new or existing project
- Designing test data and fixtures
- Setting coverage thresholds
- Auditing an existing project for test gaps

---

## Test Boundaries

| Level | What it tests | Speed | Confidence |
|-------|--------------|-------|-----------|
| **Unit** | One function or method in isolation | Fast | Narrow |
| **Integration** | Multiple components working together | Medium | Moderate |
| **End-to-end** | Full user workflow | Slow | Broad |

### Guidance

- Write **more unit tests than integration tests, and more integration tests
  than E2E tests** (testing pyramid)
- Test behavior, not implementation — assert on outputs, not internal state
- Every bug fix should come with a regression test

---

## Test Structure

Follow the **Arrange-Act-Assert** pattern:

```python
def test_expired_token_returns_401():
    # Arrange
    token = create_expired_token()
    client = create_test_client()

    # Act
    response = client.get("/api/users", headers={"Authorization": f"Bearer {token}"})

    # Assert
    assert response.status_code == 401
    assert response.json()["error"] == "token_expired"
```

### Naming

Test names should describe the scenario and expected outcome:

```
test_<scenario>_<expected_result>
```

```python
# Good
def test_empty_cart_returns_zero_total(): ...
def test_discount_code_reduces_price_by_percentage(): ...

# Bad
def test_cart(): ...
def test_discount(): ...
```

---

## Test Data

- **Factories** for complex objects: define defaults, override what matters
- **Fixtures** for shared setup: database connections, test clients
- **Inline values** for simple cases: prefer `"alice@example.com"` over a
  constant when the value is used once

### Anti-patterns

| Pattern | Problem |
|---------|---------|
| Shared mutable state between tests | Tests depend on execution order |
| Over-mocking | Tests pass but code is wrong |
| Testing only the happy path | Bugs hide in error paths |
| Asserting on implementation details | Tests break on refactors |

---

## Coverage

- **Target 80-90%** line coverage for critical paths
- **Don't chase 100%** — the last 10% is usually boilerplate that isn't worth
  testing
- Measure **branch coverage** in addition to line coverage for conditional logic
- Use coverage reports to find untested code, not as a quality metric on their own
