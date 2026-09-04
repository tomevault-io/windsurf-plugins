---
trigger: always_on
description: Provides base implementation and allows custom tenant resolution via Proc or subclass.
---

# lib/apartment/elevators/ - Rack Middleware for Tenant Switching

This directory contains Rack middleware components ("elevators") that automatically detect and switch to the appropriate tenant based on incoming HTTP requests.

## Purpose

Elevators intercept incoming requests and establish tenant context **before** the application processes the request. This eliminates the need for manual tenant switching in controllers.

## Metaphor

Like a physical elevator taking you to different floors, these middleware components "elevate" your request to the correct tenant context.

## File Structure

```
elevators/
├── generic.rb           # Base elevator with customizable logic
├── subdomain.rb         # Switch based on subdomain (e.g., acme.example.com)
├── first_subdomain.rb   # Switch based on first subdomain in chain
├── domain.rb            # Switch based on domain (excluding www and TLD)
├── host.rb              # Switch based on full hostname
├── host_hash.rb         # Switch based on hostname -> tenant hash mapping
└── header.rb            # Switch based on HTTP header (e.g., X-Tenant-Id)
```

## How Elevators Work

### Rack Middleware Pattern

All elevators are Rack middleware that intercept requests, extract tenant identifier, switch context, invoke next middleware, and ensure cleanup. See `generic.rb` for base implementation.

### v4 Constructor Pattern

v4 elevators use constructor keyword args — no class-level mutable state. The Railtie auto-inserts the elevator after `ActionDispatch::Callbacks` when `config.elevator` is set, passing `elevator_options` as keyword args.

For manual positioning (skipping `config.elevator`), options are passed at middleware insertion time:

```ruby
config.middleware.insert_before 'Warden::Manager', Apartment::Elevators::Subdomain, excluded_subdomains: %w[www api]
config.middleware.insert_before 'Warden::Manager', Apartment::Elevators::Header, header: 'X-Tenant-Id'
```

This replaces the v3 pattern of setting class attributes (`Subdomain.excluded_subdomains = [...]`) after adding to the stack. Each instance carries its own config.

### Request Lifecycle with Elevator

HTTP Request -> Elevator extracts tenant -> Switch to tenant -> Application processes -> Automatic cleanup (ensure block) -> HTTP Response

**See**: `Generic#call` method for middleware call pattern.

## Generic Elevator - Base Class

**Location**: `generic.rb`

### Purpose

Provides base implementation and allows custom tenant resolution via Proc or subclass.

### Implementation

Accepts optional Proc in initializer or expects `parse_tenant_name(request)` override in subclass. See `Generic` class implementation in `generic.rb`.

### Usage Patterns

**With Proc**: Pass Proc to Generic that extracts tenant from Rack::Request.

**Via Subclass**: Inherit from Generic and override `parse_tenant_name`.

**See**: `generic.rb` and README.md for usage examples.

## Subdomain Elevator

**Location**: `subdomain.rb`

### Strategy

Extract first subdomain from hostname.

### Implementation

Extracts subdomain via `PublicSuffix` and checks against `@excluded_subdomains` instance variable. Returns nil for excluded subdomains. See `Subdomain#parse_tenant_name` in `subdomain.rb`.

### Configuration

Pass `excluded_subdomains:` keyword arg when adding to middleware stack. See README.md for examples.

### Behavior

| Request URL                  | Subdomain | Excluded? | Tenant      |
|------------------------------|-----------|-----------|-------------|
| http://acme.example.com      | acme      | No        | acme        |
| http://widgets.example.com   | widgets   | No        | widgets     |
| http://www.example.com       | www       | Yes       | (default)   |
| http://api.example.com       | api       | Yes       | (default)   |
| http://example.com           | (empty)   | N/A       | (default)   |

### Why PublicSuffix Dependency?

**Rationale**: International domains require proper TLD parsing. Without PublicSuffix, `example.co.uk` would incorrectly parse `.uk` as the TLD rather than `.co.uk`, causing subdomain extraction to fail.

**Trade-off**: Adds gem dependency, but necessary for international domain support.

## FirstSubdomain Elevator

**Location**: `first_subdomain.rb`

### Strategy

Extract **first** subdomain from chain (for nested subdomains).

### Implementation

Splits subdomain on `.` and takes first part. See `FirstSubdomain#parse_tenant_name` in `first_subdomain.rb`.

### Configuration

Pass `excluded_subdomains:` keyword arg when adding to middleware stack. See README.md for configuration.

### Use Case

Multi-level subdomain structures where tenant is always leftmost:
- `{tenant}.api.example.com`
- `{tenant}.app.example.com`
- `{tenant}.staging.example.com`

### Note

In single-subdomain cases, `Subdomain` and `FirstSubdomain` behave identically. `FirstSubdomain` is relevant for nested structures where the tenant is always the leftmost label (e.g., `{tenant}.api.example.com`).

## Domain Elevator

**Location**: `domain.rb`

### Strategy

Use domain name (excluding 'www' and top-level domain) as tenant.

### Implementation

Extracts domain name excluding TLD and 'www' prefix. See `Domain#parse_tenant_name` in `domain.rb`.

### Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rails-on-services/apartment](https://github.com/rails-on-services/apartment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-04 -->
