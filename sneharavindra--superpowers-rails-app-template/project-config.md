---
trigger: always_on
description: > `CLAUDE.md` is a symlink to this file — edit here, not there.
---

# This Rails App

> `CLAUDE.md` is a symlink to this file — edit here, not there.

This Rails app is a Rails 8.1 web application built on the Hotwire stack (Turbo + Stimulus) with Tailwind CSS and DaisyUI. It currently provides authentication, session management, and multi-tenant account foundations.

These instructions are defaults with reasons, not law — when the code in front of you disagrees, take the better path and flag the conflict. Invariants (data loss, security, CI gates) are non-negotiable.

## The Half-Window Rule

**Core business logic must fit within half the context window of the model maintaining this codebase.**

The first 50% of context is for comprehension; the second 50% is for reasoning and generation. When code consumes the whole window, AI agents pattern-match instead of reason — subtle regressions multiply and fixes add code instead of refactoring.

The default failure mode is overproduction: unrestricted code generation creating complexity without customer value. Before adding a feature, ask whether it earns its tokens. Delete speculative abstractions. Refactor rather than append.

## Deploy

Default branch: `main`

Kamal deploys to Docker containers — see `.kamal/` for config. SQLite is the database in all environments; the `storage/` directory is a persistent Docker volume. Do not assume Postgres or MySQL conventions.

Production runs three separate SQLite databases: main (`production.sqlite3`), cache (`cache.sqlite3`), and queue (`queue.sqlite3`).

## Authentication

Custom Rails 8 authentication — no Devise, no Warden. The `Authentication` concern in `app/controllers/concerns/authentication.rb` is included in `ApplicationController` and enforces `require_authentication` globally. Controllers opt out with `allow_unauthenticated_access`.

Sessions are stored in signed cookies (`cookies.signed[:session_id]`). `Current.session` and `Current.user` (via `ActiveSupport::CurrentAttributes`) carry request-scoped identity. Rate limiting protects sign-in and password reset (10 requests per 3 minutes).

Do not introduce session or auth logic outside this concern.

## Multi-Tenancy

Users belong to Accounts. Accounts have a `slug` (unique, URL-friendly). The tenant routing shape is not yet implemented — the Account model is a foundation only. When building tenant-scoped features, scope all queries through the current account and avoid cross-tenant data access.

## Background Jobs

Solid Queue runs inside Puma (via `SOLID_QUEUE_IN_PUMA`). Jobs go in `app/jobs/`. The only scheduled job clears finished queue records hourly. Mailers use `deliver_later`.

## Frontend

Hotwire stack: Turbo Drive for navigation, Turbo Frames/Streams for partial updates, Stimulus for JavaScript behavior. No build step — importmap pins dependencies. Tailwind CSS with DaisyUI for styling.

Add Stimulus controllers in `app/javascript/controllers/`. Keep them thin; push logic to the server via Turbo.

## Icons

[Heroicons](https://heroicons.com/) via the `heroicon` gem is the standard icon library — DaisyUI does not ship icons. Default variant is `:outline` (set in `config/initializers/heroicon.rb`). Use the helper directly in views:

```erb
<%= heroicon "check", options: { class: "size-5" } %>
```

Override per-icon with `variant: :solid` or `variant: :mini` when needed.

## Testing

Minitest with fixtures and parallel execution. Tests live in `test/` mirroring `app/`. Use `SessionTestHelper` for authenticated controller tests. Run the full suite with `bin/rails test`.

## Coding Style

RuboCop with `rubocop-rails-omakase` enforces style. Run `bundle exec rubocop` before committing. Brakeman (`bundle exec brakeman`) and `bundle exec bundler-audit` are available for security checks.

## Development Practices

### Task Tracking

All tasks are managed in Basecamp. **Before writing any code, state which Basecamp task you are working on.**

Use the Basecamp CLI to read and update tasks during your session. If the CLI is not installed:

```
curl -fsSL https://basecamp.com/install-cli | bash
```

Typical workflow:
1. Pull the task to understand acceptance criteria before touching code.
2. Note the task ID in your first message when starting work.
3. Update the task status when work is complete or blocked.

### Skills

Install the `superpowers-rails` plugin if not already present:

```
/plugin install superpowers-rails@sneharavindra
```

Plugin source: https://github.com/sneharavindra/superpowers-rails

**Mandatory skills — invoke before the described action, not after:**

| Situation | Skill |
|-----------|-------|
| Starting any new feature or design decision | `/superpowers-rails:brainstorming` |
| Bug, test failure, or unexpected behavior | `/superpowers-rails:systematic-debugging` |
| Task complete or before merging to `main` | `/superpowers-rails:requesting-code-review` |

Never rationalize skipping a skill because the task feels small. If there is a 1% chance it applies, invoke it.

`daisyui` (`.claude/skills/daisyui/SKILL.md`, mirrored from https://daisyui.com/SKILL.md): DaisyUI 5 component/class reference — check it before writing any Tailwind/DaisyUI markup.

## Business Process Management with FOSM


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sneharavindra/superpowers-rails-app-template](https://github.com/sneharavindra/superpowers-rails-app-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
