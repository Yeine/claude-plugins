# Generate E2E Test

Turn a plain-English description of a user flow into a working Playwright E2E test. Navigate the running app to understand the actual DOM, then generate a test file with resilient selectors and meaningful assertions.

## Arguments

`$ARGUMENTS` describes the user flow to test and optionally specifies output:
- `/e2e-gen user logs in with valid credentials and sees dashboard`
- `/e2e-gen http://localhost:3000 user adds item to cart and checks out`
- `/e2e-gen user resets password via email link --output tests/e2e/password-reset.spec.ts`

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If Playwright is found:** note the config file location and any existing test patterns (test directory, base URL, browser projects).

**If NOT found:** tell the user and offer to install:
> Playwright is not installed in this project. Would you like me to set it up?
> This will run: `npm init playwright@latest`

If the user declines, stop — the skill requires Playwright.

## 1. Parse the Flow Description

Extract from `$ARGUMENTS`:
- **Target URL** — if a URL is present (starts with `http://` or `https://`), extract it
- **Flow description** — the plain-English description of what the user does
- **--output path** — if specified, use that path for the generated test file

If the flow description is fewer than 5 words or has no verbs, ask the user to elaborate:
> Please describe the user flow in more detail. Example: "user logs in with email and password, navigates to settings, changes their display name, and verifies the change is saved"

## 2. Determine Target URL

If a URL was extracted from arguments, use it directly.

Otherwise, discover the app URL:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` `scripts` for `dev` or `start` — infer port from common defaults (Vite → 5173, Next.js → 3000, CRA → 3000)
4. Probe common ports: `curl -s -o /dev/null -w "%{http_code}" http://localhost:PORT` for 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run, or provide the URL directly.

## 3. Analyze Existing Test Patterns

Before generating, understand the project's testing conventions:

1. Search for existing Playwright test files (`*.spec.ts`, `*.spec.js`, `*.test.ts`, `*.test.js`) in directories matching `e2e`, `playwright`, `tests`
2. Read 1–2 existing test files and note:
   - Import style (`import { test, expect }` vs `const { test, expect } = require(...)`)
   - Page Object pattern usage
   - Selector style (data-testid, role-based, CSS)
   - Assertion patterns
   - Setup/teardown (`beforeEach`, fixtures)
   - File naming convention
3. Read `playwright.config` for test directory, base URL, timeouts, browser projects

If no existing tests exist, use Playwright best practices as defaults: `@playwright/test` imports, role-based selectors.

## 4. Explore the Running App

Write and execute a temporary exploration script to capture the actual DOM state at each step of the flow.

The script should:
1. Launch a headless browser and navigate to the target URL
2. Capture the accessibility tree (`page.accessibility.snapshot()`)
3. Capture all interactive elements (buttons, links, inputs, selects, textareas, elements with `role`, `data-testid`)
4. Take a full-page screenshot

For multi-step flows, extend the script to navigate through each step based on the flow description (click buttons, fill forms, etc.), capturing the DOM and a screenshot at each stage.

Run the script with `npx tsx` (or `npx ts-node` if tsx is unavailable). Read the screenshots to visually understand the page layout.

## 5. Generate the Test File

Determine output path:
- If `--output` was specified, use it
- If an existing test directory exists, place the file there with a descriptive name
- Otherwise: `tests/e2e/<flow-slug>.spec.ts` (derive slug from flow description)

Choose selectors in this priority order:
1. **Role-based** — `page.getByRole('button', { name: 'Submit' })` (preferred)
2. **Text-based** — `page.getByText('Welcome')`
3. **Label-based** — `page.getByLabel('Email')`
4. **Placeholder-based** — `page.getByPlaceholder('Enter email')`
5. **Test ID** — `page.getByTestId('login-form')` if data-testid exists
6. **CSS** — `page.locator('.submit-btn')` only as last resort

Write the test file following project conventions discovered in step 3. Include:
- One `test.describe` per logical flow
- Comments explaining each step
- `await expect(...)` assertions at meaningful checkpoints, not just the end
- Realistic but obviously fake test data (`test@example.com`, not real emails)
- `test.beforeEach` for common setup if the pattern exists in the project

## 6. Validate

Run the generated test:
```bash
npx playwright test OUTPUT_PATH --reporter=line
```

If it fails:
1. Read the error output
2. Adjust selectors or flow based on actual page behavior
3. Re-run (up to 3 attempts)

If still failing after 3 attempts, present the test file with a note about what needs manual adjustment.

## 7. Clean Up and Report

Remove temporary exploration scripts and screenshots.

Report:

```
## E2E Test Generated

**File:** path/to/test.spec.ts
**Flow:** [user's flow description]
**Status:** PASS | NEEDS ADJUSTMENT

**Steps covered:**
1. [Step 1]
2. [Step 2]
...

**Selectors used:** role-based (N), label-based (N), testid (N)

Run: `npx playwright test path/to/test.spec.ts`
```

## Rules

- Always navigate the real app — never guess at selectors without seeing the actual DOM
- Follow existing test patterns in the project — do not introduce a new style
- Use resilient selectors (role > text > testid > CSS) — never XPath or auto-generated class names
- Generated tests must run successfully before reporting success
- Do not hardcode absolute URLs — use relative paths with `baseURL` from config
- Do not leave temporary files behind
- If the dev server is not running, say so — do not attempt to start it automatically
- Include meaningful assertions, not just navigation checks
