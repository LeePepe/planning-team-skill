---
name: teamwork
description: Coordinate a full plan-review-execute pipeline with specialized agents (`team-lead`, `planner`, `plan-reviewer`, `codex-coder`, `copilot`). Use when work spans multiple files, requires a reviewed plan before coding, or benefits from parallel execution with explicit Codex/Copilot routing. Trigger on requests like "use the planning team", "plan then implement", "multi-agent workflow", or "/teamwork:task implement auth middleware".
---

# Teamwork Skill

Run a structured multi-agent pipeline: planner writes a plan, plan-reviewer gates quality, then executors implement approved tasks.

## Dependencies

Install at least one plugin in Claude Code before running this skill (both are optional but recommended):

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
/teamwork:task <description>
```

Natural language trigger:

```text
Use the planning team to implement <feature>
```

> To install or check status, use `/teamwork:setup` (available after installing this plugin).

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
# Check which plugins are available
CODEX_SCRIPT=$(find ~/.claude/plugins -name "codex-companion.mjs" 2>/dev/null | head -1)
COPILOT_SCRIPT=$(find ~/.claude/plugins -name "copilot-companion.mjs" 2>/dev/null | head -1)

[ -n "$CODEX_SCRIPT" ]   && node "$CODEX_SCRIPT"   setup --json 2>/dev/null && CODEX_OK=true   || CODEX_OK=false
[ -n "$COPILOT_SCRIPT" ] && node "$COPILOT_SCRIPT" setup --json 2>/dev/null && COPILOT_OK=true || COPILOT_OK=false
```

At least one plugin must be available. Stop and request installation only if **both** are unavailable.

When only one plugin is installed, all tasks are routed to that executor regardless of the `executor` annotation in the plan:
- Only codex installed  → all tasks go to `codex-coder`
- Only copilot installed → all tasks go to `copilot`
- Both installed        → tasks are routed per annotation (default behavior)

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

Route by task weight and rigor requirement, not by language or file type:

| Executor | When to use |
|----------|-------------|
| `codex` | Rigorous or heavy tasks: complex algorithms, security-sensitive code, auth/authz, data migrations, strict correctness requirements, large-scale refactors with many interdependencies, critical business logic, tasks requiring deep analysis before coding |
| `copilot` | All other tasks: UI changes, simple feature additions, scripts, configuration, exploratory/experimental code, documentation, straightforward bug fixes, lightweight tooling |

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
