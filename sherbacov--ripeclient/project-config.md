---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ripeclient** is a .NET Core client library for interacting with the RIPE Database and hosted RPKI infrastructure. Published as the "RipeClient" NuGet package, it provides three main functional areas:

1. **RIPE Database Client** - Search and CRUD operations for RIPE database objects
2. **LIR Resources Client** - Access to LIR (Local Internet Registry) resource information
3. **RPKI Client** - Management of RPKI ROAs (Route Origin Authorizations)

## Development Commands

### Building
```bash
dotnet restore
dotnet build
dotnet build -c Release
```

### Testing
```bash
# Run all tests
dotnet test

# Note: Tests are integration tests requiring environment variables:
# - LIR_RESOURCES_API_KEY for LIR tests
# Tests are currently disabled in CI/CD pipeline
```

### HTTP Debugging
```csharp
// Enable HTTP tracing for any client
var client = new RipeClient(location, auth);
client.Debug = true;  // Logs all HTTP traffic to console
```

### NuGet Package
```bash
# Package is auto-generated on build (GeneratePackageOnBuild=true)
# CI/CD automatically publishes to NuGet.org on master branch pushes
dotnet build -c Release
```

## Architecture

### Core Client Components

**RipeClient** (`src/ClientsRipe/RipeClient.cs`)
- Primary interface for RIPE Database operations (search, CRUD)
- XML-based REST API communication
- Supports async and sync operations
- Authentication via `IRipeClientAuth` implementations:
  - `RipeClientAuthAnonymous` - Public searches (no credentials)
  - `RipeClientAuthPassword` - Password via query parameter (?password=xxx)
  - `RipeClientAuthApiKey` - API key via X-API-Key header (recommended)
  - `RipeClientAuthBasic` - HTTP Basic Authentication via Authorization header
- Enhanced auth interface: `IRipeClientAuthEnhanced` with `ApplyAuthentication(RestRequest)` method
- Location via `IRipeLocation` implementations:
  - `RipeSecureLocation` - HTTPS endpoint (https://rest.db.ripe.net/)
  - `RipeNonSecureLocation` - HTTP endpoint

**LirResourcesClient** (`src/ClientsRipe/LirResources/LirResourcesClient.cs`)
- Access LIR resources via RIPE NCC API (https://lirportal.ripe.net/myresources/v1/resources)
- API key authentication via `ncc-api-authorization` header
- Returns ASN and IP allocations/assignments

**RipeRpkiClient** (`src/ClientsRipe/RpkiClient/RipeRpkiClient.cs`)
- RPKI ROA management
- Production endpoint: https://my.ripe.net/api/rpki
- Test endpoint: https://localcert.ripe.net/api/rpki
- Operations: Get resources, Get ROAs, Add/Delete ROAs (batch publish)

**RipeRouteManager** (`src/ClientsRipe/RpkiClient/RipeRpkiManager.cs`)
- High-level orchestration layer coordinating RIPE Database + RPKI
- Features:
  - Cache management for ROAs (stored as `ripe.cache.json` in assembly directory)
  - RPKI validation state checking (Valid/Invalid/Unknown)
  - Automatic ROA creation when adding routes
  - Configuration-based setup via `IConfiguration` with pattern: `rpki:keys:{index}`
- Demonstrates multi-API coordination: queries multiple RPKI keys, caches results, correlates routes with ROAs

### Search API

**RipeSearchRequest** (`src/ClientsRipe/RipeSearchRequest.cs`)
- Fluent API for building search queries
- Type filtering using flag-based enum: `TypeFilter.Route | TypeFilter.Route6`
- Available filters: Route, Route6, Inetnum, Inetnum6, Autnum, Person
- Query flags:
  - `AllMore` (-M): All sub-ranges in hierarchies
  - `OneMore` (-m): One level more specific
  - `AllLess` (-L): All encompassing ranges
  - `OneLess` (-l): One level less specific
- Source filtering (default: "ripe")

### Data Models

**RipeObject** (`src/ClientsRipe/DatabaseObjects/RipeObject.cs`)
- Base class for all RIPE database objects
- List of key-value pairs with dynamic accessor: `obj["attribute"]`
- Multi-value support (multiple attributes with same key)
- Date parsing for `created` and `last-modified`
- Maintainer management (`AddMnt`)

**Serialization Strategy**
- XML: Used for RIPE Database write operations (POST/PUT)
- JSON: Used for responses and RPKI operations
- XML namespaces are stripped for cleaner output
- Response wrappers: `RipeObjects` (JSON), `WhoisResources` (XML)

### Exception Handling

Specific exception types for different scenarios:
- `RipeClientException` - General errors
- `RipeClientBadRequestException` - 400 errors with detailed error messages
- `RipeClientNotFoundException` - 404 errors
- `RipeClientConflictException` - 409 conflicts
- `RipeClientAuthPasswordException` - Authentication failures

### Key Dependencies

- `RestSharp` (v111.4.1) - HTTP client (modern async patterns)
- `Newtonsoft.Json` (v13.0.3) - JSON serialization
- `IPNetwork2` (v3.0.667) - IP address/network manipulation (used heavily for CIDR operations)
- `HttpTracer` (v2.1.1) - HTTP debugging
- `NodaTime` (v3.1.11) - Date/time handling
- `Microsoft.Extensions.Configuration.Abstractions` (v8.0.0) - Configuration

## Important Notes

### Namespace Organization
- `ClientsRipe` - Main client code
- `RipeDatabaseObjects` - Data models
- `ClientsRpki` - RPKI functionality

### Framework Targeting
- Main library: .NET 6.0
- Tests: .NET 8.0


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sherbacov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
