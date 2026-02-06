# Screenshot to Code

Take a screenshot, mockup, or design image and reverse-engineer it into working component code using the project's existing stack and components. Verify the result visually in the browser.

## Arguments

`$ARGUMENTS` provides the image path and optional output:
- `/screenshot-to-code ./designs/settings-page.png`
- `/screenshot-to-code ~/Desktop/mockup.png --output src/pages/Settings.tsx`
- `/screenshot-to-code ./figma-export.png --route /onboarding`

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Image path** — the path to the screenshot/mockup (required)
- **--output path** — where to write the generated component
- **--route /path** — if specified, wire up the route

If no image path is provided:
> Please provide the path to the screenshot or mockup image. Example: `/screenshot-to-code ./designs/page.png`

## 2. Analyze the Image

Read the provided image file and analyze:

1. **Layout structure** — grid/flex layout, number of columns, sections, header/footer/sidebar
2. **Components** — buttons, forms, cards, tables, modals, navigation, lists
3. **Typography** — heading sizes, body text, font weights, alignment
4. **Colors** — primary, secondary, background, text, accent colors
5. **Spacing** — padding, margins, gaps between elements
6. **Responsive hints** — does the layout suggest a specific breakpoint?
7. **Interactive elements** — buttons, links, inputs, dropdowns, toggles

Document each section of the page with its components and approximate layout.

## 3. Discover Project Stack

Run these checks in parallel:

**Framework:** check `package.json` for React, Next.js, Vue, Nuxt, Svelte, Angular

**CSS approach:** check for Tailwind config, styled-components, CSS Modules, global stylesheets

**Component library:** check for shadcn, MUI, Ant Design, Chakra UI, or custom `ui/` directory

**Design tokens:** check Tailwind config theme, CSS custom properties, theme files

## 4. Map to Existing Components

1. List all existing components in the project's component directory
2. Read key components to understand their props and usage
3. Map each visual element from the mockup to an existing component where possible:
   - Card-like sections → existing `Card` component
   - Buttons → existing `Button` with appropriate variant
   - Form fields → existing `Input`, `Select`, etc.
   - Tables → existing `Table` component
   - Navigation → existing `Nav`, `Sidebar` components
4. Identify elements that have no existing component match — these will need new code

## 5. Generate the Code

Write the component following project conventions:

1. **Import existing components** for every element that has a match
2. **Create inline sections** for elements without matches — do not create unnecessary new component files
3. **Match the project's CSS approach:**
   - Tailwind → use utility classes matching the project's existing class patterns
   - CSS Modules → create a matching module file
   - styled-components → use the project's existing styled pattern
4. **Use the project's color palette and spacing scale** — do not hardcode arbitrary values
5. **Include realistic placeholder data** that matches the mockup's content

Write to:
- The `--output` path if specified
- Otherwise, infer from project conventions

## 6. Wire Up the Route (if --route)

Same as `page-gen` — find the router config, add the route, follow existing patterns.

## 7. Verify Visually

If a dev server is running:

1. Navigate to the page with Playwright
2. Take a full-page screenshot of the generated page
3. Read both the original mockup and the generated screenshot
4. Compare them visually:
   - Layout matches? (column count, section order, proportions)
   - Components match? (buttons, cards, forms look right)
   - Colors approximately match? (using project palette)
   - Spacing feels right?
5. Note any discrepancies

## 8. Report

```
## Screenshot to Code Result

**Source:** [image path]
**Output:** [component file path]

### Component Mapping

| Visual Element | Mapped To | Status |
|---------------|-----------|--------|
| Header navigation | Existing `Navbar` component | Reused |
| Stats cards | Existing `Card` component | Reused |
| Data table | Existing `DataTable` component | Reused |
| Chart section | New inline code | Created |

### Visual Fidelity
- Layout: [match / minor differences / needs work]
- Colors: [using project palette — closest match]
- Typography: [matches project scale]
- Spacing: [matches project tokens]

### Discrepancies
- [any elements that could not be accurately reproduced]
```

Ask:
> How does this look? Any adjustments needed?

## Rules

- Always read the image first — do not generate code blindly from the file name
- Reuse existing components aggressively — the goal is to use the project's design system, not recreate it
- Use the project's color palette and spacing tokens, even if they don't exactly match the mockup — consistency with the codebase matters more than pixel-perfect reproduction
- Do not install new dependencies
- Do not use hardcoded pixel values when the project uses a spacing scale (Tailwind, design tokens)
- If the mockup contains text, use it as placeholder data — do not replace with "Lorem ipsum"
- If `--route` is not specified, do not modify the router
- Do not start the dev server automatically
