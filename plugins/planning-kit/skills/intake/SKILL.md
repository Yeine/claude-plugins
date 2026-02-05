---
name: intake
description: Turn a vague goal into clear requirements and acceptance criteria. No implementation.
disable-model-invocation: true
allowed-tools: Read, Grep, Glob, Edit, Bash(mkdir -p .claude/plans)
argument-hint: "<goal>"
---

You are in REQUIREMENTS mode. Do NOT propose implementation or code changes.

Goal: $ARGUMENTS

## 1. Load Project Context

Read `CLAUDE.md` at the repo root if it exists. Note:
- Architecture patterns and conventions
- Existing components and services
- Security and testing requirements
- Any constraints that affect the goal

## 2. Generate Clarifying Questions

Ask 6-12 questions grouped by:

### Users / UX
- Who is affected? What's the user-facing behavior?

### Inputs / Outputs / Data
- What data flows in and out? What contracts exist?

### Edge Cases / Failures
- What happens when things go wrong? Concurrent access? Empty data?

### Performance / Security / Compatibility
- Any performance requirements? Auth implications? Backwards compatibility?

### Non-goals
- What is explicitly out of scope?

## 3. List Assumptions

Bullet list of assumptions you're making if the user hasn't answered yet. Mark each as needing confirmation.

## 4. Write Acceptance Criteria

A checkbox list of 5-15 testable criteria. Each must be verifiable — either by a test, a command, or manual observation.

Example:
- [ ] Rate-limited endpoints return 429 after 5 requests per minute
- [ ] Rate limit is per-user, not global
- [ ] Retry-After header is included in 429 responses

## 5. List Open Questions

Anything that must be decided before planning can start.

## 6. Persist Output

Create the plans directory and save the output:
```bash
mkdir -p .claude/plans
```

Write the full intake output to `.claude/plans/<slug>-intake.md`, deriving a kebab-case slug from the goal.

## Rules

- Do NOT propose an implementation plan
- Do NOT propose code changes
- Do NOT suggest specific files to modify
- Focus entirely on understanding WHAT needs to happen, not HOW
