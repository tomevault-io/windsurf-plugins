---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repo is the **Claude Code plugin itself** (not a consumer of it). For usage, see [README.md](README.md)·[USAGE.md](USAGE.md).
For component authoring specs (command/agent/hook/skill frontmatter), verify against the official docs as the SSOT, not model knowledge:
[plugins-reference](https://code.claude.com/docs/en/plugins-reference.md) · [hooks](https://code.claude.com/docs/en/hooks.md) · [skills](https://code.claude.com/docs/en/skills.md).

## Commands

Gate scripts are Python; run tooling via `uv`.

```bash
uv sync                                                  # install dependencies
uv run pytest                                            # run all tests
uv run pytest tests/test_flow_gate_check.py::<name>      # run a single test
uv run ruff check && uv run ruff format --check          # lint + format check
uv run pre-commit run --all-files                        # full static analysis
```

When modifying `*.sh`, verify with ShellCheck (the hook runtime is Windows, so bugs are hidden as FAIL-OPEN — see Invariants).

## Folder structure

`agents/`·`hooks/hooks.json`·`skills/` declare no path in the manifest — they are **auto-discovered from their default locations** (adding a component = just adding a file).

```text
.claude-plugin/
  plugin.json              plugin manifest (minimal — name/description/version/author)
  marketplace.json         marketplace manifest (harness-tier exposes itself; plugin source=github + immutable sha pin)
agents/     harness-researcher · harness-code-analyzer · harness-critic   (harness research/analysis/critique)
hooks/      hooks.json (SessionStart rule injection + Notification) · inject-risk-tiers.sh
skills/     flow · flow-init · flow-uninstall · harness-init · doc-sync · harness-authoring · harness-insight
            playwright-scaffold · integration · performance   (/slash = skill)
rules/      risk-tiers.md  ← SSOT for tier classification & commit discipline (not auto-loaded; injected by a hook)
            harness-rules.md  ← SSOT for harness-generation discipline (loaded by the harness-init skill)
scripts/    flow_gate_check.py · precommit-runner.sh · teams_alert.py · notify-push.sh
            check-deps.sh (dependency check & guidance) · flow_init_setup.py (flow-init setup/re-run + --uninstall cleanup)
            harness_scaffold.py (harness-init scaffold generation)
            harness_insight.py (harness-insight transcript aggregation — project-agnostic, emits a temporary txt)
github/     api-contract.workflow.example.yml   contract-test SOURCE (/flow-init renders it via flow-config.contract_test)
            release.python-semantic-release.workflow.example.yml · release.semantic-release.workflow.example.yml
            branch-naming.workflow.example.yml · entropy-check.workflow.example.yml
            (the 4 above are rendered by /flow-init via flow-config.versioning — release picks one via release_tool)
.github/    workflows/ (release·branch-naming·entropy-check — harness-tier's own CI) · scripts/pin-marketplace-sha.py (pins the marketplace sha at release)
flow-tiers.yaml            tier→gates policy (plugin-owned, immutable)
flow-config.example.yaml   host environment-value slots (the real file is the host's .claude/harness-tier/config/flow-config.yaml, team-shared & git-tracked)
tests/      test_flow_gate_check.py · test_flow_init_setup.py · test_harness_scaffold.py · test_harness_insight.py
```

## Architecture (must-know)

- **The plugin is installed outside the host (in a cache) → dual paths.** `${CLAUDE_PLUGIN_ROOT}` = reads (templates/policy), `${CLAUDE_PROJECT_DIR}` = writes (host config/evidence). **Never write into the plugin directory.**
- **Host writes are grouped by purpose under `${CLAUDE_PROJECT_DIR}/.claude/harness-tier/`** (no scattering across the root): `scripts/` (copied gate scripts, plugin-owned & git-tracked) · `config/` (flow-config.yaml (team-shared & git-tracked — developers of the same repo use identical settings) · flow-tiers.yaml (tier→gates policy — plugin-owned, overwritten on every install, do not edit) · webhooks; host-owned) · `.flow/` (gate evidence, gitignored). The only exceptions are the files whose location is forced by external tools: `.gitignore` (git) · `.pre-commit-config.yaml` (pre-commit) · `.claude/settings.json` (Claude Code) · `.github/workflows/` (GitHub Actions).
- **The commit gate is registered in the host's `settings.json`** (not the plugin's hooks.json), because of deny-enforcement reliability and because `${CLAUDE_PLUGIN_ROOT}` is not resolved there. `/flow-init` **copies** the gate scripts to the host's `.claude/harness-tier/scripts/` and the `flow-tiers.yaml` policy to `.claude/harness-tier/config/`.
- **Script propagation is one-way**: `scripts/`·`flow-tiers.yaml` (SOURCE·SSOT) → cache (reinstall) → `<host>/.claude/harness-tier/scripts/` (gate scripts)·`config/flow-tiers.yaml` (policy execution copy). Fix only the SOURCE; never edit the host copies directly (they are overwritten on reinstall). After a plugin update, sync the host copies by re-running `/flow-init` (config left intact); clean up the host with `/flow-uninstall`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foryouself83/harness-tier](https://github.com/foryouself83/harness-tier) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
