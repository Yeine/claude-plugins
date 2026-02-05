---
name: approve
description: Validate and freeze a plan by setting Status to Approved.
disable-model-invocation: true
allowed-tools: Read, Edit
argument-hint: "<slug>"
---

You are in APPROVAL mode.

Input: slug = $ARGUMENTS
Target: `.claude/plans/<slug>.md`

## 1. Read and Validate

Read the plan file and check this checklist:

- [ ] Has a clear Problem statement
- [ ] Has Scope with In and Out sections
- [ ] Has Acceptance criteria (at least 3 testable items)
- [ ] Has 3-7 steps
- [ ] Every step has a concrete Verify (command or test, not vague)
- [ ] Has a Test plan with actual commands
- [ ] Has a Rollback plan
- [ ] Has no unresolved P0 issues (check Decision log for plancheck entries)
- [ ] Has no blocking Open questions

## 2. Fix Gaps

If anything from the checklist is missing:
- Add the minimum needed to satisfy it
- Add a Decision log entry: `<date> — approve: added missing <section>`

If multiple items are missing or the plan has fundamental issues, do NOT approve. Instead report what's wrong and suggest running `/plancheck` first.

## 3. Approve

- Set `Status: Draft` to `Status: Approved`
- Append this section at the end of the plan:

```markdown
## Execution notes
- Implement one step at a time in order.
- After each step, run its Verify and record the result in the Decision log.
- If reality diverges from the plan, stop and run `/planfile` or `/plancheck` to update before continuing.
- Do not skip steps or combine steps without updating the plan first.
```

- Add a Decision log entry: `<date> — Plan approved`

## 4. Confirm

Print:

```
Plan approved — .claude/plans/<slug>.md
Status: Approved
Steps: N
Ready for execution.
```

## Rules

- Only edit the plan file
- Do not start implementation
- Do not write code
- If the plan is not ready, say so clearly — do not rubber-stamp
