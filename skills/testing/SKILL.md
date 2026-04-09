---
name: testing
description: 'Testing strategy, patterns, and evaluation for software and LLM/AI systems. Use when: writing tests, choosing test boundaries, designing test data, structuring test suites, evaluating LLM outputs, building evaluation pipelines, setting coverage thresholds, auditing test coverage gaps in existing projects, or improving test quality and structure.'
tags:
    - developer
---

# Testing Standards

## When to Use

- Writing tests for any application or service
- Choosing what level of testing to apply
- Designing test data, fixtures, or factories
- Setting up CI test pipelines
- Evaluating LLM/AI agent outputs
- Building evaluation harnesses for AI systems
- Auditing an existing project's test suite for coverage gaps, weak assertions, or structural problems
- Improving test quality (naming, isolation, mocking strategy, data factories)
- Identifying untested critical paths in an existing codebase

---

## Testing Pyramid

| Level | Scope | Speed | Count |
|-------|-------|-------|-------|
| **Unit** | Single function/class, no I/O | <10ms each | Many (70%+) |
| **Integration** | Module boundaries, real DB/API calls | <1s each | Moderate (20%) |
| **E2E** | Full user workflow through the system | Seconds | Few (10%) |

- Most tests should be **unit tests**: fast, isolated, focused on logic.
- Integration tests verify that modules work together correctly.
- E2E tests prove critical user journeys work. Keep the count low — they are slow and brittle.

---

## What to Test

### Always Test

- Pure logic and business rules
- Edge cases: empty input, null, zero, boundary values, max length
- Error paths: invalid input, missing config, network failures
- State transitions and conditional branches
- Data transformations and formatting

### Don't Test

- Framework internals (React rendering lifecycle, FastAPI routing plumbing)
- Third-party library behavior
- Trivial getters/setters with no logic
- Implementation details (private methods, internal state shape)

---

## Mocking Strategy

### Mock External, Use Real Internal

- **Mock**: HTTP APIs, databases (for unit tests), file systems, time/clocks, third-party services, randomness
- **Don't mock**: Your own business logic, data transformations, utility functions

### Mock Rules

- Mock at the **boundary**, not deep inside the call chain.
- Each mock should assert it was called with expected arguments, not just that it exists.
- Use `pytest` fixtures and `unittest.mock.patch` in Python. Use `vi.mock` / `vi.fn` in Vitest.
- Never mock what you don't own unless you have integration tests covering the real thing.

---

## Test Naming

Tests should read as specifications. Name them by behavior, not implementation:

```python
# Good
class TestOrderProcessing:
    def test_applies_discount_when_coupon_is_valid(self): ...
    def test_rejects_order_when_inventory_is_insufficient(self): ...
    def test_returns_empty_list_when_no_orders_exist(self): ...

# Bad
class TestOrderService:
    def test_process(self): ...
    def test_error(self): ...
```

```typescript
// Good
describe("OrderService", () => {
  it("applies discount when coupon is valid", () => { ... });
  it("rejects order when inventory is insufficient", () => { ... });
});
```

---

## Test Data

### Fixtures and Factories

- Use **factory functions** that create valid test data with sensible defaults, allowing per-test overrides:

```python
def make_user(**overrides) -> User:
    defaults = {
        "id": "usr_test",
        "name": "Test User",
        "email": "test@example.com",
        "role": "viewer",
    }
    return User(**(defaults | overrides))

# Usage
admin = make_user(role="admin")
```

```typescript
function makeUser(overrides: Partial<User> = {}): User {
  return {
    id: "usr_test",
    name: "Test User",
    email: "test@example.com",
    role: "viewer",
    ...overrides,
  };
}
```

- Use `pytest`'s `tmp_path` for file system tests. Never write to real paths.
- Use `freezegun` or `time_machine` in Python for time-dependent tests.

### Test Isolation

- Each test must be independent. No test should depend on another test's state or execution order.
- Reset shared state (databases, caches, global variables) between tests.
- Use transactions with rollback for database tests.

---

## Snapshot Testing

- Use for complex output that is correct but tedious to assert manually: HTML output, API response shapes, serialized configs.
- **Review** snapshot diffs carefully — don't blindly update snapshots.
- Keep snapshots small. If a snapshot is >50 lines, assert specific fields instead.

---

## Coverage

- Target **80% line coverage** as a floor, not a ceiling.
- Coverage measures what your tests execute, not what they verify. High coverage with weak assertions is worthless.
- Focus coverage effort on business logic, not boilerplate.
- Never add tests purely to increase coverage numbers. Every test should verify meaningful behavior.

---

## CI Integration

- Tests run on every PR. Merging is blocked on test failure.
- Test suite must complete in **<5 minutes** for fast feedback. Parallelize if needed.
- Flaky tests are bugs. Fix or delete them — never mark them as allowed-to-fail permanently.
- Run linting and type checking before tests (fail fast on syntax/type errors).

---

## Release Validation

Tests play a critical gate role at release time (see release-management skill for full workflow):

