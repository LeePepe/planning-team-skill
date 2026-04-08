---
name: setup
description: Install planning-team-skill agents and skill file. Run after installing the plugin. Pass --global (default) or --repo to select install scope, or --check to inspect status.
argument-hint: "[--global|--repo|--check]"
allowed-tools: Bash
---

Validate the argument before running the setup script.

Accepted values for `${ARGUMENTS}` are exactly: `--global`, `--repo`, `--check`, or empty (default).

If the argument is anything other than those three values (or empty), stop immediately and tell the user:
> Invalid argument. Accepted values are: --global, --repo, --check (or leave blank for the default --global).

If the argument is valid (or empty), run:

```bash
bash "${CLAUDE_PLUGIN_ROOT}/scripts/setup.sh" ${ARGUMENTS:---global}
```

Report the output clearly, including any missing plugins and the commands needed to install them.
