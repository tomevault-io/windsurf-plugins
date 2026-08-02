---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) working in this repository.

## What this is

A monorepo of Unraid tooling — four release units plus two agent-plugin
integrations. The canonical remote is **`git@github.com:dinglebear-ai/unraid.git`**.

### Identity and history (read before touching any URL)

Consolidated on **2026-07-27**. Two formerly standalone repos — `runraid` and
`incus-unraid` — were merged in and then **deleted upstream**. This repo is now the
only home for that code; do not go looking for those repos, and do not "restore" a
reference to them.

On the same date both the local directory and the GitHub repo were renamed
`unraid-mcp` → **`unraid`**. GitHub's transfer redirect still resolves the old name
(verified, including `raw.githubusercontent.com/...` paths), so **already-deployed
`.plg` install/update URLs keep working**.

Docs, manifests, `server.json`, `Cargo.toml`, the npm launcher and the curl-able
install scripts were repointed to `dinglebear-ai/unraid`. **Four files were
deliberately left on the old URL** — the three `plugins/*/*.plg` and
`plugins/mcp/source/.../unraid-mcp-update.sh`. Installed plugins poll their own
`pluginURL` and resolve `txzURL`/`REPO` at runtime, so editing those ships a
behavioural change to live Unraid boxes; that needs a deliberate release and a test
install, not a drive-by rename. They work today via the redirect and break the day
anyone creates a new `dinglebear-ai/unraid-mcp` — load-bearing on a redirect, and
tracked follow-up work.

New in-repo references must use `dinglebear-ai/unraid`.

### "unraid-mcp" means five different things

Most `unraid-mcp` strings in this tree are **correct** and must not be swept into a
rename. Only *GitHub repo URLs* are stale.

| Occurrence | Still correct? |
|---|---|
| PyPI distribution `unraid-mcp` (import `unraid_mcp`) | ✅ unchanged |
| Claude/Codex plugin `name: unraid-mcp` (`agents/unraid-py`) | ✅ unchanged |
| Unraid OS plugin dir `plugins/mcp/source/.../unraid-mcp/` and `unraid-mcp.plg` | ✅ unchanged |
| Container image `ghcr.io/dinglebear-ai/unraid-mcp` | ✅ unchanged |
| GitHub URL `github.com/dinglebear-ai/unraid-mcp` | ⚠️ stale — redirect only |

The Rust side has its own naming split: crates.io package **`unraid-rmcp`**, binary
**`runraid`**, plugin name **`runraid`**. Service port is **40010**.

## Agent plugins ship no Claude Code hooks

`agents/unraid-py/` and `agents/unraid-rs/` previously registered `SessionStart` +
`ConfigChange` hooks (`hooks/hooks.json`) that ran `scripts/plugin-setup.sh` to
persist plugin `userConfig` credentials. **Those hooks were removed on 2026-07-27.**

- Neither `.claude-plugin/plugin.json` nor `.codex-plugin/plugin.json` may declare a
  `"hooks"` key, and no `hooks/` directory may exist. Both invariants are asserted by
  `unraid-rs/tests/setup_contract.rs` and `unraid-rs/scripts/validate-plugin-layout.sh`.
- `scripts/plugin-setup.sh` is **kept** in both plugins as the manual entry point.
- **Both plugins now carry the config in `.mcp.json`.** `agents/unraid-rs` maps 10
  `userConfig` keys (`${user_config.unraid_api_url}` → `UNRAID_API_URL`, plus the
  key, TLS skip, bearer token and four OAuth vars); `agents/unraid-py` maps
  `UNRAID_API_URL` / `UNRAID_API_KEY`. No hook and no manual step are needed for
  credentials.
- **`${user_config.*}` is the correct form in `.mcp.json` — `${CLAUDE_PLUGIN_OPTION_*}`
  is not.** The latter is what Claude Code exports to plugin *subprocesses* (hook
  scripts); it is not substituted inside `.mcp.json`. `unraid-py` used that form from
  v1.5.0 (2026-06-19) on the strength of claude-code #51573, but **that issue was
  closed 2026-04-22, two months earlier** — so the workaround targeted an
  already-fixed bug and silently broke config instead. Corrected 2026-07-28.
- **Still unwired on `unraid-py`:** `unraid_verify_ssl` → `UNRAID_VERIFY_SSL` and
  `unraid_allow_insecure_tls` → `UNRAID_ALLOW_INSECURE_TLS` reach the server through
  no path. Wire them **as a pair** — `settings.py` does a module-level `sys.exit(1)`
  when `UNRAID_VERIFY_SSL=false` without `UNRAID_ALLOW_INSECURE_TLS=true`, so wiring
  only the first lets a user hard-kill the server from the settings UI.
- `scripts/plugin-setup.sh` and `runraid setup plugin-hook` remain useful for
  non-plugin (systemd/Docker) installs that need `~/.unraid/.env` written.

## Repository layout

| Path | Component | Toolchain | Build / test from repo root |
|------|-----------|-----------|------------------------------|
| `unraid-py/` | Python MCP server (**unraid-mcp** on PyPI, import `unraid_mcp`). Self-contained: its own `pyproject.toml`, `uv.lock`, `Dockerfile`, `docs/`, `openwiki/`, `scripts/`, and tests. | Python / uv / hatchling | `cd unraid-py && uv sync && uv run pytest && uv build --wheel` (run `npm --prefix tests/mock install` once to un-skip the 9 mock-server tests) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dinglebear-ai/unraid](https://github.com/dinglebear-ai/unraid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-02 -->
