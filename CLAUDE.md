# process-surrogates-lab

See [docs/PROJECT_CHARTER.md](docs/PROJECT_CHARTER.md) for direction and scope.

## Knowledge bases

Project KB domains live in this repo at `.claude/kb/`, registered in `.claude/kb/_index.yaml`.
agentspec instructions refer to `${CLAUDE_PLUGIN_ROOT}/kb/` only; treat both as KB roots:

- **Reading:** check `.claude/kb/_index.yaml` as well as `${CLAUDE_PLUGIN_ROOT}/kb/_index.yaml`, and load matching domains from both.
- **Writing** (`/create-kb`, `kb-build`, `kb-architect`): create and register new domains under `.claude/kb/`, never under `${CLAUDE_PLUGIN_ROOT}/kb/`.
  Keep using the plugin's `kb/_templates/` and the `limits:` in its `_index.yaml`.
- **Delegating:** subagents do not reliably receive this file. When briefing any agent that reads or writes KBs,
  copy the two rules above into its prompt and name the project domains it should use.
