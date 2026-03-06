---
name: postmortem
description: Post-execution review — architecture drift, tech debt, test gaps, and lessons learned.
argument-hint: "<slug>"
---

You are in REVIEW mode. Your job is structured critique, not implementation.

Input: slug = $0

Target: `.claude/plans/<slug>.md`

## 1. Load Context

Read `.claude/plans/<slug>.md`.

**Gate check:** Status MUST be `Completed`. If not, tell the user to finish execution first.

Read `CLAUDE.md` at repo root if it exists — you'll need project conventions for context.

Gather the full changeset:
- Run `git log --oneline` to find the commits associated with this plan (use the Decision log timestamps to identify the range)
- Run `git diff <base>..HEAD` to get the complete diff
- Note the list of files added, modified, and deleted

Print:
```
Post-mortem: .claude/plans/<slug>.md
Reviewing <N> files changed, +<added> -<removed> lines
```

## 2. Architecture Review

Compare the implementation against the plan. Answer these questions:

- **Plan drift:** Did the implementation diverge from the plan? If so, where and why?
- **Abstraction quality:** Are the abstractions at the right level? Anything too abstract (overengineered) or too concrete (will need immediate refactoring)?
- **Separation of concerns:** Are responsibilities cleanly divided? Any god-files or mixed concerns?
- **API design:** Are interfaces, function signatures, and data shapes clean and consistent with the rest of the codebase?

Rate: `Clean` | `Minor drift` | `Significant drift`

## 3. Technical Debt Detection

Scan the changeset for:

- **Duplication:** Code that repeats logic already present elsewhere in the codebase
- **Missing abstractions:** Inline logic that should be extracted (but only if used 3+ times or is complex enough to warrant it)
- **Fragile logic:** Hardcoded values, implicit assumptions, tight coupling, stringly-typed patterns
- **Temporary workarounds:** Code comments with TODO/FIXME/HACK that were introduced by this plan
- **Inconsistencies:** Naming conventions, patterns, or approaches that don't match the rest of the codebase

For each item found, classify:
- `P0` — should fix before merging
- `P1` — fix soon (next sprint)
- `P2` — track for later

## 4. Test Coverage Analysis

Review the tests added or modified by the plan:

- **Untested code paths:** Identify branches, error paths, or edge cases with no test coverage
- **Missing edge cases:** Null/empty inputs, boundary values, concurrent access, error responses
- **Integration gaps:** Are components tested in isolation but not together?
- **Test quality:** Are tests actually asserting meaningful behavior, or just checking that code runs without crashing?

List specific test cases that should be added, grouped by priority.

## 5. Future Improvements

Based on the full review, suggest:

- **Refactor opportunities:** Things that work now but would benefit from restructuring
- **Performance:** Obvious N+1 queries, unnecessary re-renders, missing caching, unoptimized loops
- **Design simplifications:** Complexity that could be reduced without losing functionality
- **Missing features:** Logical next steps that the plan didn't cover but the architecture now supports

Keep this list short and actionable — max 5 items, ranked by impact.

## 6. Lessons Learned

Identify patterns worth remembering for future plans:

- What went smoothly and why?
- What required unexpected fixes during execution?
- Were any steps in the plan under-specified or wrong?
- Did any assumptions in the plan turn out to be incorrect?

## 7. Report

Print the full post-mortem report:

```
Post-mortem — .claude/plans/<slug>.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Architecture:  <Clean | Minor drift | Significant drift>
Tech debt:     <N> items (P0: <n>, P1: <n>, P2: <n>)
Test gaps:     <N> missing test cases
Improvements:  <N> suggestions

━━━ Architecture ━━━
<findings>

━━━ Technical Debt ━━━
<P0 items first, then P1, then P2>

━━━ Test Gaps ━━━
<missing tests by priority>

━━━ Improvements ━━━
<ranked suggestions>

━━━ Lessons Learned ━━━
<key takeaways>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 8. Update Knowledge (optional)

If the project has a `.claude/` directory, check if a `knowledge/` subdirectory exists.

If it does, append lessons learned to the relevant files:
- `.claude/knowledge/patterns.md` — reusable patterns discovered
- `.claude/knowledge/decisions.md` — architecture decisions made and why
- `.claude/knowledge/pitfalls.md` — mistakes to avoid

Create these files only if the directory already exists. Do not create the `knowledge/` directory itself — that's the user's choice.

Ask the user before writing:
```
Save lessons to .claude/knowledge/? (y/n)
```

## 9. Log

Append to the plan's Decision log:
- `<date> — Post-mortem completed — <Clean|Minor drift|Significant drift>, <N> debt items, <N> test gaps`

## Rules

- Do NOT make code changes. This is a review-only skill.
- Be specific — reference file names, line numbers, and function names.
- Be honest but constructive. Flag real issues, not style preferences.
- Keep the report actionable. Every finding should have a clear next step.
- Do not repeat findings from the plan's own test plan — focus on what the plan missed.
