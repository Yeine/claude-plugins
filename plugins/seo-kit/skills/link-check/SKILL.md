# Link Check

Crawl the site and detect broken links, redirect chains, orphan pages, and poor anchor text. Trace broken links to source files for fixing.

## Arguments

`$ARGUMENTS` specifies starting routes and options:
- `/link-check` — crawl from root, internal links only
- `/link-check /blog /docs` — crawl starting from specific sections
- `/link-check --external` — also check external links (slower)
- `/link-check --depth 5` — set max crawl depth (default: 3)
- `/link-check --fix` — fix broken internal links where possible

## 0. Ensure Playwright is Available

1. Check `package.json` for `@playwright/test` or `playwright` in dependencies or devDependencies
2. Look for `playwright.config.ts`, `playwright.config.js`, or `playwright.config.mjs`
3. Run `npx playwright --version` to confirm the binary resolves

**If NOT found:** offer to install (`npm init playwright@latest`). Stop if declined.

## 1. Parse Arguments

Extract from `$ARGUMENTS`:
- **Routes** — starting points for the crawl (default: `/`)
- **--external** — also check external links (default: internal only)
- **--depth N** — max crawl depth (default: 3)
- **--fix** — attempt to fix broken internal links in source

## 2. Determine Base URL

URL discovery:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` for `dev`/`start` scripts
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Discover All Routes from Source

Before crawling, gather the known routes from the project's source so orphan pages can be detected later:

- **Next.js:** list all files in `app/` or `pages/` directory, convert to routes
- **React Router:** grep for `<Route`, `path:`, or `createBrowserRouter` definitions
- **Vue Router:** grep for `routes` arrays
- **Other:** check for route config files

Record all known routes — these will be compared against crawled pages.

## 4. Crawl the Site

Starting from each specified route, crawl breadth-first:

Write and execute a Playwright script that:

1. Maintains a queue of URLs to visit and a set of already-visited URLs
2. For each URL in the queue:
   - Navigate to it with Playwright
   - Record the HTTP status code
   - If a redirect occurred, record the redirect chain (original → ... → final)
   - Extract all `<a href="...">` links with their anchor text
   - Classify each link:
     - **Internal:** same origin — add to crawl queue if not visited and within depth limit
     - **External:** different origin — add to external check list (if `--external`)
     - **Fragment:** `#anchor` — skip
     - **Non-HTTP:** `mailto:`, `tel:`, `javascript:` — skip
   - Record the source page for each link (for tracing)
3. Continue until queue is empty or depth limit is reached
4. Output all discovered data as JSON

**Crawl limits:**
- Max depth: value from `--depth` (default 3)
- Max pages: 100 (prevent runaway crawling — warn if limit reached)
- Timeout per page: 10 seconds

## 5. Check Links

### Internal Links
For each internal link discovered during the crawl:
- Already visited during crawl — use the recorded status code
- If status is 404 or error — mark as broken

### External Links (if --external)
For each external link, check with a HEAD request:
```bash
curl -sL -o /dev/null -w "%{http_code}" -m 10 "URL"
```

- 200–299: OK
- 301, 302, 307, 308: redirect (follow and note the chain)
- 403: possibly blocked (report as warning, not broken)
- 404: broken
- 000 or timeout: unreachable

Rate-limit external checks to avoid being blocked (max 5 concurrent).

### Redirect Chains
For any link that redirects, trace the full chain:
- 1 redirect: normal, just note it
- 2+ redirects: flag as a redirect chain (wastes crawl budget)

## 6. Detect Additional Issues

### Orphan Pages
Compare the known routes (from step 3) against pages that received at least one internal link during the crawl.

Routes with no incoming internal links are orphan pages — they exist in code but are unreachable via navigation.

### Anchor Text Quality
Flag links with unhelpful anchor text:
- Empty text (icon-only links without aria-label)
- Generic: "click here", "read more", "learn more", "here", "link"
- URL as text (bare URLs instead of descriptive text)

### Internal Nofollow
Flag any internal links with `rel="nofollow"` — this is almost always a mistake (it tells search engines not to follow links to your own pages).

### Mixed Content
Flag any HTTP links on HTTPS pages.

## 7. Trace to Source Files

For each issue, find the source file containing the broken/problematic link:

1. Grep for the exact `href` value in the codebase
2. If found in a component, note the file and line
3. If the link is generated dynamically (from data/CMS), note the component that renders it

## 8. Auto-Fix (if --fix)

If `--fix` is set, fix broken **internal** links where the correct URL can be inferred:

**Auto-fixable:**
- Trailing slash mismatch (`/about/` vs `/about` — normalize to the site's convention)
- Case mismatch (`/About` → `/about`)
- Old route that was renamed (check git log for recent route changes, suggest the new path)
- Simple typos in paths (levenshtein distance 1–2 from a valid route)

**NOT auto-fixable (report only):**
- Broken external links — those are external problems
- Links to deleted content — requires editorial decision
- Links with no clear correct target

After fixing, re-crawl the affected pages to verify.

## 9. Report

```
## Link Check Report

**Pages Crawled:** N
**Links Checked:** X total (Y internal, Z external)
**Crawl Depth:** N

### Summary

| Category | Count |
|----------|-------|
| Broken links (404) | X |
| Redirect chains (2+ hops) | X |
| Orphan pages | X |
| Bad anchor text | X |
| Internal nofollow | X |

### Broken Links

| Source Page | Anchor Text | Target URL | Status | Source File |
|-----------|-------------|-----------|--------|-------------|
| /blog | "old feature" | /blog/deprecated-post | 404 | `src/components/BlogList.tsx:42` |
| /docs | "API reference" | /docs/api-v1 | 404 | `src/app/docs/page.tsx:18` |

### Redirect Chains

| Original URL | Chain | Hops | Source File |
|-------------|-------|------|-------------|
| /about-us | → /about → /company | 2 | `src/components/Footer.tsx:18` |

### Orphan Pages (no incoming links)

| Page | Route Source |
|------|-------------|
| /terms | `src/app/terms/page.tsx` |
| /privacy | `src/app/privacy/page.tsx` |

### Bad Anchor Text

| Source Page | Anchor Text | Target | Source File |
|-----------|-------------|--------|-------------|
| /features | "click here" | /pricing | `src/app/features/page.tsx:55` |
| /blog/intro | "" (empty) | /contact | `src/components/CTA.tsx:12` |

### Internal Nofollow

| Source Page | Target | Source File |
|-----------|--------|-------------|
| /partners | /partner-portal | `src/app/partners/page.tsx:30` |

### Link Health: X% of links are healthy
```

## 10. Clean Up

Remove temporary scripts.

## Rules

- Respect the crawl depth limit — do not crawl infinitely
- Cap at 100 pages — warn if the limit is reached and suggest increasing depth or specifying sections
- Rate-limit external link checks — max 5 concurrent to avoid being blocked
- Do not follow external links during the crawl — only check their status
- Skip fragment-only links (`#section`), `mailto:`, `tel:`, and `javascript:` URLs
- Trace every broken link to a source file — developers need to know where to fix
- For orphan pages, distinguish between truly orphan (no links anywhere) and dynamically linked (linked via JS navigation, search, etc.)
- Do not auto-fix external broken links
- If `--fix` is not set, do not modify any files
- If the dev server is not running, say so — do not start it automatically
- Report the crawl coverage: how many of the known routes were actually reached
