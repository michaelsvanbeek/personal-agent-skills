---
name: accessibility-testing
description: 'Automated accessibility testing and WCAG compliance validation. Use when: adding accessibility tests to a project, configuring axe-core or pa11y, integrating a11y checks into CI, auditing an existing application for accessibility violations, validating ARIA usage, testing keyboard navigation, or building an accessibility regression suite. Covers axe-core, Playwright a11y testing, Lighthouse accessibility audits, CI integration, and WCAG 2.2 AA compliance.'
tags:
  - developer
  - designer
---

# Accessibility Testing

## When to Use

- Adding automated accessibility tests to a web project
- Configuring axe-core, pa11y, or Lighthouse for a11y audits
- Integrating accessibility checks into CI/CD pipelines
- Auditing an existing application for WCAG 2.2 AA violations
- Validating ARIA attributes and roles
- Testing keyboard navigation flows
- Building an accessibility regression suite
- Retrofitting a11y testing into a project that has none

---

## Core Principle: Automate the Automatable, Manually Test the Rest

Automated tools catch ~30-40% of WCAG violations (missing alt text, low contrast, invalid ARIA). The rest requires manual testing with screen readers and keyboard navigation. **Automate the detectable, then build manual test plans for the rest.**

---

## Testing Pyramid for Accessibility

| Layer | Tool | Catches | Speed |
|-------|------|---------|-------|
| **Static analysis** | eslint-plugin-jsx-a11y | Missing alt, invalid ARIA in JSX | Instant (lint) |
| **Component tests** | @axe-core/react, vitest-axe | Per-component violations | Fast (unit) |
| **Integration tests** | Playwright + axe-core | Full-page violations, focus flow | Medium |
| **Audit** | Lighthouse CI | WCAG score, best practices | Slow (CI) |
| **Manual** | Screen reader + keyboard | Context, flow, comprehension | Slowest |

**Rule**: Start from the top. Catch what you can at lint time before it reaches tests.

---

## Static Analysis (ESLint)

### Setup

```bash
npm install -D eslint-plugin-jsx-a11y
```

### Config (`eslint.config.ts`)

```typescript
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  // ... other config
  jsxA11y.flatConfigs.recommended,
];
```

### What It Catches

- Missing `alt` on `<img>`
- Invalid ARIA attributes and roles
- Missing `htmlFor` on `<label>`
- Non-interactive elements with click handlers (missing `role` and keyboard support)
- Missing `lang` on `<html>`

---

## Component-Level Testing (Vitest + axe-core)

### Setup

```bash
npm install -D @axe-core/react vitest-axe jsdom
```

### Test Pattern

```typescript
import { render } from "@testing-library/react";
import { axe, toHaveNoViolations } from "vitest-axe";
import { expect, test } from "vitest";
import { MyComponent } from "./MyComponent";

expect.extend(toHaveNoViolations);

test("MyComponent has no accessibility violations", async () => {
  const { container } = render(<MyComponent />);
  const results = await axe(container);
  expect(results).toHaveNoViolations();
});
```

### Rules

- Add an `axe` test for every major component (forms, modals, navigation, tables).
- Run axe tests as part of the standard test suite — they execute in milliseconds.
- When a violation is found, fix the component, not the test. Never disable axe rules without documenting why.

---

## Integration Testing (Playwright + axe-core)

### Setup

```bash
npm install -D @playwright/test @axe-core/playwright
```

### Full-Page Audit

```typescript
import AxeBuilder from "@axe-core/playwright";
import { expect, test } from "@playwright/test";

test("homepage has no accessibility violations", async ({ page }) => {
  await page.goto("/");
  const results = await new AxeBuilder({ page })
    .withTags(["wcag2a", "wcag2aa", "wcag22aa"])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### Scoped Audit (Specific Section)

```typescript
test("checkout form is accessible", async ({ page }) => {
  await page.goto("/checkout");
  const results = await new AxeBuilder({ page })
    .include("#checkout-form")
    .withTags(["wcag2a", "wcag2aa"])
    .analyze();

  expect(results.violations).toEqual([]);
});
```

### Keyboard Navigation Test

```typescript
test("modal can be operated with keyboard only", async ({ page }) => {
  await page.goto("/");
  await page.click('[data-testid="open-modal"]');

  // Focus should be trapped in modal
  const modal = page.locator('[role="dialog"]');
  await expect(modal).toBeFocused();

  // Tab through focusable elements
  await page.keyboard.press("Tab");
  await expect(page.locator('[data-testid="modal-input"]')).toBeFocused();

  // Escape closes the modal
  await page.keyboard.press("Escape");
  await expect(modal).not.toBeVisible();

  // Focus returns to trigger element
  await expect(page.locator('[data-testid="open-modal"]')).toBeFocused();
});
```

### Rules

- Test every route/page for WCAG 2.2 AA compliance.
- Test interactive flows (forms, modals, dropdowns) for keyboard operability.
- Test focus management: opening modals traps focus, closing returns it to trigger.
- Use `withTags(["wcag2a", "wcag2aa", "wcag22aa"])` to target the correct WCAG level.

---

## Lighthouse CI

### Setup

```bash
npm install -D @lhci/cli
```

### Config (`lighthouserc.js`)

```javascript
module.exports = {
  ci: {
    collect: {
      url: ["http://localhost:3000/", "http://localhost:3000/dashboard"],
      startServerCommand: "npm run preview",
      numberOfRuns: 3,
    },
    assert: {
      assertions: {
        "categories:accessibility": ["error", { minScore: 0.9 }],
        "color-contrast": "error",
        "image-alt": "error",
        "label": "error",
        "link-name": "error",
        "button-name": "error",
      },
    },
    upload: {
      target: "temporary-public-storage",
    },
  },
};
```

### CI Integration (Drone)

```yaml
- name: lighthouse-a11y
  image: node:22-alpine
  commands:
    - npm ci
    - npm run build
    - npx @lhci/cli autorun