- **Tag-triggered CI must re-run the full test suite** — the tag may point to a different commit than the last CI run.
- **Coverage thresholds must pass on the release commit** — don't relax standards for releases.
- **Pre-release versions** (`v2.0.0-beta.1`) should run the same test gates as stable releases.
- **Breaking changes (MAJOR bumps)** should include migration tests or integration tests that verify backward-compatibility is intentionally removed.
- **Hotfix releases** must include a regression test for the specific bug being fixed, proving it existed and is now resolved.

---

## Performance Testing

Performance testing is a specialized discipline with its own tools and strategies:

- For **API and service performance** (load tests, stress tests, regression baselines, A/B experiments): see the **performance-testing skill**.
- For **frontend page performance** (Core Web Vitals, bundle analysis, HAR analysis, memory diagnostics): see the **ui-performance skill**.

Integrate performance tests into CI using the tiered approach from the performance-testing skill: smoke tests on every PR, full load tests on release tags.

---

## LLM and AI Evaluation

Testing LLM-based systems requires a different approach because outputs are non-deterministic and correctness is often fuzzy.

### Evaluation Levels

| Level | What it tests | How |
|-------|--------------|-----|
| **Unit** | Individual tools, parsers, formatters | Standard unit tests — these are deterministic |
| **Component** | Single LLM call with prompt + expected behavior | Assertion-based eval with LLM-as-judge fallback |
| **Agent** | Multi-step workflow end-to-end | Scenario-based eval with success criteria |
| **Regression** | Known failure cases don't regress | Golden dataset of input/expected-output pairs |

### Evaluation Dimensions

Score LLM outputs across multiple dimensions, not just "correct/incorrect":

| Dimension | What it measures | How to evaluate |
|-----------|-----------------|-----------------|
| **Correctness** | Factual accuracy of the output | Compare against golden answer; LLM-as-judge |
| **Relevance** | Does the output address the input? | LLM-as-judge or embedding similarity |
| **Completeness** | Are all required elements present? | Checklist of required fields/topics |
| **Conciseness** | No unnecessary content | Token count vs. golden answer length |
| **Format compliance** | Matches expected structure | Schema validation (JSON schema, regex) |
| **Safety** | No harmful, biased, or leaked content | Content classifier + keyword blocklist |
| **Latency** | Response time | Wall-clock measurement |
| **Token efficiency** | Cost per evaluation | Input + output token count |

### Evaluation Dataset

Build and maintain a curated evaluation dataset:

```python
eval_cases = [
    {
        "id": "order-summary-001",
        "input": "Summarize order #12345",
        "context": {"order": {...}},  # relevant context provided to the LLM
        "expected": "Order #12345: 3 items, total $47.50, shipped March 18",
        "criteria": {
            "must_contain": ["#12345", "$47.50"],
            "must_not_contain": ["credit card", "password"],
            "max_tokens": 100,
        },
    },
]
```

- Start with **20-50 cases** covering happy paths and known edge cases.
- Add a new case every time you find a failure in production.
- Version the eval dataset alongside the code.

### LLM-as-Judge

For dimensions that are hard to evaluate programmatically, use a capable model as a judge:

```python
judge_prompt = """
Rate the following response on a scale of 1-5 for correctness and relevance.

Input: {input}
Expected: {expected}
Actual: {actual}

Return JSON: {"correctness": <1-5>, "relevance": <1-5>, "reasoning": "<brief explanation>"}
"""
```

- Use a **stronger model** as judge than the model being evaluated.
- Include the expected answer in the judge prompt for calibration.
- Run the judge multiple times (3-5) and take the median to reduce variance.
- Track judge agreement rate — if it varies wildly, refine the rubric.

### Regression Testing

- Maintain a **golden set** of input/output pairs that represent known-correct behavior.
- Run against the golden set on every prompt change, model update, or tool change.
- Track pass rates over time. A drop signals regression.
- Pin model versions in production. Test new model versions against the eval suite before promoting.

### Cost-Aware Testing

- Log token usage (input + output) for every eval run.
- Set budget limits per eval suite run.
- Use cheaper models (e.g., GPT-4o-mini, Claude Haiku) for high-volume regression testing.
- Reserve expensive models for final validation and judge evaluations.
- Cache LLM responses during development to avoid re-running identical calls.

---

## Property-Based Testing

For functions with broad input domains, use property-based testing to generate many random inputs and verify invariants:

```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_preserves_length(items):
    assert len(sorted(items)) == len(items)

@given(st.lists(st.integers(), min_size=1))
def test_sort_first_element_is_minimum(items):
    assert sorted(items)[0] == min(items)
```

Use when:
- Input space is large and edge cases are hard to enumerate manually.
- Function should satisfy invariants regardless of input (sort, serialize/deserialize, encode/decode).

## IDE Integration

For configuring pytest and Vitest test runners in VS Code / Cursor, including Test Explorer integration, debug configurations, and local task automation, see the **ide-setup skill**.

## iOS / Swift Testing

For iOS-specific testing patterns including Swift Testing framework (`@Suite`, `@Test`, `#expect`), ViewInspector for SwiftUI view testing, URLProtocol network mocking, SwiftData test containers, and protocol-based mock infrastructure, see the **ios-testing skill**.
