# Claude Code Plugin Marketplace

> Ship faster with battle-tested Claude Code skills — smart commits, deep reviews, PR automation, structured planning, and autonomous refactoring. One install, any project.

A collection of 26 reusable [Claude Code](https://code.claude.com) skills organized into 7 plugins. Technology-agnostic — works with any language, framework, or project.

## Plugins

### git-kit

Pure local git operations — no GitHub API needed.

| Skill | Description |
|-------|-------------|
| `smart-commit` | Atomic conventional commits — analyzes diffs, splits by domain, presents a plan before committing. Flags: `--quick`, `--push` |
| `resolve-conflicts` | Walks through merge conflicts file by file with targeted edits and verification |

### pr-kit

Pull request skills powered by GitHub CLI.

| Skill | Description |
|-------|-------------|
| `pr` | Creates a PR with auto-generated title and description from commits and diffs. Flags: `--draft`, `--review` |
| `deep-review` | Rigorous code review of the current branch. Scales sub-agents by diff size. Produces a structured report with severity levels |
| `review-feedback` | Fetches unresolved PR review threads (any reviewer), analyzes them, optionally fixes + commits + replies + resolves. Flag: `--author=` |

### dev-workflow

Dev process and methodology skills.

| Skill | Description |
|-------|-------------|
| `session` | Sets up a working branch and starts the dev environment in seconds. Infers branch prefix from first word (`fix`, `feat`, `refactor`...) |
| `quality-gate` | Discovers and runs all project quality tools (linter, static analysis, tests) in order. Flags: `--fix`, `--changed` |
| `test-debug` | Test-driven bug fixing: trace root cause, write failing test, check impact, fix, validate |

### planning-kit

A structured planning pipeline for complex tasks. Best used with a [safe planning session](#safe-planning-mode).

| Skill | Description |
|-------|-------------|
| `intake` | Turns a vague goal into clarifying questions, assumptions, and testable acceptance criteria |
| `planfile` | Creates a versioned implementation plan with steps, verification commands, risks, and rollback |
| `plancheck` | Reviews a plan like a senior engineer — assigns P0/P1/P2 issues and auto-fixes what it can |
| `approve` | Validates completeness and freezes the plan for execution |
| `execute` | Executes an approved plan step by step, verifying each before proceeding. Flags: `--step N`, `--continue` |

### ralph-task

Generates task files for [Ralph](https://github.com/Yeine/ralph), an autonomous AI loop that runs Claude Code repeatedly to complete multi-step improvement tasks. Requires [Yeine/ralph](https://github.com/Yeine/ralph) to be installed.

| Skill | Description |
|-------|-------------|
| `ralph-task` | Analyzes the codebase and generates prioritized task files. Categories: `security`, `test-coverage`, `code-quality`, `architecture`, `tech-debt`, `custom` |

### browser-kit

Browser automation skills powered by Playwright — E2E testing, accessibility, visual regression, debugging, and UI design.

| Skill | Description |
|-------|-------------|
| `e2e-gen` | Generate Playwright E2E tests from plain-English flow descriptions. Navigates the running app, discovers selectors, validates the test |
| `a11y-audit` | Accessibility audit using axe-core with WCAG compliance. Traces violations to source files. Flags: `--fix`, `--standard` |
| `browser-debug` | Investigate browser bugs: captures console errors, network failures, JS exceptions, screenshots. Traces root cause to source. Flag: `--trace` |
| `visual-regression` | Screenshot diff testing against stored baselines. Reports visual changes with source file tracing. Flags: `--update`, `--threshold`, `--mobile` |
| `page-gen` | Generate page components from plain-English descriptions using the project's existing stack and design system. Flag: `--route` |
| `screenshot-to-code` | Reverse-engineer a mockup/screenshot into working code using existing project components. Flag: `--route` |
| `design-review` | UI critique: spacing inconsistencies, alignment issues, typography problems, design system violations at desktop and mobile viewports. Flag: `--fix` |
| `clone-page` | Recreate a page from a public URL using the project's own components, palette, and conventions. Flag: `--route` |

### seo-kit

SEO analysis and generation skills powered by Playwright — audits, meta tags, structured data, and link checking.

| Skill | Description |
|-------|-------------|
| `seo-audit` | Full SEO audit: meta tags, headings, images, Open Graph, canonical, robots, structured data. Scores pages out of 100. Flags: `--fix`, `--full-crawl` |
| `meta-gen` | Generate optimized title, description, OG, and Twitter Card tags from actual page content. Framework-aware (Next.js, React Helmet, Vue, Svelte). Flag: `--dry-run` |
| `structured-data` | Add or validate JSON-LD structured data. Auto-detects schema type (Article, Product, FAQ, HowTo, etc.) from content. Flags: `--validate`, `--type` |
| `link-check` | Crawl the site for broken links, redirect chains, orphan pages, and bad anchor text. Flags: `--external`, `--depth`, `--fix` |

## Installation

```bash
# Add the marketplace
/plugin marketplace add Yeine/claude-plugins

# Install the plugins you want
/plugin install git-kit@ychabot-plugins
/plugin install pr-kit@ychabot-plugins
/plugin install dev-workflow@ychabot-plugins
/plugin install planning-kit@ychabot-plugins
/plugin install ralph-task@ychabot-plugins
/plugin install browser-kit@ychabot-plugins
/plugin install seo-kit@ychabot-plugins
```

## Usage

After installation, skills are namespaced by plugin:

```bash
# Git kit
/git-kit:smart-commit --quick
/git-kit:resolve-conflicts

# PR kit
/pr-kit:pr --draft
/pr-kit:deep-review develop
/pr-kit:review-feedback --author=coderabbitai

# Dev workflow
/dev-workflow:session fix broken login page
/dev-workflow:quality-gate --fix
/dev-workflow:test-debug users can't log in after password reset

# Planning pipeline
/planning-kit:intake add rate limiting to public API
/planning-kit:planfile api-rate-limit add rate limiting
/planning-kit:plancheck api-rate-limit
/planning-kit:approve api-rate-limit
/planning-kit:execute api-rate-limit

# Ralph tasks
/ralph-task:ralph-task security

# Browser kit
/browser-kit:e2e-gen user logs in and sees the dashboard
/browser-kit:a11y-audit /login /dashboard --fix
/browser-kit:browser-debug clicking submit shows a blank page
/browser-kit:visual-regression --update
/browser-kit:page-gen a settings page with sidebar and profile form
/browser-kit:screenshot-to-code ./designs/mockup.png
/browser-kit:design-review /login /dashboard
/browser-kit:clone-page https://example.com/pricing --route /pricing

# SEO kit
/seo-kit:seo-audit /pricing /blog --fix
/seo-kit:meta-gen /pricing /about --dry-run
/seo-kit:structured-data /blog/my-post
/seo-kit:link-check --external --depth 5
```

## Safe Planning Mode

The planning-kit works best in a restricted session that prevents accidental code changes during planning:

1. Create `~/.claude/settings.planning.json`:

```json
{
  "permissions": {
    "defaultMode": "dontAsk",
    "allow": [
      "Read", "Grep", "Glob",
      "Edit(.claude/plans/**)",
      "Write(.claude/plans/**)",
      "Bash(mkdir *)",
      "Task(Explore)"
    ],
    "deny": [
      "Read(./.env)", "Read(./.env.*)", "Read(./secrets/**)"
    ]
  }
}
```

2. Create an alias:

```bash
alias cplan='claude --permission-mode dontAsk --settings ~/.claude/settings.planning.json'
```

3. Use the full pipeline:

```bash
cplan                                    # safe mode — can only read code and edit plans
/planning-kit:intake redesign pricing
/planning-kit:planfile pricing-v2 ...
/planning-kit:plancheck pricing-v2
/planning-kit:approve pricing-v2
exit

claude                                   # normal mode — execute the plan
/planning-kit:execute pricing-v2
```

## Team Setup

Add this to your project's `.claude/settings.json` so teammates get prompted to install automatically:

```json
{
  "extraKnownMarketplaces": {
    "ychabot-plugins": {
      "source": {
        "source": "github",
        "repo": "Yeine/claude-plugins"
      }
    }
  },
  "enabledPlugins": {
    "dev-workflow@ychabot-plugins": true
  }
}
```

## License

MIT
