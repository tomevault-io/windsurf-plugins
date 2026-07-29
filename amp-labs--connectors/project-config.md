---
trigger: always_on
description: > This document is the primary onboarding reference for building a new Ampersand connector. It synthesizes all connector development guides, auth patterns, testing conventions, and operational knowledge into a single source of truth. Treat this as your starting point before writing a single line of code.
---

# Ampersand Connector Conventions

> This document is the primary onboarding reference for building a new Ampersand connector. It synthesizes all connector development guides, auth patterns, testing conventions, and operational knowledge into a single source of truth. Treat this as your starting point before writing a single line of code.

---

## Table of Contents

1. [What is Ampersand?](#what-is-ampersand)
2. [What is a Connector?](#what-is-a-connector)
3. [Repository Structure](#repository-structure)
4. [Connector Anatomy](#connector-anatomy)
5. [Authentication Schemes](#authentication-schemes)
6. [Actions: Read, Write, Subscribe, Proxy, Search](#actions)
7. [Pagination Patterns](#pagination-patterns)
8. [Field Normalization](#field-normalization)
9. [Error Handling](#error-handling)
10. [Rate Limiting](#rate-limiting)
11. [Testing Your Connector](#testing-your-connector)
12. [Subscribe / Webhook Setup](#subscribe--webhook-setup)
13. [Common Gotchas](#common-gotchas)
14. [New Connector Checklist](#new-connector-checklist)

---

## What is Ampersand?

Ampersand is a developer-first integration infrastructure platform. It enables SaaS companies to embed native, customer-facing integrations into their products without building and maintaining the underlying plumbing.

Ampersand provides:
- **Pre-built connectors** to 100+ SaaS providers (Salesforce, HubSpot, Gong, Slack, Marketo, etc.)
- **A manifest-based API** (`amp.yaml`) for defining integration actions: reads, writes, event subscriptions, proxy, and search
- **White-labeled UI components** that end users configure to connect their own accounts
- **Managed sync infrastructure** — pagination, rate limits, retries, webhook ingestion, and field normalization are handled by Ampersand, not the application developer

**Who uses it:** SaaS developers building customer-facing integrations. Builders write `amp.yaml` manifests that declare what data to sync; Ampersand handles all the provider-specific mechanics.

---

## What is a Connector?

A **connector** is the bridge between Ampersand's unified platform and a specific SaaS provider (e.g., HubSpot, Salesforce, Gong). Each connector implements:

- **Authentication** — how to obtain and refresh tokens for a given provider
- **Read** — how to fetch records from the provider's API (with pagination, field selection, incremental sync)
- **Write** — how to create/update/delete records in the provider's API
- **Subscribe** — how to receive real-time events from the provider (webhooks, CDC, EventBridge, etc.)
- **Proxy** — routing authenticated API calls through Ampersand without Ampersand modifying the payload
- **Search** — looking up records by something other than their ID

Connectors live in the `github.com/amp-labs/connectors` repository (open source, separate from the server).

---

## Repository Structure

```
connectors/
├── providers/          # One subdirectory per provider
│   ├── hubspot/
│   │   ├── connector.go     # Main connector struct and NewConnector()
│   │   ├── params.go        # Option types (WithWorkspace, WithMetadata, etc.)
│   │   ├── read.go          # Read implementation
│   │   ├── write.go         # Write implementation
│   │   ├── metadata.go      # Field/object metadata
│   │   ├── read_test.go     # Read tests
│   │   ├── write_test.go    # Write tests
│   │   └── metadata_test.go # Metadata tests
│   ├── salesforce/
│   ├── gong/
│   └── ...
├── providers/
│   ├── hubspot.go      # ProviderInfo declaration (BaseURL, auth scheme, OAuth config, etc.)
│   ├── salesforce.go
│   └── ...
├── internal/
│   └── generated/
│       └── catalog.json    # Auto-generated from all provider declarations
├── test/               # Shared test utilities
│   └── utils.go
└── common/             # Shared utilities used across connectors
    ├── urlbuilder/
    ├── catalogreplacer/
    └── ...
```

**Key rule:** The `providers/*.go` file (e.g., `providers/gong.go`) is the *declaration* — metadata, URLs, auth config. The `providers/gong/` directory is the *implementation* — actual API calls, pagination logic, field normalization.

---

## Connector Anatomy

### The `ProviderInfo` Declaration (`providers/gong.go`)

This is the first file you create. It tells Ampersand how to talk to the provider at a high level.

```go
package providers

const Gong = "gong"

func init() {
    SetInfo(Gong, ProviderInfo{
        DisplayName: "Gong",
        AuthType:    Oauth2,
        BaseURL:     "{{.api_base_url_for_customer}}",  // templated for multi-region
        Oauth2Opts: &OauthOpts{
            GrantType:                 AuthorizationCode,
            AuthURL:                   "https://app.gong.io/oauth2/authorize",
            TokenURL:                  "https://app.gong.io/oauth2/generate-customer-token",
            ExplicitScopesRequired:    true,
            ExplicitWorkspaceRequired: false,
            TokenMetadataFields: TokenMetadataFields{
                ScopesField:      "scope",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amp-labs/connectors](https://github.com/amp-labs/connectors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
