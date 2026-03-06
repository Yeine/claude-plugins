---
name: execute
description: Execute an approved plan step by step, verifying each step before proceeding.
argument-hint: "<slug> [--step N] [--continue] [--polish]"
---

You are in EXECUTION mode. Follow the plan precisely — do not improvise.

Input: slug = $0
Options parsed from $ARGUMENTS:
- `--step N` — start from step N (skip earlier steps, assume already done)
- `--continue` — resume from the first step not yet logged as done in the Decision log (reads checkpoint file if available for full context)
- `--polish` — run a frontend polish pass after completion (auto-prompted when frontend files are changed)

Target: `.claude/plans/<slug>.md`

## 1. Load and Validate

Read `.claude/plans/<slug>.md`.

**Gate checks — stop if any fail:**
- Status MUST be `Approved`. If `Draft` → tell user to run `/approve <slug>` first. If `Completed` → tell user plan is already done.
- No blocking Open questions.

Read `CLAUDE.md` at repo root if it exists — follow project conventions during implementation.

## 2. Permission Pre-flight

Before executing anything, analyze ALL steps in the plan to predict what permissions will be needed. Scan each step's **Change** and **Verify** sections.

Build a permission inventory:
- **File operations** — which files/directories will be created, modified, or deleted
- **Bash commands** — what shell commands will run (test suites, linters, build tools, package managers)
- **Git operations** — commits, branch operations
- **External tools** — API calls, database commands, Docker, etc.

Present the permission summary upfront:
```
Permission pre-flight — .claude/plans/<slug>.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

This plan will need permission to:

  Files:   Read/Write/Create in src/, tests/, config/
  Bash:    npm test, npm run lint, npm run build
  Git:     (none during execution)

  Steps: N total, estimated ~X file edits, ~Y shell commands

Approve all permissions for this execution run? (y/n)
```

If approved, proceed with all operations without further prompting for the categories listed.

**Hard deny list — NEVER auto-approve, always ask individually:**
- `rm -rf` on directories outside the project
- `git push`, `git reset --hard`, force operations
- Anything touching `~/.ssh`, `~/.aws`, `~/.config`, `/etc`, or system directories
- `DROP`, `DELETE`, `TRUNCATE` on databases
- `docker rm`, `docker system prune`
- Any command with `sudo`
- Sending messages or making external API calls (Slack, email, GitHub issues)

If an unexpected permission need arises mid-execution (not predicted in pre-flight), pause and ask for that specific permission before continuing.

## 3. Determine Starting Step

- Default: Step 1
- `--step N`: Start at step N
- `--continue`: Scan the Decision log for entries like `Step N — done` or `Step N — verified`. Start at the first step without such an entry. Also read `.claude/plans/<slug>.checkpoint.md` if it exists for detailed resume context.

Print:
```
Executing: .claude/plans/<slug>.md
Starting at: Step N of M
```

## 4. Execute Each Step

For each step in order:

### 4a. Announce
```
━━━ Step N/M: <step title> ━━━
```

### 4b. Implement
- Read the **Change** description
- Implement exactly what it says — no more, no less
- Follow project patterns from CLAUDE.md
- If the step is ambiguous, stop and ask the user rather than guessing

### 4c. Verify
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

### 4d. Checkpoint
After each verified step, check:
- Did the change introduce something unexpected (new test failures, lint errors)?
- If yes → stop and report before continuing

### 4e. Context Watch — CRITICAL

**Goal: NEVER let the system compact the conversation.** Compaction is slow, lossy, and unnecessary — the plan file already holds all the state we need.

**Track a step counter** starting at 0 when execution begins (or when resuming with `--continue`). Increment after each verified step.

**Checkpoint trigger — when the counter reaches 3 steps in this session, ALWAYS checkpoint.** Do not evaluate whether you "feel" like context is fine. 3 steps is the hard limit. This is non-negotiable.

Why 3: each step involves reading files, editing, running verification commands, and logging. That accumulates fast. 3 steps is the safe ceiling before context pressure builds.

**Checkpoint procedure:**

