---
name: assess
description: >
  Assess built features with first-principles thinking. Checks requirements
  coverage, standards compliance, integration correctness, and edge cases.
  Reports findings and stops — no auto-fix. Use after /execute.
disable-model-invocation: true
allowed-tools: Read, Write, Grep, Glob, Bash, Agent, Skill, AskUserQuestion
---

# Assess — First-Principles Verification

You are entering the Assess phase. Your job is to verify that what was built is correct, complete, and follows standards. Think from first principles — don't just pattern-match against checklists. Understand WHY each requirement exists and whether the implementation actually satisfies it.

**Report findings and stop. Do not fix anything. Do not loop.**

## CRITICAL: You MUST Dispatch Subagents

Steps 2, 3, and 5 below are **not optional** and **not inlineable**. You MUST call the `Agent` tool with the specified `subagent_type` for each one. Do NOT perform these checks yourself by reading files inline — the whole point is to keep their output out of the main context and to use the specialist's system prompt.

If you find yourself about to run `Read` / `Grep` to check requirements coverage, code standards, or runtime behavior — stop. That work belongs to the dispatched agent. Your job is to orchestrate the agents, aggregate their findings, and write the report.

## Input
Context or overrides: `$ARGUMENTS`

## Artifact Resolution
1. Read `.buildspace/current-feature` for the active feature name
2. If the file doesn't exist, look in `.buildspace/artifacts/` for feature folders containing `execution-log.md`
3. If one folder exists → use it
4. If multiple folders exist → ask the user which feature to assess
5. If no execution-log.md found → ask the user to run `/execute` first

Read from `.buildspace/artifacts/{feature-name}/`:
- `clarify.md` — requirements to assess against
- `plan.md` — planned approach and test cases
- `execution-log.md` — files created/modified

---

## Assessment Process

### Step 1 — Automated Checks

Run automated validation directly:

#### Shopify Theme Check
```bash
shopify theme check --path . --fail-level error
```
If not available, skip and note it.

#### Schema JSON Validation
For each section file from execution-log:
```bash
node -e "
const fs = require('fs');
const content = fs.readFileSync('{file-path}', 'utf-8');
const match = content.match(/\{%\s*schema\s*%\}([\s\S]*?)\{%\s*endschema\s*%\}/);
if (match) { JSON.parse(match[1]); console.log('Valid'); }
else { console.log('No schema found'); }
"
```

Report any failures.

### Step 2 — Requirements Assessment (MUST dispatch subagent)

Call the `Agent` tool with `subagent_type: "output-validator"` and the `description: "Requirements validation"`. Pass the following as the `prompt`:

```
Validate feature: {feature-name}

Files: [list exact paths from execution-log]
Requirements: .buildspace/artifacts/{feature-name}/clarify.md (if exists)
Plan: .buildspace/artifacts/{feature-name}/plan.md (if exists)

Check every file for:
1. Null/blank guards on every setting that outputs HTML
2. Empty collection handling in loops
3. Block rendering covers all block types declared in schema
4. Images have alt text, width, height attributes
5. Links have valid href
6. Conditional display works (blank settings hide elements)
7. Schema settings correctly wired to Liquid output

If requirements exist: verify each requirement — Met / Partially met / Not implemented.
If plan exists: run each test case — Pass / Fail with reason.

Return a structured report with sections: Requirements Coverage, Edge Cases, Null/Blank Guards, Block Rendering, Image/Link Validation. Include file:line references for every finding.
```

Wait for the agent to return before moving to Step 3. Store its output verbatim — it becomes the Requirements Coverage section of the report.

### Step 3 — Standards Assessment (MUST dispatch subagent)

Call the `Agent` tool with `subagent_type: "code-reviewer"` and `description: "Standards review"`. Pass the following as the `prompt`:

```
Review these files: [list exact paths from execution-log]

Execution log: .buildspace/artifacts/{feature-name}/execution-log.md

For each file, validate against the relevant skill checklist (liquid-standards, css-standards, js-standards, section-standards as applicable).
Check standards compliance, readability, maintainability.
Report issues with severity: Critical / Should Fix / Nice to Have.
Include file:line references for every issue.

Also check cross-file concerns:
- Unused snippets (created but never rendered)
- CSS class conflicts with existing files
- Schema ID collisions across sections
- Orphaned assets (CSS/JS not loaded anywhere)

Return a structured report grouped by file, followed by the cross-file section.
```

Wait for the agent to return. Its output becomes the Standards Compliance section of the report.

### Step 4 — Integration Assessment

After both agents return, check integration directly with `Grep` and `Glob`:

- **Template registration:** `Grep('{section-name}', glob='templates/*.json')` — is each new section registered?
- **CSS loading:** `Grep('{css-filename}', glob='sections/*.liquid')` — is each CSS file loaded?
- **Snippet wiring:** `Grep('render "{snippet-name}"', glob='sections/*.liquid')` — are snippets referenced?
- **Asset existence:** `Glob('assets/{filename}')` — do all referenced CSS/JS files exist on disk?

