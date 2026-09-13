---
trigger: always_on
description: This file is the working contract for AI coding agents in this repository. Keep
---

# AGENTS.md

This file is the working contract for AI coding agents in this repository. Keep
changes narrow, preserve existing runtime behavior, and never use real operator
credentials as fixtures or examples.

## Project Scope

Grok Register + Live Panel is a Camoufox-based registration workflow with a
local operations panel. It is intended for authorized research, integration,
and personal environments. Do not add features intended to evade access
controls, scrape or distribute third-party proxies, or conceal unauthorized
bulk abuse.

## Architecture And Ownership

- `register_flow.py`, `browser_session.py`, and `camoufox_adapter.py` own the
  browser registration flow, session lifecycle, exit checks, and Camoufox
  integration.
- `run_batch_headless.py` starts a batch. `batch_supervisor.py` supervises its
  child process, restarts driver crashes or idle runs, and resumes only the
  remaining slots through an atomic progress file.
- `run_until_100.py` owns multi-batch orchestration and runtime control state.
- `sso_to_auth_json.py` owns SSO/device OAuth conversion and CPA/Grok2API auth
  output. Successful output is not rolled back when a later slot fails.
- `email_providers/` contains provider adapters. Keep provider-specific API
  behavior there instead of branching throughout the registration flow.
- `webui/monitor.py` owns the HTTP server and embedded UI. Its focused stores
  and operations live in `webui/*_store.py` and `webui/*_ops.py`.
- `webui/sso_state_ops.py` owns the deprecated SSO botFlag / policy scan job.
  Reports must not include raw SSO tokens; clean exports stay in
  `log/sso_clean.txt`. Do not use this scan as a live risk gate.
- `quality_probe.py` and `webui/quality_ops.py` own the 降智测试: short streamed
  chat replies over 家宽/proxy pool (early-stop after thinking). Registration
  stamps `quality_*` onto CPA/Grok2API auth only when `quality_probe_on_register`
  is on (default off). Panel exports must not include access tokens. Prefer home
  proxies from `proxy_store.worker_proxy_details()`.
- `webui/proxy_store.py` owns proxy import, normalization, health, cooldown, and
  redacted API views. `webui/email_provider_store.py` owns provider config and
  secret-preserving updates. `webui/email_domain_store.py` owns domain rotation
  and rejection state.
- `runtime_platform.py` owns interpreter discovery, Xvfb selection, and process
  group flags. `webui/process_utils.py` owns project-scoped process discovery
  and termination.
- `static_asset_cache.py` is opt-in infrastructure. Its default-off behavior and
  request exclusions are security boundaries, not implementation details.
- `batch_traffic.py` owns the per-batch loopback proxy meter and aggregate byte
  state. It must never persist upstream proxy URLs, credentials, or content.

## Runtime Flows

1. A CLI or the panel starts `run_batch_headless.py` directly, or starts
   `run_until_100.py` for multiple rounds.
2. The supervisor isolates each browser batch and resumes incomplete work after
   a recoverable driver crash or idle timeout.
3. A worker selects an enabled, healthy proxy and keeps that same proxy for one
   account across registration, SSO, and OAuth.
4. Email provider/domain selection occurs before the registration form. Only an
   explicit xAI domain rejection increments domain rejection state.
5. Successful SSO can be converted to CPA/Grok2API auth. BFS detection is a JWT
   claim check separate from the deprecated grok.com `botFlagSource` page field.
   Account chat quality / 风控 is judged by `quality_probe` (short real replies),
   not SSO homepage scraping. New accounts are probed after OAuth write only when
   `quality_probe_on_register` is enabled.
6. The panel reads JSON/runtime state and controls only processes whose command
   line resolves to this project root.

## Setup And Run

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements.txt
.venv/bin/python -m camoufox fetch
cp config.example.json config.json

MONITOR_TOKEN="$(python3 -c 'import secrets; print(secrets.token_urlsafe(32))')" \
MONITOR_HOST=127.0.0.1 \
PANEL_INCLUDE_TAIL=0 \
  .venv/bin/python -u webui/monitor.py
```

For a headless Linux batch use:

```bash
xvfb-run -a .venv/bin/python -u run_batch_headless.py 20 3
```

Linux graphical sessions and macOS run the Python entry point directly.
Windows runs the same Python entry point without Xvfb. Pin Playwright Node
through `apply_playwright_node_env()`; never point `PLAYWRIGHT_NODEJS_PATH`
at the POSIX `scripts/playwright-node` wrapper on Windows. Default Windows
batches to `GROK_HEADLESS=1`. Proxy URLs must be reachable from the Windows
process itself (remote HTTP/SOCKS), not Linux-only loopback mixed ports.

## Required Verification

Run the full repository check for every code or UI change:

```bash
PYTHON_BIN=.venv/bin/python scripts/run_tests.sh
git diff --check
```

Before deployment also run:

```bash
.venv/bin/python -m pip check
.venv/bin/python -m camoufox version
.venv/bin/python scripts/harden_runtime_permissions.py .
```

Add or update a focused test under `tests/` when behavior changes. The release
script intentionally uses direct Python test files, `compileall`, shell syntax
checks, and `git diff --check`; do not assume pytest is required.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lij768423-svg/grok-register-panel](https://github.com/lij768423-svg/grok-register-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
