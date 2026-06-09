---
trigger: always_on
description: **swarm-llm (netllm)** is a mesh router for local LLM backends. Each host runs a lightweight agent that discovers oMLX (macOS), Ollama, LM Studio, and vLLM on localhost, finds sibling agents on the LAN via mDNS, and exposes dual API surfaces: OpenAI-compatible `http://<host>:11400/v1` and Anthropic Messages API `http://<host>:11400/v1/messages` (with translation to local backends).
---

# Agent & developer guide

## Project overview

**swarm-llm (netllm)** is a mesh router for local LLM backends. Each host runs a lightweight agent that discovers oMLX (macOS), Ollama, LM Studio, and vLLM on localhost, finds sibling agents on the LAN via mDNS, and exposes dual API surfaces: OpenAI-compatible `http://<host>:11400/v1` and Anthropic Messages API `http://<host>:11400/v1/messages` (with translation to local backends).

Tech stack: Python 3.11+, [uv](https://docs.astral.sh/uv/) workspace monorepo, FastAPI agent, Typer CLI.

## Architecture

| Package | Path | Role |
|---------|------|------|
| netllm-core | `packages/netllm-core/` | Routing, health cache, config |
| netllm-sdk-openai | `packages/netllm-sdk-openai/` | OpenAI SDK upstream adapter |
| netllm-sdk-anthropic | `packages/netllm-sdk-anthropic/` | Anthropic SDK upstream adapter |
| netllm-discovery | `packages/netllm-discovery/` | Local scan, swarm registry, mDNS |
| netllm-agent | `packages/netllm-agent/` | FastAPI: `/v1/*`, `/netllm/v1/*`, `/metrics` |
| netllm-cli | `packages/netllm-cli/` | Typer CLI |

Honcho integration: [docs/honcho-integration.md](docs/honcho-integration.md).

## Repository layout

| Path | Purpose |
|------|---------|
| `packages/` | Python source of truth (uv workspace) |
| `apps/` | Native apps: macOS menubar today (`apps/netllm-mac/`) |
| `packaging/` | Release builds per OS: [packaging/README.md](packaging/README.md) |
| `docs/` | User install/troubleshoot/editor guides: [docs/README.md](docs/README.md) |
| `tests/` | Cross-package integration tests |
| `scripts/` | CI, skill sync, install emulation |
| `Formula/` | Homebrew formula |
| `.agents/skills/` | Canonical agent skills → sync via `scripts/sync-agent-skills.sh` to `.claude/`, `.cursor/`, `.github/` |

Edit skills only under `.agents/`; run `scripts/sync-agent-skills.sh` after changes.

## Key commands

Prefer `./netllm` from the repo root, works without global PATH (`uv run` wrapper in [netllm](netllm)).

| Command | Purpose |
|---------|---------|
| `uv sync` | Install workspace dependencies |
| `./netllm init` | Write config, scan local providers, optional global CLI |
| `./netllm install` | Global `netllm` via `uv tool install` + shell PATH |
| `./netllm serve` | Start agent (foreground, default `127.0.0.1:11400`) |
| `./netllm start` / `stop` / `restart` | Background agent (macOS app, Homebrew, Linux systemd, Windows service) |
| `./netllm serve --host 0.0.0.0` | LAN + swarm: other machines can reach this agent |
| `./netllm status` | Agent, backends, swarm peers |
| `./netllm models` | Routed model catalog |
| `./netllm models --lan` | Models on remote LAN agents |
| `./netllm peers` | mDNS browse for swarm agents |
| `./netllm discover` | Probe oMLX / Ollama / LM Studio / vLLM on localhost |
| `./netllm test` | 1-token latency diagnose (OpenAI backends) |
| `./netllm test --api anthropic` | 1-token Messages API probe via agent |
| `./netllm gateway` | Promote agent role to gateway |
| `./netllm doctor` | PATH, mDNS, backend misconfig checks |
| `./netllm config-edit` | Open `config.toml` in `$EDITOR` |
| `./scripts/ci.sh` | Lint + test (same as CI) |
| `./scripts/ci.sh lint` | Ruff check + format --check |
| `./scripts/ci.sh test` | Run tests |
| `./scripts/ci.sh packaging` | Build deb/rpm (Linux) or zip (Windows) smoke artifacts |
| `scripts/verify-before-pr.sh` | Pre-push gate: lint + test + macOS `swift build -c release` |
| `scripts/verify-before-pr.sh --full` | Above + menubar e2e when Stage `.app` exists |
| `scripts/agent-verify-setup.sh` | Health + models check after setup |
| `scripts/sync-agent-skills.sh` | Sync `.agents/skills/` to other tool paths |

## Environment

Config: `~/.config/netllm/config.toml` (created by `./netllm init`). Example: [config.example.toml](config.example.toml).

Wire any OpenAI-compatible client:

```bash
export OPENAI_BASE_URL=http://127.0.0.1:11400/v1
export OPENAI_API_KEY=netllm-local
```

Native Anthropic Messages API (Claude Code, etc.):

```bash
export ANTHROPIC_BASE_URL=http://127.0.0.1:11400
export ANTHROPIC_API_KEY=netllm-local
```

Use a real `ANTHROPIC_API_KEY` only for cloud failover; local mesh uses `netllm-local`.

Default provider ports: oMLX `:8080`, Ollama `:11434`, LM Studio `:1234`, vLLM `:8000`.

## Linux and Windows

| Platform | Install | Troubleshooting | Background agent | UI |
|----------|---------|-----------------|------------------|-----|
| Linux | [docs/linux-install.md](docs/linux-install.md) | [docs/linux-troubleshooting.md](docs/linux-troubleshooting.md) | `systemctl --user enable --now netllm` (deb/rpm) | http://127.0.0.1:11400/ui/ |
| Windows | [docs/windows-install.md](docs/windows-install.md) | [docs/windows-troubleshooting.md](docs/windows-troubleshooting.md) | `NetllmAgent` service via packaged zip | http://127.0.0.1:11400/ui/ |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthewdcage/llm-swarm-router](https://github.com/matthewdcage/llm-swarm-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-09 -->
