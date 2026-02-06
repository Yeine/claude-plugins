# Visual Regression Testing

Capture screenshots of pages or routes, compare them against stored baselines, and report visual differences. Supports updating baselines and configuring sensitivity.

## Arguments

`$ARGUMENTS` specifies routes and options:
- `/visual-regression` — test all routes with existing baselines
- `/visual-regression /login /dashboard /settings` — test specific routes
- `/visual-regression --update` — capture new baselines (overwrite existing)
- `/visual-regression --update /login` — update baseline for a specific route only
- `/visual-regression --threshold 0.1` — pixel diff threshold as percentage (default: 0.1)
- `/visual-regression --mobile` — test at 375x812 (iPhone viewport)
- `/visual-regression --viewport 1920x1080` — custom viewport size

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes** — list of routes/URLs to test
- **--update** — capture new baselines instead of comparing
- **--threshold N** — pixel difference threshold as percentage (default: 0.1)
- **--viewport WxH** — viewport dimensions (default: 1280x720)
- **--mobile** — shorthand for `--viewport 375x812`

## 2. Determine Baseline Directory

Check for an existing visual regression setup:
1. Look for `__screenshots__`, `screenshots`, `visual-regression`, or `.visual-baselines` directories
2. Check `playwright.config` for `snapshotDir` or `expect.toHaveScreenshot` configuration
3. Check if the project already uses Playwright's built-in visual comparison

If nothing exists, use `tests/visual-regression/baselines/` as the default and create the directory structure:
```
tests/visual-regression/
  baselines/           ← stored reference screenshots
  diffs/               ← generated diff images (gitignored)
```

## 3. Determine Target URL

1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` `scripts` for `dev`/`start`
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 4. Discover Routes

If specific routes are provided in `$ARGUMENTS`, use those.

If no routes are provided:
1. **Check for existing baselines** — list PNGs in the baseline directory and extract routes from filenames
2. **If no baselines exist** — scan the project for route definitions:
   - **Next.js:** scan `app/` or `pages/` directory
   - **React Router:** grep for `<Route` or `createBrowserRouter`
   - **Vue Router:** grep for `routes` array definitions
3. **Present discovered routes** and ask which to include:
   > Found these routes: /, /login, /dashboard, /settings, /profile
   > Test all of them?

## 5. Capture Screenshots

Write and execute a Playwright script that for each route:

1. Sets the viewport to the specified dimensions
2. Navigates with `{ waitUntil: 'networkidle' }`
3. Waits 500ms for animations/transitions to settle
4. Hides dynamic content that causes false positives: elements with `[data-visual-regression-ignore]` are set to `visibility: hidden`
5. Takes a full-page screenshot
6. Names the file: `<route-slug>-<width>x<height>.png`

If `--update` is set, save directly to the baselines directory.
Otherwise, save to a temporary `current/` directory for comparison.

## 6. Compare Against Baselines (skip if --update)

For each route that has both a baseline and a current screenshot:

Use Playwright's built-in `toHaveScreenshot()` if the project is already configured for it. Otherwise, use a pixel comparison approach:

1. Load both images
2. Compare pixel-by-pixel, allowing for the configured threshold
3. Generate a diff image highlighting the changed pixels
4. Calculate the diff percentage

Save diff images to the `diffs/` directory.

For routes with **no baseline**, mark them as `NO BASELINE` — not as failures.

## 7. Analyze Differences

For each route where the diff exceeds the threshold:

1. Read the diff image to visually identify what changed
2. Categorize the change:
   - **Layout shift** — elements moved position
   - **Content change** — text or images differ
   - **Style change** — colors, fonts, spacing differ
   - **Missing element** — something disappeared
   - **New element** — something appeared
3. Trace to source — search for the component or CSS affecting the changed area. Check recent git changes to those files:
   ```bash
   git log -3 --oneline -- path/to/component
   ```

## 8. Report

### If --update:

```
## Visual Regression Baselines Updated

**Viewport:** WxH

| Route | Status | File |
|-------|--------|------|
| /login | Updated | baselines/login-1280x720.png |
| /dashboard | Updated | baselines/dashboard-1280x720.png |
| /settings | New | baselines/settings-1280x720.png |

Baselines saved to `tests/visual-regression/baselines/`.
Commit these files to track visual changes.
```

### If comparing:

```
## Visual Regression Report

**Viewport:** WxH
**Threshold:** 0.1%

### Summary

| Route | Status | Diff % | Details |
|-------|--------|--------|---------|
| /login | PASS | 0.00% | Identical |
| /dashboard | FAIL | 2.34% | Layout shift in header |
| /settings | FAIL | 0.85% | Color change on button |
| /profile | NO BASELINE | — | Run with --update to create |

### Failures

#### /dashboard (2.34% difference)
**Diff image:** tests/visual-regression/diffs/dashboard-1280x720-diff.png
**What changed:** [description based on visual analysis]
**Likely cause:** Recent change to `src/components/Header.tsx`
**Source:** `src/components/Header.tsx:45`

### Verdict: PASS | FAIL (N of M routes have regressions)

To accept changes as new baselines:
`/visual-regression --update /dashboard /settings`
```

## 9. Ensure Diffs are Gitignored

Check if the diffs directory is in `.gitignore`:
```bash
git check-ignore tests/visual-regression/diffs/ 2>/dev/null
```

If not, suggest adding it:
> The `diffs/` directory should be gitignored (generated output). Add `tests/visual-regression/diffs/` to `.gitignore`?

## 10. Clean Up

Remove temporary capture files (keep baselines and diffs):
```bash
rm -rf tests/visual-regression/current/
```

Remove temporary scripts.

## Rules

- Never delete baselines without `--update` flag — they are the source of truth
- Always wait for networkidle + settle time before capturing
- Use consistent viewport sizes — mismatched dimensions cause false positives
- Hide `[data-visual-regression-ignore]` elements to reduce noise from dynamic content
- Report diff percentages with 2 decimal places
- If no baseline exists for a route, report `NO BASELINE` not `FAIL`
- Keep diff images for review — do not delete them automatically
- Suggest gitignoring the diffs directory but NOT the baselines directory
- If Playwright's built-in `toHaveScreenshot` is already configured, prefer it over custom comparison
- If the dev server is not running, say so — do not start it automatically
