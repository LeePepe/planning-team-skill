---
description: Install teamwork skill bundle into your repo or global ~/.claude/. Supports default light mode, legacy eager mode, and ultra-light mode.
argument-hint: "[--global|--repo|--check|--full-agents|--ultra-light]"
allowed-tools: Bash
---

Validate the argument before running the setup script.

Accepted values for `${ARGUMENTS}` are:
- empty (default `--repo`)
- `--global`
- `--repo`
- `--check`
- `--full-agents`
- `--ultra-light`
- `--repo --full-agents`
- `--global --full-agents`
- `--repo --ultra-light`
- `--global --ultra-light`

If the argument is anything else, stop immediately and tell the user:
> Invalid argument. Accepted values are: --global, --repo, --check, --full-agents, --ultra-light, --repo --full-agents, --global --full-agents, --repo --ultra-light, --global --ultra-light (or leave blank for default --repo).

If the argument is valid (or empty), run:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/scripts/setup.sh" ${ARGUMENTS:---repo}
```

Report the output clearly, including any missing plugins and the commands needed to install them.
