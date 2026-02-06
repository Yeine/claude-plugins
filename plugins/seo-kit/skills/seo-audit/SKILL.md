# SEO Audit

Run a comprehensive SEO audit on one or more pages. Check meta tags, headings, images, Open Graph, canonical URLs, robots directives, structured data, and more. Produce a prioritized report with source file references.

## Arguments

`$ARGUMENTS` specifies the target and options:
- `/seo-audit` — audit the app's root page
- `/seo-audit /pricing /blog /about` — audit specific routes
- `/seo-audit --full-crawl` — follow all internal links from root and audit every discovered page
- `/seo-audit --fix` — auto-fix safe issues in source files

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes/URLs** — one or more paths or full URLs
- **--fix** — auto-fix safe issues in source
- **--full-crawl** — discover and audit all pages by following internal links from root

## 2. Determine Target URLs

If routes were provided, use them. Prepend the base URL to relative routes.

If `--full-crawl` is set, start from `/` and follow all internal links (breadth-first, max depth 5).

If no routes and no `--full-crawl`, audit the root page (`/`).

URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` for `dev`/`start` scripts
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Discover Meta Tag Pattern in Source

Before auditing, understand how the project manages meta tags so issues can be traced to source:

- **Next.js App Router:** `export const metadata` or `generateMetadata()` in `layout.tsx`/`page.tsx`
- **Next.js Pages Router:** `<Head>` from `next/head`
- **React (other):** `react-helmet`, `react-helmet-async`
- **Vue/Nuxt:** `useHead()`, `useSeoMeta()`, `<Head>` component
- **Svelte/SvelteKit:** `<svelte:head>`
- **Static/vanilla:** `<head>` tags in HTML files

Search for these patterns in the source to know where meta tags are defined.

## 4. Audit Each Page

For each target URL, write and execute a Playwright script that navigates to the page and extracts all SEO-relevant data:

### Meta Tags
- **Title tag** — present? text content? character count (ideal: 50–60)?
- **Meta description** — present? content? character count (ideal: 150–160)?
- **Viewport meta** — `<meta name="viewport">` present with `width=device-width, initial-scale=1`?
- **Charset** — `<meta charset="utf-8">` present?

### Heading Structure
- **H1 count** — exactly one H1 per page?
- **Heading hierarchy** — H2s follow H1, H3s follow H2, no skipped levels?
- **H1 content** — meaningful, contains primary topic?

### Images
- **Alt text** — all `<img>` have `alt` attribute?
- **Lazy loading** — images below the fold have `loading="lazy"`?
- **Dimension attributes** — `width` and `height` set (prevents CLS)?

### Open Graph
- `og:title` — present? matches or complements title tag?
- `og:description` — present?
- `og:image` — present? URL resolves? dimensions adequate (1200x630 ideal)?
- `og:url` — present? matches canonical?
- `og:type` — present? appropriate value?

### Twitter Card
- `twitter:card` — present? (`summary`, `summary_large_image`, etc.)
- `twitter:title` — present?
- `twitter:description` — present?

### Canonical & Robots
- `<link rel="canonical">` — present? correct URL? not self-referencing incorrectly?
- `<meta name="robots">` — if present, what directives?
- Check response headers for `X-Robots-Tag`
- Check `/robots.txt` for rules affecting this path

### Language
- `<html lang="...">` — set? valid language code?
- `hreflang` tags — present if multilingual?

### Structured Data
- Any `<script type="application/ld+json">` present?
- If present: valid JSON? recognized schema.org type?

### URL Quality
- URL length (keep under 75 chars)
- Hyphens vs underscores (hyphens preferred)
- Trailing slash consistency across the site
- Lowercase (no mixed case)

### Internal Links
- Count of internal links on the page
- Anchor text quality — flag "click here", "read more", empty anchors
- Any `rel="nofollow"` on internal links (usually a mistake)

Track all pages audited to detect cross-page issues:
- Duplicate title tags across pages
- Duplicate meta descriptions
- Inconsistent trailing slash usage

## 5. Trace Issues to Source Files

For each issue found:
1. Use the meta tag pattern discovered in step 3 to find where the tag is defined
2. Grep for specific text content (title text, description text) in source files
3. For heading issues, find the component rendering that heading
4. For image issues, find the `<img>` in source
5. Record file path and line number

## 6. Calculate SEO Score

Score out of 100 based on weighted categories:
- **Meta tags (25 points):** title (10), description (10), viewport (3), charset (2)
- **Headings (15 points):** single H1 (8), hierarchy (7)
- **Images (15 points):** alt text (10), lazy loading (3), dimensions (2)
- **Open Graph (15 points):** all 5 tags present (3 each)
- **Technical (15 points):** canonical (5), robots (3), lang (4), structured data (3)
- **Link quality (10 points):** anchor text (5), no nofollow on internal (5)
- **URL quality (5 points):** length (2), format (3)

Deduct points proportionally for failures.

## 7. Generate Report

```
## SEO Audit Report

