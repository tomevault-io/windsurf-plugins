---
trigger: always_on
description: A Rails app for engineering design doc review, purpose-built for AI-assisted planning. Plans get better through collaboration — domain experts leave inline feedback, AI agents respond to that feedback and apply edits automatically, and every change is versioned with full provenance. Humans comment, AI agents edit. Local agents interact via the REST API using skills (see `coplan` skill) and future CLIs.
---

# CoPlan — Agent Guidelines

## What This Is

A Rails app for engineering design doc review, purpose-built for AI-assisted planning. Plans get better through collaboration — domain experts leave inline feedback, AI agents respond to that feedback and apply edits automatically, and every change is versioned with full provenance. Humans comment, AI agents edit. Local agents interact via the REST API using skills (see `coplan` skill) and future CLIs.

## Architecture: Engine vs Host App

Most of the application logic lives in the **CoPlan Rails engine** (`engine/`), packaged as the `coplan` gem (path-based, in `Gemfile`). The top-level Rails app is a **thin host** that provides deployment configuration, ActiveAdmin, and app-specific glue.

### Engine (`engine/`) — where the code lives
- **Models** — all domain models live in `engine/app/models/coplan/` (Plan, PlanVersion, User, Comment, CommentThread, EditLease, EditSession, ApiToken, PlanCollaborator)
- **Controllers** — web UI and API controllers in `engine/app/controllers/coplan/`, including `api/v1/` for the REST API
- **Services** — all service objects in `engine/app/services/coplan/` (Plans::Create, Plans::ApplyOperations, AI providers, etc.)
- **Policies** — authorization policies in `engine/app/policies/coplan/`
- **Jobs** — background jobs in `engine/app/jobs/coplan/`
- **Views, helpers, assets, JS** — all Hotwire views and Stimulus controllers
- **Migrations** — engine-owned tables go in `engine/db/migrate/`. Data/backfill migrations supporting engine behavior also belong here (the gem only packages `engine/`, so host-only migrations don't reach gem consumers); copy them into the host with `bin/rails co_plan:install:migrations`.
- **Routes** — engine routes in `engine/config/routes.rb`, mounted by the host

### Host app (top-level) — thin deployment shell
- **ActiveAdmin** — admin registrations in `app/admin/`
- **Auth** — `SessionsController`, `User` model (legacy, being migrated to `CoPlan::User`)
- **App-specific integrations** — `SlackClient`, `SlackNotificationJob`
- **Migrations** — this app is just the example/demo host, so `db/migrate/` mostly holds the `*.co_plan.rb` copies of engine migrations (the engine owns schema for `coplan_*` tables)
- **Config** — database, deployment, environment, seeds

**When adding new features, put them in the engine** unless they are deployment- or host-specific (admin UI, external integrations, auth).

## Tech Stack & Philosophy

- **Rails** with importmaps — no Node, no bundler, no Webpack, no esbuild
- **Hotwire** — Turbo Drive, Turbo Frames, Turbo Streams, Stimulus
- **Plain CSS** — no Tailwind, no preprocessors
- **Plain JavaScript** — via importmaps and Stimulus controllers only
- **MySQL 8** — but schema must stay portable: hosts may run PostgreSQL, so no adapter-specific column options or SQL outside an adapter check (search is the worked example — see `Plan.adapter_search` and the AddSearchToCoplanPlans migration); **no `default:` on JSON columns** (use `after_initialize` in the model instead)
- **SolidQueue** for background jobs, **SolidCable** for ActionCable
- **ActiveAdmin 4 beta** + `activeadmin_assets` for admin UI — no node/tailwind needed
- **No Devise, no OmniAuth** — auth is hand-rolled (stub OIDC in dev, real OIDC later)

## Database Conventions

- All tables use **UUID primary keys** (`id: :string, limit: 36`), assigned in `ApplicationRecord#assign_uuid`
- All multi-tenant tables include `organization_id` (FK, not null)
- Enums are stored as **strings** (not integers) — validated with constants and `inclusion:` validators
- JSON columns for arrays/hashes (`tags`, `metadata`, `trigger_statuses`, `allowed_email_domains`)
- No PG-only types (`citext`, `text[]`, `ON CONFLICT ... WHERE`) — use Rails validations and `json` columns
- `Current.organization` and `Current.user` are set per-request for scoping

## Model Conventions

- Define valid values as **frozen constants** on the model (e.g., `Plan::STATUSES`, `Comment::AUTHOR_TYPES`)
- Use `inclusion:` validations against those constants
- Use `after_initialize` for defaults on JSON array columns (e.g., `self.tags ||= []`)
- Service objects live in `app/services/` namespaced by model (e.g., `Plans::Create`)
- Service objects use the `self.call` + `new(...).call` pattern
- Authorization uses plain policy objects in `app/policies/` — not Pundit

## Frontend Conventions

- JavaScript goes in `app/javascript/controllers/` as Stimulus controllers
- No npm packages — everything through importmaps or inline
- Views use Turbo Frames for partial page updates, Turbo Streams for realtime broadcasts
- Keep it simple: no React, no Vue, no component libraries

### HARD REQUIREMENT: native feel, secretly HTML

The entire goal of this stack is an app that is plain HTML over the wire but
**feels indistinguishable from a native app**. Server-rendered HTML is the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/coplan](https://github.com/block/coplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
