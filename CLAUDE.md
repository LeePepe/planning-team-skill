# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code skill (`SKILL.md`) plus five agent definitions (`agents/`) that implement a structured **plan → review → execute** pipeline. There is no compiled output — the "artifacts" are Markdown prompt files that get copied to `~/.claude/` or `.claude/`.

## Commands

```bash
# Syntax-check the setup script before committing changes to it
bash -n scripts/setup.sh

# Check current installation status (plugins, agents, skill file)
bash scripts/setup.sh --check

# Install globally
bash scripts/setup.sh --global

# Install into current git repo
bash scripts/setup.sh --repo
```

Always run `--check` before and after modifying setup logic to confirm idempotency.

## Architecture

### Pipeline Flow

```
SKILL.md  →  team-lead  →  planner  →  plan-reviewer  →  codex-coder / copilot
```

`SKILL.md` is the skill entry point: it validates plugin availability, reads repo routing config (`.claude/team.md`), then delegates entirely to `team-lead`. The team-lead orchestrates the rest — it **must not modify files directly**.

### Agent Responsibilities

| Agent | Role | May modify project files? |
|-------|------|--------------------------|
| `team-lead` | Orchestrates pipeline, routes tasks | No |
| `planner` | Writes plan files in `.claude/plan/` | Plan files only |
| `plan-reviewer` | Codex-powered plan review/iteration | Plan files only |
| `codex-coder` | Executes strict/formal tasks (TS/JS, APIs, tests) | Yes |
| `copilot` | Executes all other tasks (Swift, scripts, UI) | Yes |

### Hard Pipeline Rule

`team-lead` has `tools: Read, Glob, Agent` — no write access. Any direct file change from the lead is a pipeline violation. File changes flow exclusively through `codex-coder` or `copilot`.

### Executor Routing

Routing is determined by task type (defaults in agent files) and can be overridden per-repo via `.claude/team.md`. Only two valid executor values: `codex` and `copilot`.

### Plugin Dependency

Executors delegate to plugins: `codex-coder` uses `codex-companion.mjs`, `copilot` uses `copilot-companion.mjs`. At least one plugin must be installed; if only one is present, all tasks route to that executor regardless of the plan annotation.

## File Conventions

- Agent files: YAML front matter (`name`, `description`, `tools`) then Markdown instructions
- The `tools:` field is a hard constraint — only list tools the agent is actually permitted to use
- `scripts/setup.sh` must keep `set -euo pipefail` and quote all variable expansions
- Conventional Commits format: `type: short imperative summary`
