# Accessibility Audit

Audit web pages for accessibility violations using Playwright and axe-core. Produce a prioritized report with specific fix suggestions tied to source files.

## Arguments

`$ARGUMENTS` specifies the target and options:
- `/a11y-audit` — audit the app's root page
- `/a11y-audit http://localhost:3000/settings` — audit a specific URL
- `/a11y-audit /login /register /dashboard` — audit multiple routes
- `/a11y-audit --fix` — audit and auto-fix what can be safely fixed in source
- `/a11y-audit --standard wcag2aaa` — specify WCAG standard (default: wcag2aa)

## 0. Ensure Playwright and axe-core are Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If Playwright is NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

Additionally, check for `@axe-core/playwright`:
```bash
grep -r "axe-core" package.json
```

If not installed, offer:
> `@axe-core/playwright` is not installed. Would you like me to add it?
> This will run: `npm install -D @axe-core/playwright`

Stop if declined — axe-core is required.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes/URLs** — one or more paths (e.g., `/login /dashboard`) or full URLs
- **--fix** — whether to attempt auto-fixes in source code
- **--standard** — WCAG standard to test against (default: `wcag2aa`). Options: `wcag2a`, `wcag2aa`, `wcag2aaa`, `best-practice`

## 2. Determine Target URLs

If URLs/routes were provided in arguments, use them directly. Prepend the base URL to relative routes.

If no routes were provided, audit the root page (`/`).

To discover the base URL (when not provided explicitly):
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` `scripts` for `dev` or `start` — infer port
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Run the Audit

Write and execute a script that, for each target URL:

1. Navigates to the page with `{ waitUntil: 'networkidle' }`
2. Creates an `AxeBuilder` instance with the appropriate WCAG tags:
   - `wcag2a` → `['wcag2a']`
   - `wcag2aa` → `['wcag2a', 'wcag2aa']`
   - `wcag2aaa` → `['wcag2a', 'wcag2aa', 'wcag2aaa']`
   - `best-practice` → `['best-practice']`
3. Runs `axe.analyze()` and captures the full results
4. Takes a screenshot of the page for reference

Collect violations, passes, and incomplete checks from each page.

## 4. Trace Violations to Source Files

For each violation:

1. Extract the DOM selector from `nodes[].target` (CSS selector of the offending element)
2. Search the source code for the element — grep for unique text content, class names, aria-labels, or data attributes
3. Check component directories for files matching the route path (e.g., `/login` maps to `Login.tsx` or `login.vue`)
4. Record the source file and approximate line where the element is rendered
5. Generate a specific fix based on the violation type:
   - **Missing alt text** → add `alt` attribute with descriptive text
   - **Missing label** → add `<label>` element or `aria-label`
   - **Low contrast** → suggest specific color values meeting the required ratio
   - **Missing landmarks** → add semantic HTML (`<main>`, `<nav>`, `<header>`)
   - **Heading hierarchy** → adjust heading levels

## 5. Generate Report

```
## Accessibility Audit Report

**Standard:** WCAG 2.1 AA
**Pages Audited:** N

### Summary

| Severity | Count | Auto-fixable |
|----------|-------|-------------|
| Critical | X     | Y           |
| Serious  | X     | Y           |
| Moderate | X     | Y           |
| Minor    | X     | Y           |

### Critical Violations

#### 1. [rule-id] — [description]
**Impact:** Critical
**WCAG:** [criterion number and name]
**Pages:** [affected routes]

| Source File | Line | Element | Fix |
|-------------|------|---------|-----|
| `src/components/Foo.tsx` | 12 | `<img src="logo.png">` | Add `alt="Company Logo"` |

### Serious Violations
...

### Moderate Violations
...

### Minor Violations
...

### Passes (XX checks passed)
- [List of passing categories for confidence]
```

## 6. Auto-Fix (if --fix)

If `--fix` flag is set, apply fixes for safely auto-fixable categories:

**Auto-fixable:**
- Missing `alt` attributes (add `alt=""` for decorative, descriptive for functional images)
- Missing `lang` attribute on `<html>`
- Missing form labels (add `aria-label` where no visible label pattern exists)
- Missing button accessible names
- Duplicate IDs

**NOT auto-fixable (report only):**
- Color contrast issues — require design decisions
- Heading hierarchy — require structural understanding
- Focus management
- Custom widget ARIA patterns

After applying fixes:
1. Re-run the audit on the same pages
2. Report before/after violation counts
3. List remaining issues that need manual attention

## 7. Clean Up

Remove temporary scripts and screenshots.

## Rules

- Map violations to source files, not just DOM elements — developers fix source, not browser output
- Use axe-core severity levels (critical/serious/moderate/minor) — do not invent your own
- Include WCAG criterion references (e.g., 1.1.1, 4.1.2) for every violation
- When suggesting color fixes, provide specific hex values that meet contrast ratios
- Do not auto-fix color/contrast issues — those require design decisions
- If `--fix` is not set, do not modify any files
- Always report what passed, not just what failed — gives confidence in coverage
- Be specific: "3 images missing alt on /login" not "images need alt text"
