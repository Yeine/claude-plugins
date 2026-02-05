# Claude Code Plugin Marketplace

A collection of reusable [Claude Code](https://code.claude.com) skills for everyday developer workflows. Technology-agnostic — works with any language, framework, or project.

## Plugins

### dev-workflow

Daily development skills for commits, reviews, PRs, and quality checks.

| Skill | Description |
|-------|-------------|
| `smart-commit` | Atomic conventional commits — analyzes diffs, splits by domain, presents a plan before committing. Flags: `--quick`, `--push` |
| `deep-review` | Rigorous code review of the current branch. Scales sub-agents by diff size. Produces a structured report with severity levels |
| `review-feedback` | Fetches unresolved PR review threads (any reviewer), analyzes them, optionally fixes + commits + replies + resolves. Flag: `--author=` |
| `session` | Sets up a working branch and starts the dev environment in seconds. Infers branch prefix from first word (`fix`, `feat`, `refactor`...) |
| `pr` | Creates a PR with auto-generated title and description from commits and diffs. Flags: `--draft`, `--review` |
| `quality-gate` | Discovers and runs all project quality tools (linter, static analysis, tests) in order. Flags: `--fix`, `--changed` |
| `resolve-conflicts` | Walks through merge conflicts file by file with targeted edits and verification |
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

Generates task files for [Ralph](https://github.com/Yeine/ralph), an autonomous AI loop that runs Claude Code repeatedly to complete multi-step improvement tasks.

| Skill | Description |
|-------|-------------|
| `ralph-task` | Analyzes the codebase and generates prioritized task files. Categories: `security`, `test-coverage`, `code-quality`, `architecture`, `tech-debt`, `custom` |

## Installation

```bash
# Add the marketplace
/plugin marketplace add Yeine/claude-plugins

# Install the plugins you want
/plugin install dev-workflow@ychabot-plugins
/plugin install planning-kit@ychabot-plugins
/plugin install ralph-task@ychabot-plugins
```

## Usage

After installation, skills are namespaced by plugin:

```bash
# Dev workflow
/dev-workflow:smart-commit --quick
/dev-workflow:deep-review develop
/dev-workflow:session fix broken login page
/dev-workflow:pr --draft
/dev-workflow:quality-gate --fix
/dev-workflow:review-feedback --author=coderabbitai

# Planning pipeline
/planning-kit:intake add rate limiting to public API
/planning-kit:planfile api-rate-limit add rate limiting
/planning-kit:plancheck api-rate-limit
/planning-kit:approve api-rate-limit
/planning-kit:execute api-rate-limit

# Ralph tasks
/ralph-task:ralph-task security
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
