# Test-Driven Bug Fix

When the user reports a bug, follow this exact process. Do NOT write any fix until steps 1-3 are complete.

## 1. Trace the Root Cause

- Read the relevant code path end-to-end (route/handler → service → data layer)
- Identify the **exact line** where incorrect behavior originates
- Present a root cause diagnosis with file paths and line numbers before proceeding
- If the issue spans multiple layers, identify ALL contributing points — do not stop at the first symptom

## 2. Write a Failing Test

- Write a focused test (unit or integration, whichever is more appropriate) that reproduces the broken behavior
- Run the test and confirm it **fails for the right reason**
- If the test passes unexpectedly, re-examine your root cause analysis — your understanding is wrong

## 3. Check Impact

- Search the codebase for all usages of the functions/methods you plan to change
- List every caller or dependent that could be affected by the fix
- If the fix touches shared utilities, validation logic, or base classes, flag this explicitly

## 4. Implement the Fix

- Make the minimal change that addresses the root cause
- Do not refactor surrounding code unless it is part of the bug

## 5. Validate

- Run the new test — it must pass
- Run the full related test suite to check for regressions
- If anything fails, iterate on the fix (do not skip failing tests)

## 6. Report

Only present the solution once ALL tests pass. Include:
- **Root cause**: what was wrong and why
- **Fix**: what changed and why
- **Test added**: what the new test covers
- **Impact**: any callers or dependents you verified still work