### Step 5 — Runtime Testing (MUST dispatch subagent)

There is no dedicated runtime-testing subagent. Dispatch the built-in `general-purpose` agent and point it at the `runtime-test` skill inside this plugin. Call the `Agent` tool with `subagent_type: "general-purpose"` and `description: "Runtime testing"`. Pass the following as the `prompt`:

```
Run runtime tests for feature: {feature-name}

Read and follow the runtime-test skill at:
${CLAUDE_SKILL_DIR}/../runtime-test/SKILL.md

Inputs:
- Feature artifacts dir: .buildspace/artifacts/{feature-name}/
- Files from execution-log: [list exact paths]
- Selectors hint: .buildspace/artifacts/{feature-name}/selectors.json (if exists)

Required process (follow the skill verbatim):
1. Pre-flight — ensure Playwright is installed (install if missing; do not ask).
   Detect shopify theme dev at http://localhost:9292. If no response, ask the
   user whether it's running and for the preview URL.
2. Determine which page URL exercises the new section(s) by reading templates/*.json.
   For index.json use "/". For product/collection/page templates, ask the user.
3. Analyze each section file: schema (settings, blocks), Liquid (wrapper selector,
   setting-to-DOM mapping), template JSON (section key, current values).
4. Generate a Playwright test script covering:
   - Section renders on page (wrapper selector visible)
   - No JavaScript console errors on page load
   - Accessibility scan (axe-core) scoped to the section
   - Setting-to-DOM wiring for each text-outputting setting
   - Empty state (blank all text settings, verify no broken HTML)
   - Block rendering (if section has blocks)
5. Execute: npx playwright test --config=.buildspace/artifacts/{feature-name}/playwright.config.js
6. Write results to .buildspace/artifacts/{feature-name}/runtime-test-results.md
7. Append a manual test checklist for non-automatable scenarios (theme customizer,
   visual/responsive, edge cases) to that file.
8. Always restore template JSON files to their original state after testing.

If shopify theme dev is not reachable, skip automated tests and generate only
the manual test checklist. Return a summary of results and the path to the
results file.
```

When the agent returns, read `.buildspace/artifacts/{feature-name}/runtime-test-results.md` so you can fold its contents into the final assessment report.

### Step 6 — First-Principles Questions

Think about the built feature from first principles. Ask yourself context-appropriate questions like:

- Would this break on a store with zero products in a collection?
- Would this look correct with no image uploaded?
- Would this survive a theme editor session? (section independence, live preview)
- Does every block type render something meaningful?
- Is every schema setting actually wired to output, or are there orphans?
- Would a merchant understand these setting labels?
- Would this break if a merchant reorders blocks?

These are examples — adapt to the specific feature. The point is to think beyond checklists about whether this would actually work in production.

---

## Report

Write the assessment report to `.buildspace/artifacts/{feature-name}/assessment-report.md`.

Read the template from `${CLAUDE_SKILL_DIR}/templates/assessment-report-template.md` and fill it in with findings from all steps.

Combine the output-validator's findings into the Requirements Coverage section.
Combine the code-reviewer's findings into the Standards Compliance section.
Combine the runtime-test results into the Runtime Testing section.
Add your own integration and first-principles findings.

Tell the user:
- Where the report was saved
- Overall verdict
- Count of issues by severity

**Do NOT output the full report in conversation. The artifact file is the source of truth.**

---

## Auto-Capture Learnings

If verdict is **PASS**, check if something non-obvious was learned during this feature's development. Apply the filter: "If I knew this before starting, would it have made the task faster or avoided a mistake?"

If nothing non-obvious → skip. Don't force it.

If learnings exist, append to `.claude/patterns-learned.md` in the project directory:

```markdown
### {Brief Title}
**Type:** Pattern / Mistake & Fix / Convention / Codebase Context / Platform Gotcha
**Category:** Liquid / Sections / Schema / CSS / JavaScript / Architecture
**Date:** {YYYY-MM-DD}

[2-4 sentences: what was learned, why it matters, when to apply it.]
```

Before writing:
- Read existing file to avoid duplicates
- If a new learning conflicts with an existing one, update the existing entry
- Maximum 3 learnings per feature — keep it focused

---

## Next Step

After presenting the report, tell the user:

If verdict is **PASS**:
```
Pipeline complete. Feature is ready for deployment.
```
If verdict is **NEEDS WORK**:
```
→ Run /fix to resolve issues. The assessment report includes root cause analysis for each issue.
```

---

## Rules
- **Always dispatch subagents for Steps 2, 3, 5** — use the `Agent` tool with the exact `subagent_type` specified. Never inline this work by reading files yourself.
- **Never fix issues during assessment** — only identify and report them
- **Think from first principles** — don't just check boxes. Ask "would this actually work?"
- **Be honest** — if the code works, say PASS. Don't invent issues. Don't soften real issues.
- **Root causes, not symptoms** — in the report, explain WHY something is wrong, not just WHAT
- **One pass only** — assess once, report, stop. No loops, no retries.
- **PROJECT standards override generic best practices** — use the skill checklists as authority
