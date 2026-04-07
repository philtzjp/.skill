---
name: test-responsibility
description: 新機能の実装後、agent-browserまたはPlaywrightでE2Eテストを自動作成・実行する。新機能実装時に自動発動。
---

# Test Responsibility

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Overview

When a new feature is implemented, this skill ensures E2E test coverage is created and executed. Uses agent-browser as the primary testing tool, falling back to Playwright for interactions agent-browser cannot handle.

## Trigger

This skill activates automatically when:

- A new user-facing feature has been implemented
- A significant UI change or new page/route has been added
- A user flow has been modified (e.g., auth, checkout, form submission)

MUST NOT trigger for:

- Pure backend/API changes with no UI surface
- Documentation-only changes
- Config or dependency updates
- Refactors with no behavioral change

## Tool Selection

### agent-browser (Primary)

Use agent-browser for all standard web interactions:

- Page navigation and routing
- Clicking buttons, links, and interactive elements
- Filling forms and selecting options
- Reading text content and verifying visibility
- Waiting for network/load states
- Taking screenshots for visual verification
- Drag and drop operations
- File uploads
- Snapshot-based element discovery (`@e1`, `@e2` refs)

### Playwright (Fallback)

Fall back to Playwright when agent-browser cannot cover the interaction:

- **Canvas element manipulation** — swipe, draw, pinch-zoom inside `<canvas>`
- **Fine-grained mouse sequences** — `mousedown → mousemove → mouseup` paths that require precise coordinate control
- **Touch gesture simulation** on desktop browsers (multi-touch, rotation)
- **Low-level network interception** with request/response modification
- **Multi-page/cross-origin flows** requiring synchronized state across tabs
- **WebSocket or SSE assertion** requiring message-level inspection
- **Accessibility tree assertions** via `toMatchAriaSnapshot`

When falling back to Playwright, MUST note in the test file why agent-browser was insufficient:

```typescript
// Playwright fallback: canvas swipe gesture not supported by agent-browser
```

## Procedure

### 1. Identify What to Test

After implementing a feature, MUST analyze the changes to determine:

- **User journeys affected** — which flows does this feature touch?
- **Entry points** — what URL or action starts the flow?
- **Expected outcomes** — what should the user see/experience?
- **Edge cases** — empty states, error states, boundary inputs

### 2. Determine Test Location

Check for existing test infrastructure:

```
1. e2e/ or tests/e2e/ directory exists → use it
2. __tests__/ or tests/ with playwright config → use it
3. Neither exists → create e2e/ at project root
```

MUST follow the project's existing test file naming convention. If none exists, use:
- agent-browser tests: `e2e/<feature>.test.sh` (shell scripts calling agent-browser CLI)
- Playwright tests: `e2e/<feature>.spec.ts`

### 3. Write Tests with agent-browser

Follow the snapshot-first workflow:

```bash
#!/usr/bin/env bash
# e2e/user-registration.test.sh
set -euo pipefail

echo "=== Test: User Registration ==="

# Navigate
agent-browser open http://localhost:3000/register

# Discover elements
agent-browser snapshot -i

# Fill form
agent-browser fill @e1 "testuser@example.com"
agent-browser fill @e2 "SecurePass123!"
agent-browser fill @e3 "SecurePass123!"

# Submit
agent-browser click @e4

# Wait for navigation
agent-browser wait --url "**/dashboard"
agent-browser wait --load networkidle

# Verify
TEXT=$(agent-browser get text @e1)
if [[ "$TEXT" == *"Welcome"* ]]; then
  echo "PASS: Registration successful"
else
  echo "FAIL: Expected welcome message"
  agent-browser screenshot
  exit 1
fi

agent-browser close
```

#### Key Patterns

- **Always snapshot before interacting** — refs are invalidated after DOM changes
- **Re-snapshot after navigation or state changes** — stale refs cause failures
- **Use `wait` before assertions** — ensure the page has settled
- **Screenshot on failure** — capture visual state for debugging
- **Use `snapshot -i`** — interactive elements only, reduces noise

### 4. Write Tests with Playwright (When Needed)

```typescript
// e2e/canvas-drawing.spec.ts
// Playwright fallback: canvas drawing requires precise mouse coordinate sequences
import { test, expect } from '@playwright/test';

test('user can draw on canvas', async ({ page }) => {
  await page.goto('/editor');

  const canvas = page.locator('canvas');
  const box = await canvas.boundingBox();

  // Simulate swipe/draw gesture
  await page.mouse.move(box!.x + 50, box!.y + 50);
  await page.mouse.down();
  await page.mouse.move(box!.x + 200, box!.y + 200, { steps: 20 });
  await page.mouse.up();

  // Verify canvas state changed
  await expect(page.locator('[data-testid="stroke-count"]')).toHaveText('1');
});
```

### 5. Run Tests

#### agent-browser tests

```bash
# Single test
bash e2e/user-registration.test.sh

# All tests
for f in e2e/*.test.sh; do bash "$f"; done
```

#### Playwright tests

```bash
npx playwright test e2e/canvas-drawing.spec.ts
```

### 6. Report Results

After running tests, MUST report:

```
## Test Results

| Test | Tool | Result |
|------|------|--------|
| User registration | agent-browser | ✅ PASS |
| Canvas drawing | Playwright | ✅ PASS |
| Login flow | agent-browser | ❌ FAIL |

### Failures
- **Login flow**: Expected redirect to /dashboard, got /error
  Screenshot: /tmp/agent-browser/screenshot-20260404-123456.png

### Coverage
- New feature: user registration → 3 tests
- Affected flow: login → 1 test (FAILING)
```

If any test fails, MUST:
1. Analyze the failure
2. Determine if it's a test issue or a feature bug
3. Fix the test if the test is wrong, or report the bug if the feature is broken

## Prohibitions

- NEVER skip writing tests for a new user-facing feature
- NEVER use Playwright when agent-browser can handle the interaction
- NEVER hardcode element selectors when snapshot refs are available — refs adapt to DOM changes
- NEVER write tests that depend on timing (`sleep 3`) — use explicit wait conditions
- NEVER leave the browser open after test completion — always `agent-browser close` or let Playwright cleanup
- NEVER commit tests that are known to be flaky without documenting the flakiness reason
