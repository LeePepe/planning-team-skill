---
name: teamwork
description: Install, check, and maintain the Teamwork Claude pipeline in a repository, including plugin readiness and overload diagnostics.
metadata:
  author: LeePepe
  version: "0.1.0"
---

# Teamwork

Use this skill when the user asks to install, update, verify, or troubleshoot this repository's teamwork setup.

## Scope

This skill manages the Claude-side teamwork package from Codex by running the local setup script and reporting state.

## Standard workflow

1. Run status check first:

```bash
bash scripts/setup.sh --check
```

2. If install/update is requested, run one of:

```bash
bash scripts/setup.sh --repo
bash scripts/setup.sh --global
```

3. Re-run check and report:
- plugin availability (`codex` / `copilot`)
- whether fallback mode is active
- next command if user still has missing dependencies

## Team-Lead planning from Codex

When the user asks to "start team-lead" or plan fallback/routing behavior:

1. Preflight plugin availability using companion scripts (same logic as `commands/task.md`):

```bash
CODEX_SCRIPT=$(find ~/.claude/plugins -name "codex-companion.mjs" 2>/dev/null | head -1)
COPILOT_SCRIPT=$(find ~/.claude/plugins -name "copilot-companion.mjs" 2>/dev/null | head -1)
echo "codex=$([ -n "$CODEX_SCRIPT" ] && echo true || echo false) copilot=$([ -n "$COPILOT_SCRIPT" ] && echo true || echo false)"
```

2. Ensure `team-lead` is present using the loader snippet in `commands/task.md` (Step 2.5).
3. Derive executor constraint and planning path:
- both true -> route by plan task annotation (`executor: codex|copilot`)
- codex=true, copilot=false -> force `codex-coder`
- codex=false, copilot=true -> force `copilot`
- both false -> force `claude-coder` and choose `haiku|sonnet|opus` by complexity
4. Return a concrete execution plan before implementation.

## Overload diagnostics

If user reports `529 overloaded_error` after setup:

1. Re-run `bash scripts/setup.sh --check`
2. Detect recursive cache paths:

```bash
find ~/.claude/plugins/cache/teamwork -type d -path "*/teamwork/*/teamwork/*" | head
```

3. If recursion exists, clean cache and ask user to reload plugins:

```bash
rm -rf ~/.claude/plugins/cache/teamwork
```

Then tell user to run `/reload-plugins` in Claude Code.
