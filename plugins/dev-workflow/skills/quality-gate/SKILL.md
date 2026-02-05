# Quality Gate

Run all project quality checks in sequence and report pass/fail. Discovers which tools are available from the project's own configuration.

## Arguments

`$ARGUMENTS` optionally limits the scope:
- `/quality-gate` — run all discovered checks
- `/quality-gate --fix` — auto-fix what can be fixed (linter, formatter), then report remaining issues
- `/quality-gate --changed` — only check files changed on the current branch (faster)

## 1. Discover Quality Tools

Check these sources in order to build the list of available checks:

**CLAUDE.md** — read `CLAUDE.md` at repo root. Look for quality/testing commands. This is the most reliable source since it documents the project's actual workflow.

**Makefile** — check for quality-related targets:
```bash
make -qp 2>/dev/null | grep -E '^[a-zA-Z_-]+:' | grep -iE 'test|lint|check|stan|cs-fix|format|type'
```

**package.json** — check for scripts:
- `lint`, `typecheck`, `test`, `format`, `check`

**composer.json** — check for scripts or dev dependencies:
- `phpstan`, `php-cs-fixer`, `phpunit`, `psalm`

**Config files** — presence of these indicates available tools:
- `.eslintrc*`, `eslint.config.*` → ESLint
- `tsconfig.json` → TypeScript type check
- `phpstan.neon*` → PHPStan
- `.php-cs-fixer*` → PHP CS Fixer
- `pytest.ini`, `pyproject.toml` → pytest
- `rustfmt.toml`, `clippy.toml` → Rust tools

Build an ordered checklist of checks to run.

## 2. Determine Scope

If `--changed` flag is set, get the list of changed files:
```bash
git diff --name-only $(git merge-base HEAD origin/$(git remote show origin | sed -n '/HEAD branch/s/.*: //p'))..HEAD
```

Where possible, pass only these files to the quality tools (many linters support file arguments).

## 3. Run Checks in Order

Run checks in this priority order (skip any that aren't available):

1. **Formatter / Code Style** (fastest, auto-fixable)
   - If `--fix`: run the fixer, then check for remaining issues
   - If no `--fix`: run in dry-run/check mode only

2. **Static Analysis / Type Checking** (catches bugs without running code)

3. **Unit Tests** (fast tests first)

4. **Integration Tests** (slower, run last)

For each check, run the command and capture the output.

## 4. Report Results

Present results as they complete. Final summary:

```
## Quality Gate Results

| Check | Status | Details |
|-------|--------|---------|
| Code Style | PASS | 0 issues |
| Static Analysis | FAIL | 3 errors |
| Unit Tests | PASS | 142 tests, 0 failures |
| Integration Tests | SKIP | (not run — use full mode) |

### Failures

**Static Analysis — 3 errors:**
- `src/Service/Foo.php:42` — Parameter $x has no type hint
- `src/Service/Foo.php:58` — Method bar() should return int but returns string
- `src/Controller/BazController.php:15` — Unused import App\Entity\Qux

### Verdict: FAIL
Fix the 3 static analysis errors before pushing.
```

## 5. Offer to Fix (if failures found)

If there are failures, ask:

> Would you like me to fix the issues that can be auto-fixed?

Auto-fixable: code style, unused imports, simple type issues.
Not auto-fixable: logic errors, failing tests, complex type mismatches — just report these.

## Rules

- Never skip a check silently — if a tool is found but fails to run, report the error
- Run checks in order from fastest to slowest
- Do not modify any files unless `--fix` was specified or the user approves
- Report exact file paths and line numbers for every failure
- If all checks pass, just say "All checks passed" — don't over-explain
