# Planning Team Skill for Claude Code

A Claude Code skill that orchestrates a full **plan → review → execute** pipeline using a team of agents.

Claude plans, Codex reviews, and tasks are automatically routed to **Codex** (strict/formal work) or **Copilot** (everything else).

## How It Works

```
/planning-team <your feature or fix>
        │
        ▼
   team-lead (orchestrates only — never modifies files directly)
        ├── planner      → creates .claude/plan/<slug>.md
        │                   each task annotated: executor: codex | copilot
        ├── plan-reviewer (Codex)
        │                → reviews or adversarially challenges the plan
        └── executors (parallel where possible)
              codex-coder  ← TypeScript, APIs, tests, business logic
              copilot      ← Swift, Kotlin, UI, scripts, platform code
```

## Dependencies

Install at least one plugin in Claude Code (both are optional but recommended):

**Codex plugin** (by OpenAI):
```
/plugin marketplace add openai/codex-plugin-cc
/plugin install codex@openai-codex
```

**Copilot plugin** (this org):
```
/plugin marketplace add LeePepe/copilot-plugin-cc
/plugin install copilot@copilot-local
```

Then reload:
```
/reload-plugins
```

Run setup only for the plugins you installed:
```
/codex:setup     # only if codex plugin was installed
/copilot:setup   # only if copilot plugin was installed
```

## Install This Skill

```bash
git clone https://github.com/LeePepe/planning-team-skill.git
cd planning-team-skill

# Global install (agents + skill → ~/.claude/)
bash scripts/setup.sh --global

# Or project-local install (agents + skill → .claude/ in current repo)
bash scripts/setup.sh --repo
```

Check status before or after install:

```bash
bash scripts/setup.sh --check
```

The script installs agent definitions, the skill file, and registers plugin marketplaces in `~/.claude/settings.json`. In `--repo` mode it also creates a `.claude/team.md` template for per-repo customization.

## Usage

```
/planning-team implement a JWT auth middleware for the Express API
```

```
/planning-team refactor the payment module to use the new Stripe SDK
```

## Per-Repo Customization

### Routing preferences

Run `bash scripts/setup.sh --repo` inside your repo — it copies a `team.md` template to `.claude/team.md` automatically. Then edit it to set routing rules and review mode:

```markdown
## Executor Routing
- *.swift → copilot
- *.ts    → codex

## Review Mode
default: adversarial-review
```

### Project-specific executors

Add repo-aware agent definitions to `.claude/agents/` in your repo:

- `.claude/agents/codex-coder.md` — knows your TS conventions, test setup, etc.
- `.claude/agents/copilot.md` — knows your xcodebuild commands, Sapphire structure, etc.

Project-level agents automatically override global ones.

## Executor Routing Defaults

| Executor | Handles |
|----------|---------|
| `codex` | TypeScript/JS · APIs · types · tests · DB migrations · algorithms |
| `copilot` | Swift/SwiftUI · Kotlin/Android · UI · scripts · exploratory refactoring |

## Related

- [codex-plugin-cc](https://github.com/openai/codex-plugin-cc) — OpenAI's Codex plugin
- [copilot-plugin-cc](https://github.com/LeePepe/copilot-plugin-cc) — Copilot CLI plugin
