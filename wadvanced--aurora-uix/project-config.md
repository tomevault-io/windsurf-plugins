---
trigger: always_on
description: Canonical guidance for any AI coding agent working in this repository (Claude Code, GitHub Copilot, opencode, qwen-code, Cursor, Aider, etc.). `CLAUDE.md` and `.github/copilot-instructions.md` are symlinks to this file.
---

# AGENTS.md

Canonical guidance for any AI coding agent working in this repository (Claude Code, GitHub Copilot, opencode, qwen-code, Cursor, Aider, etc.). `CLAUDE.md` and `.github/copilot-instructions.md` are symlinks to this file.

## Overview

LogaMoney is a Phoenix/Elixir financial management application for loan management, targeting the Dominican Republic market. It handles loan lifecycle, payment processing, employer payroll deduction integration, user KYC, and multi-tenant role-based access (borrower, investor, employer).

## Commands

```bash
# Development
mix setup                    # Install deps, create and migrate DB (first time)
iex -S mix phx.server        # Start dev server with interactive shell (port 4000)

# Testing
mix test                     # Run all tests
mix test path/to/test.exs    # Run a specific test file
mix test --failed            # Re-run previously failed tests
mix test --cover             # Run with coverage report

# Code quality (run before committing)
mix precommit                # Full CI compliance check: format, compile, docs, credo, doctor, dialyzer
mix format                   # Auto-format code
mix credo --strict           # Lint
mix dialyzer                 # Static analysis (first run ~10 minutes)

# Database
mix ecto.reset               # Drop, recreate, and migrate DB
mix ecto.gen.migration NAME  # Create a new migration
mix ecto.psql                # Open PostgreSQL console

# Localization
mix localise                 # Extract and merge gettext translations (en, es)

# Assets
mix assets.build             # Build Tailwind + esbuild
```
## Naming Conventions section near the top of CLAUDE.md

When referencing UI components in documentation or code, use simple names (e.g., 'Phone', 'Email') not prefixed/namespaced names (e.g., 'EmbeddedPhone', 'EmbeddedEmail') unless explicitly asked.

## Domain Glossary

Two terms are frequently confused in this codebase. Use them precisely:

| Term | Definition | Example correct usage |
|---|---|---|
| **Customer / User** | Any authenticated person on the platform. Every account holder is a customer the moment they sign up, regardless of which account types they hold. | "Contact Customer Support", "Unlock your potential as a Loga Money customer", KYC ("Know Your Customer") |
| **Borrower** | The `:borrower` `account_type` value on an `Account`. One of three account types (`:borrower`, `:investor`, `:employer`). A single user may hold all three simultaneously. | `account_type: :borrower`, `BorrowerDetails`, `borrower_account_id`, `ValidateBorrowerRole` |

**Key rule:** A user is a customer the moment they sign up. They become a borrower only when they hold an account with `account_type: :borrower`.

**When to write "customer":** Generic product/marketing context where you mean any platform user, not a specific account type. Example: learn pages, support copy, onboarding headline text.

**When to write "borrower":** Anywhere you are referring specifically to the borrower account type: Ash resource names, field names, route segments under the borrower flow, UI labels on the borrower account card.

## Architecture

### Core Structure

```
lib/
  loga_money/           # Business logic (Ash resources)
    models/             # Domain models: accounts, core, settings
    mail/               # Email system (Swoosh)
    oban_jobs/          # Background jobs (Oban)
    application.ex      # OTP supervision tree
  loga_money_web/       # Web interface (Phoenix)
    live/               # LiveView modules
    controllers/        # HTTP controllers
    components/         # Reusable UI components (core_components.ex)
priv/
  repo/migrations/      # Ecto migrations
  gettext/              # Translation files (es_DO default, en supported)
docs/                   # Comprehensive architecture documentation
```

### Key Patterns

**Ash Framework**: Business logic is defined as declarative Ash resources (not plain Ecto schemas) — never raw Ecto queries. Ash automatically generates GraphQL from resources.

**Domain code interfaces (STRICT)**: All data access goes through the **domain**. Outside a domain module, **never** call the `Ash.*` data API directly — this applies to LiveViews, controllers, components, Oban jobs, channels, plugs, and tests. Every read, write, and relationship load must be exposed either as a domain *code interface* (`define :name, action: :action` inside the domain's `resources` block) or as a plain public function on the domain module. The raw `Ash.*` API is used **only inside** domain modules and resource definitions.

Forbidden outside domain modules:
- Actions: `Ash.create/2`, `Ash.update/2`, `Ash.destroy/2`, `Ash.run_action/2`
- Reads: `Ash.read/2`, `Ash.read_one/2`, `Ash.get/3`, `Ash.load/3`
- Builders: `Ash.Changeset.for_*`, `Ash.Query.for_*` / `filter` / `sort` / `load`

```elixir
# ❌ Bad — direct Ash API in a LiveView
Notification
|> Ash.Query.for_read(:read_by_id, %{id: id})
|> Ash.read_one!()

# ✅ Good — define the interface once in the domain `resources` block
resource Core.Notification do

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wadvanced/aurora_uix](https://github.com/wadvanced/aurora_uix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
