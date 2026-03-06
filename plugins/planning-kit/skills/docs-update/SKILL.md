---
name: docs-update
description: Update documentation, changelogs, and migration notes to reflect a completed plan's changes.
argument-hint: "<slug>"
---

You are in DOCUMENTATION mode. Your job is to bring all project docs in sync with the changes made by a completed plan.

Input: slug = $0

Target: `.claude/plans/<slug>.md`

## 1. Load Context

Read `.claude/plans/<slug>.md`.

**Gate check:** Status MUST be `Completed`. If not, tell the user to finish execution first.

Read `CLAUDE.md` at repo root if it exists — follow project conventions for documentation style.

Gather the changeset:
- Run `git log --oneline` to identify commits from this plan
- Run `git diff <base>..HEAD --name-only` to get the list of changed files
- Read the plan's acceptance criteria and step descriptions to understand what was delivered

Print:
```
Docs update: .claude/plans/<slug>.md
Scanning for documentation to update...
```

## 2. Discover Documentation

Search the project for existing documentation files and patterns:

### 2a. Changelog
Look for (in order): `CHANGELOG.md`, `CHANGES.md`, `HISTORY.md`, `changelog.md`, release notes in `docs/`

### 2b. README
Check if `README.md` (root or relevant subdirectories) references features, APIs, or configuration that was changed by the plan.

### 2c. API documentation
Look for: OpenAPI/Swagger specs (`openapi.yaml`, `swagger.json`), API doc generators (JSDoc, TypeDoc, PHPDoc), `docs/api/` directories, Postman collections.

### 2d. Migration / upgrade guides
Look for: `MIGRATION.md`, `UPGRADING.md`, `docs/migrations/`, version-specific upgrade notes.

### 2e. In-code documentation
Check if changed files have doc comments (JSDoc, docstrings, etc.) that are now outdated.

### 2f. Other docs
Look for: `docs/` directory, wiki references, `CONTRIBUTING.md`, architecture docs (`docs/architecture.md`, ADRs in `docs/adr/`).

Print a summary of what was found:
```
Found:
  Changelog:    CHANGELOG.md
  README:       README.md (references affected features)
  API docs:     docs/api/ (OpenAPI spec)
  Migration:    none
  Other:        docs/architecture.md
```

## 3. Determine What Needs Updating

For each discovered doc, check if the plan's changes make it outdated:

- **Changelog:** Does the project maintain one? If yes, it always needs an entry.
- **README:** Were new features, commands, config options, or setup steps added?
- **API docs:** Were endpoints, parameters, or response shapes changed?
- **Migration guide:** Were there breaking changes, renamed fields, removed features, or changed defaults?
- **In-code docs:** Do doc comments on changed functions still match their behavior?
- **Architecture docs:** Did the plan introduce new components, services, or change data flow?

Present the update plan:
```
Updates needed:
  1. CHANGELOG.md — add entry for <feature summary>
  2. README.md — update CLI usage section
  3. docs/api/openapi.yaml — add new endpoint
  4. No migration guide needed (no breaking changes)
```

Ask the user to confirm before proceeding:
```
Proceed with these updates? (y/n)
```

## 4. Apply Updates

For each doc that needs updating:

### Changelog
- Follow the existing format exactly (keep-a-changelog, custom, etc.)
- Add entry under `Unreleased` or the current version section
- Categorize changes: `Added`, `Changed`, `Fixed`, `Removed`, `Deprecated`
- Reference the plan slug or PR number if the project convention includes references

### README
- Update only the sections affected by the plan
- Match the existing tone, formatting, and level of detail
- Add new sections only if the feature warrants visibility in the README

### API docs
- Update specs to match the actual implementation
- If OpenAPI: update paths, schemas, examples
- If doc comments: update JSDoc/docstrings to match new signatures

### Migration guide
- Only create if there are breaking changes
- Include: what changed, why, and how to migrate (before/after code examples)
- Follow existing migration doc format if one exists

### In-code docs
- Update doc comments on functions/classes whose behavior changed
- Don't add new doc comments to code that didn't have them before

### Architecture docs
- Update diagrams or descriptions if the plan changed system structure
- Keep it factual — document what is, not what should be

## 5. Verify

For each updated doc:
- Re-read the file to confirm the edit is clean and consistent
- If the project has a doc build step (e.g., `npm run docs`, `mkdocs build`), run it and check for errors
- If the project has doc linting (e.g., `markdownlint`), run it

## 6. Summary

Print:
```
Docs update — .claude/plans/<slug>.md
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Updated:
  - CHANGELOG.md (added entry under Unreleased)
  - README.md (updated usage section)
  - docs/api/openapi.yaml (added /widgets endpoint)
Skipped:
  - Migration guide (no breaking changes)
Created:
  - (none)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

## 7. Log

Append to the plan's Decision log:
- `<date> — Docs update completed — <N> files updated`

## Rules

- Follow existing documentation conventions exactly. Match format, tone, heading levels, and structure.
- Never create documentation files that the project doesn't already maintain. If there's no changelog, don't create one — mention it in the summary as a suggestion instead.
- Don't rewrite docs that aren't affected by the plan. Only touch what's outdated.
- Ask before proceeding. Always show the update plan and get confirmation.
- Keep changelog entries concise — one line per change, not paragraphs.
- If you're unsure whether a doc needs updating, include it in the update plan with a note and let the user decide.
