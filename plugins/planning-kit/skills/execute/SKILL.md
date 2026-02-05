---
name: execute
description: Execute an approved plan step by step, verifying each step before proceeding.
argument-hint: "<slug> [--step N] [--continue]"
---

You are in EXECUTION mode. Follow the plan precisely — do not improvise.

Input: slug = $0
Options parsed from $ARGUMENTS:
- `--step N` — start from step N (skip earlier steps, assume already done)
- `--continue` — resume from the first step not yet logged as done in the Decision log

Target: `.claude/plans/<slug>.md`

## 1. Load and Validate

Read `.claude/plans/<slug>.md`.

**Gate checks — stop if any fail:**
- Status MUST be `Approved`. If `Draft` → tell user to run `/approve <slug>` first. If `Completed` → tell user plan is already done.
- No blocking Open questions.

Read `CLAUDE.md` at repo root if it exists — follow project conventions during implementation.

## 2. Determine Starting Step

- Default: Step 1
- `--step N`: Start at step N
- `--continue`: Scan the Decision log for entries like `Step N — done` or `Step N — verified`. Start at the first step without such an entry.

Print:
```
Executing: .claude/plans/<slug>.md
Starting at: Step N of M
```

## 3. Execute Each Step

For each step in order:

### 3a. Announce
```
━━━ Step N/M: <step title> ━━━
```

### 3b. Implement
- Read the **Change** description
- Implement exactly what it says — no more, no less
- Follow project patterns from CLAUDE.md
- If the step is ambiguous, stop and ask the user rather than guessing

### 3c. Verify
- Run the **Verify** command or check described in the step
- If verification **passes**: log `<date> — Step N — verified ✓` in the Decision log
- If verification **fails**:
  1. Attempt ONE fix based on the error output
  2. Re-run verify
  3. If it fails again → stop execution, log `<date> — Step N — BLOCKED: <reason>`, update Status to `Blocked`, and print:
     ```
     BLOCKED at Step N: <step title>
     Reason: <what failed>
     Options:
       /plancheck <slug>    — review and adjust the plan
       /execute <slug> --continue  — resume after fixing manually
     ```

### 3d. Checkpoint
After each verified step, check:
- Did the change introduce something unexpected (new test failures, lint errors)?
- If yes → stop and report before continuing

## 4. Run Test Plan

After all steps pass:
- Execute the **Test plan** commands from the plan
- If tests pass: log `<date> — Test plan — passed ✓`
- If tests fail: log `<date> — Test plan — FAILED: <summary>`, do NOT mark as completed

## 5. Check Acceptance Criteria

Review each acceptance criterion in the plan:
- Check the box `[x]` if satisfied
- If any criterion is NOT met, list what's missing and stop

## 6. Complete

When all steps verified + test plan passed + acceptance criteria met:

- Set `Status: Approved` to `Status: Completed`
- Log `<date> — Plan completed`
- Print:
  ```
  Plan completed — .claude/plans/<slug>.md
  Steps: N/N verified
  Test plan: passed
  Acceptance criteria: all met
  ```

## Rules

- Follow the plan exactly. Do not add features, refactor surrounding code, or "improve" things not in the plan.
- If reality diverges from the plan (e.g., a file doesn't exist, an API changed), STOP and tell the user. Suggest running `/plancheck <slug>` to update the plan.
- One step at a time. Do not batch or skip steps.
- Every code change must be tied to a specific step in the plan.
- Update the Decision log as you go — it's the audit trail.
- Do not combine the execute skill with other skills in the same invocation.
