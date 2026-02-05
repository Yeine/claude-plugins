# Smart Commit

Create atomic conventional commits from the current working tree changes. Analyze diffs, split by domain, present a plan, and commit after approval.

## Arguments

`$ARGUMENTS` supports these flags:
- `/smart-commit` — full workflow: analyze, plan, approve, commit
- `/smart-commit --quick` — single commit, skip the plan/approval step
- `/smart-commit --push` — push to remote after committing
- `/smart-commit --quick --push` — single commit and push

## Commit Message Format

```
<type>(<scope>): <description>

<optional body>
```

**Types:**
- `feat` — new feature
- `fix` — bug fix
- `refactor` — code restructuring (no behavior change)
- `perf` — performance improvement
- `test` — adding or updating tests
- `docs` — documentation changes
- `chore` — maintenance (dependencies, build, config)
- `style` — formatting, whitespace, semicolons (no logic change)

**Scope** is optional. Auto-detect from file paths when changes cluster in one area:
- All files in `auth/` → `feat(auth): ...`
- All files in `api/payments/` → `fix(payments): ...`
- Files spread across many areas → no scope, just `feat: ...`

**Rules:**
- Max 72 characters for the description line
- Present tense, lowercase after the colon: "add" not "Added"
- NEVER include "Co-Authored-By: Claude" or any AI attribution
- NEVER use `--no-verify` to skip hooks

## 1. Gather Context

Run these in parallel:

```bash
git status --porcelain
git diff --stat
git diff --staged --stat
git log --oneline -20
```

This gives you: untracked files, unstaged changes, staged changes, and recent commit style.

## 2. Check for Suspicious Files

Scan the changed/untracked files for anything that should NOT be committed:
- Credentials or secrets (`.env`, `*.key`, `*.pem`, `credentials.*`)
- Large binaries, build artifacts, `node_modules/`, `vendor/`
- Temporary or debug files (`.log`, `.tmp`, `.DS_Store`)

If found, **flag them to the user** with a recommendation (add to `.gitignore`, or just skip). Do NOT delete or ignore them silently — let the user decide.

## 3. Read the Diffs

For every changed file, read the actual diff to understand what changed:

```bash
git diff HEAD -- <file>
```

For untracked files, read their content. You cannot write a good commit message without understanding the changes.

## 4. Plan Commits

Group changes into atomic commits. Each commit should represent **one logical change**.

**Grouping strategy — by domain/feature, not file proximity:**
- Changes that implement one feature together → one commit
- A bug fix and a separate refactor → two commits
- Test files go with the code they test, not in a separate "test" commit
- Config/dependency changes that enable a feature → same commit as the feature

**If `--quick` flag is set:** Skip the plan. Create a single commit with all changes. Jump to step 6.

## 5. Present Plan and Wait for Approval

Show the plan in this format:

```
## Commit Plan

### Commit 1: feat(auth): add login rate limiting
**Files:** (+42/-8)
- src/middleware/rate-limit.ts (+35/-0)
- src/routes/auth.ts (+7/-8)

**Why:** Adds rate limiting to the login endpoint to prevent brute force attacks.

### Commit 2: test: add rate limiting test suite
**Files:** (+67/-0)
- tests/middleware/rate-limit.test.ts (+67/-0)

**Why:** Covers the new rate limiting middleware with unit tests.

---
Approve this commit plan?
```

Wait for the user to approve, modify, or reject before proceeding.

## 6. Execute Commits

For each planned commit:

1. Stage only the files for that commit: `git add <file1> <file2> ...`
2. Create the commit with the message. For multi-file commits with non-obvious changes, add a body:
   ```
   feat(auth): add login rate limiting

   - Add RateLimitMiddleware with sliding window algorithm
   - Apply rate limit of 5 attempts per minute on /login
   - Return 429 with Retry-After header when exceeded
   ```
3. Verify the commit was created: `git log -1 --oneline`

For simple single-file commits, a description-only message (no body) is fine.

## 7. Post-Commit

After all commits are created:

- Show a summary: number of commits, files touched, lines changed
- If `--push` flag was set, push to the current branch's remote
- If no `--push` flag, do NOT push — just confirm the commits are ready

```
3 commits created:
  abc1234 feat(auth): add login rate limiting
  def5678 test: add rate limiting test suite
  ghi9012 docs: update API rate limiting documentation

Not pushed. Run `git push` when ready.
```
