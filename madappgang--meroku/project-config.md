---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## !! THIS REPO IS PUBLIC - NEVER COMMIT SECRETS !!

**ABSOLUTE RULE: No real infrastructure data, investigation files, or debug output may EVER be committed to this repository.**

This repo is **public on GitHub**. Any committed secret is immediately exposed to the world.

**NEVER commit:**
- Real Terraform plan/state JSON files (they contain embedded secrets: SSH keys, API keys, passwords, account IDs)
- AWS credentials, SSH keys, API tokens, or any secret material
- Debug/investigation output from real infrastructure
- Real YAML configs with account IDs or credentials
- Any file from a real project's `.terraform/` directory

**For test data:** Always use **synthetic/fake data** with dummy values. Never copy-paste from real infrastructure output.

**If you need test fixtures:** Generate them manually with placeholder values like `REDACTED`, `000000000000`, `fake-key-do-not-use`.

## Project Overview

This is a comprehensive Terraform Infrastructure as Code (IaC) repository that provides a reusable, modular AWS infrastructure setup. It includes:

- Terraform modules for AWS services (20+ modules including ECS, RDS, ALB, Cognito, Lambda, etc.)
- Go CLI application ("meroku") for interactive infrastructure management using Bubble Tea TUI framework
- React+TypeScript web frontend for visual infrastructure management
- Handlebars-based templating system (using Raymond Go package) for environment configuration
- GitHub Actions CI/CD workflows with OIDC authentication

## Template and Configuration Management Strategy

### Dual Approach: Migrations + Default Helper

We use **both** migrations and the `default` Handlebars helper for managing configuration values:

#### When to Use Migrations

**Use migrations for core infrastructure fields that should always be explicit in YAML:**

```go
// migrations.go - Example v12
booleanDefaults := map[string]interface{}{
    "multi_az":                             false,
    "storage_encrypted":                    true,
    "iam_database_authentication_enabled":  false,
}
```

**Template:**
```handlebars
multi_az = {{postgres.multi_az}}
storage_encrypted = {{postgres.storage_encrypted}}
```

**Benefits:**
- YAML is self-documenting (all values visible)
- Single source of truth for defaults (migration code)
- Type safety at load time
- Fields always exist when template renders

**Use for:**
- Core infrastructure booleans (security, HA, encryption)
- Required fields that should never be missing
- Fields where explicit values improve clarity

#### When to Use Default Helper

**Use `default` helper for optional fields with sensible fallbacks:**

```handlebars
instance_class = "{{default postgres.instance_class "db.t4g.micro"}}"
allocated_storage = {{default postgres.allocated_storage 20}}
backend_cpu = "{{default workload.backend_cpu "256"}}"
```

**Benefits:**
- Defaults visible in template (documentation)
- Less YAML clutter for optional fields
- Flexibility to override when needed
- Works correctly with `false`, `0`, `0.0` (fixed in our implementation)

**Use for:**
- Optional configuration with sensible defaults
- Fields that vary by use case
- Non-critical settings

#### Default Helper Behavior

The `default` helper in `app/raymond.go` has been fixed to handle all types correctly:

- `nil` (field missing from YAML) → returns default
- `false` → returns `false` (valid value)
- `0` → returns `0` (valid value)
- `0.0` → returns `0.0` (valid value)
- `""` (empty string) → returns default
- `[]` (empty array) → returns default

**Example:**
```yaml
# YAML
postgres:
  min_capacity: 0  # Explicitly set to 0 (valid for Aurora pausing)

# Template
min_capacity = {{default postgres.min_capacity 0.5}}

# Result: 0 (uses explicit value, not default)
```

#### When to Use Exists Helper

**Use `exists` helper when you need to distinguish "value is 0/false" from "value is missing":**

```handlebars
{{!-- Aurora capacity: 0 is valid (pause when idle), distinguish from missing --}}
{{#if (exists postgres.min_capacity)}}
  min_capacity = {{postgres.min_capacity}}
{{/if}}
```

**Why `exists` vs `{{#if}}`:**
- `{{#if value}}` checks **truthiness** → 0, false, "", [] are all falsy
- `{{#if (exists value)}}` checks **presence** → only nil is false

**Use cases:**
- Aurora Serverless v2 capacity where 0 means "pause when idle"
- Numeric fields where 0 is a valid configuration
- Boolean fields where false needs different handling than missing

**Pattern comparison:**

```handlebars
{{!-- OLD PATTERN (complex, hard to read) --}}
{{#if (or min_capacity (eq min_capacity 0))}}
  min_capacity = {{min_capacity}}
{{/if}}

{{!-- NEW PATTERN (simple, clear intent) --}}
{{#if (exists min_capacity)}}
  min_capacity = {{min_capacity}}
{{/if}}
```

**Other available helpers:**
- `or` - Logical OR: `{{#if (or a b)}}` (checks truthiness)
- `eq` - Equality: `{{#if (eq value 0)}}` (compares values)
- `default` - Fallback values: `{{default value 10}}`

**Tests:** All helpers have comprehensive test coverage in `app/raymond_test.go`

## Important Architecture Decisions

### VPC Configuration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MadAppGang) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
