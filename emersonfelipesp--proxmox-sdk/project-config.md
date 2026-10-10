---
trigger: always_on
description: This file mirrors the sibling `CLAUDE.md` guidance for agents that read `AGENTS.md`. Treat `CLAUDE.md` as the source material; the content below preserves the current guide.
---

# proxmox-sdk Agent Index — AGENTS.md Mirror

This file mirrors the sibling `CLAUDE.md` guidance for agents that read `AGENTS.md`. Treat `CLAUDE.md` as the source material; the content below preserves the current guide.

## Source

@CLAUDE.md

---

# proxmox-sdk Agent Index

## Workspace Context

This file lives at `/root/personal-context/nmulticloud-context/proxmox-sdk/CLAUDE.md` inside the `personal-context` workspace.
Workspace guidance: `/root/personal-context/CLAUDE.md`.
Per-repo deep-dive: `/root/personal-context/claude-reference/proxmox-sdk.md`.
Submodule layout and cross-repo links: `/root/personal-context/claude-reference/dependency-map.md`.

---

## Overview

Package version: `0.0.15`. Python requirement: `>=3.11`.

Generated inventories are authoritative: PVE `9.2` and `latest` each contain
444 paths / 675 operations; retained PVE `9.1.11` contains 431 paths / 649
operations; PDM `latest` contains 246 paths / 318 operations. Preserve older
release statements as historical records when these current inventories move.

`proxmox-sdk` is a schema-driven FastAPI package for Proxmox API that provides:

1. **Dual-mode operation** - Mock mode (default, SQLite/WAL CRUD) or Real mode (proxy to actual Proxmox)
2. **Standalone Python SDK** - Production-ready SDK without FastAPI server (async + sync)
3. **CLI + TUI** - Typer CLI and Textual terminal UI for interactive use
4. **Codegen pipeline** - Automatically crawl Proxmox API Viewer and convert to OpenAPI schema
5. **675 operations / 444 paths** - Pre-generated Proxmox VE 9.2 API with full OpenAPI schema (`latest` exposes the same API surface; 9.1.11 retains 649 operations / 431 paths)
6. **318 operations / 246 paths** - Pre-generated Proxmox Datacenter Manager (PDM) API with full OpenAPI schema
7. **Rate limiting** - Built-in protection via SlowAPI

## Package Structure

```
proxmox_sdk/
├── __init__.py               # Package exports and public API
├── main.py                   # Full API server (mock OR real mode)
├── mock_main.py              # Standalone mock-only server entrypoint
├── schema.py                 # Schema management (load/save OpenAPI)
├── generated/pdm/latest/     # Packaged PDM OpenAPI + raw capture artifacts
├── rate_limit.py             # SlowAPI rate limiting configuration
├── exception.py              # Exception classes
├── logger.py                 # Logging utilities
├── telemetry.py              # Optional OpenTelemetry tracing
├── routes/
│   ├── codegen.py            # Code generation endpoints (protected)
│   ├── helpers.py            # Shared route utilities
│   ├── mock.py               # Mock route handlers
│   └── versions.py           # Version management endpoints
├── proxmox/                  # Real API proxy
│   ├── routes.py             # Proxmox API proxy routes with validation
│   ├── config.py             # ProxmoxConfig dataclass
│   └── client.py             # FastAPI adapter wrapping the SDK HTTPS backend
├── ceph/                     # Ceph facade (PVE service)
│   ├── client.py             # CephClient / SyncCephClient
│   ├── models.py             # Pydantic models for Ceph responses
│   ├── _confirm.py           # confirm_destroy gate (shared)
│   ├── domains/              # Cluster, Node, Write domain helpers
│   └── providers/            # Direct provider clients (Dashboard, RGW, RBD)
│       ├── capability.py     # ProviderCapability descriptor
│       ├── dashboard.py      # DashboardCephClient
│       ├── rgw.py            # RGWAdminClient
│       └── rbd.py            # RBDClient
├── pbs/                      # Proxmox Backup Server facade
│   ├── client.py             # PBSClient / SyncPBSClient
│   ├── models.py             # Pydantic models for PBS responses
│   └── domains/              # Datastores, Jobs, Nodes, Snapshots
├── pdm/                      # Proxmox Datacenter Manager facade
│   ├── client.py             # PDMClient / SyncPDMClient
│   ├── models.py             # Pydantic models for PDM responses
│   ├── domains/              # Access, Metrics, PBS, PVE, Remotes, Resources, Views
│   └── mock/                 # PDM mock routes for proxmox-sdk-pdm-mock
├── node/                     # Node-level helpers
│   └── hardware/             # Hardware discovery
├── ssh/                      # SSH utilities
│   └── __init__.py
├── proxmox_cli/              # CLI + TUI application
│   ├── app.py                # Typer app construction and setup_logging
│   ├── batch.py              # Batch request execution
│   ├── cache.py              # Response caching layer
│   ├── cli.py                # CLI entrypoint (proxmox, pbx, proxmox-cli)
│   ├── completion.py         # Shell completion helpers
│   ├── config.py             # Config file management
│   ├── config_commands.py    # `config` subcommand group
│   ├── doc_commands.py       # `docs` subcommand group
│   ├── docgen_capture.py     # CLI docs capture pipeline
│   ├── exceptions.py         # CLI-specific exceptions
│   ├── install.py            # Self-install helpers
│   ├── output.py             # Shared output formatting
│   ├── pdm_tui_app.py        # PDM TUI entrypoint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emersonfelipesp/proxmox-sdk](https://github.com/emersonfelipesp/proxmox-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
