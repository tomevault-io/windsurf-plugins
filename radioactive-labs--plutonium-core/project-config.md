---
trigger: always_on
description: This guide helps AI assistants contribute to the Plutonium framework itself.
---

# Plutonium Core Development Guide

This guide helps AI assistants contribute to the Plutonium framework itself.

## Project Overview

Plutonium is a Rails RAD framework distributed as a Ruby gem. It provides resource-oriented architecture with automatic CRUD, policies, definitions, and multi-tenancy.

## Repository Structure

```
plutonium-core/
├── lib/plutonium/              # Core framework code
│   ├── resource/               # Resource system (controllers, policies, definitions)
│   ├── portal/                 # Portal engines and multi-tenancy
│   ├── interaction/            # Business logic encapsulation
│   ├── ui/                     # Phlex view components
│   ├── definition/             # Definition DSL and field types
│   ├── query/                  # Search, filters, scopes
│   ├── auth/                   # Authentication (Rodauth integration)
│   ├── routing/                # Route helpers and extensions
│   └── generators/             # Rails generators (pu:res:scaffold, etc.)
├── app/                        # Rails app components
│   ├── assets/                 # JavaScript, CSS
│   ├── controllers/            # Base controllers
│   └── views/                  # Phlex components
├── test/                       # Test suite
│   ├── plutonium/              # Unit tests mirroring lib/ structure
│   ├── dummy/                  # Test Rails app
│   └── system/                 # System/integration tests
├── docs/                       # VitePress documentation site
│   ├── getting-started/
│   ├── guides/
│   └── reference/
└── .claude/skills/             # AI assistant skills (documentation for Claude)
```

## Key Abstractions

### Resource System
- `Plutonium::Resource::Record` - Model mixin with `associated_with` scopes
- `Plutonium::Resource::Policy` - ActionPolicy-based authorization
- `Plutonium::Resource::Definition` - Declarative UI configuration
- `Plutonium::Resource::Controller` - CRUD controller mixin
- `Plutonium::Resource::Interaction` - Business logic encapsulation

### Portal System
- `Plutonium::Portal::Engine` - Rails engine mixin for web interfaces
- `Plutonium::Package::Engine` - Rails engine mixin for feature packages
- Entity scoping for multi-tenancy (path or custom strategy)

### UI Components
Located in `lib/plutonium/ui/` - Phlex-based components:
- `page/` - Page layouts (index, show, new, edit)
- `form/` - Form builders and field components
- `table/` - Data tables with sorting, pagination
- `display/` - Field display components

## Development Workflow

### Environment Setup

When working on Plutonium itself, set:

```bash
export PLUTONIUM_DEV=1
```

This enables development mode which:
- Uses local assets instead of packaged ones
- Enables hot reloading of components
- Shows more detailed error messages

### Building Assets

When working on JavaScript or CSS in `src/`:

```bash
# Watch mode (rebuilds on changes to src/build/)
yarn dev

# Production build (to app/assets/)
yarn build
```

- `yarn dev` - watches and rebuilds to `src/build/` for development
- `yarn build` - compiles to `app/assets/` for release

**Always run `yarn dev`** in a terminal when working on frontend code.

### Running Tests

Tests use [Appraisal](https://github.com/thoughtbot/appraisal) for multiple Rails versions:

```bash
# Full test suite (all Rails versions)
bundle exec appraisal rake test

# Specific Rails version
bundle exec appraisal rails-8.1 rake test

# Specific test file
bundle exec appraisal rails-8.1 ruby -Itest test/plutonium/resource/policy_test.rb
```

Available: `rails-7`, `rails-8.0`, `rails-8.1`

### Testing Generators

Generators are in `lib/generators/pu/`. Test by:
1. Running in the dummy app: `cd test/dummy && rails g pu:res:scaffold ...`
2. Generator tests in `test/generators/`

### Documentation

```bash
yarn docs:dev      # Local preview at localhost:5173
yarn docs:build    # Build for production
```

## Code Conventions

### Ruby Style
- Follow Rails conventions
- Use `with_connection` for database operations (fiber-safe)
- Avoid excessive defensive programming (no `respond_to?` checks)
- Use `Rails.logger.warn { "message" }` block syntax in production code

### Generators
- Always provide `--dest=` flag support to avoid interactive prompts
- Use `--dest=main_app` for main application, `--dest=package_name` for packages
- Quote shell arguments with special characters: `'field:type?'`, `'field:decimal{10,2}'`

### Migrations
- Inline indexes and constraints in `create_table` blocks
- Use `foreign_key: {on_delete: :cascade}` where appropriate

### Frontend
- Stimulus controllers for interactivity (always register controllers)
- TailwindCSS 4 for styling
- Phlex for view components

## Adding New Features

### New Field Type
1. Add renderer in `lib/plutonium/ui/display/components/`
2. Add input in `lib/plutonium/ui/form/components/`
3. Register in field type mappings
4. Add tests and documentation

### New Generator
1. Create in `lib/generators/pu/category/name_generator.rb`
2. Add templates in `lib/generators/pu/category/name/templates/`
3. Add tests in `test/generators/`
4. Document in `docs/reference/generators/`

### New Interaction Response
1. Add response class in `lib/plutonium/interaction/response/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radioactive-labs/plutonium-core](https://github.com/radioactive-labs/plutonium-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
