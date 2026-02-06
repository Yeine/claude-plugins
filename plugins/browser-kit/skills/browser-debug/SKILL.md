# Browser Bug Investigation

Given a bug description, use Playwright to navigate the app, reproduce the issue, capture diagnostic data (console errors, network failures, JS exceptions, screenshots), trace the root cause to source code, and suggest fixes.

## Arguments

`$ARGUMENTS` describes the bug and optionally provides a URL:
- `/browser-debug clicking submit on the contact form shows a blank page`
- `/browser-debug http://localhost:3000/checkout payment form does not submit`
- `/browser-debug dropdown menu flickers on hover`
- `/browser-debug --trace login page takes 10 seconds to load`

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse the Bug Report

Extract from `$ARGUMENTS`:
- **URL** — if present (starts with `http://` or `https://`)
- **Bug description** — the plain-English description
- **--trace** — if set, enable Playwright tracing for detailed performance analysis

If the description is under 5 words, ask:
> Please describe the bug in more detail. What should happen vs. what actually happens? Which page is affected?

## 2. Determine Target URL

If a URL was extracted, use it directly.

Otherwise, try to infer the page from the bug description:
1. Search route definitions in the project (React Router, Next.js pages/app, Vue Router, etc.)
2. Match keywords in the bug description to route paths

Then fall back to URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` `scripts` for `dev`/`start`
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Capture Diagnostics

Write and execute a Playwright script that:

1. **Sets up listeners** before navigation:
   - `page.on('console')` — capture all console messages, flag errors
   - `page.on('pageerror')` — capture unhandled JS exceptions with stack traces
   - `page.on('requestfailed')` — capture failed network requests with error text
   - `page.on('response')` — capture all responses with status codes and timing

2. **Optionally enables tracing** (if `--trace`):
   - `context.tracing.start({ screenshots: true, snapshots: true, sources: true })`

3. **Navigates to the target URL** with `{ waitUntil: 'networkidle', timeout: 30000 }`

4. **Takes an initial screenshot** (baseline state)

5. **Executes reproduction steps** based on the bug description:
   - If "clicking X" → find and click element X
   - If "form submission" → fill the form and submit
   - If "hover" → hover over the element
   - If "scroll" → scroll to the relevant section
   - If "page load" → already captured, focus on timing

6. **Takes a post-action screenshot**

7. **Saves the trace** (if `--trace`): `context.tracing.stop({ path: '.tmp-debug-trace.zip' })`

8. **Outputs all diagnostics** as JSON

Read the screenshots to visually confirm the bug.

## 4. Analyze Diagnostics

### Console Errors
For each error:
- Parse the error message and stack trace
- Look for source map references (file:line:column)
- If the error references a source file, read it

### Network Failures
For each failed request:
- Identify the endpoint (API route, static asset)
- Check if the backend route exists in the codebase
- Check for CORS issues (`Access-Control` headers)
- Note 4xx/5xx responses

### JS Exceptions
For each unhandled exception:
- Parse the stack trace
- Map to source files
- Read the code at the error location

### Performance (if --trace)
- Identify long tasks
- Flag slow network requests (> 1s)
- Check for blocking resources

### Visual Analysis
Read the before/after screenshots:
- Is the page blank?
- Are elements mispositioned?
- Are there visual glitches or missing content?

## 5. Trace to Source Code

For each issue identified:

1. Parse stack traces for file paths and line numbers
2. Search for the component that renders the problematic element — grep for unique text, class names, IDs
3. Search route definitions to find the page component
4. Read the source file and understand the code around the error
5. Identify the root cause:
   - Missing null/undefined check?
   - Incorrect API endpoint?
   - Race condition in state management?
   - Missing error boundary?
   - CSS issue (z-index, overflow, display)?
   - Missing dependency in a hook?

## 6. Report

```
## Browser Debug Report

**Bug:** [user's description]
**URL:** [target URL]

### Findings

#### Console Errors (N found)

| # | Error | Source | Line |
|---|-------|--------|------|
| 1 | TypeError: Cannot read property 'map' of undefined | `src/components/UserList.tsx` | 23 |

#### Network Failures (N found)

| # | URL | Method | Status | Cause |
|---|-----|--------|--------|-------|
| 1 | /api/users | GET | 500 | Missing auth header |

#### JS Exceptions (N found)

| # | Exception | Source | Line |
|---|-----------|--------|------|
| 1 | Unhandled Promise Rejection | `src/hooks/useAuth.ts` | 45 |

### Root Cause Analysis

**Primary cause:** [clear explanation]
**Source file:** `path/to/file:line`
**Why it happens:** [chain of events]

### Suggested Fix

**File:** `path/to/file`
[Specific code change with before/after]

**Additional recommendations:**
- [related improvements]
```

## 7. Offer to Fix

After presenting the report:
> Would you like me to apply the fix? I will:
> 1. Apply the suggested change to `path/to/file`
> 2. Re-run diagnostics to verify
> 3. Check for similar patterns elsewhere in the codebase

If the user approves, apply the fix, re-run the diagnostic script, and confirm the issue is resolved.

## 8. Clean Up

Remove temporary scripts, screenshots, and trace files.

## Rules

- Capture diagnostics BEFORE reproduction — baseline matters
- Read screenshots to visually confirm the bug
- Trace every error back to a source file — do not just report browser output
- Suggest fixes in the actual source code, not the browser console
- If the bug cannot be reproduced, report what was observed and suggest manual steps
- Do not modify source files unless the user explicitly approves
- Check for similar patterns in the codebase after finding a bug — the same mistake may exist elsewhere
- If the dev server is not running, say so — do not start it automatically
- Clean up all temporary files