```

### Rules

- Accessibility score must be ≥90 (0.9). Regression below this blocks the build.
- Specific checks (`color-contrast`, `image-alt`, `label`, `link-name`, `button-name`) are always errors.
- Run Lighthouse 3 times and take the median score to reduce flakiness.

---

## WCAG 2.2 AA Compliance Checklist

The most commonly violated WCAG criteria:

### Perceivable

- [ ] All images have descriptive `alt` text (or `alt=""` for decorative images)
- [ ] Color contrast ratio ≥4.5:1 for normal text, ≥3:1 for large text
- [ ] Information is not conveyed by color alone
- [ ] Video has captions; audio has transcripts
- [ ] Content is readable at 200% zoom without horizontal scrolling

### Operable

- [ ] All interactive elements are reachable and operable via keyboard
- [ ] Focus order follows logical reading order
- [ ] Focus indicator is visible (never `outline: none` without replacement)
- [ ] No keyboard traps (user can always Tab/Escape out)
- [ ] Touch targets are ≥24x24 CSS pixels (WCAG 2.2 Level AA)
- [ ] Motion respects `prefers-reduced-motion` media query

### Understandable

- [ ] `<html lang="...">` is set on every page
- [ ] Form inputs have visible `<label>` elements
- [ ] Error messages identify the field and describe the fix
- [ ] Navigation is consistent across pages

### Robust

- [ ] Valid HTML (no duplicate IDs, proper nesting)
- [ ] ARIA roles and attributes are valid and correctly applied
- [ ] Custom components have appropriate ARIA roles, states, and properties
- [ ] Status messages use `role="status"` or `aria-live` regions

---

## Screen Reader Testing (Manual)

### Tools

| OS | Screen Reader | Browser |
|----|--------------|---------|
| macOS | VoiceOver (built-in) | Safari |
| Windows | NVDA (free) | Firefox or Chrome |
| Windows | JAWS (paid) | Chrome |

### Test Script

1. Navigate to the page using only keyboard (Tab, Enter, Arrow keys, Escape).
2. Listen to how the screen reader announces page landmarks, headings, and interactive elements.
3. Complete the primary user flow (e.g., submit a form, navigate to a detail page) without using the mouse.
4. Verify that dynamic content changes (modals, toasts, live regions) are announced.
5. Check that images are described meaningfully and decorative images are skipped.

---

## Handling Violations

### Severity Triage

| axe-core Impact | Action |
|----------------|--------|
| **critical** | Fix before merge. Blocks users entirely. |
| **serious** | Fix before merge. Significant barrier. |
| **moderate** | Fix within current sprint. Degraded experience. |
| **minor** | Add to backlog. Low-impact inconvenience. |

### Disabling Rules

When a rule must be disabled (rare), document it:

```typescript
const results = await new AxeBuilder({ page })
  .withTags(["wcag2a", "wcag2aa"])
  .disableRules(["color-contrast"]) // Known issue: #1234, tracking upstream fix
  .analyze();
```

**Rule**: Never disable a rule without a tracking issue and a comment explaining why.

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| `aria-label` on everything | Overrides visible text, confuses screen readers | Use visible labels; ARIA is a last resort |
| `outline: none` without replacement | Keyboard users can't see focus | Use `:focus-visible` with custom styles |
| `role="button"` on `<div>` | Missing keyboard support, missing semantics | Use `<button>` element instead |
| Testing only with automation | Misses flow, context, and comprehension issues | Pair automation with manual screen reader testing |
| Fixing tests instead of components | Violations persist for users | Fix the component, then verify the test passes |
| `tabindex="1"` or higher | Breaks natural focus order | Only use `tabindex="0"` or `tabindex="-1"` |

---

## Audit Checklist

When auditing an existing project for accessibility testing:

- [ ] ESLint jsx-a11y plugin is configured and enforced in CI
- [ ] Major components have axe-core unit tests
- [ ] Every route has a Playwright + axe-core integration test
- [ ] Lighthouse CI runs with accessibility score ≥0.9 threshold
- [ ] Critical and serious axe violations block the build
- [ ] Keyboard navigation is tested for all interactive flows (modals, forms, menus)
- [ ] Focus management is tested (modal trap, return to trigger)
- [ ] Manual screen reader testing is scheduled for major releases
- [ ] `prefers-reduced-motion` is respected for animations
- [ ] Disabled axe rules have tracking issues and comments
