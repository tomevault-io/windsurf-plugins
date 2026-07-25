---
trigger: always_on
description: - **Main Module**: `msal4j-sdk/` (focus here for most work)
---

# MSAL Java - Copilot Instructions

## Quick Reference

- **Main Module**: `msal4j-sdk/` (focus here for most work)
- **Main Package**: `com.microsoft.aad.msal4j`
- **Three Application Types**: `PublicClientApplication`, `ConfidentialClientApplication`, `ManagedIdentityApplication`
- **Pattern**: Each auth flow has `*Parameters` (public API), `*Request` (internal), `*Supplier` (executor)
- **Current Version**: 1.25.1

---

## Project Overview

**Microsoft Authentication Library for Java (MSAL4J)** is a library that enables applications to integrate with the Microsoft identity platform. It provides APIs for acquiring security tokens from Azure Active Directory (Azure AD), Microsoft Accounts, and Azure AD B2C.

- **Language**: Java 8+
- **Build Tool**: Maven
- **Current Version**: 1.25.1
- **Artifact**: `com.microsoft.azure:msal4j`
- **Key Protocols**: OAuth2, OpenID Connect

### Core Capabilities
- Sign in users with Microsoft identities
- Acquire access tokens for protected APIs (Microsoft Graph, custom APIs)
- Token caching and automatic refresh
- Support for various authentication flows (interactive, silent, client credentials, on-behalf-of, device code, managed identity)
- Multi-cloud and B2C support

### Repository Structure
This repository contains three Maven modules:
- **`msal4j-sdk/`** - The main MSAL Java library (focus of development)
- **`msal4j-brokers/`** - Broker integration for native authentication (Windows WAM)
- **`msal4j-persistence-extension/`** - Cross-platform token cache persistence helpers

For most work, focus on **`msal4j-sdk/`**.

---

## Architecture & Structure

### Application Hierarchy

MSAL4J provides three main application types, all extending from a common base:

```
AbstractApplicationBase (base for all applications)
├── AbstractClientApplicationBase (base for client applications)
│   ├── PublicClientApplication (desktop, mobile apps)
│   └── ConfidentialClientApplication (web apps, web APIs, daemons)
└── ManagedIdentityApplication (Azure managed identity)
```

**Key Classes:**
- `PublicClientApplication` - For apps that cannot securely store secrets (desktop/mobile). Supports interactive, device code, integrated Windows auth flows.
- `ConfidentialClientApplication` - For apps that can securely store secrets (web apps, APIs, daemons). Supports client credentials and on-behalf-of flows.
- `ManagedIdentityApplication` - For Azure resources using managed identities (VMs, App Service, Functions, etc.)

### Important Directories

**Main source code:**
- `msal4j-sdk/src/main/java/com/microsoft/aad/msal4j/` - All library code
  - Core application classes: `PublicClientApplication`, `ConfidentialClientApplication`, `ManagedIdentityApplication`
  - Token cache: `TokenCache`, `TokenCacheAccessContext`, cache entity classes
  - Authentication flows: `*Request.java`, `*Parameters.java`, `*Supplier.java` classes
  - Authority handling: `Authority`, `AADAuthority`, `B2CAuthority`, `ADFSAuthority`
  - Token request execution: `TokenRequestExecutor`, `OAuthHttpRequest`
  - HTTP layer: `HttpHelper`, `DefaultHttpClient`, `IHttpClient`
  - Exceptions: `MsalException` and subclasses

**Test code:**
- `msal4j-sdk/src/test/java/` - Unit tests
- `msal4j-sdk/src/integrationtest/java/` - Integration tests (require test infrastructure)

**Samples:**
- `msal4j-sdk/src/samples/` - Example applications demonstrating various scenarios

### Key Architectural Patterns

**Builder Pattern**: All application types use fluent builders (e.g., `PublicClientApplication.builder(clientId).authority(...).build()`)

**Request/Parameters Pattern**: Each token acquisition flow has:
- A `*Parameters` class (e.g., `InteractiveRequestParameters`) - User-facing API
- A `*Request` class (e.g., `InteractiveRequest`) - Internal request representation
- A `*Supplier` class (e.g., `AcquireTokenByInteractiveFlowSupplier`) - Executes the flow

**Token Flow Execution**:
1. Application receives `*Parameters` from developer
2. Converts to internal `MsalRequest` (via `*Request` classes)
3. Routes to appropriate `AuthenticationResultSupplier` implementation
4. Supplier checks cache, executes HTTP requests via `TokenRequestExecutor`
5. Returns `AuthenticationResult` with tokens and account info

**Token Cache**: 
- In-memory cache with five entity types: `AccessTokenCacheEntity`, `RefreshTokenCacheEntity`, `IdTokenCacheEntity`, `AccountCacheEntity`, `AppMetadataCacheEntity`
- Implements `ITokenCache` with serialization hooks via `ITokenCacheAccessAspect`
- Thread-safe with read-write locks

**Service Bundle**: Internal `ServiceBundle` class provides shared services (HTTP client, telemetry, executor service) to all requests

---

## Authentication Flows & Public APIs

MSAL4J supports multiple authentication flows, each with a public `*Parameters` class (used by developers) and an internal `*Supplier` class (executes the flow logic). Each flow is available only on specific application types.

### Public Client Flows (PublicClientApplication)

**Interactive Flow**
- **Public API**: `acquireToken(InteractiveRequestParameters)`
- **Parameters**: `InteractiveRequestParameters` - Opens system browser for user authentication
- **Internal**: `InteractiveRequest` → `AcquireTokenByInteractiveFlowSupplier`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-for-java](https://github.com/AzureAD/microsoft-authentication-library-for-java) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
