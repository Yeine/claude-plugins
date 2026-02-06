# Deep Code Review

Perform a rigorous, thorough code review of the current branch against its base branch. Review EVERY changed file and produce a structured report with actionable findings.

## Arguments

`$ARGUMENTS` optionally specifies the base branch:
- `/deep-review` — auto-detect base branch
- `/deep-review develop` — use `develop` as the base
- `/deep-review --base=main` — use `main` as the base

## 1. Determine Base Branch and Scope

If `$ARGUMENTS` is provided and non-empty, extract the branch name (strip any `--base=` prefix) and use it as `BASE`.

Otherwise, auto-detect:
```bash
git remote show origin | sed -n '/HEAD branch/s/.*: //p'
```
If that fails, fall back to the first local branch matching `develop`, `main`, or `master` (in that order).

Run these in parallel:
```bash
git branch --show-current
git diff $BASE..HEAD --name-status
git diff $BASE..HEAD --stat
git log $BASE..HEAD --oneline
```

Record: current branch, list of changed files with status (A/M/D/R), total line counts, and commit history.

## 2. Load Project Conventions

Before analyzing code, check for project-level instructions:
- Read `CLAUDE.md` at the repository root if it exists
- Read `.claude/CLAUDE.md` if it exists
- Note architecture patterns, naming conventions, security requirements, and testing standards

Use these conventions as additional review criteria throughout the analysis.

## 3. Deep Analysis

Scale the review strategy based on diff size:

- **1-5 files:** Review directly in context. Do not spawn sub-agents.
- **6-15 files:** Spawn 2 sub-agents, splitting files evenly.
- **16+ files:** Spawn 3 sub-agents, splitting files evenly.

For each sub-agent, use the Task tool with `subagent_type="general-purpose"` and provide:

```
You are performing a deep code review. Be thorough and rigorous.

Base Branch: {BASE}

Files to Review:
{list of files assigned to this agent}

Project Conventions:
{summary of relevant conventions from CLAUDE.md, or "None found"}

## Instructions

For EACH file:

1. Read the FULL current file content using the Read tool
2. Read the diff: `git diff {BASE}..HEAD -- {filepath}`
3. Analyze against ALL six categories below

### A. Correctness and Logic
- Does the code do what it intends?
- Logic errors, missed edge cases, off-by-one errors?
- Null/undefined/empty handling?
- Race conditions or timing issues?

### B. Security
- Input validation present where needed?
- No hardcoded secrets or credentials?
- Protection against injection (SQL, XSS, command, path traversal)?
- Proper authentication and authorization checks?

### C. Performance
- Unnecessary loops, redundant operations, or re-computation?
- N+1 query problems?
- Memory leaks (unclosed resources, growing collections)?
- Efficient algorithms and data structures?

### D. Code Quality
- Follows existing project patterns and conventions?
- Proper error handling and propagation?
- No dead code or commented-out code?
- Clear naming, single responsibility, no duplication?

### E. Maintainability
- Readable and self-documenting?
- Complex logic has comments explaining WHY?
- Appropriate level of abstraction?

### F. Testing Considerations
- Are new code paths testable?
- Should new tests accompany these changes?
- Are edge cases covered by existing tests?

## Output Format

For each file:

### {filepath}
**Status:** Clean | Has Issues

**Critical Issues:**
- `line:XX` - {description} -- {suggested fix}

**Warnings:**
- `line:XX` - {description} -- {suggested fix}

**Suggestions:**
- `line:XX` - {description}

Omit empty severity sections. If a file is clean, just report Status: Clean.
At the end, list every file you reviewed to confirm completeness.
```

When reviewing directly (small diffs), follow the same analysis criteria without spawning agents.

## 4. Completeness Check

After all reviews complete, compare the set of reviewed files against the full changed-file list from step 1. If any files are missing, review them directly before proceeding.

## 5. Cross-File Analysis

Perform these checks across the entire diff:

1. **Consistency** — Do changes across files use the same patterns, naming, and error handling? Are similar problems solved the same way?
2. **Integration** — Do the changed components work together? Are interfaces and contracts between them compatible?
3. **Missing changes** — Are there files that SHOULD have been changed but were not? Missing tests, configuration, migrations?
4. **Dependencies** — Any circular dependencies introduced? Are new imports valid? Are new external dependencies justified?

Use Grep to search the codebase when verifying integration points, callers, or missing changes. Do not guess — check.

## 6. Generate Report

Compile all findings into this format:

```
## Deep Code Review Report

**Branch:** {current} -> {BASE}
**Files Changed:** {N} | **Commits:** {M}
**Review Date:** {date}

---

### Critical Issues (Must Fix)
Issues that will cause bugs, security vulnerabilities, or data loss.

- `path/to/file:42` - {description} -- {fix}

### Warnings (Should Fix)
Issues that may cause problems or violate best practices.

- `path/to/file:87` - {description} -- {fix}

### Suggestions (Nice to Have)
Improvements that would make the code better but are not blocking.

- `path/to/file:123` - {description}

### Cross-File Concerns
- {description}

### Questions for Author
- {question}

---

### Files Reviewed
- [x] `path/to/file1` - 2 critical, 1 warning
- [x] `path/to/file2` - Clean
- [x] `path/to/file3` - 1 suggestion

---

### Summary
{2-3 sentences on overall quality}

**Recommendation:** APPROVE | REQUEST CHANGES | NEEDS DISCUSSION

{If REQUEST CHANGES: list the critical issues that must be addressed}
```

## Severity Guidelines

**Critical** (must fix before merge):
- Security vulnerabilities
- Data loss or corruption risks
- Crashes or unhandled exceptions in mainline paths
- Logic errors that produce wrong results
- Breaking changes without migration path

**Warning** (should fix):
- Performance problems
- Missing error handling
- Violations of project conventions
- Missing input validation
- Hard-to-maintain code

**Suggestion** (nice to have):
- Better naming or structure
- Additional documentation
- Refactoring opportunities
- Test coverage improvements

## Rules

- Review EVERY changed file — no shortcuts
- Read full file context, not just diffs
- Always include file paths and line numbers
- Every issue must have a clear, actionable fix
- No false positives — only report real problems, not style preferences
- Understand the author's intent before criticizing
- If you find one bug pattern, search for similar ones elsewhere
- Consider edge cases: empty collections, null values, concurrent access, failures
