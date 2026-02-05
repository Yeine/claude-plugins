# Start Dev Session

Set up a working branch and start the dev environment. Be fast and opinionated — no unnecessary questions.

## Arguments

`$ARGUMENTS` describes what you're working on. The first word determines the branch prefix:
- `/session fix broken login page` → branch `fix/broken-login-page`
- `/session feature add dark mode` → branch `feature/add-dark-mode`
- `/session refactor auth middleware` → branch `refactor/auth-middleware`
- `/session` → start dev environment only, ask what to work on for branch

## 1. Check for Dirty Working Tree

```bash
git status --porcelain
```

If there are uncommitted changes, **stop and warn the user**. Do not stash, commit, or switch branches. Tell them what's dirty and let them decide how to handle it.

If the tree is clean, proceed.

## 2. Determine Branch Name

If `$ARGUMENTS` is empty, skip to step 5 (dev environment only). Ask the user what they want to work on before creating a branch.

Otherwise, parse the arguments:

**Extract prefix from first word:**

| First word | Branch prefix |
|---|---|
| fix, bug, bugfix | `fix/` |
| feat, feature, add | `feature/` |
| hotfix | `hotfix/` |
| refactor, refact | `refactor/` |
| chore | `chore/` |
| docs | `docs/` |
| test | `test/` |
| anything else | `feature/` |

**Build branch name:**
- Strip the prefix word from the rest of the arguments
- Convert remaining words to kebab-case
- Example: `fix broken login page` → `fix/broken-login-page`
- Example: `add user authentication` → `feature/user-authentication`

## 3. Create Branch from Latest Base

Auto-detect the base branch:
```bash
git remote show origin | sed -n '/HEAD branch/s/.*: //p'
```
Fall back to `develop`, `main`, or `master` (first found) if that fails.

Fetch and create the branch:
```bash
git fetch origin
git checkout -b BRANCH_NAME origin/BASE
git push -u origin BRANCH_NAME
```

## 4. Start Dev Environment

Detect and start the dev environment using this priority:

1. **CLAUDE.md** — read `CLAUDE.md` at repo root. Look for startup commands (e.g., `make up`, `docker compose up`, `npm run dev`). Use what the project documents.
2. **Makefile** — check for targets: `up`, `start`, `dev`. Run the first one found.
3. **docker-compose.yml** — if present, run `docker compose up -d`.
4. **package.json** — check for scripts: `dev`, `start`, `serve`. Run the first one found.
5. **Nothing found** — skip and report "No dev environment detected."

Wait a few seconds after starting, then verify the environment is running (check process/container status). Do not exhaustively report every port and container — just confirm it's up.

## 5. Summary

Print a concise summary:

```
Branch: fix/broken-login-page (from develop)
Environment: running (docker compose)
Ready to work.
```

Or if environment only:

```
Environment: running (docker compose)
Branch: develop (no task branch created)
```

That's it. No further questions. Start working.

## Rules

- Be fast — this should complete in seconds, not minutes
- Do not ask "should I create the branch?" — the user already told you by providing arguments
- Do not spawn sub-agents for environment detection — just check files directly
- Do not report every running container and port — just confirm up or down
- If branch already exists remotely, warn and ask before overwriting
- Never force-push or delete existing branches
