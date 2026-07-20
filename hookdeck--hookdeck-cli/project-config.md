---
trigger: always_on
description: This repository contains the Hookdeck CLI, a Go-based command-line tool for managing webhook infrastructure. When working with this codebase, please follow these guidelines to maintain consistency and ensure proper functionality.
---

# AGENTS Guidelines for Hookdeck CLI

This repository contains the Hookdeck CLI, a Go-based command-line tool for managing webhook infrastructure. When working with this codebase, please follow these guidelines to maintain consistency and ensure proper functionality.

## 1. Project Structure & Navigation

### Core Directories
- `pkg/cmd/` - All CLI commands (Cobra-based)
- `pkg/hookdeck/` - API client and models
- `pkg/config/` - Configuration management
- `pkg/listen/` - Local webhook forwarding functionality
- `cmd/hookdeck/` - Main entry point
- `REFERENCE.md` - Complete CLI documentation and examples

### Key Files
- `https://api.hookdeck.com/2025-07-01/openapi` - API specification (source of truth for all API interactions)
- `pkg/cmd/sources/` - Fetches and caches the OpenAPI spec for source type enum and auth rules; use for validation and help in source and connection management
- `pkg/cmd/helptext.go` - Shared Short/Long help for resource commands (sources, connections); use when adding or editing command help to avoid duplication
- `.plans/` - Implementation plans and architectural decisions
- `AGENTS.md` - This file (guidelines for AI agents)

## 2. OpenAPI to CLI Conversion Standards

When adding new CLI commands that interact with the Hookdeck API, follow these conversion patterns:

### Parameter Mapping Rules
```bash
# Nested JSON objects → Flat CLI flags
API: { "configs": { "strategy": "final_attempt" } }
CLI: --strategy final_attempt

# Arrays → Comma-separated values
API: { "connections": ["conn_1", "conn_2"] }
CLI: --connections "conn_1,conn_2"

# Boolean presence → Presence flags
API: { "channels": { "email": {} } }
CLI: --email

# Complex objects with values → Value flags
API: { "channels": { "slack": { "channel_name": "#alerts" } } }
CLI: --slack-channel "#alerts"
```

### Flag Naming Conventions
- **Resource identifiers**: Always `--name` for human-readable names
- **Type parameters**: 
  - **Individual resource commands**: Use `--type` (clear context)
    - Sources: `hookdeck source create --type STRIPE`
    - Destinations: `hookdeck destination create --type HTTP`
    - Issue Triggers: `hookdeck issue-trigger create --type delivery`
  - **Connection creation**: Use prefixed flags to avoid ambiguity when creating inline resources
    - `--source-type STRIPE` when creating source inline
    - `--destination-type HTTP` when creating destination inline
    - This prevents confusion between source and destination types in single command
- **Authentication**: Standard patterns (`--api-key`, `--webhook-secret`, `--basic-auth`)
  - **Connection creation**: Use prefixed authentication to avoid collisions
    - `--source-webhook-secret` for source authentication
    - `--destination-api-key` for destination authentication
- **Collections**: Use comma-separated values (`--connections "a,b,c"`)
- **Booleans**: Use presence flags (`--email`, `--pagerduty`, `--force`)

### Ordered Array Configurations

For API arrays where **order matters** (e.g., rules, processing steps, middleware):

**Pattern:** Use flag position to determine array order
```bash
# Flag naming: --<category>-<type>-<property>
API: { "rules": [{"type": "retry", ...}, {"type": "filter", ...}] }
CLI: --rule-retry-strategy exponential --rule-filter-body '{...}'

# Order determined by first flag of each type
--rule-filter-body '{...}' \      # Filter is first (index 0)
  --rule-transform-name "tx1" \   # Transform is second (index 1)
  --rule-filter-headers '{...}'   # Modifies first filter rule
```

**Implementation Guidelines:**
- First occurrence of `--<category>-<type>-*` flag establishes that item's position
- Subsequent flags for same type modify the existing item (don't create new one)
- Only one item of each type allowed (per API constraints)
- Provide JSON fallback for complex scenarios: `--<category>` or `--<category>-file`

**Example: Connection Rules (5 rule types)**
```bash
# Retry → Filter → Transform execution order
hookdeck connection create \
  --rule-retry-strategy exponential --rule-retry-count 3 \
  --rule-filter-body '{"event_type":"payment"}' \
  --rule-transform-name "my-transform"

# JSON fallback for complex configurations
hookdeck connection create --rules-file rules.json
```

**Validation:**
- If any `--rule-*` flag is used, corresponding rule object is constructed
- Type-specific required fields validated (e.g., `--rule-retry-strategy` required if any `--rule-retry-*` flag present)
- JSON fallback takes precedence and ignores all individual flags

### Command Structure Standards
```bash
# Standard CRUD pattern
hookdeck <resource> <action> [resource-id] [flags]

# Examples

# Individual resource creation (clear context)
hookdeck source create --type STRIPE --webhook-secret abc123
hookdeck destination create --type HTTP --url https://api.example.com

# Connection creation with inline resources (requires prefixed flags)
hookdeck connection create \
  --source-type STRIPE --source-name "stripe-prod" \
  --source-webhook-secret "whsec_abc123" \
  --destination-type HTTP --destination-name "my-api" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hookdeck/hookdeck-cli](https://github.com/hookdeck/hookdeck-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
