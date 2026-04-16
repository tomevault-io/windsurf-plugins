---
trigger: always_on
description: This repository is a **progressive workshop** demonstrating modern .NET 10 patterns across 8 incremental demos:
---

# .NET 10 demos — Copilot coding-agent notes

## Overview & Scope

This repository is a **progressive workshop** demonstrating modern .NET 10 patterns across 8 incremental demos:

| Demo        | Focus                        | Key Patterns                                                   |
| ----------- | ---------------------------- | -------------------------------------------------------------- |
| **demo1**   | Identity Foundation          | ASP.NET Core Identity, Bootstrap integration                   |
| **demo2**   | Dual-Mode Handoff (Baseline) | Blazor Server ↔ WASM transition, prerender-safe DI             |
| **demo3**   | BFF + RBAC                   | Permission-based authorization, role-permission junction table |
| **demo4**   | Entra ID Integration         | OBO flow, Microsoft Graph, enterprise auth                     |
| **demo5**   | Downstream API               | Server-to-server communication, protected scopes               |
| **demo5.1** | Distributed Modular Monolith | .NET Aspire, YARP Proxy, "Two Locks" security                  |
| **demo6**   | Multi-Tenant SaaS            | Finbuckle, Multi-Identity, Data Isolation                      |
| **demo7**   | Feature Flags & Hardening    | Microsoft.FeatureManagement, Azure AppConfig                   |

**Key assumption:** demo2 is the baseline for monolithic demos; demo5.1 is the baseline for Aspire-based distributed demos.

## Workshop Project Structure

### Standard Layout (Demos 1-5)

```
demo<N>/
├── .docs/                      # Demo-specific documentation
│   ├── issues/                 # Demo-specific markdown-based issue (deprecated)
│   └── research/               # Demo-specific implementation notes
├── Demo<N>.<Name>/             # Server (Blazor Server, APIs, Identity)
├── Demo<N>.<Name>.Client/      # Client (Blazor WASM, prerendered)
├── Demo<N>.<Name>.Shared/      # Shared DTOs, models, interfaces
└── README.md                   # Demo-specific walkthrough
```

### Aspire-Based Layout (Demos 5.1+)

```
demo<N>/
├── Demo<N>.<Name>.AppHost/          # .NET Aspire Orchestrator
├── Demo<N>.<Name>.ServiceDefaults/  # Shared telemetry/discovery
├── Demo<N>.<Name>.ApiService/       # Modular Monolith Backend
├── Demo<N>.<Name>.Web/              # Blazor Frontend (BFF with YARP)
├── Demo<N>.<Name>.Web.Client/       # Blazor WASM Client
└── Demo<N>.<Name>.Shared/           # Shared models/interfaces
```

## Creating a New Demo

### Quick Start

Use the VS Code task:
1. `Tasks: Run Task` → "Create New Demo (Copy Previous)"
2. Enter demo number and name (PascalCase)
3. Task auto-copies `demo<n-1>` → `demo<n>`, renames namespaces, cleans build artifacts

### Manual Script Approach

```powershell
scripts/copy-demo.ps1 -NewDemoNumber <n> -DemoName <PascalCase>
```

### Post-Copy Steps

1. Update namespaces to match new project name exactly
2. Update `demo<n>/README.md` with goal, prerequisites, and what's new
3. Update root `README.md` demo table with new demo entry
4. Test the build: `dotnet build demo<n>/Demo<n>.<Name>.slnx`

### Pattern Selection Rules (Required)

- Every new demo **must** introduce at least one pattern from the patterns catalog.
- Pattern selection **must** build on prior demos and reflect a strategic progression (foundation → integration → distribution → hardening).
- Document the chosen patterns in the demo README and link back to the catalog entry for each pattern.

### Namespace Convention

- Namespace follows project name exactly: `Demo<N>.<Name>` → `namespace Demo<N>.<Name>;`
- Permissions use lowercase dot-notation: `weather.read`, `users.delete`, `forecast.write`

## Authentication & Authorization Patterns

### Identity Foundation (demo1+)

- **Auth Method:** ASP.NET Core Identity cookie-based authentication
- **Schema Version:** Must use `IdentitySchemaVersions.Version3` for passkey support (demo2+)
- **Endpoints:** Keep `app.MapAdditionalIdentityEndpoints()` to wire passkey endpoints and `/Account/*` pages
- **Default Users (demo3+):**
  | Email             | Password    | Role    |
  | ----------------- | ----------- | ------- |
  | admin@local.app   | Admin123!   | Admin   |
  | manager@local.app | Manager123! | Manager |
  | user@local.app    | User123!    | User    |

### Permission-Based RBAC (demo3+)

**Data Model:**
- Role → Permissions via junction table (`Data/RolePermission.cs`)
- Permissions seeded in `Data/DbSeeder.cs`

**Request Flow:**
```
HTTP Request
  ↓
IClaimsTransformation (adds "permission" claims)
  ↓
PermissionAuthorizationHandler (enforces PermissionRequirement)
  ↓
Minimal API / Controller returns response
```

**Implementation Checklist** when adding a new permission:
1. Add to seed data in `Data/DbSeeder.cs`
2. Add server policy in `Program.cs`: `AddAuthorizationBuilder().AddPolicy("...", ...)`
3. Add client policy in `.Client/Program.cs`: `AddAuthorizationCore()` requires the `permission` claim
4. Reference in minimal APIs: `.RequirePermission("...")`  (see `Authorization/AuthorizationExtensions.cs`)
5. Reference in Razor components: `<AuthorizeView Resource="..." Action="...">`

### Entra ID Integration (demo4+)

- **Protocol:** OpenID Connect (OIDC) with Microsoft Identity Web

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arisng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-14 -->
