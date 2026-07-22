---
trigger: always_on
description: Resgrid is a logistics and resource management platform for emergency services (fire, EMS, SAR). It's a .NET (C#) monolith solution organized into 30+ projects across 7 areas.
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
Config  →  Model  →  Services  →  Repositories/Providers  →  Web/Workers
```

Each layer depends only on the layer(s) to its left:
- **Config** (`Resgrid.Config`): Static configuration classes, no dependencies
- **Model** (`Resgrid.Model`): Entities, enums, interfaces — no external deps
- **Services** (`Resgrid.Services`): Business logic — depends on Model
- **Repositories** (`Resgrid.Repositories.*`): Data access — depends on Model
- **Providers** (`Resgrid.Providers.*`): External integrations — depends on Model
- **Web/Workers**: Entry points — depend on everything

### Dependency Injection (Autofac + Service Locator)

This codebase uses **Service Locator** pattern, NOT constructor injection:

```csharp
// How services are resolved throughout the codebase:
var service = Bootstrapper.GetKernel().Resolve<ISomeService>();
```

The `Bootstrapper` class (in `Resgrid.Workers.Framework/Bootstrapper.cs`) initializes Autofac with module-based registration:
```csharp
var builder = new ContainerBuilder();
builder.RegisterModule(new DataModule());
builder.RegisterModule(new ServicesModule());
builder.RegisterModule(new CacheProviderModule());
// ... more modules
_container = builder.Build();
```

**When adding new services, you MUST update the Autofac module files** (typically `DataModule.cs` or `ServicesModule.cs`) to register your new type against its interface.

### Configuration System

Configuration is NOT in `appsettings.json`. It uses **static classes with mutable fields** loaded via reflection:

1. Individual static classes in `Core/Resgrid.Config/` — one per domain (e.g., `SystemBehaviorConfig`, `CacheConfig`, `ApiConfig`)
2. All config fields are `public static` (NOT properties with getters/setters)
3. `ConfigProcessor.LoadAndProcessConfig()` uses reflection to find classes in the `Resgrid.Config` namespace and set their static fields
4. Values come from a JSON file (keyed as `"ClassName.FieldName"`) or environment variables (keyed as `RESGRID:ClassName:FieldName`)

**Usage:** `Config.SystemBehaviorConfig.CacheEnabled`, `Config.CacheConfig.RedisConnectionString`

### Caching (Redis Cache-Aside)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Resgrid/Core](https://github.com/Resgrid/Core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
