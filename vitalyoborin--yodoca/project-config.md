---
trigger: always_on
description: This file provides **default working protocol and context** for AI coding agents (Cursor, Copilot, Codex, etc.) in this repository. It complements `.cursor/rules/` and `.cursor/skills/`; read it first for a project snapshot, then follow the referenced rules and skills for implementation details.
---

# AGENTS.md — Yodoca Agent Engineering Protocol

This file provides **default working protocol and context** for AI coding agents (Cursor, Copilot, Codex, etc.) in this repository. It complements `.cursor/rules/` and `.cursor/skills/`; read it first for a project snapshot, then follow the referenced rules and skills for implementation details.

**Scope:** Entire repository. Keep instructions actionable and under ~400 lines; reference docs and skills instead of duplicating them.

---

## 1. Project snapshot (read first)

**Yodoca** is an autonomous AI agent runtime with:

- **All-is-extension** — All user-facing functionality (channels, memory, tools, schedulers, agents) lives in extensions under `sandbox/extensions/<id>/`. The core is a nano-kernel: Loader, EventBus, MessageRouter, ModelRouter, Orchestrator.
- **Core must not depend on extensions** — Core discovers and wires extensions by protocol only; no imports from `sandbox` in `core/`. Extensions depend only on contracts and `ExtensionContext`. See [.cursor/skills/core_boundary/SKILL.md](.cursor/skills/core_boundary/SKILL.md).
- **Event-driven** — Durable SQLite event journal; extensions publish/subscribe via `context.emit()` and `context.subscribe_event()`.
- **Stack** — Python 3.12+, uv for deps, OpenAI Agents SDK, Pydantic, SQLite (events, memory, tasks). Config: `config/settings.yaml`; secrets: keyring or `.env`.

**Entry points:** `uv run python -m supervisor` (production); `uv run python -m core` (agent process, usually spawned by supervisor). Onboarding: `uv run python -m onboarding` when config is missing.

**Key docs:** [docs/README.md](docs/README.md) (index), [docs/architecture.md](docs/architecture.md) (bootstrap, components), [docs/extensions.md](docs/extensions.md) (extension contract, manifest, context API).

---

## 2. Dev environment tips

- **Setup:** `uv sync` (or `uv sync --extra dev` for lint/test deps). Use `uv run <command>` so the correct venv is used.
- **Run the app:** `uv run python -m supervisor`. If config is missing, onboarding runs automatically; or run `uv run python -m onboarding` manually.
- **Config:** `config/settings.yaml` (create from `config/settings.example.yaml` if present). Never put API keys in YAML; use keyring or `.env`. Reset everything: `uv run python scripts/reset.py`.
- **Extensions:** Each extension is `sandbox/extensions/<id>/` with `manifest.yaml` and `main.py` (or declarative agent with `agent` section only). Add new extensions there; core does not reference extension IDs for behavior.
- **Prompts:** System agent prompts live in `sandbox/prompts/`. Extension-specific prompts can live in `sandbox/extensions/<id>/` (e.g. `prompt.jinja2`). See [.cursor/rules/development.mdc](.cursor/rules/development.mdc).
- **Jump to code:** Core bootstrap: `core/runner.py`. Extension contracts: `core/extensions/contract.py`. Loader: `core/extensions/loader.py`. Example extensions: `sandbox/extensions/cli_channel/`, `sandbox/extensions/memory/`, `sandbox/extensions/task_engine/`.

---

## 3. Testing and quality checks

Install dev dependencies first: `uv sync --extra dev`.

| Check | Command | Notes |
|-------|---------|--------|
| **Lint** | `uv run ruff check .` | Fix auto-fixable: `uv run ruff check . --fix` |
| **Format** | `uv run ruff format .` | Check only: `uv run ruff format . --check` |
| **Import layers** | `uv run lint-imports` | Enforces: core must not import extensions (see pyproject `importlinter`) |
| **Types** | `uv run mypy` | Strict mode; overrides for tests/scripts |
| **Security** | `uv run bandit -r core onboarding sandbox` | Basic scan (B101 skipped) |
| **Tests** | `uv run pytest` | Asyncio mode auto; tests in `tests/` |

**Before committing / PR:** Run at least `ruff check .`, `ruff format --check .`, and `lint-imports`. Fix test and type errors; add or update tests for changed behavior. Do not commit with failing checks.

---

## 4. PR and commit expectations

- **Title:** Clear, concise. Prefer conventional style where it helps (e.g. `feat(ext): add X`, `docs: update Y`). No strict format required.
- **Checks:** All of §3 must pass. CI may run the same commands.
- **Architecture changes:** Must be documented in `docs/adr/` **before** implementation. Create `docs/adr/NNN-short-slug.md` (e.g. `016-new-feature.md`) with Status, Context, Decision, Consequences. See [.cursor/skills/adr/SKILL.md](.cursor/skills/adr/SKILL.md) and [.cursor/rules/adr.mdc](.cursor/rules/adr.mdc).
- **Documentation:** If you change behavior or contracts, update the relevant doc in `docs/` (and ADR if applicable). Required by project rules.

---

## 5. Coding conventions and rules

These align with `.cursor/rules/` and `.cursor/skills/`. When in doubt, follow the referenced rule or skill.

### Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VitalyOborin/yodoca](https://github.com/VitalyOborin/yodoca) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
