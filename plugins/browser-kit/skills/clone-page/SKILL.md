# Clone Page

Take a public URL, analyze its layout and design, and generate an equivalent page using the project's existing stack and components. No copy-paste of external code — rebuild from scratch using your own design system.

## Arguments

`$ARGUMENTS` provides the source URL and options:
- `/clone-page https://example.com/pricing`
- `/clone-page https://stripe.com/pricing --output src/pages/Pricing.tsx`
- `/clone-page https://linear.app/features --route /features`

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Source URL** — the public URL to clone (required, starts with `http://` or `https://`)
- **--output path** — where to write the generated component
- **--route /path** — if specified, wire up this route in the project's router

If no URL is provided:
> Please provide the URL of the page to clone. Example: `/clone-page https://example.com/pricing`

## 2. Capture the Source Page

Write and execute a Playwright script that navigates to the source URL and captures:

1. **Full-page screenshot** at 1280x720 (desktop)
2. **Full-page screenshot** at 375x812 (mobile)
3. **Accessibility tree** — `page.accessibility.snapshot()` for semantic structure
4. **DOM structure** — extract the page's section hierarchy:
   - All landmark elements (`header`, `main`, `footer`, `nav`, `section`, `aside`)
   - Heading hierarchy (H1–H6 with text content)
   - Key interactive elements (buttons, links, forms)
5. **Computed styles** for key elements:
   - Colors (background, text, accent)
   - Typography (font sizes, weights, line heights)
   - Spacing (padding, margins, gaps)
   - Layout (flex/grid properties, max-widths)

## 3. Analyze the Design

Read the captured screenshots and data. Document:

1. **Page structure** — header, hero, sections, footer; number of columns; layout flow
2. **Visual hierarchy** — what draws attention first, heading sizes, emphasis
3. **Component inventory** — cards, buttons, navigation, lists, forms, CTAs
4. **Color palette** — extract the 5–8 most used colors
5. **Typography** — heading scale, body size, font families
6. **Responsive behavior** — compare desktop vs mobile layouts

## 4. Discover Project Stack

Same as `page-gen`:
- Framework (React, Next.js, Vue, Svelte, etc.)
- CSS approach (Tailwind, CSS Modules, styled-components)
- Component library (shadcn, MUI, custom)
- Design tokens (Tailwind theme, CSS variables, theme files)

## 5. Map to Project Components

1. List existing components in the project
2. Map each visual element from the source page to existing components:
   - Navigation bar → existing `Navbar`/`Header`
   - Cards → existing `Card`
   - Buttons → existing `Button` with appropriate variant
   - Pricing tables → existing `Table` or `Card` grid
   - Feature grids → existing grid/card patterns
3. For elements without matches, plan new inline code using the project's CSS approach
4. Map the source page's colors to the closest values in the project's palette

## 6. Generate the Page

Write the component following project conventions:

1. **Reuse existing components** for every element that has a match
2. **Adapt the layout** to use the project's grid/layout system
3. **Map colors** to the project's palette — use the closest semantic match, not exact hex values
4. **Match typography** to the project's type scale
5. **Use the project's spacing scale** for all padding, margins, and gaps
6. **Include realistic content** — use the source page's text as placeholder, or write equivalent copy
7. **Implement responsive behavior** using the project's breakpoint system

Write to:
- The `--output` path if specified
- Otherwise, infer from project conventions

## 7. Wire Up the Route (if --route)

If `--route` is specified:
1. Find the project's router configuration
2. Add the new route
3. Follow existing routing patterns

## 8. Verify Visually

If a dev server is running for the project:

1. Navigate to the generated page with Playwright
2. Take a full-page screenshot at desktop and mobile
3. Read all screenshots: original source (desktop + mobile) and generated (desktop + mobile)
4. Compare:
   - Does the layout structure match? (sections in correct order, similar proportions)
   - Do components serve the same purpose? (even if styled differently)
   - Is the responsive behavior similar?

## 9. Report

```
## Clone Page Result

**Source:** [URL]
**Output:** [component file path]

### Structure Mapping

| Source Section | Implementation | Status |
|---------------|---------------|--------|
| Hero with CTA | Reused `Hero` + `Button` | Matched |
| Features grid | Reused `Card` in CSS Grid | Matched |
| Pricing table | New code using project CSS | Adapted |
| Footer | Reused `Footer` component | Matched |

### Adaptation Notes
- Colors mapped to project palette: source blue (#0066FF) → project `primary` (#0055EE)
- Source uses 3-column grid → project uses 12-col system, adapted to `col-span-4`
- Source font (Inter) → project font (already Inter, direct match)

### Visual Comparison
- Layout structure: [match / adapted]
- Component parity: [N of M sections mapped to existing components]
- Responsive: [both viewports working]
```

Ask:
> How does this look? Any sections that need adjustment?

## Rules

- Never copy-paste HTML, CSS, or JS from the source page — generate fresh code using the project's stack
- Reuse existing project components wherever possible
- Map colors to the project's palette — consistency with the codebase matters more than exact replication
- Use the project's spacing scale and typography — do not hardcode values from the source
- The goal is "same structure and purpose" not "pixel-perfect copy"
- Do not install new dependencies
- Do not include external assets (images, fonts) from the source — use placeholders or project assets
- If `--route` is not specified, do not modify the router
- Do not start the dev server automatically
- Respect copyright — this skill recreates the design pattern, not the content. Use placeholder text for marketing copy.
