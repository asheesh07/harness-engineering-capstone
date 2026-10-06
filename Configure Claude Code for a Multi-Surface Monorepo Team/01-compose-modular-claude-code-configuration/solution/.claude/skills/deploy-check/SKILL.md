---
name: deploy-check
description: Run read-only pre-deployment checks and report whether the change is ready
context: fork
allowed-tools:
  - Read
  - Glob
  - Grep
---

# Deploy Check

Use this skill for a read-only pre-deployment assessment.

The goal is to keep verbose output out of the **main session** while the forked context performs the detailed checks. The `context: fork` setting is a Claude Code product feature analogous to the Playbook **Branching Reality** pattern: detailed investigation can happen in an isolated context and return a concise result.

## Checks

### Tests

Detect: inspect the relevant test files and available test configuration.

Pass: the relevant tests are present and provide evidence that the changed behavior is covered.

Fail: required tests are missing or the available evidence indicates a regression.

### Type and static checks

Detect: inspect the repository configuration and changed source files for the applicable type-checking or static-analysis command.

Pass: applicable type or static checks have no reported errors.

Fail: applicable type or static checks report errors that affect the change.

### Deployment configuration

Detect: inspect deployment configuration and changed files for missing required settings, paths, or incompatible configuration.

Pass: the required deployment configuration is present and consistent with the change.

Fail: a required setting is missing, contradictory, or clearly incompatible with deployment.

## Skill vs CLAUDE.md

Use a **skill** when the guidance is task-specific, on-demand, or benefits from a forked context and detailed investigation. Keep broadly applicable team rules in `CLAUDE.md` when they should be loaded in every session.

Use a skill for a task-specific workflow that is **on-demand** or **forked**. Use `CLAUDE.md` for rules that are **always-loaded** or universal across sessions.

## Personal customization

Personal customizations can be added under `~/.claude/skills/` without changing the project-scoped skill.
