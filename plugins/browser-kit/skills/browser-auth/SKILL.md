---
name: browser-auth
description: Authenticate once and save the session for all browser-kit skills. Supports interactive login and config-based automation.
argument-hint: "[url] [--interactive] [--reset]"
---

Save an authenticated browser session so all other browser-kit skills can access protected pages.

## Arguments

`$ARGUMENTS` specifies options:
- `/browser-auth` — auto-detect mode (use config if exists, otherwise interactive)
- `/browser-auth http://localhost:3000` — specify the app URL
- `/browser-auth --interactive` — force interactive mode (manual login)
- `/browser-auth --reset` — delete saved auth state and re-authenticate

## 1. Check Existing Auth State

Look for saved auth state at `.browser-auth-state.json` in the project root.

If it exists and `--reset` is NOT set:
1. Write a quick Playwright script that loads the state and navigates to the app
2. Check if the session is still valid (no redirect to login, no 401)
3. If valid:
   ```
   Auth state is still valid. Session loaded successfully.
   To force re-authentication: /browser-auth --reset
   ```
   Stop here.
4. If expired, continue to re-authenticate.

If `--reset` is set, delete `.browser-auth-state.json` and continue.

## 2. Determine Target URL

If a URL was provided in arguments, use it.

Otherwise, discover the app URL:
1. Check `playwright.config` for `use.baseURL`
2. Check `CLAUDE.md` for documented dev server URLs
3. Check `package.json` `scripts` for `dev`/`start`
4. Probe common ports: 3000, 3001, 4200, 5173, 5174, 8000, 8080

If nothing responds:
> No running dev server detected. Please start your dev server and re-run.

## 3. Choose Auth Mode

### 3a. Check for config file

Look for `.claude/browser-auth.yaml` or `.claude/browser-auth.json`.

Config format (YAML example):
```yaml
url: http://localhost:3000/login
steps:
  - fill: "[name=email]"
    value: "${APP_EMAIL}"         # reads from environment variable
  - fill: "[name=password]"
    value: "${APP_PASSWORD}"      # reads from environment variable
  - click: "button[type=submit]"
wait_for: "url:**/dashboard*"    # glob pattern — auth is done when URL matches
```

Config format (JSON example):
```json
{
  "url": "http://localhost:3000/login",
  "steps": [
    { "fill": "[name=email]", "value": "${APP_EMAIL}" },
    { "fill": "[name=password]", "value": "${APP_PASSWORD}" },
    { "click": "button[type=submit]" }
  ],
  "wait_for": "url:**/dashboard*"
}
```

**Step types:**
- `fill` — fill a form field (selector + value)
- `click` — click an element (selector)
- `wait` — wait for a selector to appear
- `navigate` — go to a URL

**Variable resolution:** Values starting with `${...}` are resolved from environment variables. If the env var is not set, stop and tell the user:
> Environment variable `APP_PASSWORD` is not set. Set it before running browser-auth.

**If config exists and `--interactive` is NOT set:** use config mode (step 4).
**If `--interactive` is set or no config exists:** use interactive mode (step 5).

## 4. Config-Based Authentication

1. Navigate to the config's `url`
2. Execute each step in order:
   - `fill`: `page.fill(selector, resolvedValue)`
   - `click`: `page.click(selector)`
   - `wait`: `page.waitForSelector(selector)`
   - `navigate`: `page.goto(url)`
3. Wait for the `wait_for` condition:
   - `url:pattern` — wait for URL to match the glob pattern
   - `selector:css` — wait for a CSS selector to appear
4. If any step fails, report the error and suggest using `--interactive` mode instead

After successful login, proceed to step 6 (Save State).

## 5. Interactive Authentication

Print:
```
Opening browser for manual login...

1. A browser window will open at <target URL>
2. Log in normally (SSO, MFA, whatever your app requires)
3. Once you are logged in and see your app's authenticated page, come back here and confirm

Waiting for you to log in...
```

1. Launch Playwright in **headed** mode (`headless: false`)
2. Navigate to the target URL
3. Wait for the user to confirm they've logged in (use `page.pause()` or ask the user to press Enter)
4. After confirmation, verify the session is valid:
   - Check that the URL is no longer the login page
   - Check for typical authenticated indicators (navigation menus, user profile elements, dashboard content)
5. If it looks like the user is still on the login page, ask them to try again

Proceed to step 6 (Save State).

## 6. Save Auth State

1. Save the browser context's storage state:
   ```js
   await context.storageState({ path: '.browser-auth-state.json' })
   ```
2. Verify the file was written and is non-empty
3. Check if `.browser-auth-state.json` is in `.gitignore`
   - If not, warn:
     ```
     WARNING: .browser-auth-state.json contains session cookies.
     Add it to .gitignore to avoid committing credentials:
       echo '.browser-auth-state.json' >> .gitignore
     ```
     Offer to add it automatically.

Print:
```
Auth state saved to .browser-auth-state.json
All browser-kit skills will now use this session automatically.

To re-authenticate:  /browser-auth --reset
To set up automated login:  create .claude/browser-auth.yaml
```

## 7. Generate Config Template (if interactive and no config exists)

If the user just did an interactive login and no `.claude/browser-auth.yaml` exists, offer:
```
Would you like me to generate a browser-auth config for automated login next time?
I'll create .claude/browser-auth.yaml with your login page's form fields.
Credentials will reference environment variables, not stored in plain text.
```

If the user agrees:
1. Navigate back to the login page
2. Inspect the form: find input selectors, submit button
3. Generate the YAML config with `${ENV_VAR}` placeholders
4. Write to `.claude/browser-auth.yaml`
5. Tell the user which env vars to set

## Rules

- NEVER store credentials in plain text. Config files must use `${ENV_VAR}` references.
- NEVER commit auth state files. Always check and warn about `.gitignore`.
- The auth state file (`.browser-auth-state.json`) must be in the project root so all skills can find it.
- If the session has expired, re-authenticate automatically — don't make the user figure out why skills are failing.
- Interactive mode must use headed browser (`headless: false`) — the user needs to see and interact with it.
- Support any auth flow in interactive mode: username/password, SSO, OAuth, MFA, certificate-based — the user handles it manually.
- Config mode is for simple flows only. If the login requires MFA or captcha, tell the user to use interactive mode.
