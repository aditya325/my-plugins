# Shopify Theme Toolkit

A Claude Code plugin for orchestrated Shopify theme development. Supports feature development, bug fixing, assessment, and code understanding through an artifact-based workflow. Version 3.0.0.

**Author:** Aditya Pasikanti

## Installation

### Step 1: Add the Marketplace

Inside Claude Code, run:

```
/plugin marketplace add aditya325/my-plugins
```

### Step 2: Install the Plugin

```
/plugin install shopify-theme-toolkit@my-plugins
```

### Step 3: Verify

Restart Claude Code and type `/shopify-theme-toolkit:clarify` — if it responds, the plugin is working.

## Workflow

### Full Pipeline (Feature Development)

```
/figma → /clarify → /plan → /execute → /compare → /assess → /fix (if needed)
```

Start with `/figma` when building from a Figma design. Skip it if working from text requirements only.

`/assess` automatically runs Playwright-based runtime tests (section rendering, JS errors, accessibility, setting wiring) when `shopify theme dev` is running.

### Standalone Commands

```
/figma         — Extract design context from Figma (via MCP)
/fix           — Bug fixing with first-principles Root Cause Analysis
/assess        — First-principles verification against requirements and standards
/runtime-test  — Playwright runtime tests (also dispatched by /assess)
/compare       — Visual comparison of code vs Figma screenshots
/research      — Shopify topic research
/understand    — Deep code explanation
```

### Use Cases

| Use Case | Entry Point | Flow |
|----------|-------------|------|
| Figma → Feature | `/figma` | /figma → /clarify → /plan → /execute → /compare → /assess → /fix |
| Feature Development | `/clarify` | /clarify → /plan → /execute → /assess → /fix |
| Bug Fixing | `/fix` | standalone with first-principles RCA |
| Assessment | `/assess` | standalone or after /execute (includes runtime tests) |
| Runtime Testing | `/runtime-test` | standalone or auto-dispatched by /assess |
| Visual Comparison | `/compare` | after /execute when Figma screenshots exist |
| Research | `/research` | standalone web search |
| Understand Code | `/understand` | standalone deep trace |

## Skills

### Workflow Skills (user-invoked)

| Skill | Purpose | Input Artifact | Output Artifact |
|-------|---------|----------------|-----------------|
| `/figma` | Extract design context from Figma via MCP | Figma URL(s) | `design-context.md` + screenshots |
| `/clarify` | Define requirements, research, challenge user | User request | `clarify.md` |
| `/plan` | Technical specification with per-file decisions | `clarify.md` | `plan.md` |
| `/execute` | Build all files in-context with full visibility | `plan.md` | code files + `execution-log.md` |
| `/compare` | Visual comparison of code vs Figma screenshots | `selectors.json` + Figma screenshots | `comparison-report.md` |
| `/assess` | First-principles verification (requirements + standards + integration + runtime) | `execution-log.md` + `clarify.md` | `assessment-report.md` |
| `/runtime-test` | Playwright runtime tests — DOM rendering, JS errors, accessibility, setting wiring | `execution-log.md` + `selectors.json` | `runtime-test-results.md` |
| `/fix` | First-principles RCA + fix all instances (waits for approval) | `assessment-report.md` or bug report | `fix-log.md` |
| `/understand` | Deep code explanation | file/section/feature name | conversation output |
| `/research` | Shopify topic research | topic query | conversation output |

### Standard Skills (auto-triggered by Claude)

| Skill | Triggers On |
|-------|------------|
| `liquid-standards` | Any `.liquid` file |
| `section-standards` | Section `.liquid` files and `{% schema %}` blocks |
| `css-standards` | Any `.css` file |
| `js-standards` | Any `.js` file |
| `theme-architecture` | File creation and organization |

## Agents

| Agent | Dispatched By | Model | Role |
|-------|--------------|-------|------|
| `codebase-analyzer` | `/plan` | sonnet | Discovers naming conventions, reusable code, potential conflicts |
| `output-validator` | `/assess` | sonnet | Validates requirements coverage, edge cases, integration |
| `code-reviewer` | `/assess` | sonnet | Reviews code quality against skill checklists |

**Direct build pattern:** `/execute` builds all files directly in the main context with full visibility across files. No agent dispatch during execution — standards are loaded via the Skill tool before each file type.

