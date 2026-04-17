# Runtime Test Results: {Feature Name}

## Environment
- **Dev Server:** {preview-url} ({running / not running})
- **Test Page:** {page-path}
- **Browser:** Chromium (headless)
- **Playwright:** {version}

## Automated Test Results

| # | Test | Status | Details |
|---|---|---|---|
| 1 | Section renders on page | PASS / FAIL | {details} |
| 2 | No JavaScript console errors | PASS / FAIL | {error list if any} |
| 3 | Accessibility (axe-core) | PASS / FAIL / SKIPPED | {violation count or skip reason} |
| 4 | Setting: {id} wiring | PASS / FAIL | {per setting} |
| ... | ... | ... | ... |
| N | Empty state handling | PASS / FAIL | {details} |
| N+1 | Block rendering | PASS / FAIL / N/A | {details or "no blocks in schema"} |

**Summary:** {passed}/{total} passed, {failed} failed, {skipped} skipped

## Accessibility Violations
<!-- Only include if axe-core found violations. Remove section if clean. -->
| Rule | Impact | Elements | Description |
|---|---|---|---|
| {rule-id} | {critical/serious/moderate/minor} | {count} | {help text} |

**Fix guidance:** {brief guidance per violation}

## Failed Test Details
<!-- Only include if tests failed. Remove section if all passed. -->

### {Test Name}
- **Expected:** {what should have happened}
- **Actual:** {what happened}
- **Root Cause Hint:** {brief analysis — e.g., "setting 'heading' outputs in Liquid but the null guard prevents rendering when the template JSON has no default value"}

## Test Artifacts
- **Test script:** `.buildspace/artifacts/{feature-name}/runtime-tests.spec.js`
- **Playwright config:** `.buildspace/artifacts/{feature-name}/playwright.config.js`
- **Re-run command:** `npx playwright test --config=.buildspace/artifacts/{feature-name}/playwright.config.js`

---

## Manual Test Checklist

> These tests require human verification — theme customizer interaction, visual judgment, and edge cases that can't be automated.

### Theme Customizer
<!-- Derive from the feature's schema settings and blocks -->
- [ ] {Scenario} → Expected: {result}

### Visual / Responsive
<!-- Derive from the feature's layout and content -->
- [ ] {Scenario} → Expected: {result}

### Edge Cases
<!-- Derive from the feature's specific requirements and schema -->
- [ ] {Scenario} → Expected: {result}
