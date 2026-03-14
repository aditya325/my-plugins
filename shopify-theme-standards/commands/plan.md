---
description: Create a detailed execution plan for a task. Use after /clarify when requirements are confirmed. Researches codebase and produces a step-by-step TODO plan.
allowed-tools: Read, Write, Grep, Glob, Task
---

# Plan — Execution Planning

You are entering the Plan phase. Your job is to create a detailed, directionally correct execution plan. Do not write code yet. Plan thoroughly so that building becomes straightforward execution.

## Input
Context or overrides: `$ARGUMENTS`

## Artifact Resolution
1. Look in `.buildspace/artifacts/` for feature folders containing `task-spec.md`
2. If one folder exists → use it
3. If multiple folders exist → ask the user which feature to plan for
4. If no task-spec.md found → ask the user to run `/clarify` first

Read `.buildspace/artifacts/{feature-name}/task-spec.md` as your primary input.
If `design-context.md` exists in the same folder, read it for visual context.

## Process

### Step 1: Read Project Context
Read `CLAUDE.md` if it exists for project overview and conventions.

### Step 2: Research the Codebase
Use the Task tool to dispatch an **Explore** subagent:

> Research the codebase to understand how to implement: [task description from task-spec]
>
> Find:
> 1. Existing files that will need to be modified or that relate to this task
> 2. Existing patterns in the codebase that this task should follow
> 3. Any dependencies, imports, or utilities already available that should be reused
> 4. Potential conflicts or areas that might break
>
> Return a structured summary of your findings.

### Step 3: Assess Knowledge Gaps
Based on the task spec and codebase research, check if there are Shopify-specific topics you are not confident about — unfamiliar APIs, Liquid tags, platform behaviors, or patterns you haven't seen before.

If knowledge gaps exist, present them to the user:

> I'm not confident about [specific topic]. Should I research this before planning?

If the user agrees, use the Task tool to dispatch a research subagent:

> Research the following Shopify topic: [specific topic]
>
> Search shopify.dev first, then github.com/Shopify/dawn, then Shopify Community forums.
> For each promising result, use WebFetch to read the full page — not just search snippets.
> Return a structured summary: what you found, code examples, and any gotchas.

Incorporate findings into the plan. If no gaps, skip this step.

### Step 4: Draft the Execution Plan
Based on the Task Spec, codebase research, and any research findings, draft the plan:

```
## Execution Plan

**Task:** [One-line summary]
**Feature:** {feature-name}
**Estimated Complexity:** [Low / Medium / High]

### Approach
[2-3 sentences explaining the overall approach and WHY this approach was chosen over alternatives. Reference specific codebase patterns discovered during research.]

### Files to Create/Modify
- `path/to/file.liquid` — [what changes and why]
- `path/to/file.css` — [what changes and why]

### TODO Steps
Each TODO must be specific enough to execute without ambiguity.

- [ ] **TODO 1:** [Specific action]
  - Details: [What exactly to do]
  - Files: [Which files this TODO touches]

- [ ] **TODO 2:** [Specific action]
  - Details: [What exactly to do]
  - Files: [Which files this TODO touches]

[Continue for all steps...]

### Risks & Considerations
- [Anything that could go wrong or needs careful attention]

### Validation Checks
- [How to verify each part works correctly after building]
```

### Step 5: Validate the Plan
Before presenting to the user, self-check:
- Are the steps in the right order (dependencies handled)?
- Is anything missing between current state and desired state?
- Is this plan specific enough that building is just execution?

### Step 6: Present and Save
Present the plan to the user. Wait for confirmation or adjustments.

Once confirmed, write the plan to `.buildspace/artifacts/{feature-name}/plan.md`.

## Rules
- Never write implementation code during planning — pseudocode or brief snippets for clarity are acceptable
- Every TODO must be actionable and specific — "implement the feature" is not a TODO
- If the task is too large, suggest breaking it into smaller plans
- Include the WHY behind approach decisions, not just the WHAT
- Do not read skill files — skills are loaded during the build phase when code is being written
- If you find the existing codebase contradicts conventions, flag it to the user
