---
trigger: always_on
description: Notes for Claude / contributors working on **grafana-agento11y-hermes**.
---

# CLAUDE.md

Notes for Claude / contributors working on **grafana-agento11y-hermes**.

## What this is

A Hermes Agent plugin that exports observability data to Grafana Cloud's Agent Observability (agento11y, formerly Sigil). Distributed as a pip package via the `hermes_agent.plugins` entry point under the key `agento11y`. Hermes auto-discovers it, and users opt in via `~/.hermes/config.yaml`. See the README: `hermes plugins enable` does not currently see pip-installed plugins.

## Two channels

The plugin records to two independent destinations, each with its own endpoint and basic-auth pair:

| Channel | Endpoint | What flows | Token scope |
|---|---|---|---|
| Generations | `<cloud>/api/v1/generations:export` | Normalized generation/tool-execution records (the AI Observability UI reads from here) | the setup page token |
| OTel | `OTEL_EXPORTER_OTLP_ENDPOINT` (the OTLP HTTP exporters append `/v1/traces` and `/v1/metrics` themselves) | Traces + metrics (`gen_ai.client.*`) | the same token (set via `OTEL_EXPORTER_OTLP_HEADERS=Authorization=Basic …`) |

Each channel is independently optional. If only one is configured, only that one runs. Generations are configured by `AGENTO11Y_AUTH_TOKEN` (or `AGENTO11Y_AUTH_MODE` non-`none`); OTel is configured by `OTEL_EXPORTER_OTLP_ENDPOINT`.

## Layout

- `__init__.py`: `register(ctx)` wires the hook handlers into Hermes's plugin context and applies the legacy env shim first.
- `_hooks.py`: `pre_api_request` / `post_api_request` (LLM call to generation), `api_request_error` (failed LLM call to a marked generation, then a bounded flush), `post_tool_call` (tool to tool execution), `on_session_end` / `on_session_finalize` (flush). Every handler is wrapped in `@_fail_open`. Mints the generation id at pre time, chains the generations of a turn, and starts each tool execution inside the span context of the call that requested it. See "Tags, linking and generation mode". Seeds each generation with the system prompt, tool definitions and sampling params `_request.py` read, falling back per field to this session's capture: an empty field takes the cached one, a clipped field takes a complete cached one, and between two clipped copies the longer one wins, because they are prefixes of the same value. The sampling params are the exception: they only carry forward from a capture read on the same model, because each model resolves its own cap and temperature from its own profile, while the prompt and the toolset belong to the agent and survive a fallback to another provider. A request on another model that read no params of its own leaves the cached model in place rather than taking the entry over, because hermes restores the primary runtime at the top of every turn, so a fallback lasts one turn and the session comes back to a model whose payload is by then clipped. The merged result goes back into the cache, so it can only improve. `hermes.request_facts_reused` on the record says whether any field came from an earlier request. The read runs before the sampling gate: the payloads that arrive complete are the earliest of a session, so gating them would leave every sampled-in request with an empty capture behind it.
- `_client.py`: lazy singleton `Client`. Build via the SDK's `ClientConfig` plus `GenerationExportConfig(protocol="http", auth=AuthConfig(mode="basic", ...))` when generation creds are set, `protocol="none"` otherwise. Also carries the identity tags, which reach spans and metrics from here only. Init failure is cached.
- `_otel.py`: installs `TracerProvider` and `MeterProvider` with OTLP HTTP exporters that read the `OTEL_EXPORTER_OTLP_*` envs themselves. The only kwargs passed are the derived auth headers and, for `AGENTO11Y_OTEL_EXPORTER_OTLP_ENDPOINT`, the per-signal endpoint the exporters cannot resolve. Each provider is checked independently, so the host can own one and let the plugin install the other. Force-flushes only providers we installed.
- `_config.py`: reads plugin-only knobs under `AGENTO11Y_HERMES_*` and tracks two presence flags, `generations_configured` (from `AGENTO11Y_AUTH_TOKEN` / `AGENTO11Y_AUTH_MODE`) and `otel_configured` (from `OTEL_EXPORTER_OTLP_ENDPOINT` or its `AGENTO11Y_`-prefixed alias, which `otel_endpoint_override` carries to `_otel.py` because only the standard name reaches the exporters). Channel decisions in `_client.py` and `_otel.py` are driven by these flags.
- `_compat.py`: copies retired `SIGIL_*` env vars to their `AGENTO11Y_*` names in `os.environ` before anything reads config. The old name stays, because hermes spawns subprocesses for tool calls and a telemetry plugin must not change what the host passes to its children; the cost is a duplicate SDK warning. Reads the SDK's own `_LEGACY_ENV_RENAMES` so the table cannot drift, and adds the few it omits. Temporary: delete it once the SDK reads the old names itself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexander-akhmetov/grafana-agento11y-hermes](https://github.com/alexander-akhmetov/grafana-agento11y-hermes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
