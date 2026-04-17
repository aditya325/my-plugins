---
name: runtime-test
description: >
  Run Playwright-based runtime tests against shopify theme dev. Checks section
  rendering, JS errors, accessibility (axe-core), and setting-to-DOM wiring.
  Generates test scripts from schema and Liquid analysis, executes them, and
  reports results. Can be invoked standalone or dispatched by /assess.
disable-model-invocation: true
allowed-tools: Read, Write, Bash, Grep, Glob, AskUserQuestion
---

# Runtime Test — Playwright-Based Verification

You are running runtime tests against a live Shopify theme dev server. Your job is to generate targeted Playwright tests from the built code, execute them, and report results.

**This is NOT a code review. You are testing what the browser actually renders.**

## Input
Context or overrides: `$ARGUMENTS`

## Artifact Resolution
1. Read `.buildspace/current-feature` for the active feature name
2. If the file doesn't exist, look in `.buildspace/artifacts/` for feature folders containing `execution-log.md`
3. If one folder exists → use it
4. If multiple folders exist → ask the user which feature to test
5. If no execution-log.md found → tell the user to run `/execute` first

Read from `.buildspace/artifacts/{feature-name}/`:
- `execution-log.md` — files created/modified
- `selectors.json` — wrapper CSS selectors for sections (if exists)

---

## Pre-Flight Checks

### Step 1 — Ensure Playwright is installed

Check if Playwright is available:
```bash
npx playwright --version 2>/dev/null
```

If NOT installed:
```bash
# Initialize package.json if it doesn't exist
[ ! -f package.json ] && npm init -y

# Install Playwright and axe-core
npm install -D @playwright/test @axe-core/playwright

# Install only Chromium browser
npx playwright install chromium
```

Do not ask the user. Just install it.

If installation fails for any reason, report the error and skip automated tests — generate only the manual test checklist (jump to Step 9).

### Step 2 — Detect shopify theme dev

Try the default dev server URL:
```bash
curl -s -o /dev/null -w "%{http_code}" http://localhost:9292 --max-time 3
```

If response is `200` or `301` or `302` → dev server is running at `http://localhost:9292`. Store as `PREVIEW_URL`. Proceed.

If no response → Ask the user:
```
Is shopify theme dev running?
```

- If **no** → skip automated tests. Write results as "Skipped — shopify theme dev not running". Generate only the manual test checklist (jump to Step 9).
- If **yes** → Ask: `What's the preview URL? (e.g., http://localhost:9292)`

Store the URL as `PREVIEW_URL`.

### Step 3 — Determine test page URL

Identify which template file(s) include the new sections by searching for the section type name in template JSON files:
```
Grep('{section-filename-without-extension}', glob='templates/*.json')
```

Map the template file to a page path:

| Template file | Page path |
|---|---|
| `templates/index.json` | `/` |
| `templates/cart.json` | `/cart` |
| `templates/404.json` | `/not-a-real-page` |
| `templates/collection.json` or `templates/collection.*.json` | Ask user: "What collection URL should I test? (e.g., /collections/all)" |
| `templates/product.json` or `templates/product.*.json` | Ask user: "What product URL should I test? (e.g., /products/example-product)" |
| `templates/page.json` or `templates/page.*.json` | Ask user: "What page URL should I test? (e.g., /pages/about)" |
| `templates/blog.json` or `templates/article.json` | Ask user: "What blog/article URL should I test?" |

If section is not found in any template JSON → ask user: "Which page URL has the {section-name} section?"

Store as `PAGE_PATH`.

---

## Test Generation

### Step 4 — Analyze the built code

For each **section file** from execution-log:

**A. Read the schema** — extract:
- All settings: `id`, `type`, `default`, `label`
- All block types: `type`, and their settings
- Section `name`, `class`, `tag`

**B. Read the Liquid** — identify:
- Wrapper element and its CSS class (cross-reference with `selectors.json` if available)
- Which settings map to which DOM elements — trace `{{ section.settings.{id} }}` to the nearest HTML element and note the element/class
- Block rendering structure — the `{% case block.type %}` or iteration pattern, and CSS classes for each block type
- Any JavaScript files loaded via `<script>` tags or `{{ 'file.js' | asset_url }}`

**C. Read the template JSON** — identify:
- The section's key in the `sections` object (e.g., `"featured_collection_abc123"`)
- Current setting values
- Current blocks and their `block_order`

Collect this analysis into a structured understanding before generating tests.

### Step 5 — Generate Playwright test script

Write a Playwright config to `.buildspace/artifacts/{feature-name}/playwright.config.js`:

```js
const { defineConfig } = require('@playwright/test');
module.exports = defineConfig({
  testDir: './',
  timeout: 30000,
  retries: 0,
  reporter: [['list'], ['json', { outputFile: 'test-results.json' }]],
  use: {
    baseURL: '{PREVIEW_URL}',
    headless: true,
  },
});
```

Replace `{PREVIEW_URL}` with the actual detected/provided URL.

