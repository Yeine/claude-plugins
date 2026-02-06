# Create Pull Request

Create a pull request from the current branch with an auto-generated title and description based on the commits and changes.

## Arguments

`$ARGUMENTS` optionally specifies the base branch or flags:
- `/pr` — auto-detect base branch, generate PR
- `/pr develop` — target `develop` as base
- `/pr --draft` — create as draft PR
- `/pr --review` — run `/deep-review` before creating the PR
- `/pr develop --draft` — combined

## 1. Gather Context

Run these in parallel:

```bash
git branch --show-current
git remote show origin | sed -n '/HEAD branch/s/.*: //p'
git status --porcelain
```

If `$ARGUMENTS` specifies a base branch, use it. Otherwise use the auto-detected default.

Check that:
- The working tree is clean — if there are uncommitted changes, warn the user and stop
- The current branch is NOT the base branch — if it is, warn and stop
- The branch has been pushed — if not, push it with `git push -u origin HEAD`

## 2. Analyze the Changes

Run these in parallel:

```bash
git log BASE..HEAD --oneline
git log BASE..HEAD --format='%s%n%n%b'
git diff BASE..HEAD --stat
git diff BASE..HEAD --name-status
```

Read the full diff to understand what changed:
```bash
git diff BASE..HEAD
```

Also check for project PR conventions:
- Read `CLAUDE.md` at repo root if it exists — look for PR templates or conventions
- Check `.github/PULL_REQUEST_TEMPLATE.md` if it exists

## 3. Optional: Deep Review

If `--review` flag is set, run a deep review of the branch before creating the PR. Use the same analysis criteria as the `/deep-review` skill.

If critical issues are found, present them and ask the user whether to proceed with PR creation or fix the issues first.

## 4. Generate PR Title and Description

**Title:**
- If the branch has a single commit, use its message as the title
- If multiple commits, synthesize a title that captures the overall change
- Keep under 72 characters
- Use conventional format if commits follow it: `feat: add rate limiting`

**Description:**
Build from the commit messages and diff analysis:

```markdown
## Summary
- [2-4 bullet points describing what changed and why]

## Changes
- [Grouped by logical area — not a raw file list]

## Test Plan
- [How to verify these changes work — based on what tests exist, what was added]
```

If a PR template exists, use its structure instead.

## 5. Create the PR

```bash
gh pr create --title "TITLE" --base BASE --body "$(cat <<'EOF'
BODY_CONTENT
EOF
)"
```

Add `--draft` flag if `--draft` was in `$ARGUMENTS`.

## 6. Summary

Print:

```
PR created: #NUMBER
URL: https://github.com/owner/repo/pull/NUMBER
Base: BASE ← BRANCH
Commits: N | Files: M | +X/-Y lines
```

Do NOT add reviewers or labels unless the user explicitly asks.

## Rules

- Never create a PR from a dirty working tree
- Never create a PR targeting the current branch
- Push the branch before creating the PR if it hasn't been pushed
- Use the project's PR template if one exists
- Do not add "Generated with Claude" or similar attribution
- Do not assign reviewers or labels automatically
- Keep the description concise — developers read PRs, not novels
