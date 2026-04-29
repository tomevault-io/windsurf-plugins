---
trigger: always_on
description: This file provides context for AI agents working on this codebase.
---

# Agent Context — ARC (Agent Remote Control)

This file provides context for AI agents working on this codebase.

**IMPORTANT FOR AGENTS:** Update this file on EVERY commit that changes features,
architecture, CLI commands, configuration, build steps, or known issues.
This is the canonical context document — future agents and humans rely on it.
If you're unsure whether to update it, update it.

## Project Overview

ARC is a universal remote control for AI agent frameworks. It connects any agent to a browser-based interface for real-time monitoring, interaction, and remote command injection via WebSocket relay.

**Architecture:** Agents → WebSocket Relay → Web Viewers. Commands flow back the same path.

**Beta relay:** `arc-beta.axolotl.ai` — open access with prefix-based tokens (`axolotl_beta_*`).

## Repository Structure

```
├── relay/                     # OSS relay server (Python/FastAPI)
│   ├── relay.py               # Main app: WebSocket relay + create_app() factory + web viewer serving
│   ├── beta_app.py            # Beta relay entrypoint (Redis session store + prefix auth)
│   ├── protocols.py           # Extension interfaces (AuthProvider, SessionStore, SessionPolicy, LifecycleHooks)
│   ├── defaults.py            # OSS defaults (token auth, prefix tokens, in-memory store)
│   ├── models.py              # Session, SessionInfo dataclasses (includes e2e field)
│   └── tests/                 # Unit tests (test_relay.py, test_prefix_auth.py)
├── packages/                  # TypeScript (npm workspaces, @axolotlai scope)
│   ├── protocol/              # @axolotlai/arc-protocol — wire types, RemoteControlClient, BaseAdapter, crypto
│   ├── cli/                   # @axolotlai/arc-cli — `arc` command
│   ├── web-client/            # Static React SPA viewer (auto-connect from URL params, E2E decrypt)
│   ├── adapter-hermes/        # @axolotlai/arc-adapter-hermes (unused — native plugin is primary)
│   ├── adapter-deepagent/     # @axolotlai/arc-adapter-deepagent
│   └── adapter-openclaw/      # @axolotlai/arc-adapter-openclaw
├── hermes-plugin/             # Native Hermes Agent plugin (PRIMARY Hermes integration)
│   ├── arc-remote-control/    # Plugin: tools, lifecycle hooks, clarify monkeypatch, E2E encryption
│   │   ├── __init__.py        # Main plugin code (ArcRelay, hooks, clarify patch, E2E)
│   │   └── plugin.yaml        # Hermes plugin manifest
│   └── tests/                 # Plugin tests (test_reconnect.py, test_e2e.py)
├── hosted/                    # Hosted SaaS version (see hosted/AGENTS.md)
│   ├── backend/               # Python: WorkOS auth, Stripe billing, Postgres, Redis pub/sub
│   │   └── store.py           # RedisSessionStore with pub/sub for multi-instance
│   ├── deploy.sh              # Fly.io deploy script (beta/staging/prod)
│   ├── Dockerfile.beta        # Beta: relay + Redis + web viewer (multi-stage Node + Python)
│   ├── fly.beta.toml          # Beta Fly.io config (arc-beta.axolotl.ai)
│   └── AGENTS.md              # Hosted-specific agent context
├── .github/workflows/
│   ├── ci.yml                 # Lint (ESLint + ruff) + parallel Node/Python test jobs
│   ├── deploy-web-client.yml  # Deploy web client to GitHub Pages on push to main
│   ├── publish-npm.yml        # Publish @axolotlai/* packages on vX.X.X tags
│   └── publish-pypi.yml       # Publish arc-relay to PyPI on vX.X.X tags
├── eslint.config.mjs          # TypeScript linter config
├── ruff.toml                  # Python linter config (black-compatible)
├── .husky/pre-commit          # Pre-commit: lint + build
├── install.sh                 # curl | sh installer (--local flag for dev installs)
├── docker-compose.yml         # Local dev: relay + Postgres 16 + Redis 7
└── pyproject.toml             # PyPI config (setuptools-scm for version from git tags)
```

## CLI Commands

```bash
arc setup [--hermes|--self-hosted]
    # Defaults to beta relay (arc-beta.axolotl.ai), generates prefix token
    # --hermes: installs plugin, skill, patches hermes entrypoint
    # --self-hosted or ARC_ENV=dev: local relay with auto-start

arc connect [--json] [--quiet] [--framework hermes] [--name my-agent]
    # Start a remote control session
    # Writes ~/.arc/session.json and ~/.arc/viewer-url
    # Copies viewer URL to clipboard, opens browser
    # --json outputs machine-readable JSON

arc update
    # Local: git pull + npm install + npm run build + reinstall skills
    # Global: npm install -g @axolotlai/arc-cli@latest + reinstall skills
    # Re-patches hermes entrypoint, restarts local relay if dev mode

arc install-skill [framework]    # Install /remote-control skill
arc status                       # Show current config
```

## Key Design Patterns

### Extension Protocol
The relay is extensible via four protocol interfaces in `relay/protocols.py`:
- **AuthProvider** — authenticate agents, viewers, admins
- **SessionStore** — persist session state
- **SessionPolicy** — enforce session limits
- **LifecycleHooks** — react to session events

### E2E Encryption — See `hosted/AGENTS.md`
E2E encryption is a hosted feature. The wire protocol supports encrypted envelopes
(`EncryptedField`, `encrypted: true` flag) and the relay passes them through without
inspection. Implementation details are in the hosted docs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [axolotl-ai-cloud/arc](https://github.com/axolotl-ai-cloud/arc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
