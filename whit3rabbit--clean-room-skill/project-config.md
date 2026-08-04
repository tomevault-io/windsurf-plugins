---
trigger: always_on
description: - npm package: `clean-room-skill`.
---

# Clean Room Repo Guide

## Repo Quick Facts

- npm package: `clean-room-skill`.
- CLI entrypoint: `bin/install.js`.
- Executable inner-loop runner: `clean-room-skill run`.
- Full local verifier: `bin/verify.sh`.
- Node requirement: `>=22`.
- CI runs Node 24 with Python 3.12 on macOS.
- This repo installs clean-room skills, role agents, hooks, schemas, examples, and optional verification templates for multiple agent runtimes.
- Installer runtime flags: Codex, Claude Code, Antigravity, Gemini CLI, OpenCode, Kilo, Cursor, GitHub Copilot, Windsurf, Augment, Trae, Qwen Code, Hermes Agent, and CodeBuddy.
- Hook registration is verified for Codex, Claude Code, and OpenCode. Other runtime layouts are best-effort installs unless code and tests prove otherwise.
- Dynamic workflows (`workflows/*.js` -> `.claude/workflows/`) are Claude Code only and local-scope only, installed alongside their `/clean-room:<name>` front-door command wrapper. No other runtime ships a `Workflow()` engine; global installs stay hooks-only, so a triggerless global workflow is never shipped.
- The workflow creates clean behavioral spec packages and clean implementation outputs. It does not generate replacement code directly from source.
- Full docs: [README.md](README.md), [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md), and [docs/REFERENCE.md](docs/REFERENCE.md).

## Repo Map

- `bin/`: installer CLI and local verification script.
- `lib/`: installer helpers, TUI, hook config helpers, runtime layout logic, preflight helpers, doctor checks, and inner-loop runner.
- `skills/`: skill entrypoints, schemas, references, source and visual index scripts, and example spec packages.
- `agents/`: Claude role-agent prompts.
- `examples/codex/`: Codex role-agent templates.
- `hooks/`: Python guardrail hooks.
- `templates/docker/`: optional Agent 3 verification images and example verification command policies.
- `tests/`: Node tests, JSON Schema fixtures, schema parity fixtures, and validation helper.
- `.github/workflows/`: CI and npm publish workflows.
- `.agents/plugins/marketplace.json`: Codex repo marketplace.
- `.agents/skills/synrepo/`: repo-bundled synrepo skill metadata.
- `.codex/`: repo-local Codex config and hook config used for this workspace.
- `.claude-plugin/marketplace.json`: Claude Code marketplace.
- `plugin.json`, `.codex-plugin/plugin.json`, and `.claude-plugin/plugin.json`: runtime-specific plugin manifests.

## Commands

- Search with `st`, not ripgrep or grep.
- Run a single test file: `node --test tests/install.test.js`.
- Install deps: `npm ci --ignore-scripts`.
- Run all Node tests: `npm test`.
- Run installer tests only: `npm run test:install`.
- Set up Python verifier deps on macOS/Homebrew Python: `python3 -m venv .venv && .venv/bin/python -m pip install "jsonschema[format]>=4.18,<5"`.
- Run full local checks: `npm run verify`.
- Dry-run bootstrap folder creation: `node bin/install.js init --dry-run`.
- Show preflight helper options: `node bin/install.js preflight --help`.
- Dry-run installer: `node bin/install.js --dry-run --all --global`.
- Check runtime install state: `node bin/install.js status --global`.
- Dry-run runtime updates: `node bin/install.js update --global --dry-run`.
- Smoke-test generated hooks: `node bin/install.js doctor --runtime codex --hooks=safe --config-dir <path>`.
- Smoke-test strict OpenCode hook bridge coverage: `node bin/install.js doctor --runtime opencode --hooks=strict --coverage --config-dir <path>`.
- Dry-run inner-loop unit selection: `node bin/install.js run --task-manifest <path> --dry-run`.
- Run inner-loop with adapter commands: `node bin/install.js run --task-manifest <path> --agent-commands <path>`.
- Run inner-loop with built-in Claude role-agent dispatch: `node bin/install.js run --task-manifest <path> --agent-runtime claude [--agent-config-dir <path>]`.
- Create or validate canonical artifacts: `node bin/install.js artifact <kinds|template|validate> ...` (`lib/artifact.cjs`, `lib/artifact-cli.cjs`).
- Build source index help: `python3 skills/clean-room/scripts/build_source_index.py --help`.
- Build visual index help: `python3 skills/clean-room/scripts/build_visual_index.py --help`.
- Build optional Docker verification profiles: `docker compose -f templates/docker/compose.clean-room.yml build`.
- No lint script exists. Do not invent one.

## Verification

- JS changes: run `node --check` on touched JS/CJS files and `npm test`.
- CLI/preflight/doctor/runner changes: run `node --check bin/install.js lib/doctor.cjs lib/preflight.cjs lib/run.cjs` plus touched JS/CJS files, then `npm test`.
- Runner changes: run `node --check lib/run.cjs lib/run-cli.cjs lib/run-controller.cjs bin/install.js`, `node --test tests/run.test.js`, and `npm test`.
- Installer/runtime layout changes: run `npm run test:install` and `npm run verify`.
- Installer TUI changes: use the MCP TUI test tool for at least one representative interactive flow.
- Python hook or script changes: run `python3 -m compileall -q hooks skills/clean-room/scripts`.
- Schema or example changes: run `.venv/bin/python tests/validate_jsonschema.py` if `.venv` exists, otherwise use a Python with `jsonschema[format]>=4.18,<5`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whit3rabbit/clean-room-skill](https://github.com/whit3rabbit/clean-room-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
