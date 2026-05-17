---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AIP (ATProtocol Identity Provider) is a web application that manages ATProtocol access tokens using OAuth and app-passwords. The project is built with Rust using the Axum web framework and integrates with the ATProtocol ecosystem.

## OAuth and OAuth Integrations

AIP provides OAuth login features that are backed by integrations, including ATProtocol OAuth. The following naming convention is used:

* "Base OAuth" refers to OAuth authentication process directly against AIP itself.
* "ATProtocol OAuth" (aka "atpoauth") refers to using ATProtocol OAuth authentication as a part of "Base OAuth" to create an active ATProtocol OAuth session that can be accessed with "Base OAuth" credentials.

Code pertaining to base OAuth features and functionality is organized in the `src/oauth/` package. This includes OAuth helpers and tooling that is not specific to ATProtocol OAuth.

Code pertaining to ATProtocol OAuth features and functionality is organized in the `src/oauth/atprotocol/` package.

### Client Management API

AIP includes optional client management API endpoints for dynamic client registration and management. These endpoints can be enabled or disabled using the `ENABLE_CLIENT_API` environment variable:

* When `ENABLE_CLIENT_API=true`, the following endpoints are available:
  - `POST /oauth/clients/register` - Dynamic Client Registration (RFC 7591)
  - `GET /oauth/clients/{client_id}` - Retrieve client information
  - `PUT /oauth/clients/{client_id}` - Update client configuration
  - `DELETE /oauth/clients/{client_id}` - Delete client registration

* When `ENABLE_CLIENT_API` is not set or set to any value other than "true", these client management endpoints are disabled and will return 404 responses.

The core OAuth endpoints (authorize, token, PAR) remain available regardless of this setting.

### XRPC Client Management

AIP includes an XRPC endpoint for administrative client management:

- `POST /xrpc/tools.graze.aip.clients.Update` - Update client configuration via XRPC

This endpoint requires authorization with a DID that is included in the `ADMIN_DIDS` configuration.

### Client Token Expiration

Client token expiration durations can be configured globally using environment variables:

- `CLIENT_DEFAULT_ACCESS_TOKEN_EXPIRATION` - Default access token lifetime (default: "1d")
- `CLIENT_DEFAULT_REFRESH_TOKEN_EXPIRATION` - Default refresh token lifetime (default: "14d")

These values use the `duration_str` format (e.g., "1d", "12h", "3600s").

## Common Development Commands

### Build Commands
```bash
# Build the project in debug mode
cargo build

# Build the project in release mode
cargo build --release

# Run the project
cargo run

# Run in release mode
cargo run --release
```

### Testing and Quality Assurance
```bash
# Run tests
cargo test

# Run tests with output displayed
cargo test -- --nocapture

# Run a specific test
cargo test test_name

# Check code without building
cargo check

# Format code
cargo fmt

# Check formatting without modifying files
cargo fmt -- --check

# Run clippy linter
cargo clippy

# Run clippy with all targets
cargo clippy --all-targets --all-features
```

### Documentation
```bash
# Generate and open documentation
cargo doc --open

# Generate documentation without dependencies
cargo doc --no-deps
```

## Architecture Notes

The project structure:
- Binary crate at `src/bin/aip.rs` - The main application entry point
- Library modules:
  - `src/config.rs` - Configuration types and environment variable handling
  - `src/http.rs` - HTTP server configuration and request handlers
  - `src/templates.rs` - Template engine configuration (embed/reload modes)
  - `src/errors.rs` - Error types using standardized error codes
  - `src/storage/` - Storage trait definitions and implementations
- Static files in `static/` directory
- Templates in `templates/` directory
- Database migrations in `migrations/` directory (organized by database type)
- Using Rust edition 2021 with async/await patterns
- Example applications in `examples/` directory:
  - `simple-website` - Minimal OAuth 2.1 + PAR demo with dynamic client registration
  - `dpop-website` - Example demonstrating DPoP (Demonstrating Proof of Possession)
  - `lifecycle-website` - Example showing OAuth lifecycle management
  - `react-website` - React-based frontend example

Key dependencies:
- Axum for the web framework
- ATProtocol crates for identity and OAuth support
- Minijinja for templating
- Tower for middleware
- Tokio for async runtime
- SQLx for database access (with optional SQLite and PostgreSQL support)

## Storage Implementations

AIP supports multiple storage backends through a trait-based architecture. Storage traits are defined in `src/storage/traits.rs` and can be implemented for different backends.

### Storage Traits

The following storage traits are defined:
- `OAuthClientStore` - Stores OAuth client registrations
- `AuthorizationCodeStore` - Stores OAuth authorization codes
- `AccessTokenStore` - Stores OAuth access tokens
- `RefreshTokenStore` - Stores OAuth refresh tokens
- `KeyStore` - Stores cryptographic keys for JWT signing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [graze-social/aip](https://github.com/graze-social/aip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
