---
trigger: always_on
description: **Gem Name**: `ros-apartment`
---

# CLAUDE.md - Apartment

**Gem Name**: `ros-apartment`
**Maintained by**: CampusESP
**Active work**: v4 rewrite (phased, PR-per-sub-phase off `main`)

## Design & Plan Documents

Planning artifacts live in `docs/` with no date prefixes (git handles temporal tracking):

- `docs/designs/<feature>.md` — Design specs (what and why). Living docs, one per feature, updated in place.
- `docs/plans/<feature>/` — Implementation plans (how and in what order). Can have multiple files for phased plans.

Do NOT use `docs/superpowers/specs/` or `docs/superpowers/plans/` — those are plugin defaults that we override with the paths above.

**Key documents:**
- `docs/designs/apartment-v4.md` — v4 design spec
- `docs/designs/v4-railtie-test-infra.md` — Railtie + test infrastructure design
- `docs/designs/elevator-tenant-validation.md` — Elevator tenant validation + missing-tenant fail-safe (shipped; see Key Patterns)
- `docs/plans/apartment-v4/phase-2-adapters.md` — Current phase plan (includes deferred review items)

## Where to Start

1. **README.md** - Installation, basic usage, configuration options
2. **docs/architecture.md** - Core design decisions and WHY they were made (v3)
3. **docs/designs/apartment-v4.md** - v4 architecture and motivation
4. **lib/apartment/CLAUDE.md** - Implementation file guide
5. **spec/CLAUDE.md** - Test organization and patterns

## Commands

```bash
# Unit tests (no database required)
bundle exec rspec spec/unit/

# Unit tests across Rails versions
bundle exec appraisal install                              # first time only
bundle exec appraisal rails-8.1-sqlite3 rspec spec/unit/   # single version
bundle exec appraisal rspec spec/unit/                     # all versions

# v4 integration tests (requires real databases)
bundle exec appraisal rails-8.1-sqlite3 rspec spec/integration/v4/                        # SQLite
DATABASE_ENGINE=postgresql bundle exec appraisal rails-8.1-postgresql rspec spec/integration/v4/  # PostgreSQL
DATABASE_ENGINE=mysql bundle exec appraisal rails-8.1-mysql2 rspec spec/integration/v4/          # MySQL

# Lint. Update the plugins first, or a green local run tells you nothing about CI:
# no Gemfile.lock is committed, so CI resolves rubocop's plugins fresh while a local
# checkout keeps whatever it last resolved. AllCops.NewCops is `enable`, so a cop
# added in a plugin minor is enforced the moment it ships — CI fails on a commit that
# linted clean locally. Deliberate: 247 cops are pending across the loaded plugins and
# the alternative drops all of them.
bundle update rubocop rubocop-rspec rubocop-rails rubocop-performance rubocop-rake rubocop-thread_safety
bundle exec rubocop

# Build gem
gem build ros-apartment.gemspec

# Coverage report (opt-in)
COVERAGE=1 bundle exec rspec spec/unit/

# Test profiling
FPROF=1 bundle exec appraisal rails-8.1-sqlite3 rspec spec/integration/v4/
EVENT_PROF=sql.active_record bundle exec appraisal rails-8.1-sqlite3 rspec spec/integration/v4/

# Request lifecycle tests (requires PostgreSQL)
DATABASE_ENGINE=postgresql bundle exec appraisal rails-8.1-postgresql rspec spec/integration/v4/request_lifecycle_spec.rb

# RBAC integration tests (requires provisioned PG/MySQL roles; see docs/designs/v4-phase5.2-rbac-integration-tests.md)
DATABASE_ENGINE=postgresql bundle exec appraisal rails-8.1-postgresql rspec spec/integration/v4/ --tag rbac
DATABASE_ENGINE=mysql bundle exec appraisal rails-8.1-mysql2 rspec spec/integration/v4/ --tag rbac
```

**CI matrix**: Ruby 3.3/3.4/4.0 × Rails 7.2/8.0/8.1/main × PG 16+18, MySQL 8.4, SQLite3. Rails main is a canary (`continue-on-error`). See `.github/workflows/ci.yml`.

## Core Concepts

**Multi-tenancy via database isolation**: One app, many customers, data fully separated.
- **PostgreSQL (schemas)**: Namespaces in single DB. Fast (<1ms switch), scales to 100+ tenants.
- **MySQL (databases)**: Separate DB per tenant. Complete isolation, slower switching.
- **Elevators**: Rack middleware extracts tenant from request. Auto-inserted after `ActionDispatch::Callbacks` (before sessions/auth).
- **Pinned models**: Global tables declared with `Apartment::Model` + `pin_tenant`. Bypasses tenant routing. Use `has_many :through`, not HABTM. Replaces `excluded_models` (deprecated in v4).

See `docs/architecture.md` for v3 design decisions, `docs/adapters.md` for strategy trade-offs, `docs/elevators.md` for middleware rationale.

## Key Patterns

- **Block-based switching**: Always prefer `switch(tenant) { ... }` over `switch!`. Ensure block guarantees cleanup on exceptions.
- **Adapter pattern**: Abstract base class with database-specific subclasses. Unified API hides DB differences.
- **Callbacks**: `ActiveSupport::Callbacks` on `:create` and `:switch` for logging/notification hooks.
- **Dynamic tenant discovery**: `tenants_provider` is a callable (proc/lambda) that queries the database at runtime.
- **Tenant name validation**: `TenantNameValidator` does pure in-memory format checks (no DB queries). Enforced in `AbstractAdapter#create` and `ConnectionHandling#connection_pool`. Engine-specific rules for PG identifiers, MySQL names, SQLite paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rails-on-services/apartment](https://github.com/rails-on-services/apartment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