Write the test script to `.buildspace/artifacts/{feature-name}/runtime-tests.spec.js`.

The script MUST follow this structure — adapt the specifics to the feature:

```js
const { test, expect } = require('@playwright/test');
const AxeBuilder = require('@axe-core/playwright');
const fs = require('fs');
const path = require('path');

const TEMPLATE_PATH = path.resolve('{template-json-path}');
const PAGE_PATH = '{page-path}';

let originalTemplate;

test.beforeAll(async () => {
  originalTemplate = fs.readFileSync(TEMPLATE_PATH, 'utf-8');
});

test.afterAll(async () => {
  fs.writeFileSync(TEMPLATE_PATH, originalTemplate);
});

test.afterEach(async () => {
  // Restore after every test that modifies settings — defense in depth
  fs.writeFileSync(TEMPLATE_PATH, originalTemplate);
});

test.describe('{Section Name} — Runtime Tests', () => {

  // ── GROUP 1: Section Renders ──
  test('section renders on page', async ({ page }) => {
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');
    const section = page.locator('{wrapper-selector}');
    await expect(section).toBeVisible();
  });

  // ── GROUP 2: No JavaScript Errors ──
  test('no JavaScript console errors on page load', async ({ page }) => {
    const errors = [];
    page.on('pageerror', err => errors.push(err.message));
    page.on('console', msg => {
      if (msg.type() === 'error') errors.push(msg.text());
    });
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');
    expect(errors).toEqual([]);
  });

  // ── GROUP 3: Accessibility ──
  test('accessibility scan passes (axe-core)', async ({ page }) => {
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');
    const results = await new AxeBuilder({ page })
      .include('{wrapper-selector}')
      .analyze();
    // Store violations for reporting even if test passes
    if (results.violations.length > 0) {
      const summary = results.violations.map(v =>
        `[${v.impact}] ${v.id}: ${v.help} (${v.nodes.length} elements)`
      ).join('\n');
      console.log('AXE_VIOLATIONS:' + JSON.stringify(results.violations));
      expect(results.violations, summary).toEqual([]);
    }
  });

  // ── GROUP 4: Setting-to-DOM Wiring ──
  // Generate ONE test per text-outputting setting (text, richtext, inline_richtext, html)
  // Example for a single setting — repeat this pattern for each:

  test('setting "{setting-id}" renders in DOM', async ({ page }) => {
    const testValue = 'RT_{setting-id}_test';

    const template = JSON.parse(fs.readFileSync(TEMPLATE_PATH, 'utf-8'));
    template.sections['{section-key}'].settings['{setting-id}'] = testValue;
    fs.writeFileSync(TEMPLATE_PATH, JSON.stringify(template, null, 2));

    await page.waitForTimeout(1500);
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');

    const section = page.locator('{wrapper-selector}');
    await expect(section).toContainText(testValue);
  });

  // ── GROUP 5: Empty State — Blank Settings ──
  test('handles blank settings without breaking', async ({ page }) => {
    const errors = [];
    page.on('pageerror', err => errors.push(err.message));

    const template = JSON.parse(fs.readFileSync(TEMPLATE_PATH, 'utf-8'));
    const settings = template.sections['{section-key}'].settings;

    // Blank out all text-type settings
    // {For each text/richtext/inline_richtext/html setting: settings['{id}'] = '';}

    fs.writeFileSync(TEMPLATE_PATH, JSON.stringify(template, null, 2));

    await page.waitForTimeout(1500);
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');

    // Section wrapper should still exist (not crash)
    const section = page.locator('{wrapper-selector}');
    await expect(section).toBeAttached();

    // No "undefined" or "null" leaked into visible text
    const text = await section.textContent();
    expect(text).not.toContain('undefined');
    expect(text).not.toContain('null');

    // No JS errors from blank settings
    expect(errors).toEqual([]);
  });

  // ── GROUP 6: Block Rendering ──
  // Generate ONLY if section has blocks defined in schema

  test('all declared block types render', async ({ page }) => {
    const template = JSON.parse(fs.readFileSync(TEMPLATE_PATH, 'utf-8'));
    const sectionData = template.sections['{section-key}'];

    // Ensure at least one block of each declared type exists
    // {Generate block objects with required settings based on schema}
    sectionData.blocks = {
      // 'test_block_1': { type: '{block-type-1}', settings: {...} },
      // 'test_block_2': { type: '{block-type-2}', settings: {...} },
    };
    sectionData.block_order = [/* 'test_block_1', 'test_block_2' */];

    fs.writeFileSync(TEMPLATE_PATH, JSON.stringify(template, null, 2));

    await page.waitForTimeout(1500);
    await page.goto(PAGE_PATH);
    await page.waitForLoadState('networkidle');

    // Assert each block type rendered something
    // {For each block type, check its specific CSS class or element is visible}
  });

});
```

**Generation rules:**

1. **Replace ALL placeholders** — `{wrapper-selector}`, `{section-key}`, `{setting-id}`, `{template-json-path}`, `{page-path}` must be actual values from Step 4 analysis.

