# Design Review

Navigate the running app and critique the UI for spacing inconsistencies, alignment issues, typography problems, and design system violations. Trace issues to source files with fix suggestions.

## Arguments

`$ARGUMENTS` specifies routes and options:
- `/design-review` — review the app's root page
- `/design-review /login /dashboard /settings` — review specific routes
- `/design-review --fix` — review and auto-fix objective issues (spacing, alignment)

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes/URLs** — one or more paths or full URLs
- **--fix** — auto-fix objective issues in source

## 2. Determine Target URLs

If routes were provided, use them. Prepend the base URL to relative routes.

If no routes were provided, review the root page (`/`).

URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` for `dev`/`start` scripts
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Load Design System Context

Before reviewing, understand the project's design system:

1. **Tailwind config** — read `tailwind.config.*` for the spacing scale, color palette, font sizes, breakpoints
2. **Theme files** — look for `theme.ts`, `tokens.ts`, `variables.css`, or similar
3. **CSS custom properties** — grep for `--color-`, `--spacing-`, `--font-` in global stylesheets
4. **Component library config** — MUI theme, Chakra theme, etc.
5. **CLAUDE.md** — check for documented design conventions

Record the project's:
- Spacing scale (e.g., 4px increments, Tailwind's default scale)
- Color palette (named colors, semantic tokens)
- Typography scale (heading sizes, body text, weights)
- Breakpoints

If no design system is found, use general UI best practices as the baseline.

## 4. Capture Pages

For each target route, write and execute a Playwright script that:

1. Captures a screenshot at **desktop** (1280x720) viewport
2. Captures a screenshot at **mobile** (375x812) viewport
3. Extracts computed styles for key elements:
   - All headings (tag, font-size, font-weight, margin, color)
   - All buttons (padding, font-size, border-radius, colors)
   - All form inputs (height, padding, font-size, border)
   - Container/section padding and margins
   - Gap values in flex/grid containers

## 5. Analyze for Issues

Review each page at both viewports. Check for:

### Spacing
- Inconsistent padding/margins between similar sections
- Spacing values that don't match the project's scale (e.g., 13px when scale is 4px-based)
- Missing or excessive whitespace between elements
- Sections touching viewport edges on mobile

### Alignment
- Elements that should be aligned but aren't (text baselines, icons with text, form labels)
- Off-center content in centered containers
- Inconsistent left/right margins across sections

### Typography
- Heading hierarchy violations (H3 larger than H2, skipped levels)
- Inconsistent font sizes for similar elements across pages
- Line height issues (text too cramped or too spread)
- Inconsistent font weights

### Color and Contrast
- Colors not in the project's palette (one-off hex values)
- Insufficient contrast between text and background (WCAG AA: 4.5:1 for body, 3:1 for large text)
- Inconsistent use of semantic colors (different shades for the same purpose)

### Responsive
- Content overflowing on mobile
- Elements too small to tap (< 44x44px touch targets)
- Text too small on mobile (< 16px)
- Horizontal scroll on mobile
- Layout not adapting appropriately between breakpoints

### Consistency Across Pages
- Same component styled differently on different pages
- Inconsistent button sizes or styles
- Different heading styles for the same level across pages

## 6. Trace Issues to Source

For each issue:

1. Identify the element by its text content, class, or role
2. Grep the source for the component rendering that element
3. Find the specific style declaration causing the issue (CSS class, inline style, Tailwind class)
4. Record the file path and line number

## 7. Report

```
## Design Review Report

**Pages Reviewed:** N
**Viewports:** Desktop (1280x720), Mobile (375x812)

### Summary

| Category | Issues |
|----------|--------|
| Spacing | X |
| Alignment | X |
| Typography | X |
| Color/Contrast | X |
| Responsive | X |
| Consistency | X |

### Spacing Issues

#### 1. Inconsistent section padding on /dashboard
**Severity:** Moderate
**Source:** `src/pages/Dashboard.tsx:34`
**Problem:** Hero section uses `p-8` but feature section uses `p-6 py-10` — inconsistent vertical rhythm
**Fix:** Use `p-8` for both sections to match

### Alignment Issues
...

### Typography Issues
...

### Color/Contrast Issues
...

### Responsive Issues
...

### Cross-Page Consistency Issues
...
```

## 8. Auto-Fix (if --fix)

If `--fix` is set, fix **objective** issues only:

**Auto-fixable:**
- Spacing values not matching the design scale (round to nearest scale value)
- Inconsistent padding/margins on similar elements (standardize to the more common value)
- Missing responsive classes causing overflow

**NOT auto-fixable (report only):**
- Color choices — subjective
- Layout structure changes — require design decisions
- Typography scale changes — may be intentional
- Component redesign

After fixing:
1. Re-capture screenshots at both viewports
2. Report before/after comparison
3. List remaining issues that need manual design decisions

## 9. Clean Up

Remove temporary scripts and captured screenshots.

## Rules

- Use the project's design system as the source of truth — do not impose external standards
- If no design system exists, use general best practices but flag suggestions as opinions, not violations
- Distinguish between objective issues (contrast ratio fails, overflow, broken alignment) and subjective opinions (spacing preferences, color choices)
- Check both desktop and mobile — responsive issues are real bugs
- Report issues with specific source file paths and line numbers
- When suggesting spacing fixes, use the project's scale values (e.g., `p-4` not `padding: 16px`)
- Do not modify files unless `--fix` was specified
- Do not fix subjective issues automatically — only objective ones
- If the dev server is not running, say so — do not start it automatically
