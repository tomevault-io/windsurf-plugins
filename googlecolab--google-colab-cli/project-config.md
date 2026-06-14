---
trigger: always_on
description: - **CLI**: Modular `Typer` based entry point in `cli.py` with subcommands in `commands/`.
---

# Colab CLI: Agent Guidelines

## Architecture Overview
- **CLI**: Modular `Typer` based entry point in `cli.py` with subcommands in `commands/`.
- **Common**: `common.py` centralizes shared `State` (lazy-loading) and session resolution.
- **Client**: `ColabClient` handles API interactions (assignment, unassignment).
- **Auth**: `auth.py` exposes a single `get_credentials(config_path, provider)` facade that dispatches on the `AuthProvider` enum. Two providers are supported, selected via the global `--auth=oauth2|adc` flag (default `oauth2`):
  - `oauth2`: public `google-auth-oauthlib` `InstalledAppFlow`, token cached at `~/.config/colab-cli/token.json`. Reads the client OAuth config from `-c/--client-oauth-config` (default `~/.colab-cli-oauth-config.json`), falling back to the **bundled** `src/colab_cli/oauth_config.json` resource (re-added in PR #41 / `9f44fe2`, 2026-05-29 — the earlier "removed in `20eb88e`" note was stale/incorrect; the file exists and `auth.py:_get_google_auth_credentials` loads it via `importlib.resources`). As of 2026-06-11 the flow is a **remote copy-paste flow**, not a localhost server: `_run_remote_flow` sets `redirect_uri=https://sdk.cloud.google.com/applicationdefaultauthcode.html` + `token_usage=remote`, prints the URL, and reads the pasted code via `input()`. NEVER revert to OOB (`urn:ietf:wg:oauth:2.0:oob`) — Google blocked it in 2022 ("OOB flow has been blocked"); the `sdk.cloud.google.com` redirect is registered only to the bundled cloud-SDK client (`764086051850-...`), so any other client id gets `redirect_uri_mismatch`. Server-side acceptance/rejection of these variants is verifiable GET-only by building the authorization URL and inspecting whether Google reaches sign-in vs. an OAuth error page (no resources allocated).
  - `adc`: Google Application Default Credentials via `google.auth.default()`. The CLI passes `scopes=PUBLIC_SCOPES` (which includes `colaboratory`) and re-applies via `creds.with_scopes()` for credential types that support it. **User credentials minted by `gcloud auth application-default login` ignore the `scopes=` kwarg AND raise `NotImplementedError` on `with_scopes`**: ADC users must explicitly re-authenticate with `gcloud auth application-default login --scopes=openid,https://www.googleapis.com/auth/cloud-platform,https://www.googleapis.com/auth/userinfo.email,https://www.googleapis.com/auth/colaboratory`. `userinfo.email` is required by the session backend at `colab.research.google.com` (assign/unassign/sessions return 401 without it); `colaboratory` is required by the `RuntimeService` at `colab.pa.googleapis.com` (keep-alive returns 403 without it); `openid` and `cloud-platform` are mandated by `gcloud` itself, which rejects scope lists that omit `cloud-platform` with `Invalid value for [--scopes]`. Service-account / GCE / GKE / impersonated creds get the right scopes transparently via `with_scopes`.
- **Backend Hosts**: Two distinct backends with different requirements:
  - `colab.research.google.com` (session backend / `tun/m/...`): accepts the `userinfo.email` scope.
  - `colab.pa.googleapis.com` (`RuntimeService`, used by `KeepAliveAssignment`): requires (a) the `colaboratory` OAuth scope, AND (b) `X-Goog-Api-Client` header containing the substring `grpc-web`. Both are enforced server-side; missing either yields HTTP 400 / 403 with descriptive `google.rpc.DebugInfo` payloads. Always log `response_body` on failure for these RPCs to avoid silent debugging.
- **Runtime**: `ColabRuntime` wraps `jupyter-kernel-client` for execution.
- **State**:
  - `StateStore` persists session metadata in `~/.config/colab-cli/sessions.json`.
  - Persistent settings are in `~/.config/colab-cli/settings.json`.
- **History**: `HistoryLogger` records structured events in `~/.config/colab-cli/history/*.jsonl`.

## Core Mandates
- **Minimalism**: Favor standard library where possible (e.g., `urllib`) while utilizing `Typer` for CLI ergonomics.
- **Piping**: Always consider piped input (`stdin`) vs. interactive TTY.
- **Trace Alignment**: When implementing new endpoints, validate against captured browser traces (HAR files).
- **TDD (Test-Driven Development)**: Always implement tests first. Verify they fail before implementing the solution to make them pass. Every design must include a testing strategy and specific test cases.

- **Jupyter Protocol Deviations**: Google Colab uses custom extensions to the Jupyter protocol. Examples include `colab_request` messages over the `iopub` channel and `input_reply` wrapping `colab_reply` payloads on the `stdin` channel. These require monkey-patching or specialized handlers within `jupyter-kernel-client` (e.g., `wsclient.kernel_socket.on_message` interceptors).

- **Integration Testing**: Unit tests and mocks are not enough. Before declaring any feature complete, you MUST perform a real-world, end-to-end integration test against a live Colab environment using the CLI. Never rely solely on mocked unit tests to verify a feature's correctness.
    - Integration tests are located in `integration/` (e.g., `integration/repro_plot_redirection/test.sh`).
    - To run an integration test, use: `uv run bash integration/repro_<name>/test.sh`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [googlecolab/google-colab-cli](https://github.com/googlecolab/google-colab-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
