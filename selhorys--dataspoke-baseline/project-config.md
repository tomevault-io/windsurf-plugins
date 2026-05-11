---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

DataSpoke is a sidecar extension to DataHub that ships a five-feature baseline (Ingestion Control, Validation, Ontology Generation, Metadata Generation, Governance) plus a Productized Scaffold (AI Scaffold + Development Scaffold) for building custom Spokes. User-group framing (DE / DA / DG) is a UI and API extensibility surface only: `/spoke/dg/` carries baseline governance routes; `/spoke/de/` and `/spoke/da/` are reserved for organization-specific extensions. Application source code (`src/`) will be generated using the scaffold's subagents. Read `spec/MANIFESTO_en.md`, `spec/ARCHITECTURE.md`, and `spec/AI_SCAFFOLD.md` for the full picture.

## Shell Commands

Run every command from the directory it expects (usually project root). Do not `cd` away mid-session — use relative paths instead.

## Dev Environment

```bash
cd dev_env && ./install.sh    # Install infrastructure (DataHub, PostgreSQL, Redis, Airflow)
cd dev_env && ./uninstall.sh  # Tear down everything
# Component reinstall: run the component's uninstall.sh then install.sh
cd dev_env && bash dataspoke-infra/uninstall.sh && bash dataspoke-infra/install.sh
```

Settings in `dev_env/.env`. See `dev_env/README.md` for access details and ingress endpoints.

The API runs **in-cluster** alongside Airflow so that workflow callbacks work via cluster DNS. Developers access it via nginx-ingress (`http://app.<INGRESS_IP>.nip.io/api/v1/`). Code changes require `docker build` + `helm upgrade` (automated by `dev_env/dataspoke-test-mode.sh`). For optional host-mode development (no Airflow callbacks): `uv run -m src.cli`.

The dev environment uses the same umbrella Helm chart as production (`helm-charts/dataspoke/`) with a dev overlay (`values-dev.yaml`). See `spec/TESTING.md §Testing Modes`.

## Key Design Decisions

- **DataHub-backed SSOT**: DataHub stores metadata; DataSpoke extends without modifying core
- **API-first**: FastAPI implementation in `src/api/` is the SSOT for the API contract; all APIs follow `spec/API_DESIGN_PRINCIPLE_en.md`
- **Three-tier API routing**: `/api/v1/spoke/common/…`, `/api/v1/spoke/[de|da|dg]/…`, `/api/v1/hub/…`
- **Airflow 3.1.8** for workflow orchestration (fixed schedule tiers + on-demand HTTP triggers, LocalExecutor); **PostgreSQL 17** (with `pgvector` for vector search and Apache `age` for ontogen triple graph materialization) for operational DB
- **Headless / API-first**: backend's primary task is to support `spec/API.md`; frontend is a thin reference UI that consumes API routes verbatim (no invented endpoints); per `spec/feature/FRONTEND_BASIC.md` no streaming surface exists in the baseline — clients poll `event/...` and `attr/.../result`
- **No DataHub CLI**: The `datahub` CLI requires Python ≤ 3.11 and is incompatible with the project's Python 3.13 runtime. Use Python scripts with the `acryl-datahub` SDK instead.
- **DataHub debugging protocol**: For any DataHub integration or infrastructure issue, consult `ref/github/datahub/` source code and use the `/datahub-api` skill before guessing configs or iterating through Helm upgrades.
- **Reference when implementing**: `spec/DATAHUB_INTEGRATION.md` for DataHub interactions; `spec/API.md` for routes, auth, middleware, error codes; `spec/feature/BACKEND.md` for backend services, workflows; `spec/feature/BACKEND_SCHEMA.md` for DB schema (relational + pgvector tables); `spec/feature/FRONTEND_*.md` for UI layout, workspace pages, shared components

## Spec Convention

Specs must not contradict each other — propagate changes up and down. Priority order:

| Priority | Documents | Role |
|----------|-----------|------|
| 1 | `MANIFESTO_en/kr.md`, `API.md`, `USE_CASE_en/kr.md` | Golden product identity, API contract, and scenario set. Never modify unless explicitly requested; everything else syncs to these. |
| 2 | `API_DESIGN_PRINCIPLE_en/kr.md`, `DATAHUB_INTEGRATION.md` | Binding conventions. |
| 3 | `ARCHITECTURE.md`, `TESTING.md` | System architecture and testing conventions. |
| 4 | `AI_SCAFFOLD.md`, `AI_PRAUTO.md` | Claude Code scaffold conventions; autonomous PR worker. |
| 5 | `feature/<FEATURE>.md` | Common feature specs and user-group-specific FRONTEND specs (`FRONTEND_DE/DA/DG.md`). |

When both `_en.md` and `_kr.md` exist, read only English unless directed otherwise. Write Korean in plain style (-다/-한다).

In spec, focus on architecture, decisions, and constraints. From spec, remove verbatim template code, full code blocks, and script snippets that duplicate the impl files.

## Git Commit Convention

- Conventional Commits: `<type>: <subject>` (e.g. `feat:`, `fix:`, `docs:`, `refactor:`)
- **Always run `git diff` (or `git diff --staged`) and base the commit message on the actual diff output**, not on prior conversation context or memory of what was changed
- Body optional, **max 5 lines** if included

## Implementation Workflow

The scaffold uses a **plan → approve → generate → evaluate** architecture. This separation prevents the self-praise failure mode where agents approve their own mediocre work.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selhorys/dataspoke-baseline](https://github.com/selhorys/dataspoke-baseline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
