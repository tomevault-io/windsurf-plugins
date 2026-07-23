---
trigger: always_on
description: Enterprise-grade CMS built on .NET 10.0. This repository contains 21 production projects organized in a layered architecture with clear separation of concerns.
---

# Umbraco CMS - Multi-Project Repository

Enterprise-grade CMS built on .NET 10.0. This repository contains 21 production projects organized in a layered architecture with clear separation of concerns.

**Repository**: https://github.com/umbraco/Umbraco-CMS
**License**: MIT
**Main Branch**: `main`

---

## 1. Overview

### What This Repository Contains

**21 Production Projects** organized in 3 main categories:

1. **Core Architecture** (Domain & Infrastructure)
   - `Umbraco.Core` - Interface contracts, domain models, notifications
   - `Umbraco.Infrastructure` - Service implementations, data access, caching

2. **Web & APIs** (Presentation Layer)
   - `Umbraco.Web.UI` - Main ASP.NET Core web application
   - `Umbraco.Web.Common` - Shared web functionality, controllers, middleware
   - `Umbraco.Cms.Api.Management` - Backoffice Management API (REST)
   - `Umbraco.Cms.Api.Delivery` - Content Delivery API (headless)
   - `Umbraco.Cms.Api.Common` - Shared API infrastructure

3. **Specialized Features** (Pluggable Modules)
   - Persistence: EF Core (modern), NPoco (legacy) for SQL Server & SQLite
   - Caching: `PublishedCache.HybridCache` (in-memory + distributed)
   - Search: `Examine.Lucene` (full-text search)
   - Imaging: `Imaging.ImageSharp` v1 & v2 (image processing)
   - Other: Static assets, targets, development tools

**6 Test Projects**:
- `Umbraco.Tests.Common` - Shared test utilities
- `Umbraco.Tests.UnitTests` - Unit tests
- `Umbraco.Tests.Integration` - Integration tests
- `Umbraco.Tests.Benchmarks` - Performance benchmarks
- `Umbraco.Tests.AcceptanceTest` - E2E tests
- `Umbraco.Tests.AcceptanceTest.UmbracoProject` - Test instance

### Key Technologies

- **.NET 10.0** - Target framework for all projects
- **ASP.NET Core** - Web framework
- **Entity Framework Core** - Modern ORM
- **OpenIddict** - OAuth 2.0/OpenID Connect authentication
- **Microsoft.AspNetCore.OpenApi** - OpenAPI document generation
- **Swashbuckle.AspNetCore.SwaggerUI** - Swagger UI for API documentation
- **Lucene.NET** - Full-text search via Examine
- **ImageSharp** - Image processing

---

## 2. Repository Structure

```
Umbraco-CMS/
├── src/                                    # 21 production projects
│   ├── Umbraco.Core/                      # Domain contracts (interfaces only)
│   │   └── CLAUDE.md                      # ⭐ Core architecture guide
│   ├── Umbraco.Infrastructure/            # Service implementations
│   ├── Umbraco.Web.Common/                # Web utilities
│   ├── Umbraco.Web.UI/                    # Main web application
│   ├── Umbraco.Cms.Api.Management/        # Management API
│   ├── Umbraco.Cms.Api.Delivery/          # Delivery API (headless)
│   ├── Umbraco.Cms.Api.Common/            # Shared API infrastructure
│   │   └── CLAUDE.md                      # ⭐ API patterns guide
│   ├── Umbraco.PublishedCache.HybridCache/ # Content caching
│   ├── Umbraco.Examine.Lucene/            # Search indexing
│   ├── Umbraco.Cms.Persistence.EFCore/    # EF Core data access
│   ├── Umbraco.Cms.Persistence.EFCore.Sqlite/
│   ├── Umbraco.Cms.Persistence.EFCore.SqlServer/
│   ├── Umbraco.Cms.Persistence.Sqlite/    # Legacy SQLite
│   ├── Umbraco.Cms.Persistence.SqlServer/ # Legacy SQL Server
│   ├── Umbraco.Cms.Imaging.ImageSharp/    # Image processing v1
│   ├── Umbraco.Cms.Imaging.ImageSharp2/   # Image processing v2
│   ├── Umbraco.Cms.StaticAssets/          # Embedded assets
│   ├── Umbraco.Cms.DevelopmentMode.Backoffice/
│   ├── Umbraco.Cms.Targets/               # NuGet targets
│   └── Umbraco.Cms/                       # Meta-package
│
├── tests/                                  # 6 test projects
│   ├── Umbraco.Tests.Common/
│   ├── Umbraco.Tests.UnitTests/
│   ├── Umbraco.Tests.Integration/
│   ├── Umbraco.Tests.Benchmarks/
│   ├── Umbraco.Tests.AcceptanceTest/
│   └── Umbraco.Tests.AcceptanceTest.UmbracoProject/
│
├── templates/                              # Project templates
│   └── Umbraco.Templates/
│
├── tools/                                  # Build tools
│   └── Umbraco.JsonSchema/
│
├── umbraco.sln                            # Main solution file
├── Directory.Build.props                  # Shared build configuration
├── Directory.Packages.props               # Centralized package versions
├── .editorconfig                          # Code style
└── .globalconfig                          # Roslyn analyzers
```

### Architecture Layers

**Dependency Flow** (unidirectional, always flows inward):

```
Web.UI → Web.Common → Infrastructure → Core
                ↓
          Api.Management → Api.Common → Infrastructure → Core
                ↓
          Api.Delivery → Api.Common → Infrastructure → Core
```

**Key Principle**: Core has NO dependencies (pure contracts). Infrastructure implements Core. Web/APIs depend on Infrastructure.

### Project Dependencies

**Core Layer**:
- `Umbraco.Core` → No dependencies (only Microsoft.Extensions.*)

**Infrastructure Layer**:
- `Umbraco.Infrastructure` → `Umbraco.Core`
- `Umbraco.PublishedCache.*` → `Umbraco.Infrastructure`
- `Umbraco.Examine.Lucene` → `Umbraco.Infrastructure`
- `Umbraco.Cms.Persistence.*` → `Umbraco.Infrastructure`

**Web Layer**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umbraco/Umbraco-CMS](https://github.com/umbraco/Umbraco-CMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
