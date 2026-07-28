---
trigger: always_on
description: **NorFab** (Network Automations Fabric) is a Service-Oriented Architecture (SOA) framework for extreme network automation. It runs equally on Windows, Linux, and macOS — locally on a laptop or distributed across servers.
---

# CLAUDE.md - NorFab Repository Guide

## Project Overview

**NorFab** (Network Automations Fabric) is a Service-Oriented Architecture (SOA) framework for extreme network automation. It runs equally on Windows, Linux, and macOS — locally on a laptop or distributed across servers.

- **License**: Apache-2.0
- **Python**: 3.10–3.14
- **Docs**: https://docs.norfablabs.com
- **Repo**: https://github.com/norfablabs/NORFAB

## Architecture

Three core components communicate via ZeroMQ using the NFP (NorFab Protocol):

```
Clients  ──►  Broker  ──►  Workers
```

- **Broker** (`norfab/core/broker.py`) — Central message router. Distributes jobs from clients to available service workers.
- **Workers** (`norfab/workers/`) — Service processes that execute automation tasks. Multiple workers can form a service (load-balanced).
- **Clients** (`norfab/clients/`) — Interfaces to submit jobs and retrieve results (Python API, CLI, Robot Framework).
- **Inventory** (`norfab/core/inventory.py`) — YAML-based configuration loaded at startup; supports glob patterns for worker config mapping.

### NFP Protocol

The communication protocol is defined in `norfab/core/NFP.py`. Workers, broker, and clients speak NFP over ZeroMQ sockets. Jobs are tracked by UUID in a SQLite database on the client side (`norfab/core/client.py`).

### Job Lifecycle

`NEW` → `SUBMITTING` → `DISPATCHED` → `STARTED` → `COMPLETED` / `FAILED` / `STALE`

## Directory Structure

```
norfab/
├── core/
│   ├── nfapi.py        # NorFab main class — starts broker + workers + client
│   ├── broker.py       # NFPBroker — routes jobs between clients and workers
│   ├── worker.py       # NFPWorker base class — all service workers extend this
│   ├── client.py       # NFPClient — submits jobs, stores results in SQLite
│   ├── inventory.py    # NorFabInventory — loads YAML inventory files
│   ├── NFP.py          # Protocol constants and message builders
│   ├── keepalives.py   # Keepalive heartbeat implementation
│   ├── security.py     # ZeroMQ certificate generation
│   └── exceptions.py   # Custom exceptions
├── workers/
│   ├── nornir_worker/  # Nornir network automation service
│   ├── netbox_worker/  # NetBox DCIM/IPAM integration service
│   ├── agent_worker/   # AI/LLM agent service (LangChain/Ollama)
│   ├── fastapi_worker/      # REST API service (FastAPI + Uvicorn)
│   ├── fastmcp_worker/      # Model Context Protocol (MCP) service
│   ├── workflow_worker/     # Workflow orchestration service
│   ├── containerlab_worker/  # ContainerLab integration
│   └── filesharing_worker/ # File sharing service
├── clients/
│   ├── nfcli_shell/nfcli_shell_client.py  # Interactive CLI (nfcli)
│   ├── robot_client.py        # Robot Framework library
│   ├── textual_client.py      # TUI client (Textual)
│   └── streamlit_client.py    # Web UI client (Streamlit)
├── models/
│   ├── norfab_configuration.py          # Pydantic models for inventory config
│   ├── norfab_configuration_logging.py  # Logging config models
│   ├── fastapi/                          # FastAPI response models
│   └── containerlab/                     # ContainerLab models
└── utils/
    └── nfcli.py   # CLI entry point script
tests/
├── conftest.py                      # pytest fixtures (NorFab start/teardown)
├── nf_tests_inventory/              # Test inventory (inventory.yaml + service configs)
├── services/
│   ├── containerlab/                 # Containerlab service tests by task area
│   ├── dummy/                        # Dummy plugin service tests
│   ├── fakenos/                      # FakeNOS service tests by task area
│   ├── fastapi/                      # FastAPI service tests by task area
│   ├── fastmcp/                      # FastMCP service tests by task area
│   ├── filesharing/                  # FileSharing service tests by task area
│   ├── netbox/                       # NetBox service tests by task area plus common.py helpers
│   ├── nornir/                       # Nornir service tests by task area
│   └── workflow/                     # Workflow service tests by task area
└── nfcli/                            # Interactive CLI shell tests
    ├── test_shell_client.py
    └── test_shell_common.py
docs/                  # MkDocs documentation source (Material theme)
docker/                # Docker deployment configs
```

## Common Commands

### Installation

```bash
# Core only
poetry run pip install norfab

# With CLI
poetry run pip install norfab[nfcli]

# With Nornir service
poetry run pip install norfab[nornirservice]

# Everything
poetry run pip install norfab[full]

# Development (using Poetry)
poetry install
```

### Running

```bash
# Start interactive CLI (from directory containing inventory.yaml)
poetry run nfcli

# Create a new NorFab environment scaffold
poetry run nfcli --create-env norfab
```

### Testing

```bash
# Run all tests (from repo root, requires a running/startable NorFab)
cd tests && poetry run pytest

# Run a specific service test suite
cd tests && poetry run pytest services/nornir

# Run NFCLI shell tests
cd tests && poetry run pytest nfcli
cd tests && poetry run pytest -m nfcli


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [norfablabs/NORFAB](https://github.com/norfablabs/NORFAB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
