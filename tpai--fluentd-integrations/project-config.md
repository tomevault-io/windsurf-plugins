---
trigger: always_on
description: This file documents conventions for coding agents (Claude Code, Copilot, etc.) working in this repository.
---

# AGENTS.md — Coding Agent Guidelines

This file documents conventions for coding agents (Claude Code, Copilot, etc.) working in this repository.

## Repository Overview

A collection of self-contained Docker Compose examples showing how to ship logs from a containerized app (nginx) through Fluentd to a downstream log aggregation backend. Each example lives in its own top-level directory.

```
<backend>/
  docker-compose.yml          # full stack definition
  fluentd/
    Dockerfile                # extends fluent/fluentd base, installs plugin
    fluent.conf               # Fluentd pipeline config
  README.md
  screenshot.png
```

Current backends: `elasticsearch`, `loki`, `opensearch`, `splunk-hec`, `graylog`.

The `graylog` example is a CEF-focused lab with an extended layout:

```
graylog/
  docker-compose.yml
  Makefile                     # up / down / clean / send-tcp / health / fluentd targets
  .env.example                 # credential placeholders (GRAYLOG_PASSWORD_SECRET, etc.)
  fluentd/
    Dockerfile
    fluent.conf                # tail → CEF transform → out_exec (nc) to Graylog TCP
  docs/
    architecture.md
    graylog-input-setup.md     # step-by-step: create CEF TCP input in the UI
    cef-field-mapping.md
    troubleshooting.md
  samples/
    app-logs.txt               # JSON fixtures read by Fluentd tail source
    cef-events.txt             # raw CEF lines for manual nc tests
  scripts/
    gen-secret.sh
    hash-password.sh
    send-cef-tcp.sh            # feeds cef-events.txt to nc over TCP
    healthcheck.sh
```

Fluentd in the Graylog example runs under the `fluentd` Compose profile (`docker compose --profile fluentd up fluentd`) and is optional — the stack validates CEF delivery via direct `nc` commands without Fluentd.

---

## Adding a New Backend Example

Follow this checklist exactly — every example must be structurally identical so a reader can diff them mentally.

1. Create `<backend>/docker-compose.yml`
2. Create `<backend>/fluentd/Dockerfile`
3. Create `<backend>/fluentd/fluent.conf`
4. Create `<backend>/README.md`
5. Add an entry to the root `README.md` under **Available Examples**

Do not skip any of these files. Do not add extra files unless the backend genuinely requires them (e.g., a provisioning config that cannot be inlined).

---

## Naming Conventions

| Thing | Convention | Example |
|---|---|---|
| Top-level directory | `kebab-case`, lowercase, matches the plugin/product name | `splunk-hec`, `opensearch` |
| `container_name` in Compose | match the service role, lowercase, no prefix | `fluentd`, `elasticsearch`, `kibana` |
| Fluentd log tags | `<category>.<sub>` dot-notation | `httpd.access`, `app.error` |
| Environment variables | `SCREAMING_SNAKE_CASE` | `LOKI_URL`, `SPLUNK_HEC_TOKEN` |
| Compose named volumes | `<backend>-data` | `elasticsearch-data`, `opensearch-data` |
| Index / dataset names | lowercase, singular or product-default | `logs`, `main` |

---

## fluent.conf Conventions

Every `fluent.conf` must follow this exact section order:

```
1. <source> forward    (port 24224)
2. <source> http       (port 9880)
3. <match fluent.**>   stdout (Fluentd internal logs)
4. <match *.**>        copy → <store> destination + <store> stdout
```

**Exception — Graylog example:** The Graylog `fluent.conf` is a CEF lab, not a general-purpose pipeline. It uses `<source> monitor_agent` (port 24220) and `<source> tail` instead of `forward`/`http`, and a single `<match app.logs>` with `out_exec` (netcat) instead of `out_copy`. That deviation is intentional and must not be "corrected" to match the standard layout.

Rules:
- Always include both `forward` and `http` sources — they are the two standard ingestion paths (standard backends only; see Graylog exception above).
- Always mirror logs to `stdout` inside a `<store>` block so `docker compose logs fluentd` is useful during development.
- Use `flush_interval 5s` for all output plugins unless the backend has a documented reason to differ.
- Set `include_tag_key true` on every output plugin.
- Prefer `include_timestamp true` over relying on the backend to infer time.
- Read secrets (tokens, passwords) from environment variables using `"#{ENV['VAR']}"` — never hardcode credentials in `fluent.conf`.
- Keep inline comments to a minimum; only annotate non-obvious flags (e.g., `verify_es_version_at_startup false # disable version check`).

---

## Dockerfile Conventions

```dockerfile
FROM fluent/fluentd:v1.19-2      # pin the base tag; do not use :latest
USER root
# <Backend> plugin
RUN gem install <dependency>      # only if a gem dependency is required first
RUN gem install fluent-plugin-<name>
USER fluent                       # always drop back to fluent user
```

Rules:
- Pin the base image tag (`v1.19-2`, not `latest`). Current latest: `v1.19-2` (= `v1.19.2-2.3`, updated 2026-05-01).
- One `RUN gem install` per gem — do not chain with `&&` unless order matters.
- Drop privileges back to `USER fluent` as the last instruction.
- The single comment above the install block names the backend (`# Elasticsearch plugin`) — keep it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tpai/fluentd-integrations](https://github.com/tpai/fluentd-integrations) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
