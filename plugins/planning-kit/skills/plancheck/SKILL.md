---
name: plancheck
description: Review a plan for gaps, risks, and weak verification steps. Tighten it.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Edit
argument-hint: "<slug>"
---

You are in PLAN REVIEW mode. Critique the plan like a senior engineer reviewing an RFC.

Input: slug = $ARGUMENTS
Target: `.claude/plans/<slug>.md`

## 1. Read the Plan

Read `.claude/plans/<slug>.md` in full. Also read `CLAUDE.md` at repo root to understand project patterns.

## 2. Critique

Evaluate each of these dimensions and assign P0/P1/P2 to issues found:

### Scope
- Is the problem statement clear and specific?
- Are In/Out boundaries well-defined?
- Is the scope too big for a single plan? Should it be split?

### Steps
- Are they in the right order? Could any run in parallel?
- Is any step doing too much? (Should be split)
- Are there missing steps between existing ones?

### Verification
- Does EVERY step have a concrete Verify?
- Are Verify steps actual commands/tests, not vague "confirm it works"?
- Does the test plan cover the acceptance criteria?

### Files
- Are all files that will change listed?
- Are there files that SHOULD change but are missing? (Use Grep to check)
- Is the plan consistent with project patterns from CLAUDE.md?

### Risks
- Are risks realistic and mitigations actionable?
- Is there a rollback plan?
- What happens if the plan is partially completed and abandoned?

### Missing pieces
- Any acceptance criteria without a corresponding step?
- Any open questions that block execution?

## 3. Output Issues

```
## Plan Review: <slug>

### P0 — Must fix before approval
- [issue description + suggested fix]

### P1 — Should fix
- [issue description + suggested fix]

### P2 — Nice to have
- [issue description + suggested fix]
```

## 4. Apply Fixes

- Apply P0 and straightforward P1 fixes directly to the plan file
- If a fix would change the plan's scope materially, ask the user first — do not auto-apply
- Add a Decision log entry for each change made: `<date> — plancheck: <what was changed>`

## Rules

- Only edit the plan file
- Do not propose code changes
- Do not approve the plan — that's what `/approve` is for
- Be rigorous but practical — don't invent problems that aren't there
