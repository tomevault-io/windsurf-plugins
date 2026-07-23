---
trigger: always_on
description: <!-- dgc-policy-v1 -->
---

<!-- dgc-policy-v1 -->
# Dual-Graph Context Policy

This project uses a local dual-graph MCP server (graperoot-pro) for efficient,
budget-aware context retrieval. Always prefer it over native file exploration.

## MANDATORY: Always follow this order

1. **Call `graph_continue` first** -- before any file exploration, grep, or code reading.

2. **If `graph_continue` returns `needs_project=true`**: call `graph_scan` with the
   current project directory (`pwd`). Do NOT ask the user.

3. **If `graph_continue` returns `skip=true`**: project is too small for the graph to
   help. Skip all graph tools and explore normally.

4. **Read `recommended_files`** using `graph_read` -- one call per file.
   - `recommended_files` may contain `file::symbol` entries (e.g. `src/auth.ts::handleLogin`).
     Pass them verbatim to `graph_read(file: "src/auth.ts::handleLogin")` -- it reads only
     that symbol's lines, not the full file.

5. **Check `confidence` and obey the caps strictly:**
   - `confidence=high` -> Stop. Do NOT grep or explore further.
   - `confidence=medium` -> If recommended files are insufficient, call `fallback_rg`
     at most `max_supplementary_greps` time(s) with specific terms, then `graph_read`
     at most `max_supplementary_files` additional file(s). Then stop.
   - `confidence=low` -> Call `fallback_rg` at most `max_supplementary_greps` time(s),
     then `graph_read` at most `max_supplementary_files` file(s). Then stop.

## Exhaustive enumeration tasks

Some tasks require scanning **every file** -- e.g. "find all dead exports", "list every
.find() without a limit", "audit all test files". Use these tools first:

- **`graph_dead_exports()`** -- pre-computed at scan time. Use for any dead-export task.
- **`graph_grep_all(pattern, file_glob?, max_hits?)`** -- exhaustive grep, no call cap.

## Rules

- Do NOT use `rg`, `grep`, or bash file exploration before calling `graph_continue`.
- Do NOT do broad/recursive exploration at any confidence level.
- After edits, call `graph_register_edit(files: ["path/to/file"])`. The parameter is
  `files` (plural, always an array). Use `file::symbol` notation when the edit targets
  a specific function, class, or hook.
<!-- /dgc-policy-v1 -->

---

# Resgrid Project Guide

## Overview

Resgrid is a logistics and resource management platform for emergency services (fire, EMS, SAR). It's a .NET (C#) monolith solution organized into 30+ projects across 7 areas.

## Solution Structure

```
Resgrid.sln
├── Web/                          # ASP.NET web apps
│   ├── Resgrid.Web/              # Main MVC web application
│   ├── Resgrid.Web.Services/     # REST API (v4 controllers)
│   ├── Resgrid.Web.Eventing/     # Webhook/event endpoint
│   ├── Resgrid.Web.Mcp/          # MCP endpoint
│   └── Resgrid.Web.Tts/          # Text-to-speech
├── Core/                         # Core business logic
│   ├── Resgrid.Config/           # Static config classes (one per domain)
│   ├── Resgrid.Framework/        # Utilities: Logging, Serialization, Hashing
│   ├── Resgrid.Localization/     # Localization strings
│   ├── Resgrid.Model/            # Entities, enums, interfaces (Services, Repositories, Providers)
│   └── Resgrid.Services/         # Service implementations
├── Repositories/                 # Data access
│   ├── Resgrid.Repositories.DataRepository/   # SQL Server / Dapper
│   └── Resgrid.Repositories.NoSqlRepository/  # MongoDB
├── Providers/                    # Infrastructure implementations
│   ├── Resgrid.Providers.Cache/       # Redis caching (AzureRedisCacheProvider)
│   ├── Resgrid.Providers.Bus/         # Azure Service Bus
│   ├── Resgrid.Providers.Bus.Rabbit/  # RabbitMQ alternative
│   ├── Resgrid.Providers.Email/       # Email delivery
│   ├── Resgrid.Providers.Geo/         # Geolocation
│   ├── Resgrid.Providers.Marketing/   # Marketing/CRM
│   ├── Resgrid.Providers.Messaging/   # Push notifications
│   ├── Resgrid.Providers.Migrations/  # SQL Server migrations
│   ├── Resgrid.Providers.MigrationsPg/# PostgreSQL migrations
│   ├── Resgrid.Providers.Number/      # Phone number provisioning
│   ├── Resgrid.Providers.Pdf/         # PDF generation
│   ├── Resgrid.Providers.Voip/        # VoIP/SIP
│   ├── Resgrid.Providers.Weather/     # Weather data
│   ├── Resgrid.Providers.Workflow/    # Workflow execution
│   ├── Resgrid.Providers.Claims/      # Custom auth claims
│   └── Resgrid.Providers.AddressVerification/
├── Workers/                      # Background job processing
│   ├── Resgrid.Workers.Framework/     # Worker logic + Bootstrapper
│   ├── Resgrid.Workers.Console/       # Worker host (console app)
│   └── Support/Quidjibo.Postgres/     # Queue backend for PostgreSQL
├── Tests/                        # Test projects
│   ├── Resgrid.Tests/
│   ├── Resgrid.SmokeTests/
│   └── Resgrid.Intergration.Tests/
└── Tools/
    └── Resgrid.Console/          # Admin CLI tools
```

## Build Configurations

7 solution configurations: `Debug`, `Release`, `Docker`, `Azure`, `Cloud`, `Staging`, plus `x86`/`x64` variants.

Build command: `dotnet build Resgrid.sln`

The `Directory.Build.props` sets OS-conditional intermediate output paths:
- Windows: `obj/windows/`
- Linux/Unix: `obj/unix/`

## Architecture & Conventions

### Layered Architecture

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Resgrid/Core](https://github.com/Resgrid/Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
