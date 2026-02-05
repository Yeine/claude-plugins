---
name: planfile
description: Create or update a versioned implementation plan at .claude/plans/<slug>.md.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Edit, Bash(mkdir -p .claude/plans)
argument-hint: "<slug> <goal>"
---

You are in PLANNING mode. Do NOT edit any file except the plan file. Do NOT write code.

Inputs:
- slug = $0 (required, kebab-case)
- goal = everything after the slug in $ARGUMENTS

If slug is missing or looks invalid (contains spaces, uppercase, or special chars):
- Propose a kebab-case slug and ask the user to re-run. Do not edit files yet.

## 1. Setup

```bash
mkdir -p .claude/plans
```

Target file: `.claude/plans/<slug>.md`

If the file already exists, read it and preserve:
- Completed checklist items `[x]`
- Decision log entries
- Any sections marked as final

## 2. Load Context

- Read `CLAUDE.md` at repo root if it exists — note architecture patterns, conventions, and testing standards
- Check if `.claude/plans/<slug>-intake.md` exists (from `/intake`) — incorporate its acceptance criteria and decisions
- Grep/read relevant source files to understand the current state of what the plan will change

## 3. Write the Plan

Write or update `.claude/plans/<slug>.md` using this exact structure:

```markdown
# Plan: <slug> — <goal>

Date: <today>
Status: Draft

## Problem statement
What problem does this solve? Why now?

## Scope
### In
- What IS included in this change

### Out
- What is explicitly NOT included

## Acceptance criteria
(Copy from intake if available, otherwise define here)
- [ ] Criterion 1
- [ ] Criterion 2

## Constraints
Technical, time, or business constraints.

## Assumptions
What are we assuming to be true?

## Files likely to change
- `path/to/file` — why this file needs changes

## Step-by-step plan

### Step 1: <title>
- **Change:** What will change
- **Verify:** Exact command or check to confirm this step worked

### Step 2: <title>
- **Change:** What will change
- **Verify:** How to verify

(3-7 steps total)

## Test plan
How to verify the entire change works end-to-end. Include specific commands.

## Rollback plan
How to undo this change if something goes wrong.

## Risks and mitigations
| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|

## Open questions
Anything still unresolved.

## Decision log
- <date> — <decision made>
```

## 4. Summarize

After writing the plan, print a 5-10 bullet summary of what the plan covers and why.

## Rules

- Keep to 3-7 steps. If you need more, the scope is too big — split it.
- Every step MUST have a concrete Verify (a command, a test, or a specific check — not "verify it works")
- Do NOT edit any file except the plan file
- Do NOT write code
- Reference existing project patterns from CLAUDE.md when deciding approach
