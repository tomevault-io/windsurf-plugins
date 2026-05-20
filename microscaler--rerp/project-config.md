---
trigger: always_on
description: > **Desktop dev environment** — before doing anything in this repo, read the
---

# Agent Guide for RERP Development

> **Desktop dev environment** — before doing anything in this repo, read the
> Microscaler-wide topology brief. It explains that you are on a Mac but the
> code lives on `ms02` (NFS), where commands execute for this environment, how
> the Kind cluster and vLLM fit in, and the network constraints behind the SSH
> tunneling. Do not duplicate its contents here — link to it. If reality drifts,
> fix the canonical doc, not this copy.
>
> - GitHub: [`cylon-local-infra/docs/desktop-dev-environment.md`](https://github.com/microscaler/cylon-local-infra/blob/main/docs/desktop-dev-environment.md)
> - On ms02 NFS: `~/Workspace/microscaler/cylon-local-infra/docs/desktop-dev-environment.md`

---

This document provides agentic AI systems with essential context and references for working on the RERP project.

---

## Project Overview

**RERP** (Rust Enterprise Resource Planning) is a cloud-native, microservices-based ERP system built with Rust. The project consists of **71 independent microservices**, each with its own OpenAPI specification, organized into a Rust workspace with 142 crates (71 generated + 71 implementation).

---

## Suites and BFF (Important)

**RERP is composed of suites of systems.** Each **suite** has:

- **Microservices**: `openapi/{suite}/{name}/openapi.yaml` and corresponding `microservices/` crates
- **One BFF per suite**: `openapi/{suite}/bff-suite-config.yaml` and generated `openapi/{suite}/openapi_bff.yaml` that aggregates that suite’s services

Example: the **accounting** suite has microservices (general-ledger, invoice, accounts-receivable, accounts-payable, bank-sync, asset, budget, edi, financial-reports) and one **BFF** that fronts them. Other suites (e.g. HR, sales) will have their own BFF when implemented.

Suites are inferred **dynamically**: `rerp` and related tooling list `openapi/` subdirs that contain `bff-suite-config.yaml`, read `bff_service_name` from each config, and walk `openapi/{suite}/{name}/openapi.yaml` for microservices. **No hardcoded suite names or BFF mappings** in `rerp ports`. When adding a new suite with a BFF, add `openapi/{suite}/bff-suite-config.yaml` with `bff_service_name` and tooling will pick it up.

---

## Key Architecture Principles

1. **OpenAPI-First**: All services are defined in OpenAPI 3.1.0 specifications
2. **Code Generation**: Services are generated from OpenAPI specs using BRRTRouter
3. **Two-Crate Model**: Each service has a generated crate and an implementation crate
4. **Microservices**: Independent, deployable services with well-defined APIs
5. **Suite-Level BFF**: Each suite has one BFF; `openapi/{suite}/bff-suite-config.yaml` and `openapi_bff.yaml` are per-suite

---

## Agentic AI Planning & Analysis Documents

**⚠️ CRITICAL RULE: ALL planning, analysis, design proposals, and implementation status documents MUST be created in `./docs/` or its subdirectories. Creating planning documents in the project root or other locations is NOT ALLOWED.**

### LLM Wiki

RERP has a persistent LLM wiki at [`docs/llmwiki/`](docs/llmwiki/). For substantive tooling, architecture, generator, or operational changes:

1. Read [`docs/llmwiki/SCHEMA.md`](docs/llmwiki/SCHEMA.md) and [`docs/llmwiki/index.md`](docs/llmwiki/index.md).
2. Update or create the relevant topic/entity page under `docs/llmwiki/`.
3. Update `docs/llmwiki/index.md` when adding pages.
4. Append an entry to `docs/llmwiki/log.md`.

The wiki is not a replacement for source code or ADRs. It is the reconciled, agent-facing synthesis that prevents future sessions from rediscovering and undoing known fixes.

### Document Organization

Planning documents are organized in `docs/` as follows:

- **`docs/ai/`** - AI-generated planning, analysis, and implementation status documents
- **`docs/adrs/`** - Architecture Decision Records (ADRs)
- **`docs/`** (root) - Design proposals, PRDs, and other planning documents

### Implementation Status (docs/ai/)

- **[docs/ai/OPENAPI_GENERATION_COMPLETE.md](docs/ai/OPENAPI_GENERATION_COMPLETE.md)** - OpenAPI specification generation status for all 71 services
- **[docs/ai/BFF_GENERATION_COMPLETE.md](docs/ai/BFF_GENERATION_COMPLETE.md)** - System-level BFF generation completion status
- **[docs/ai/FIRST_CI_AUTOMATION.md](docs/ai/FIRST_CI_AUTOMATION.md)** - First CI automation implementation and status

### Architecture & Planning (docs/ai/)

- **[docs/ai/SYSTEM_BFF_GENERATION.md](docs/ai/SYSTEM_BFF_GENERATION.md)** - System-level BFF generation architecture and process
- **[docs/ai/TOP_LEVEL_SPECS_PLAN.md](docs/ai/TOP_LEVEL_SPECS_PLAN.md)** - Planning document for top-level OpenAPI specifications
- **[docs/ai/CI_AUTOMATION_SETUP.md](docs/ai/CI_AUTOMATION_SETUP.md)** - CI/CD automation setup and configuration

### Analysis & Research (docs/ai/)

- **[docs/ai/ODOO_MODULES_ANALYSIS.md](docs/ai/ODOO_MODULES_ANALYSIS.md)** - Comprehensive analysis of Odoo modules and architecture patterns
- **[docs/ai/MICROSERVICE_MATRIX_AUDIT.md](docs/ai/MICROSERVICE_MATRIX_AUDIT.md)** - Microservice matrix audit and service organization analysis

### Design Proposals & PRDs (docs/)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microscaler/rerp](https://github.com/microscaler/rerp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