2. **Setting wiring tests** — only generate for settings that produce **visible text content**:
   - YES: `text`, `richtext`, `inline_richtext`, `html`
   - NO: `color`, `color_background`, `range`, `number`, `checkbox`, `select`, `radio`, `font_picker`, `image_picker`, `url`, `video_url`, `collection`, `product`, `blog`, `page`, `link_list`

3. **Image setting tests** — for `image_picker` settings, instead of text assertion, test that an `<img>` element exists inside the wrapper when set and doesn't render an empty/broken `<img>` when blank. Generate as a separate test if the section has image settings.

4. **Block tests** — only generate if the schema declares blocks. Populate test blocks with minimal required settings from the schema defaults.

5. **Wrapper selector** — use from `selectors.json` first. If not available, derive from the section's outermost element CSS class by reading the Liquid.

6. **afterEach restore** — every test that modifies the template JSON gets a restore in `afterEach`. This ensures a failed test doesn't corrupt state for subsequent tests.

7. **Wait timing** — use `page.waitForTimeout(1500)` after writing the template file to give `shopify theme dev` time to detect the file change and rebuild. Then `page.goto()` + `waitForLoadState('networkidle')` ensures the page is fully loaded.

8. **axe-core import** — wrap in a try/catch at the top of the file. If `@axe-core/playwright` is not available, skip the accessibility test gracefully:
   ```js
   let AxeBuilder;
   try { AxeBuilder = require('@axe-core/playwright').default || require('@axe-core/playwright'); } catch (e) { /* skip */ }
   ```
   And in the test:
   ```js
   test('accessibility scan passes (axe-core)', async ({ page }) => {
     test.skip(!AxeBuilder, 'axe-core not installed');
     // ...
   });
   ```

---

## Test Execution

### Step 6 — Run the tests

```bash
cd {project-root} && npx playwright test --config=.buildspace/artifacts/{feature-name}/playwright.config.js 2>&1
```

Capture the full output.

### Step 7 — Parse results

From the Playwright output, extract:
- Total tests run, passed, failed, skipped
- For each failure: test name, error message, expected vs actual
- For accessibility: violation details if any

Also check if `.buildspace/artifacts/{feature-name}/test-results.json` was generated (from the JSON reporter) and read it for structured results.

---

## Report

### Step 8 — Write results

Write results to `.buildspace/artifacts/{feature-name}/runtime-test-results.md`.

Read the template from `${CLAUDE_SKILL_DIR}/templates/runtime-test-results-template.md` and fill it in with findings from all tests.

Tell the user:
- Where results were saved
- Pass/fail count
- Summary of any failures

**Do NOT output the full report in conversation. The artifact file is the source of truth.**

---

## Manual Test Checklist

### Step 9 — Generate manual test checklist

After automated tests complete (or if automated tests were skipped), generate a manual test checklist and **append it to the runtime test results file**.

Derive the checklist from the specific feature — use the schema settings, block types, and requirements (from `clarify.md` if it exists) to generate targeted scenarios. Do not use generic checklists.

**Theme Customizer Tests:**
- Add the section via theme editor → does it appear in the preview?
- Remove the section → does the page render without errors?
- Toggle each boolean/checkbox setting → does the visual change match the label?
- Reorder blocks via drag-and-drop → does the preview reflect the new order?
- Change a text setting in the editor → does the live preview update without a full reload?

**Visual / Responsive Tests:**
- View section at mobile (375px), tablet (768px), desktop (1440px)
- Verify spacing and alignment with the sections above and below
- Test with very long text content (3x expected length) → no overflow or layout break
- Test with minimum content (shortest possible text, fewest items)

**Edge Case Tests (derive from the feature):**
These are feature-specific. Think about what could realistically break:
- e.g., "Set collection to one with 0 products → verify empty state message shows"
- e.g., "Upload a very wide panoramic image → verify it's contained within the section"
- e.g., "Set all optional settings to blank → verify no broken HTML or empty wrappers"
- e.g., "Add maximum number of blocks → verify layout handles it"

Format each as a checkbox:
```markdown
- [ ] {Scenario} → Expected: {what should happen}
```

---

## Cleanup

### Step 10 — Verify project state

After all tests and reporting:
1. Read the template JSON file that was modified during tests
2. Compare with the original content from before tests ran
3. If different, restore the original content
4. Confirm restoration in the results file

This is critical — **never leave the project in a modified state after testing**.

---

## Rules
- **Never fix issues** — only identify and report them
- **Always restore template files** — leave the project exactly as you found it
- **Install Playwright without asking** — just do it
- **Ask about dev server only if default port doesn't respond** — don't ask if it's already reachable
- **Unique test values** — use identifiable strings (prefixed with `RT_`) so failures are traceable
- **One execution** — run tests once, report, stop. No retries, no fix loops
- **Generated scripts stay** — save to artifacts so the user can inspect and re-run manually
- **Skip gracefully** — if any pre-flight check fails, skip automated tests and generate the manual checklist. Never error out entirely.
