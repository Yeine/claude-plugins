# Generate Ralph Task Files

Create task files for Ralph, an autonomous AI loop that runs Claude Code repeatedly to complete multi-step improvement tasks. All output goes in a `ralph/` subfolder to keep the repo root clean.

## Arguments

`$ARGUMENTS` optionally specifies the improvement category to skip the interactive question:
- `/ralph-task` — ask the user what kind of improvements they want
- `/ralph-task security` — security audit
- `/ralph-task test-coverage` — add tests for untested code
- `/ralph-task code-quality` — fix lint errors, remove dead code, improve naming
- `/ralph-task architecture` — ensure project patterns are followed consistently
- `/ralph-task tech-debt` — complete TODOs, remove deprecated code, fix warnings
- `/ralph-task custom` — analyze a specific area (ask user for details)

## 1. Determine Improvement Category

If `$ARGUMENTS` is provided and matches a category above, use it directly.

Otherwise, ask the user what kind of improvements they want Ralph to work on:
1. **Security** — find and fix vulnerabilities (injection, auth bypasses, input validation)
2. **Test coverage** — add tests for untested code paths
3. **Code quality** — fix static analysis errors, remove dead code, improve naming
4. **Architecture** — ensure project patterns and conventions are followed
5. **Tech debt** — complete TODOs, remove deprecated code, address warnings
6. **Custom** — user specifies the area to analyze

## 2. Load Project Context

Before analyzing the codebase:
- Read `CLAUDE.md` at the repo root if it exists — note architecture patterns, conventions, and testing standards
- Identify the project's language(s), framework, and test runner
- Note these for use in task guidance and verification commands

## 3. Analyze the Codebase

Search systematically based on the chosen category:

**Security:**
- Grep for raw SQL queries, unescaped user input, hardcoded credentials
- Search for missing authorization checks, open endpoints
- Look for injection vectors (SQL, XSS, command, path traversal)

**Test coverage:**
- Identify service classes, controllers, and utilities that lack corresponding test files
- Check existing test patterns to understand what the project's tests look like
- Prioritize code with complex logic or high business impact

**Code quality:**
- Run the project's linter/static analysis if available (check Makefile, package.json, composer.json for commands)
- Search for TODO/FIXME/HACK comments, dead code, duplicated logic
- Look for overly broad exception handling

**Architecture:**
- Compare implementation against CLAUDE.md patterns
- Find code that bypasses the project's established conventions
- Identify business logic in wrong layers (e.g., logic in controllers/handlers)

**Tech debt:**
- Grep for TODO, FIXME, DEPRECATED, @deprecated
- Search for commented-out code blocks
- Identify unused imports, variables, or functions

**Custom:**
- Ask the user to describe the specific area, then search accordingly

## 4. Setup and Generate Files

### 4a. Create the ralph/ directory
```bash
mkdir -p ralph
```

### 4b. Add to .gitignore
Check if `ralph/` is already in `.gitignore`. If not, append it:
```
###> ralph ###
ralph/
###< ralph ###
```

### 4c. Write `ralph/IMPROVEMENT_TASKS.md`

Use this format:

````markdown
# Ralph Improvement Loop — Task List

## How This File Works
1. Ralph picks the first unchecked `[ ]` task
2. Implements the fix following the guidance
3. Runs verification
4. Marks the task `[x]`
5. Moves to the next task

---

## CRITICAL PRIORITY

### Task 1: [Specific task name]
- [ ] **Status:** Not Started

**File:** `path/to/file`

**Problem:** Clear description of the issue

**Solution:** Step-by-step fix with code patterns to follow

**Verification:**
```
[exact command to verify the fix — tests, linter, grep, etc.]
```

---

## HIGH PRIORITY

### Task N: [...]
...

---

## Progress Tracking

| Priority | Total | Completed | Remaining |
|----------|-------|-----------|-----------|
| Critical | X     | 0         | X         |
| High     | X     | 0         | X         |
| Medium   | X     | 0         | X         |
| **Total**| **X** | **0**     | **X**     |
````

**Rules for tasks:**
- Each task must be specific and self-contained — completable in one Ralph iteration
- Include exact file paths
- Provide code patterns to follow (reference existing project code, not invented examples)
- Include a concrete verification command
- Order by priority: Critical > High > Medium > Low
- Aim for 5-20 tasks per run (too many = state file becomes unwieldy)

### 4d. Write `ralph/RALPH_TASK.md`

Use this format, filling in the `[type]` and `[verification command]` based on the project context discovered in step 2:

````markdown
# Ralph [Type] Loop

You are an autonomous [type] agent. Your progress is tracked in `ralph/IMPROVEMENT_TASKS.md`.

**CRITICAL: Do ONE task per run, then STOP.**

## Phase 1: Read State
Read `ralph/IMPROVEMENT_TASKS.md` and find the FIRST unchecked `[ ]` task.
**Output:** `PICKING: <task name>`

If no unchecked tasks remain, skip to EXIT.

## Phase 2: Implement
Make the changes described in the task guidance.
**Output:** `IMPLEMENTING: <description>`

## Phase 3: Verify
Run the verification command specified in the task.
**Output:** `TESTING: <command>`

If verification fails, attempt to fix. If still failing after a reasonable effort:
**Output:** `ATTEMPT_FAILED: <task name>`
Then STOP. Do not move to the next task.

## Phase 4: Mark Complete
Edit `ralph/IMPROVEMENT_TASKS.md`:
- Change the task's `[ ]` to `[x]`
- Update the Progress Tracking table (increment Completed, decrement Remaining)

**Output:** `MARKING COMPLETE: <task name>`

## STOP
**Output:**
```
DONE: <task name>
REMAINING: <count> tasks
```

## EXIT
If no unchecked tasks remain:
**Output:** `EXIT_SIGNAL: true`
````

## 5. Present the Run Command

After generating both files, tell the user:

```
Files created:
  ralph/RALPH_TASK.md         — prompt file
  ralph/IMPROVEMENT_TASKS.md  — task checklist

Run with:
  bin/ralph -p ralph/RALPH_TASK.md

Options:
  bin/ralph -p ralph/RALPH_TASK.md --caffeinate --log    # long-running, with log
  bin/ralph -p ralph/RALPH_TASK.md -j 4                  # 4 parallel workers
  bin/ralph -p ralph/RALPH_TASK.md --max 10              # cap at 10 iterations
```
