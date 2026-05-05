---
trigger: always_on
description: - shinyOAuth is an S7-based OAuth/OIDC toolkit for Shiny; top-level flow lives in R/oauth_module_server.R with supporting UI glue in R/use_shinyOAuth.R.
---

# AI Coding Guide

## Project Snapshot
- shinyOAuth is an S7-based OAuth/OIDC toolkit for Shiny; top-level flow lives in R/oauth_module_server.R with supporting UI glue in R/use_shinyOAuth.R.
- Core domain objects are S7 classes in R/classes__OAuthProvider.R, R/classes__OAuthClient.R, R/classes__OAuthToken.R; prefer helper constructors (`oauth_provider_*()`, `oauth_client()`) over manual `new_class` calls.
- S7 is a modern OOP system in R; follow existing patterns for generics, methods, and validation when extending or adding classes. Access properties with `@` (e.g., `client@state_key`), define with `S7::new_property()`, and add class-level checks with `validator` arguments. See <https://rconsortium.github.io/S7/> for documentation.

## File Organization & Naming
- **Classes**: `R/classes__*.R` - S7 class definitions with `S7::new_class()`, validators, and properties
- **Methods**: `R/methods__*.R` - S7 generic implementations (login, token, userinfo, client_bearer_req)
- **Utilities**: `R/utils__*.R` - Topic-organized helpers (http, jwt, jwks, crypt, state, url, scopes, shiny, random, base64url)
- **Providers**: `R/providers.R` and `R/providers__oidc_discovery.R` - Built-in provider configs and OIDC discovery
- **Development**: `playground/*.R` - Full working examples for manual provider testing (not shipped with package)
- **Examples**: `inst/examples/*.R` - Minimal code snippets illustrating specific features (shipped with package)
- Common R idioms: use `%||%` for null coalescing, `compact_list()` to drop NULL/NA from lists, `is_valid_string()` for non-empty string checks

## Core Code Paths
- `oauth_module_server()` orchestrates redirect→callback→token→refresh, exposing a reactiveValues API (`request_login()`, `logout()`, `build_auth_url()`) and watchdogs for missing JS/browser tokens.
- Token exchange, refresh, and userinfo logic live in R/methods__login.R, R/methods__token.R, R/methods__userinfo.R; these rely on `swap_code_for_token_set()` and expect httr2 requests to pass through `add_req_defaults()` and `req_with_retry()`.
- UI resources ship via inst/www/shinyOAuth.js and must be loaded once per app via `use_shinyOAuth()`; the watchdog warning fires if `oauth_module_server()` runs before this helper sets the flag.

## Security & State
- OAuthClient instances seal state payloads with AES-GCM using `client@state_key` and single-use cache entries (`state_store_get_remove()`); share both key and cache across workers in production deployments.
- Host validation is centralized in `is_ok_host()` and enforced by OAuthProvider validators; always pipe new endpoints through these helpers and avoid bypassing the checks.
- Provider objects gate PKCE/nonce/id-token policies and `token_type` enforcement; align new provider helpers with existing defaults in R/providers.R (e.g., `allowed_algs`, `allowed_token_types`).

## HTTP & External Providers
- Every outbound call should wrap `httr2::request()` with `add_req_defaults()` for timeout/UA and `req_with_retry()` for transient handling; tune via `options(shinyOAuth.timeout, shinyOAuth.retry_*)`.
- `oauth_provider_oidc_discover()` intersects discovery metadata with caller `allowed_algs` and infers `token_auth_style`; surface configuration failures with `err_config()`/`err_http()` to retain trace ids.
- Built-in providers (`oauth_provider_github/google/microsoft/...`) illustrate `extra_token_headers`, JWKS pinning, and fingerprinting; mirror their structure when adding providers so tests can stub with `with_mocked_bindings()`.

## Shiny Integration
- Module cookies bind browser sessions using Web Crypto; tests and headless contexts can skip the requirement with `options(shinyOAuth.skip_browser_token = TRUE)` or by stubbing values via helper functions.
- Async flows require promises, future, and later; configure a plan (e.g., `future::plan(multisession)`) when enabling `async = TRUE` or tests will degrade to synchronous warnings.
- Tab title cleanup, cookie scope, and proactive refresh are all configurable arguments to `oauth_module_server()`; document new parameters with roxygen comments and guard them with `stopifnot` validations.

## Auditing & Diagnostics
- `audit_event()`/`emit_trace_event()` in R/errors.R send redacted telemetry to `options(shinyOAuth.trace_hook)` and `options(shinyOAuth.audit_hook)`; preserve hashed identifiers via `string_digest()` when logging new fields.
- All error paths should raise via `err_abort` wrappers (`err_token()`, `err_invalid_state()`, `err_userinfo()`, etc.) so trace ids and structured context propagate to Shiny logs and audit hooks.
- Options like `shinyOAuth.print_errors` and `shinyOAuth.print_traceback` let operators tune verbosity; respect these flags instead of printing directly.

## Error Handling
- Throw failures with the typed helpers in R/errors.R (`err_abort()` plus `err_token()`/`err_invalid_state()`/`err_http()`, etc.); they wrap `rlang::abort` with package-specific classes and inject trace ids, so avoid base `stop()`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lukakoning/shinyOAuth](https://github.com/lukakoning/shinyOAuth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
