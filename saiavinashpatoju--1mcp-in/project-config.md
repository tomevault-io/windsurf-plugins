---
trigger: always_on
description: > This file is the single source of truth for any AI agent or developer working on this codebase.
---

# AGENTS.md — 1mcp.in

> This file is the single source of truth for any AI agent or developer working on this codebase.
> Read this entirely before writing any code, making any architecture decision, or suggesting any change.

---

Application Name : 1mcp.in
centralMcp Name : mach1

## What This Product Is

**1mcp.in** is a local-first MCP (Model Context Protocol) hub and router for teams and individuals.

It solves two distinct problems:

**For individuals (OSS free tier):**
Managing multiple MCP servers is painful — separate installs, separate credentials, separate configs per AI client. 1mcp replaces all of that with a single router process (`mach1`) and a desktop Hub UI. One install. Every AI client (Codex, Cursor, VS Code, Codex Desktop) connects to one endpoint.

**For teams (Team Pro):**
Every developer on a team configures MCPs individually — 10 developers means 10 copies of the same setup, 10x credential sprawl, zero visibility into what AI is doing. 1mcp Team Pro gives an admin one control plane: install once, credential once, every member gets it automatically. Add agents and shared workflows on top. The team's AI clients become a unified, governed agentic layer.

---

## What This Product Is NOT

- Not a replacement for Codex, Cursor, or any AI client. It is infrastructure beneath them.
- Not an LLM. It does not generate responses.
- Not a workflow builder (like n8n or Zapier). Workflows are composed from agents, not visual flowcharts.
- Not enterprise-first. The GTM target is SMB teams of 5–50 people.

---

## North Star

> Every member of a team should be able to use every AI tool the team has approved — with zero individual setup, full admin visibility, and pre-built agents for their job function — from inside the AI client they already use.

---

## Repository Structure

```
1mcp.in/                          ← PUBLIC (open source, Apache 2.0)
├── services/
│   ├── mach1/              ← Go router, supervisor, sandbox, CLI
│   │   ├── cmd/
│   │   │   ├── mach1/      ← Main router binary
│   │   │   ├── mcpapiserver/     ← Local API (auth bridge, marketplace)
│   │   │   └── mach1ctl/        ← CLI (install, list, env)
│   │   └── internal/
│   │       ├── router/           ← Request routing, tool namespacing
│   │       ├── supervisor/       ← Process lifecycle, semantic ranking
│   │       ├── sandbox/          ← Process isolation, stdio tunneling
│   │       ├── transport/        ← stdio (current), HTTP streamable (add)
│   │       └── clouddb/          ← PostgreSQL schema (marketplace, auth)
│   └── web-ui/                   ← SvelteKit + Tauri desktop app
│       ├── src/                  ← Svelte pages, components, stores
│       ├── src-tauri/            ← Rust backend (auto-update, sqlite)
│       └── vite.config.ts
├── packages/
│   ├── mcp-manifest/             ← JSON Schema for MCP config
│   └── registry-index/           ← Curated MCP catalog (18 servers)
├── scripts/
│   ├── build.ps1                 ← Windows build (existing)
│   └── build.sh                  ← macOS/Linux build (ADD THIS)
└── .github/workflows/
    ├── ci.yml
    └── release.yml

1mcp-cloud/                       ← PRIVATE (closed source, Team Pro backend)
├── services/
│   ├── team-api/                 ← Go — workspace, members, vault, agents
│   │   ├── cmd/teamd/
│   │   └── internal/
│   │       ├── workspace/        ← Org management
│   │       ├── members/          ← Invites, roles, permissions
│   │       ├── vault/            ← Encrypted credential store
│   │       ├── agents/           ← Agent library, custom builder
│   │       ├── workflows/        ← Workflow templates
│   │       ├── activitylog/      ← Immutable audit trail
│   │       └── sync/             ← Config push to local routers
│   └── team-ui/                  ← SvelteKit web-only (no Tauri)
│       └── src/
│           ├── workspace/        ← Admin dashboard
│           ├── members/          ← Seat management
│           ├── agents/           ← Agent library UI
│           ├── vault/            ← Credential management
│           └── activity/         ← Feed + analytics
```

Railway project: 1mcp
├── Service: mcpapiserver    ← OSS cloud API (marketplace, auth)
├── Service: teamd           ← Team Pro API (workspace, vault, agents)  [ADD]
├── Service: PostgreSQL      ← Shared DB (two schemas: public + team)
└── Service: Redis           ← Job queue + config pub/sub              [ADD]
---

## Tech Stack

| Layer | Tech | Notes |
|---|---|---|
| Router | Go 1.22+ | `mach1` binary |
| CLI | Go | `mach1ctl` binary |
| Desktop UI | SvelteKit + Tauri (Rust) | Local Hub UI |
| Team API | Go | `teamd` binary, private repo |
| Team UI | SvelteKit (web) | No Tauri, browser only |
| Local DB | SQLite | Router registry, session state |
| Cloud DB | PostgreSQL (Railway) | Marketplace, auth, team data |
| Credential Vault | AES-256-GCM | Keys stored encrypted server-side |
| Transport (current) | stdio | MCP 2024-11-05 spec |
| Transport (add) | Streamable HTTP | MCP 2025-03-26 spec |
| Auth | JWT (current) → OAuth 2.1 (add) | Per MCP 2025-06-18 spec |
| Telemetry | OpenTelemetry (add) | Spans per tool call |
| CI/CD | GitHub Actions | Build + release on vX.Y.Z tag |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaiAvinashPatoju/1mcp.in](https://github.com/SaiAvinashPatoju/1mcp.in) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
