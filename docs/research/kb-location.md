# Where project knowledge bases live — findings (spike M0.3)

| Field   | Value |
|---------|-------|
| Issue   | #1 |
| Date    | 2026-10-25 |
| Plugin  | agentspec 3.6.0 (commit `74fab99`) |

## Question

Can agentspec use KB domains stored in this repository, and by what mechanism?

## How agentspec discovers KBs

Plugin paths below are relative to the plugin root (`${CLAUDE_PLUGIN_ROOT}`).

- **There is no discovery code.** KB locations are prose instructions inside agent, skill and command
  markdown files, and the model follows them. Every one names `${CLAUDE_PLUGIN_ROOT}/kb/`:
  - `agents/architect/kb-architect.md`: reads `kb/_index.yaml`, globs `kb/{domain}/**`, writes new
    domains to `kb/{domain}/` and registers them in `kb/_index.yaml`.
  - `skills/kb-build/SKILL.md` (`/create-kb --validated`): same root for templates, registry and output.
  - `commands/knowledge/create-kb.md`: checks `kb/_templates/` and `kb/_index.yaml` exist.
  - `skills/sdd-brainstorm/SKILL.md`, `skills/sdd-define/SKILL.md`: list available domains from `kb/_index.yaml`.
  - Specialist agents hardcode their domains, e.g. `agents/python/python-developer.md` reads
    `kb/python/`, `kb/pydantic/`, `kb/testing/` (also declared as `kb_domains:` in its frontmatter).
- **`kb/_index.yaml`** is the registry: file-size `limits:` (the single source of truth for line
  budgets), the templates list, and a `domains:` map of path, entry points, concepts and patterns.
- **No project-level KB support.** No configuration, environment variable or project path is read.
  The only project-aware hook, `scripts/init-workspace.sh` (SessionStart), creates `.claude/sdd/` and
  `.claude/agents/` and writes stack hints; it does not touch KBs.
- **The install folder is per-version:** `~/.claude/plugins/cache/agentspec/agentspec/3.6.0/`.
  An update installs a new versioned folder, so a domain written into the current one would be left
  behind, and its `_index.yaml` entry with it.
- **Local agents override plugin agents.** `.claude/agents/<name>.md` wins over a plugin agent with the
  same `name:` (Claude Code's native precedence, documented in the plugin's generated
  `.claude/agents/README.md`).

## Options

| Option | How | Survives plugin update | Upkeep | Verdict |
|--------|-----|-----------------------|--------|---------|
| A. Plugin default | Write into `${CLAUDE_PLUGIN_ROOT}/kb/` | No | None, until content is lost | Rejected: violates charter §7.3 |
| B. Symlink / sync | Repo `kb/<domain>` symlinked or copied into the plugin `kb/`, plus `_index.yaml` edits | No (redo after every update) | A script to re-run; edits to a plugin-owned file | Rejected: fragile |
| C. Project KB root + instructions | Repo `.claude/kb/` with its own `_index.yaml`; `CLAUDE.md` declares it as a second KB root | Yes | One rule in `CLAUDE.md`; brief delegated agents | **Chosen** |
| D. Override agents | Copy KB-using agents into `.claude/agents/` and repoint them at `.claude/kb/` | Yes | Copies drift from upstream on every update | Kept for project-owned agents only |

## Experiment

A throwaway domain `.claude/kb/zz-spike-test/` held a made-up fact found nowhere else and was
registered in `.claude/kb/_index.yaml`. `CLAUDE.md` declared `.claude/kb/` as a second KB root.
The `kb-architect` agent was asked a question only that fact answers, without being told where to look.

Result: **found.** The agent read the plugin registry first (as its definition instructs), found
nothing, then read `CLAUDE.md`, followed it to `.claude/kb/_index.yaml`, and answered from the test
domain with the correct value.

Caveat: the agent reported that `CLAUDE.md` was **not** loaded into its context automatically; it
opened the file on its own initiative. So a subagent cannot be relied on to see the rule. The
main session does load `CLAUDE.md`, and it is the main session that briefs subagents (including
`kb-build`, which runs in the main session and launches `kb-architect` subagents). Hence the
mitigation: whoever delegates passes the KB rules in the prompt.

The test domain was deleted after the experiment.

## Conclusion

Option C, with a delegation rule. Recorded as ADR-001 (#4).
