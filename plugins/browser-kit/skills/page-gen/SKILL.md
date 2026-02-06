# Generate Page

Generate a page component from a plain-English description, using the project's existing stack, design system, and component library. Verify the result in the browser.

## Arguments

`$ARGUMENTS` describes the page to create:
- `/page-gen a settings page with a sidebar nav, profile form, and danger zone`
- `/page-gen a dashboard with stats cards, recent activity table, and a chart`
- `/page-gen landing page with hero, features grid, and pricing section --route /pricing`

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Page description** — the plain-English description of the page
- **--route /path** — if specified, wire up this route in the project's router
- **--output path** — if specified, write the component to this path

If the description is fewer than 5 words, ask:
> Please describe the page in more detail. What sections should it include? What data does it display?

## 2. Discover Project Stack

Run these checks in parallel to understand the project's technology:

**Framework:**
- `package.json` dependencies: `react`, `next`, `vue`, `nuxt`, `svelte`, `@sveltejs/kit`, `@angular/core`
- Directory structure: `app/`, `pages/`, `src/routes/`

**CSS approach:**
- `tailwind.config.*` → Tailwind CSS
- `styled-components` or `@emotion` in dependencies → CSS-in-JS
- `.module.css` or `.module.scss` files → CSS Modules
- Global stylesheets

**Component library:**
- `@shadcn/ui`, `@radix-ui` → shadcn/Radix
- `@mui/material` → Material UI
- `antd` → Ant Design
- `@chakra-ui` → Chakra UI
- Custom component library in `src/components/ui/` or similar

**Design tokens:**
- `tailwind.config` theme section → colors, spacing, fonts
- CSS custom properties in global styles
- Theme files (`theme.ts`, `tokens.ts`)

## 3. Analyze Existing Pages

Read 2–3 existing page components to learn the project's patterns:

1. Search for page/view components in the project
2. Note:
   - File naming convention (PascalCase, kebab-case, index files)
   - Layout patterns (shared layout components, sidebar patterns)
   - How data is fetched (hooks, loaders, server components)
   - Import organization
   - Component composition style
   - How routing is set up

## 4. Generate the Page

Based on the description and discovered patterns:

1. **Plan the component structure** — break the description into sections and map each to existing components or new sub-components
2. **Reuse existing components** wherever possible — if the project has a `Card`, `Table`, `Form`, `Button`, etc., use them
3. **Follow the project's conventions** exactly — same import style, same file structure, same naming
4. **Use realistic placeholder data** — not "Lorem ipsum" but plausible content matching the page's purpose
5. **Include responsive behavior** if the project uses responsive patterns

Write the component file to:
- The `--output` path if specified
- Otherwise, infer from project conventions (e.g., `src/pages/Settings.tsx`, `app/settings/page.tsx`)

If new sub-components are needed, create them in the project's component directory following existing patterns.

## 5. Wire Up the Route (if --route)

If `--route` is specified:
1. Find the project's router configuration
2. Add the new route pointing to the generated component
3. Follow the existing routing pattern (lazy loading, layout nesting, etc.)

If no `--route` flag, skip this step — just create the component.

## 6. Verify in Browser

Determine the app URL (same URL discovery as other skills). If a dev server is running:

1. Write a Playwright script to navigate to the page
2. Take a full-page screenshot
3. Read the screenshot and present it to the user

If no dev server is running, skip browser verification and note:
> Start your dev server and navigate to [route] to see the result.

## 7. Present and Iterate

Show the user:
- The screenshot (if captured)
- A summary of files created/modified
- Which existing components were reused

Ask:
> How does this look? Any adjustments?

If the user requests changes, apply them and re-verify.

## Rules

- Always discover and use the project's existing stack — never assume React, Tailwind, etc.
- Reuse existing components before creating new ones
- Follow the project's file naming, import style, and directory structure exactly
- Use realistic placeholder data, not "Lorem ipsum"
- Do not install new dependencies — use what the project already has
- If `--route` is not specified, do not modify the router
- Do not start the dev server automatically
- If existing page patterns use data fetching (hooks, loaders), include appropriate data fetching in the generated page with placeholder implementations