1. **Write checkpoint file** to `.claude/plans/<slug>.checkpoint.md`:
   ```markdown
   # Checkpoint: <slug>
   ## Session ended after
   Step N: <title> — verified

   ## Resume at
   Step N+1: <title>

   ## Next step details
   <Copy the FULL Change and Verify sections for step N+1 from the plan>

   ## Decisions & deviations
   - <any decisions made during execution that affect remaining steps>
   - <any files modified differently than the plan expected>
   - <any workarounds applied>

   ## Remaining
   Steps N+1 through M:
   - Step N+1: <title>
   - Step N+2: <title>
   - ...

   ## Permissions (pre-approved)
   <Copy the permission inventory so the next session skips the approval prompt>
   ```

2. **Log** in the Decision log: `<date> — Context checkpoint after Step N — continuing in next session`

3. **Print:**
   ```
   Checkpoint saved — 3 steps completed this session.

   Continue immediately:
     /execute <slug> --continue

   Progress: N/M steps verified
   Next up: Step N+1 — <title>
   ```

4. **STOP.** Do not execute another step. Do not say "let me try one more." Stop.

**On resume with `--continue`:**
- Read `.claude/plans/<slug>.checkpoint.md` FIRST — it has everything needed
- Read the plan file for the remaining step details
- Re-read `CLAUDE.md` for project conventions
- Skip the permission pre-flight if the checkpoint includes pre-approved permissions — just print: `Resuming with previously approved permissions.`
- Reset the step counter to 0 — you get another 3 steps before the next checkpoint
- Delete the previous checkpoint file before writing a new one (keep only the latest)

## 5. Run Test Plan

After all steps pass:
- Execute the **Test plan** commands from the plan
- If tests pass: log `<date> — Test plan — passed ✓`
- If tests fail: log `<date> — Test plan — FAILED: <summary>`, do NOT mark as completed

## 6. Check Acceptance Criteria

Review each acceptance criterion in the plan:
- Check the box `[x]` if satisfied
- If any criterion is NOT met, list what's missing and stop

## 7. Complete

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

## 8. Frontend Polish

**Trigger:** runs when `--polish` is passed. If `--polish` is NOT passed, auto-detect by checking if any changed files match frontend patterns (`.tsx`, `.jsx`, `.vue`, `.svelte`, `.html`, `.css`, `.scss`, component/page directories). If frontend files were touched, ask:
```
Frontend files were changed. Run UI/UX polish pass? (y/n)
```
Skip this phase entirely if the user declines or no frontend files were touched.

When triggered:

### 8a. Detect scope
- List all files created or modified during execution
- Identify new pages/routes added by the plan
- Identify existing navigation components (navbars, sidebars, menus, routers)

### 8b. Polish UI/UX
Use the `/frontend-design` skill with the following prompt:

> Improve the UI and UX of the pages touched by this plan. Focus on:
> - Visual consistency with the existing design system
> - Responsive layout, spacing, and typography
> - Interactive states (hover, focus, loading, empty, error)
> - Accessibility basics (contrast, labels, keyboard nav)

### 8c. Ensure navigation
- Verify that every new page/route added by the plan is reachable from the existing frontend navigation (sidebar, navbar, menu, router config)
- If any new page is orphaned (not linked from anywhere), add the appropriate navigation entry
- Run the app or relevant tests to confirm the links work

### 8d. Log
- Log `<date> — Frontend polish — done` in the Decision log

## Rules

- Follow the plan exactly. Do not add features, refactor surrounding code, or "improve" things not in the plan.
- If reality diverges from the plan (e.g., a file doesn't exist, an API changed), STOP and tell the user. Suggest running `/plancheck <slug>` to update the plan.
- One step at a time. Do not batch or skip steps.
- Every code change must be tied to a specific step in the plan.
- Update the Decision log as you go — it's the audit trail.
- Do not combine the execute skill with other skills in the same invocation.
- When resuming with `--continue`, read the checkpoint file first — it has the context you need without re-reading the whole conversation.
- Delete the checkpoint file (`.claude/plans/<slug>.checkpoint.md`) when the plan reaches Completed status.
- Prefer checkpointing early over running out of context. A clean handoff is always better than a degraded session.