**Agent-assisted assessment:** `/assess` dispatches `output-validator` for functional checks and `code-reviewer` for standards checks. Verbose agent output stays in forked contexts.

## Artifact Structure

```
.buildspace/
  artifacts/
    {feature-name}/
      design-context.md      <- /figma output (structured design specs)
      clarify.md             <- /clarify output
      plan.md                <- /plan output
      execution-log.md       <- /execute output
      selectors.json         <- /execute output (section->CSS selector map)
      runtime-tests.spec.js  <- /runtime-test output (Playwright test script)
      playwright.config.js   <- /runtime-test output (Playwright config)
      runtime-test-results.md <- /runtime-test output
      screenshots/           <- /figma + /compare output
        figma-{section}-desktop.png
        figma-{section}-mobile.png
        code-{section}-desktop.png
        code-{section}-mobile.png
      comparison-report.md   <- /compare output
      assessment-report.md   <- /assess output
      fix-log.md             <- /fix output
```

Add `.buildspace/` to your project's `.gitignore` — artifacts are working files, not source code.

## Context & Cost Efficiency

This plugin is designed to minimize token usage and API costs:

### Token Optimization
- **Direct build in /execute.** No agent dispatch overhead. Standards loaded once per file type via Skill tool, not per-agent.
- **Single assessment pass.** /assess replaces separate /test + /code-review — one command, one report, no duplicate file reads.
- **Agent-assisted assessment.** output-validator and code-reviewer run in forked contexts. Verbose output stays localized.
- **Externalized artifact templates.** Markdown output templates live in `skills/{name}/templates/` and are loaded via Read on demand.
- **Artifacts replace conversation.** Each stage reads a small, structured artifact file. You can `/clear` between stages.

### Cost Optimization
- **Context fork on /assess, /research.** Verbose agent output and WebFetch results stay in forked context. Only summaries return.
- **disable-model-invocation on workflow skills.** Skill descriptions not loaded for auto-trigger, saving context budget.
- **allowed-tools restriction.** Each skill has only the tools it needs, preventing off-track tool calls.
- **Session boundary guidance.** Each pipeline stage reminds users they can `/clear` between stages since artifacts are the handoff mechanism.

### Standards Authority
- **Plugin standards are the authority** for code patterns, structure, and architecture.
- **Codebase analyzer informs naming only** — file names, setting ID prefixes, CSS class prefixes. Never overrides standards.
- **New code follows standards** even if the existing codebase doesn't. Bad patterns are not replicated.

## Skills & Enforcement

Plugin skill descriptions are loaded into Claude's context, and Claude auto-invokes skills it deems relevant. To guarantee skills are invoked every time, add the following to your project's `CLAUDE.md`:

```markdown
## Project Standards — MANDATORY

Coding standards are provided as **plugin skills** from the `shopify-theme-toolkit` plugin.
They must be invoked using the **Skill tool** before writing any code.

**Before writing or modifying any file, invoke the relevant skill(s):**
- **`.liquid` files** -> invoke `shopify-theme-toolkit:liquid-standards`
- **`.js` files** -> invoke `shopify-theme-toolkit:js-standards`
- **CSS / styling** -> invoke `shopify-theme-toolkit:css-standards`
- **Section files** -> invoke `shopify-theme-toolkit:section-standards`
- **New files / architecture decisions** -> invoke `shopify-theme-toolkit:theme-architecture`

Do not skip this step. The plugin skills have detailed rules and checklists that must be followed.
```

## Theme Check Hook

The plugin includes a PostToolUse hook that auto-runs `shopify theme check` on `.liquid` files after Write/Edit operations. Copy `hooks/hooks.json` into your theme project's `.claude/settings.json` to enable it.

Requires Shopify CLI installed (`npm install -g @shopify/cli`).

## Prerequisites

| What | Required? | Install |
|---|---|---|
| Claude Code | Yes | `npm install -g @anthropic-ai/claude-code` |
| Figma MCP Server | For /figma skill | `claude mcp add --transport http figma https://mcp.figma.com/mcp` |
| Figma Pro+ plan | For /figma skill | Free plan = 6 calls/month; Pro = 200/day |
| Shopify CLI | For theme check hook | `npm install -g @shopify/cli` |
| Node.js 18+ | For screenshot capture | nodejs.org |
| Playwright | For /runtime-test | Auto-installed by the skill if missing |
| shopify theme dev | For /runtime-test | Must be running during runtime tests |
