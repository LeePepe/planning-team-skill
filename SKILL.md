---
name: planning-team
description: Coordinate a full plan-review-execute pipeline with specialized agents (`team-lead`, `planner`, `plan-reviewer`, `codex-coder`, `copilot`). Use when work spans multiple files, requires a reviewed plan before coding, or benefits from parallel execution with explicit Codex/Copilot routing. Trigger on requests like "use the planning team", "plan then implement", "multi-agent workflow", "/planning-team setup", or "/planning-team implement auth middleware".
---

# Planning Team Skill

Run a structured multi-agent pipeline: planner writes a plan, plan-reviewer gates quality, then executors implement approved tasks.

## Dependencies

Install both plugins in Claude Code before running this skill:

- **[codex-plugin-cc](https://github.com/openai/codex-plugin-cc)** for Codex rescue/review integration
- **[copilot-plugin-cc](https://github.com/LeePepe/copilot-plugin-cc)** for local Copilot rescue integration

Use these commands:

```bash
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex

/plugin marketplace add LeePepe/copilot-plugin-cc
/plugin install copilot@copilot-local

/reload-plugins
```

## Triggers

```text
/planning-team setup [--global|--repo]
/planning-team <description>
```

Natural language trigger:

```text
Use the planning team to implement <feature>
```

## Setup

When the user asks for `/planning-team setup`, perform these steps.

### 1) Locate the skill directory

```bash
SKILL_DIR=$(find ~/.claude/skills ~/Development -name "planning-team-skill" -maxdepth 4 -type d 2>/dev/null | head -1)
if [ -z "$SKILL_DIR" ]; then
  git clone https://github.com/LeePepe/planning-team-skill.git /tmp/planning-team-skill
  SKILL_DIR="/tmp/planning-team-skill"
fi
```

### 2) Run status check and install

```bash
bash "$SKILL_DIR/scripts/setup.sh" --check
bash "$SKILL_DIR/scripts/setup.sh" [--global|--repo]
```

Pick install mode with this rule:

- Use `--repo` when the user wants project-local install and current shell is inside a git repo.
- Use `--global` otherwise.

### 3) Install missing plugins

If the check output reports missing plugins, run:

```text
/plugin install codex@openai-codex
/plugin install copilot@copilot-local
/reload-plugins
```

### 4) Verify setup

```bash
bash "$SKILL_DIR/scripts/setup.sh" --check
node $(find ~/.claude/plugins -name "codex-companion.mjs" | head -1) setup --json 2>/dev/null
node $(find ~/.claude/plugins -name "copilot-companion.mjs" | head -1) setup --json 2>/dev/null
```

Report clear status: installed components and remaining gaps.

## Pipeline

```text
team-lead
  ├── planner        → writes .claude/plan/<slug>.md with executor annotations
  ├── plan-reviewer  → reviews plan (review or adversarial-review)
  └── executors (parallel where possible):
        executor: codex   → codex-coder
        executor: copilot → copilot
```

## Workflow

### 1) Validate plugin readiness

```bash
# Verify codex plugin
node $(find ~/.claude/plugins -name "codex-companion.mjs" | head -1) setup --json

# Verify copilot plugin
node $(find ~/.claude/plugins -name "copilot-companion.mjs" | head -1) setup --json
```

Stop and request installation if either plugin is unavailable.

### 2) Read repo routing config

```bash
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
cat "$REPO_ROOT/.claude/team.md" 2>/dev/null
```

If `.claude/team.md` exists, read:

- Executor routing overrides
- Preferred review mode (`review` or `adversarial-review`)

### 3) Delegate orchestration to `team-lead`

Pass:

```text
Agent: team-lead
Prompt: <user's description>
        Routing preferences: <from .claude/team.md, or "use defaults">
```

Let `team-lead` run:

1. `planner` creates the plan
2. `plan-reviewer` reviews and iterates plan quality
3. executors implement approved tasks

### 4) Report outcome

Return:

- plan path
- modified files grouped by executor
- failed/skipped tasks
- follow-up actions

## Per-Repo Customization

Drop a `.claude/team.md` in the repo to override defaults:

```markdown
## Executor Routing
- *.swift, *.m → copilot
- *.ts, *.tsx  → codex
- tests/**     → codex

## Review Mode
default: adversarial-review
```

Optionally provide project-specific executor prompts in `.claude/agents/`:

- `.claude/agents/codex-coder.md`
- `.claude/agents/copilot.md`

Project-level agents automatically take priority over global ones.

## Executor Routing Defaults

| Executor | Task Types |
|----------|-----------|
| `codex` | TypeScript/JS, APIs, types, tests, DB migrations, algorithms, business logic |
| `copilot` | Swift/SwiftUI, Kotlin/Android, UI, exploratory refactoring, platform code, scripts |

## Constraints

- Keep task routing values to `codex` or `copilot`.
- Require review pass before any execution phase.
- Keep planner and reviewer scoped to plan files; avoid direct project-code edits there.
- Keep executor prompts concrete: scope, dependencies, verification.

## Shipped Agents

- `team-lead.md`
- `planner.md`
- `plan-reviewer.md`
- `codex-coder.md`
- `copilot.md`

Install manually when needed:

```bash
cp agents/*.md ~/.claude/agents/
```