**Pages Audited:** N
**SEO Score:** X/100

### Summary

| Category | Pass | Fail | Warning |
|----------|------|------|---------|
| Meta Tags | X | X | X |
| Headings | X | X | X |
| Images | X | X | X |
| Open Graph | X | X | X |
| Twitter Card | X | X | X |
| Technical | X | X | X |
| Links | X | X | X |

### Critical Issues (must fix)

| # | Issue | Page | Source | Fix |
|---|-------|------|--------|-----|
| 1 | Missing meta description | /pricing | `src/app/pricing/page.tsx` | Add description via metadata export |
| 2 | No H1 tag | /about | `src/app/about/page.tsx:12` | Change H2 "About Us" to H1 |

### Warnings (should fix)

| # | Issue | Page | Source | Fix |
|---|-------|------|--------|-----|
| 1 | Title too long (73 chars) | /blog/post-1 | `src/app/blog/[slug]/page.tsx:8` | Trim to under 60 chars |

### Info (nice to have)

| # | Issue | Page |
|---|-------|------|
| 1 | No structured data | /pricing |

### Cross-Page Issues
- Duplicate meta description on /blog/post-1 and /blog/post-2
- Inconsistent trailing slashes: /about/ vs /pricing

### Per-Page Detail

#### / (Score: 85/100)
- Title: "Acme Inc — Build Better Software" (34 chars) — PASS
- Description: "Acme helps teams..." (142 chars) — PASS
- H1: "Build Better Software" — PASS
- OG: all tags present — PASS
- Images: 2/3 have alt — FAIL

[Repeat for each page]
```

## 8. Auto-Fix (if --fix)

If `--fix` is set, fix **safe** issues only:

**Auto-fixable:**
- Missing `<meta name="viewport">` — add standard viewport tag
- Missing `<html lang>` — add `lang="en"` (or infer from content)
- Missing `<meta charset>` — add `charset="utf-8"`
- Missing image `alt` — add empty `alt=""` for decorative, flag functional images for manual alt
- Heading hierarchy gaps — adjust heading levels to fix nesting
- Missing canonical — add self-referencing canonical

**NOT auto-fixable (report only):**
- Meta description content — requires understanding of page purpose
- Title tag content — requires keyword research
- OG image — requires actual image asset
- Structured data — requires content analysis (use `/structured-data` skill instead)

After fixing, re-run the audit on fixed pages and report before/after scores.

## 9. Clean Up

Remove temporary scripts.

## Rules

- Audit the rendered page, not just source — SPAs may inject meta tags at runtime
- Flag severity correctly: missing title is Critical, missing OG is Warning, URL style is Info
- Include source file paths for every issue — developers fix source, not HTML output
- Report what passes too — gives confidence in coverage
- Check cross-page issues (duplicate titles, inconsistent patterns) not just per-page
- Do not auto-fix content decisions (descriptions, titles, OG images)
- If `--fix` is not set, do not modify any files
- Use actual character counts, not estimates
- Check robots.txt only once, not per page
- If the dev server is not running, say so — do not start it automatically
