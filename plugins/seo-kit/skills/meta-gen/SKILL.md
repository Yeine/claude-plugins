# Generate Meta Tags

Analyze actual page content and generate optimized meta tags (title, description, Open Graph, Twitter Card, canonical) using the project's existing meta tag pattern. Writes tags directly into source files.

## Arguments

`$ARGUMENTS` specifies routes and options:
- `/meta-gen` — generate meta tags for the root page
- `/meta-gen /pricing /blog /about` — generate for specific routes
- `/meta-gen --dry-run` — show what would be generated without writing files

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes/URLs** — one or more paths or full URLs
- **--dry-run** — preview generated tags without modifying source files

## 2. Determine Target URLs

If routes were provided, use them.

If no routes provided, generate for the root page (`/`).

URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` for `dev`/`start` scripts
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Discover the Project's Meta Tag Pattern

Search the codebase to understand how meta tags are managed:

**Next.js App Router:**
- Look for `export const metadata` or `export async function generateMetadata` in `layout.tsx` or `page.tsx` files
- Check for a root layout with default metadata

**Next.js Pages Router:**
- Look for imports from `next/head`
- Check for `<Head>` components in page files

**React (other):**
- Check for `react-helmet` or `react-helmet-async` in `package.json`
- Search for `<Helmet>` components in pages

**Vue/Nuxt:**
- Look for `useHead()`, `useSeoMeta()` calls
- Check for `<Head>` component usage

**Svelte/SvelteKit:**
- Look for `<svelte:head>` blocks in page components

**Static/vanilla:**
- Check for `<head>` sections in HTML files

Read 1–2 existing pages with meta tags to learn the exact pattern, import style, and naming convention.

Also check for:
- A site-wide title template (e.g., `%s | Acme Inc`)
- Default OG image path
- Site name / brand name used in titles

## 4. Analyze Page Content

For each target URL, navigate with Playwright and extract:

1. **H1 content** — the primary heading, usually the best basis for the title
2. **First meaningful paragraph** — the intro text, best basis for the description
3. **Primary image** — the largest above-the-fold image, or an existing `og:image`
4. **Page type** — infer from content:
   - Has article date/author → blog post / article
   - Has pricing/plans → pricing page
   - Has product details → product page
   - Has FAQ accordion → FAQ page
   - Generic content → landing / info page
5. **Existing meta tags** — check what's already present (to update rather than duplicate)
6. **Site context** — the site name, base URL, any brand tagline

## 5. Generate Meta Tags

For each page, generate:

### Title Tag
- Under 60 characters
- Include the primary topic (from H1)
- Append site name if a title template exists (e.g., "Pricing Plans | Acme Inc")
- Front-load the most important words
- Avoid keyword stuffing — write for humans

### Meta Description
- 150–160 characters
- Summarize what the page offers (from first paragraph)
- Include a subtle call-to-action where appropriate ("Learn more", "Get started", "Compare plans")
- Make it compelling — this is the search snippet

### Open Graph
- `og:title` — can match title tag or be slightly more descriptive
- `og:description` — can match meta description or be slightly different
- `og:image` — use the primary image found, or the site's default OG image
- `og:url` — the canonical URL of the page
- `og:type` — `website` for landing pages, `article` for blog posts

### Twitter Card
- `twitter:card` — `summary_large_image` if there's a good image, `summary` otherwise
- `twitter:title` — match `og:title`
- `twitter:description` — match `og:description`

### Canonical URL
- Self-referencing canonical with the preferred URL format (trailing slash or not, www or not)

## 6. Present Generated Tags

For each page, show the generated tags clearly:

```
### /pricing

**Title:** "Pricing Plans — Start Free, Scale as You Grow | Acme" (52 chars)
**Description:** "Compare Acme plans from free to enterprise. All plans include core features. Upgrade anytime with no lock-in." (109 chars)
**OG Image:** /images/og-pricing.png (existing)
**Canonical:** https://acme.com/pricing

**Source file:** `src/app/pricing/page.tsx`
**Status:** New (no existing meta) | Update (replacing current meta)
```

## 7. Write to Source (unless --dry-run)

If `--dry-run`, stop here — present the report only.

Otherwise, for each page:

1. Open the source file identified in step 3
2. Write the meta tags following the project's exact pattern:
   - **Next.js App Router:** add or update `export const metadata = { ... }` or `generateMetadata()`
   - **Next.js Pages Router:** add or update `<Head>` block
   - **React Helmet:** add or update `<Helmet>` component
   - **Vue/Nuxt:** add or update `useHead()` / `useSeoMeta()` call
   - **Svelte:** add or update `<svelte:head>` block
   - **Static HTML:** edit the `<head>` section directly
3. If updating existing tags, preserve any tags not being generated (e.g., custom meta tags, script tags)

## 8. Verify

After writing, navigate to each page again with Playwright and confirm:
- Title tag renders correctly in `<head>`
- Meta description renders correctly
- OG tags are present
- Twitter Card tags are present
- Canonical URL is correct

Report any tags that failed to render (might indicate a framework-specific issue).

## 9. Report

```
## Meta Tags Generated

| Route | Title | Description | OG | Twitter | Canonical | Source |
|-------|-------|-------------|-----|---------|-----------|--------|
| / | Updated | Generated | Generated | Generated | Added | `src/app/page.tsx` |
| /pricing | Generated | Generated | Generated | Generated | Added | `src/app/pricing/page.tsx` |
| /blog | Kept | Updated | Generated | Generated | Added | `src/app/blog/page.tsx` |

### Details

#### /pricing
**Title:** "Pricing Plans — Start Free, Scale as You Grow | Acme" (52 chars)
**Description:** "Compare Acme plans from free to enterprise..." (109 chars)
**Verified:** All tags rendering correctly
```

## 10. Clean Up

Remove temporary scripts.

## Rules

- Analyze the actual rendered content — do not guess what a page is about from its file name
- Follow the project's existing meta tag pattern exactly — do not introduce a new approach
- Write for humans, not search engines — no keyword stuffing, no clickbait
- Keep titles under 60 chars and descriptions under 160 chars — search engines truncate beyond this
- If a page already has good meta tags, say so and skip it — do not regenerate for the sake of it
- Preserve existing meta tags that are not being generated (custom tags, verification codes, etc.)
- If `--dry-run`, do not modify any files
- Do not fabricate content — base titles and descriptions on actual page content
- If the dev server is not running, say so — do not start it automatically
- Use the site's existing title template if one exists
