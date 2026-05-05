---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**shepard-obs-stack** ("The Eye") — Docker-based observability for AI coding assistants (Claude Code, Codex, Gemini CLI). 
Hybrid telemetry: bash hooks emit OTLP metrics (git context + tool/event counters); native OTel export provides logs, traces, and richer provider-specific metrics. 
All data flows through OTel Collector into Prometheus, Loki, and Tempo; 9 Grafana dashboards auto-provision on startup.

## Quick Start

```bash
./scripts/init.sh               # bootstrap: env, docker compose up, health check
./hooks/install.sh              # inject hooks + native OTel into CLI configs
./scripts/test-signal.sh        # verify pipeline (11 checks)
```

Open http://localhost:3000 (admin / shepherd).

## Common Commands

```bash
# Stack lifecycle
docker compose up -d                          # start all 6 services
docker compose down                           # stop (preserves volumes)
docker compose down -v                        # stop + delete all data
docker compose restart otel-collector         # restart single service
docker compose logs -f loki --tail=50         # tail service logs

# Verify services
curl -s http://localhost:3100/ready           # Loki health
curl -s http://localhost:3000/api/health      # Grafana health
curl -s http://localhost:9090/-/healthy       # Prometheus health

# Hook management
./hooks/install.sh claude                     # install for specific CLI
./hooks/uninstall.sh                          # remove all hooks + native OTel
./hooks/install.sh codex gemini               # selective install

# Test a hook manually (simulate Claude PostToolUse)
echo '{"tool_name":"Read","tool_response":"ok","cwd":"/tmp"}' | bash hooks/claude/post-tool-use.sh

# Query Prometheus directly
curl -s 'http://localhost:9090/api/v1/query?query=shepherd_tool_calls_total' | jq .
curl -s 'http://localhost:9090/api/v1/query?query=shepherd_events_total' | jq .

# Query Loki directly
curl -s 'http://localhost:3100/loki/api/v1/query_range' \
  --data-urlencode 'query={service_name="claude-code"}' --data-urlencode 'limit=5' | jq .

# Render C4 architecture diagrams (requires Docker)
./scripts/render-c4.sh
```

## Port Map

| Port | Service        | Description          |
|------|----------------|----------------------|
| 3000 | Grafana        | Dashboards & explore |
| 3100 | Loki           | Log aggregation      |
| 9090 | Prometheus     | Metrics & alerts     |
| 9093 | Alertmanager   | Alert routing        |
| 3200 | Tempo          | Distributed tracing      |
| 9095 | Tempo          | gRPC (internal)          |
| 4317 | OTel Collector | OTLP gRPC receiver       |
| 4318 | OTel Collector | OTLP HTTP receiver       |
| 8888 | OTel Collector | Collector metrics        |
| 8889 | OTel Collector | Prometheus exporter (internal, not host-exposed) |

## Architecture & Data Flow

```
AI CLI (Claude Code / Codex / Gemini)
    ├── hooks/*.sh (git context + tool/event counters)
    │   └── curl POST → OTel Collector :4318 (OTLP HTTP → metrics)
    └── native OTel → OTel Collector :4317 (gRPC)
        ├── metrics → Prometheus (claude_code.*, gen_ai.client.*)
        ├── logs → Loki ({service_name="claude-code"}, {service_name="codex_cli_rs"}, {service_name="gemini-cli"})
        └── traces → Tempo
    ▼
Loki :3100
    └── recording rules → Prometheus :9090 (Codex metrics, 15 rules, 1m interval)
    ▼
Grafana dashboards:
    Unified (01-04):   hook metrics (tools/events) + native OTel metrics (cost/tokens)
    Deep-Dive (10-12): native OTel metrics + logs (provider-specific)
    Session Timeline (13): synthetic traces from JSONL parser
    ▲
Prometheus :9090 ← scrapes OTel Collector :8889 + Tempo :3200
    └─→ Alertmanager :9093 → Telegram/Slack/Discord
```

**Key pipeline detail:** Hooks emit DELTA sum metrics. OTel Collector's `deltatocumulative` processor converts them to cumulative counters before Prometheus scrapes them. 
The Prometheus exporter applies `shepherd` namespace, so all metrics get the `shepherd_` prefix.

## Key Conventions

**Metrics naming:** All metrics in Prometheus have `shepherd_` prefix (applied by OTel Collector's Prometheus exporter namespace). 
Hook metrics additionally have `_total` suffix (counters). 
Native OTel metrics: dots become underscores (e.g., `claude_code.cost_usage.USD` → `shepherd_claude_code_cost_usage_USD_total`).

**Fire-and-forget hooks:** `hooks/lib/metrics.sh:emit_counter()` uses `curl -s & disown` to avoid blocking the CLI.
Hooks must never block or slow down the AI assistant.

**Rust accelerator resolution:** All hooks source `hooks/lib/accelerator.sh` which sets `$SHEPARD_HOOK` via 3-step lookup:
`hooks/bin/shepard-hook` (project-local) → `command -v shepard-hook` (PATH) → empty (bash fallback).
Install with `./scripts/install-accelerator.sh` — downloads to `hooks/bin/` (gitignored, no sudo).

**Dashboard provisioning:** Dashboards in `configs/grafana/dashboards/*.json` are auto-loaded by Grafana on startup. 
Edits made in the Grafana UI are **lost on container restart**. 
Always edit the JSON files directly. 
Tools and Operations dashboards use `$source` and `$git_repo` variables. 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shepard-system/shepard-obs-stack](https://github.com/shepard-system/shepard-obs-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
