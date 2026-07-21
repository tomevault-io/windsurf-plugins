---
trigger: always_on
description: > **READ THIS FIRST**: This is the entry point for AI agents working on the **Krafter template project** itself.
---

# Krafter AI Agent Instructions (Template Project)

> **READ THIS FIRST**: This is the entry point for AI agents working on the **Krafter template project** itself.
> This file is for **template developers** — it is NOT shipped in either template output.

## 1. Overview

Krafter is a .NET 10 full-stack **project template** that generates applications in two hosting variants:
- **Split Host** (`dotnet new krafter`): Separate Backend API + Blazor UI hosts
- **Single Host** (`dotnet new krafter-single`): Combined Backend + Blazor in one process

The template project itself contains both variants. Shared business code lives in `src/`, while hosting-specific files live in `src-single/` and `aspire-single/` (overlaid by the template engine).

**Stack**: ASP.NET Core Minimal APIs + Vertical Slice Architecture (VSA), Hybrid Blazor (WebAssembly + Server) + Radzen Components, .NET Aspire, OpenTelemetry, PostgreSQL/MySQL.

## 2. Which Instructions to Read?

```
┌──────────────────────────────────────────────────────────────────────┐
│                         DECISION TREE                                │
├──────────────────────────────────────────────────────────────────────┤
│ What are you working on?                                             │
│                                                                      │
│ ├── API endpoint, Handler, Entity, Database?                        │
│ │   └── READ: src/AditiKraft.Krafter.Backend/Agents.md              │
│ │                                                                    │
│ ├── Blazor page, Component, UI Service?                             │
│ │   └── READ: src/UI/Agents.md                                      │
│ │                                                                    │
│ ├── Shared DTOs, Requests, Responses (Contracts)?                   │
│ │   └── READ: src/AditiKraft.Krafter.Contracts/Agents.md            │
│ │                                                                    │
│ ├── Aspire orchestration, Docker, CI/CD?                            │
│ │   └── Use patterns from aspire/ and .github/workflows/            │
│ │                                                                    │
│ ├── Cross-cutting (affects both Backend + UI)?                      │
│ │   └── READ BOTH Backend + UI sub-files                            │
│ │                                                                    │
│ ├── Template configuration, overlay mechanism?                      │
│ │   └── See .template.config/ (split) and .template.config-single/  │
│ │       Single-host overlays: src-single/ and aspire-single/        │
│ │       READ: Section 4 (Template Mechanism) below                  │
│ │                                                                    │
│ └── Agents.md variant files (root-level AI instructions)?           │
│     └── Agents.md        — Template developers (this file)          │
│         Agents.split.md  — Shipped as Agents.md in krafter output   │
│         Agents.single.md — Shipped as Agents.md in krafter-single   │
│         READ: Section 5 (Agents.md Variant Strategy) below          │
└──────────────────────────────────────────────────────────────────────┘
```

## 2.1 New Feature Flow (Short Version)
1. If a feature-level `Agents.md` exists, read it first.
2. Add contracts + validators in `src/AditiKraft.Krafter.Contracts/Contracts/<Feature>/`.
3. Add permissions/routes in `src/AditiKraft.Krafter.Contracts/Common/`.
4. Add Backend operations in `src/AditiKraft.Krafter.Backend/Features/<Feature>/`.
5. Add UI Refit + pages in `src/UI/AditiKraft.Krafter.UI.Web.Client/`.

## 2.2 Deep Dives
- Backend persistence: `src/AditiKraft.Krafter.Backend/Infrastructure/Persistence/Agents.md`
- Backend background jobs: `src/AditiKraft.Krafter.Backend/Infrastructure/Jobs/Agents.md`
- Backend auth: `src/AditiKraft.Krafter.Backend/Features/Auth/Agents.md`
- Backend Users feature: `src/AditiKraft.Krafter.Backend/Features/Users/Agents.md`
- Backend roles: `src/AditiKraft.Krafter.Backend/Features/Roles/Agents.md`
- Backend tenants: `src/AditiKraft.Krafter.Backend/Features/Tenants/Agents.md`
- UI Refit: `src/UI/AditiKraft.Krafter.UI.Web.Client/Infrastructure/Refit/Agents.md`
- UI auth: `src/UI/AditiKraft.Krafter.UI.Web.Client/Features/Auth/Agents.md`
- UI users: `src/UI/AditiKraft.Krafter.UI.Web.Client/Features/Users/Agents.md`
- UI roles: `src/UI/AditiKraft.Krafter.UI.Web.Client/Features/Roles/Agents.md`
- UI tenants: `src/UI/AditiKraft.Krafter.UI.Web.Client/Features/Tenants/Agents.md`

## 2.3 Bugfix Fast Path
1. Locate the closest matching feature `Agents.md` and read it.
2. Find the exact operation file (one file per operation) and trace request/response types in Shared.
3. Apply minimal diff; verify response shape and route/permission usage.

## 3. Solution Structure

```
AditiKraft.Krafter/
├── Agents.md                    ← YOU ARE HERE (template-project version)
├── Agents.split.md              ← Split-host variant (→ Agents.md in krafter output)
├── Agents.single.md             ← Single-host variant (→ Agents.md in krafter-single output)
├── .template.config/            # Split-host template config
│   └── template.json

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AditiKraft/Krafter](https://github.com/AditiKraft/Krafter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
