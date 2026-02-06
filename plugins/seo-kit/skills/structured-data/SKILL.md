# Structured Data

Add or validate JSON-LD structured data on pages. Auto-detects the appropriate schema.org type from page content, generates the JSON-LD, and writes it into source files.

## Arguments

`$ARGUMENTS` specifies routes and options:
- `/structured-data /blog/my-post` — add structured data to a specific page
- `/structured-data /pricing /about /blog` — add to multiple pages
- `/structured-data --validate` — check existing structured data without adding new
- `/structured-data /product/widget --type Product` — force a specific schema type

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes/URLs** — one or more paths or full URLs
- **--validate** — only validate existing structured data, do not add new
- **--type TypeName** — force a specific schema.org type (overrides auto-detection)

If no routes provided:
> Please specify which routes to add structured data to. Example: `/structured-data /blog/my-post /about`

## 2. Determine Target URLs

Use provided routes. Prepend the base URL to relative routes.

URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` for `dev`/`start` scripts
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Check Existing Structured Data

For each target URL, navigate with Playwright and:

1. Extract all `<script type="application/ld+json">` elements
2. Parse each as JSON
3. Record the `@type`, properties, and any validation issues

If `--validate` is set, proceed to step 7 (validation report) — skip generation.

## 4. Analyze Page Content

For each page without existing structured data (or with incomplete data), extract:

1. **Page type indicators:**
   - Article/blog: publication date, author, article body, categories/tags
   - Product: price, availability, reviews/ratings, SKU, brand
   - FAQ: question/answer pairs (accordion, definition lists, Q&A sections)
   - HowTo: numbered steps, time estimates, materials lists
   - Breadcrumb: breadcrumb navigation trail
   - Organization: logo, contact info, social profiles, address
   - Local business: address, phone, hours, map
   - Event: date, location, ticket info
   - Recipe: ingredients, instructions, cook time, nutrition

2. **Content extraction:**
   - Headings and their text
   - Paragraph text (for descriptions)
   - Images with alt text (for image properties)
   - Dates (publication, modification)
   - Author information
   - Prices and currency
   - Ratings/reviews
   - Lists (ordered/unordered — may indicate steps or items)

## 5. Determine Schema Type

If `--type` is specified, use that type.

Otherwise, auto-detect based on content analysis:

| Content Signals | Schema Type |
|----------------|-------------|
| Article date, author, body text | `Article` or `BlogPosting` |
| Price, availability, product images | `Product` |
| Question/answer pairs | `FAQPage` |
| Numbered steps, instructions | `HowTo` |
| Breadcrumb navigation | `BreadcrumbList` |
| Company info, logo, social links | `Organization` |
| Address, phone, hours | `LocalBusiness` |
| Event date, venue, tickets | `Event` |
| Recipe steps, ingredients | `Recipe` |
| Home page / general | `WebSite` + `Organization` |

If the type is ambiguous, present options to the user:
> This page could be an Article or a HowTo. Which fits better?

A single page can have multiple schemas (e.g., `BreadcrumbList` + `Article`).

## 6. Generate JSON-LD

For each page, generate valid JSON-LD following schema.org specifications.

### Article / BlogPosting
```json
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "[from H1]",
  "description": "[from meta description or first paragraph]",
  "image": "[primary image URL]",
  "datePublished": "[from page content]",
  "dateModified": "[if available]",
  "author": {
    "@type": "Person",
    "name": "[author name]"
  },
  "publisher": {
    "@type": "Organization",
    "name": "[site name]",
    "logo": { "@type": "ImageObject", "url": "[logo URL]" }
  }
}
```

### Product
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "[product name]",
  "description": "[product description]",
  "image": "[product image]",
  "offers": {
    "@type": "Offer",
    "price": "[extracted price]",
    "priceCurrency": "[currency code]",
    "availability": "https://schema.org/InStock"
  }
}
```

### FAQPage
```json
{
  "@context": "https://schema.org",
  "@type": "FAQPage",
  "mainEntity": [
    {
      "@type": "Question",
      "name": "[question text]",
      "acceptedAnswer": {
        "@type": "Answer",
        "text": "[answer text]"
      }
    }
  ]
}
```

Follow the same patterns for HowTo, BreadcrumbList, Organization, LocalBusiness, Event, and Recipe — using schema.org required and recommended properties.

### Validation Rules
Before writing, validate the generated JSON-LD:
- Valid JSON syntax
- `@context` is `https://schema.org`
- `@type` is a recognized schema.org type
- All required properties for the type are present
- URLs are absolute, not relative
- Dates are in ISO 8601 format
- No empty or placeholder values

## 7. Discover Where to Place JSON-LD in Source

Same meta tag pattern discovery as `meta-gen`:
- **Next.js App Router:** add to metadata export or as a `<script>` in the page component
- **Next.js Pages Router:** add inside `<Head>` component
- **React (other):** add via Helmet or as a component in the page
- **Vue/Nuxt:** add via `useHead()` with `script` property
- **Svelte:** add inside `<svelte:head>`
- **Static HTML:** add directly in `<head>`

If the project already has a pattern for structured data (e.g., a `JsonLd` component), use that.

## 8. Write to Source (unless --validate)

If `--validate`, skip this step.

For each page:
1. Open the source file
2. Add the JSON-LD following the project's pattern
3. If structured data already exists, merge or replace (preserving manually added properties)

## 9. Verify

After writing, navigate to each page and confirm:
- The `<script type="application/ld+json">` is present in the rendered HTML
- The JSON is valid (parseable)
- The `@type` matches what was intended

## 10. Report

### If --validate:

```
## Structured Data Validation Report

| Route | Type | Status | Issues |
|-------|------|--------|--------|
| /blog/intro | Article | Valid | 0 |
| /pricing | Product | Invalid | 2 errors |
| /about | None | Missing | — |

### Issues

#### /pricing — Product
1. Missing required property `offers.price`
2. `image` is a relative URL (should be absolute)

### Rich Results Eligibility
| Route | Type | Eligible | Missing for Eligibility |
|-------|------|----------|------------------------|
| /blog/intro | Article | Yes | — |
| /pricing | Product | No | Fix `offers.price` and `image` |
```

### If generating:

```
## Structured Data Added

| Route | Type | Properties | Source | Rich Result |
|-------|------|-----------|--------|-------------|
| /blog/intro | Article | headline, author, datePublished, image | `src/app/blog/[slug]/page.tsx` | Article snippet |
| /pricing | FAQPage | 5 Q&A pairs | `src/app/pricing/page.tsx` | FAQ rich result |
| / | Organization | name, logo, url, sameAs | `src/app/layout.tsx` | Knowledge panel |

### /blog/intro — Article
Source: `src/app/blog/[slug]/page.tsx:45`
Properties: headline, description, image, datePublished, dateModified, author, publisher
Rich Result: Article snippet in search results
```

## 11. Clean Up

Remove temporary scripts.

## Rules

- Extract data from the actual rendered page — do not hardcode or fabricate values
- Use absolute URLs for all URL properties (images, pages)
- Dates must be ISO 8601 format
- A page can have multiple schema types (e.g., BreadcrumbList + Article) — add all appropriate types
- Do not add structured data that contains placeholder or empty values — every property must have real content
- Follow the project's existing structured data pattern if one exists
- If `--validate` is set, do not modify any files
- If `--type` is specified but doesn't match the page content, warn the user
- Check Google's Rich Results documentation for eligibility requirements per type
- If the dev server is not running, say so — do not start it automatically
