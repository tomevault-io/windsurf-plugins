---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# Context for coding agents

This file provides guidance to coding agents when working with code in this repository.

## About AlchemyCMS

AlchemyCMS is an open source Rails CMS engine with a flexible, YAML-driven content architecture. It uses a three-tier content model: Pages → Elements → Ingredients, with multi-language support, versioning, and a modern admin interface built with Rails + Web Components.

## Development Commands

### Initial Setup

```bash
bin/setup
```

This installs dependencies (Ruby gems + pnpm packages) and sets up the dummy app in `spec/dummy/`.

### Running Tests

```bash
# Run all tests (prepares database + runs RSpec suite)
bundle exec rake

# Run RSpec only
bin/rspec

# Run specific test file
bin/rspec spec/models/alchemy/page_spec.rb

# Prepare test database (required before first test run)
bundle exec rake alchemy:spec:prepare

# Run JavaScript tests
pnpm run test
```

### Building Frontend Assets

```bash
# Build all assets (JavaScript, CSS, Handlebars templates, icons)
pnpm run build

# Build individual components
pnpm run build:admin      # Bundle admin JavaScript (app/javascript/alchemy_admin/**)
pnpm run build:js         # Bundle vendored dependencies (sortablejs, shoelace, tinymce, etc.)
pnpm run build:css        # Sass compilation
pnpm run handlebars:compile  # Compile Handlebars templates
pnpm run build:icons      # Generate icon sprite
```

### Running the Dummy App

```bash
# Start the full stack in Docker: builds images and runs the Rails server,
# Sass watcher, and JS bundle watcher (docker compose up --build)
bin/start
```

This starts a Rails server and pnpm watch task in the dummy application for running a full dev environment.

After changing dependencies (`Gemfile` or `package.json`), rebuild the image and
refresh the baked `node_modules` in one step:

```bash
bin/start --refresh
```

### Starting the Rails console

```bash
cd spec/dummy; bin/rails console; cd -
```

This starts the Rails console for running code in the context of the dummy application.

### Running a one off script

```bash
cd spec/dummy; bin/rails runner 'puts Some.ruby.code'; cd -
```

This starts the Rails console for running code in the context of the dummy application.

### Linting

```bash
# Ruby linting (uses Standard/RuboCop)
bundle exec standardrb

# JavaScript linting
pnpm run eslint

# Prettier linting
pnpm run lint
```

### Code Formatting

```bash
# Ruby linting (uses Standard/RuboCop)
bundle exec standardrb --fix

# Prettier formatting
pnpm exec prettier --write app/javascript/**/*.js
```

## Architecture Overview

### Content Model: Three-Tier Hierarchy

Alchemy organizes content in a three-tier hierarchy:

**Pages** (`Alchemy::Page`)
- Top-level content containers organized in a tree structure (using `awesome_nested_set`)
- Each page has a `page_layout` (defined in `config/alchemy/page_layouts.yml`)
- Multi-language support: each page belongs to one `Alchemy::Language`
- **Dual-version system**:
  - `draft_version` - Working copy (no `public_on` date)
  - `public_version` - Published content (has `public_on` date)
- Page locking prevents concurrent editing (`locked_by`, `locked_at`)

**Elements** (`Alchemy::Element`)
- Content blocks that live on specific `PageVersion` records
- Defined in `config/alchemy/elements.yml`
- Can be **fixed** (permanent) or **mutable** (user-removable)
- Support nesting via `parent_element_id`
- Positioned using `acts_as_list` within scope

**Ingredients** (`Alchemy::Ingredient`)
- The actual content values (text, images, links, etc.)
- Uses **Single Table Inheritance (STI)** with 15+ types:
  - Value types: `Text`, `Headline`, `Html`, `Richtext`, `Number`, `Boolean`
  - Media types: `Picture`, `File`, `Audio`, `Video`
  - Reference types: `Page`, `Node`, `Link`
  - Special types: `Select`, `Datetime`
- Each has a `role` (semantic identifier within element) and optional `related_object` (polymorphic)
- Flexible metadata stored in JSON `data` column

### YAML-Driven Configuration

Alchemy uses declarative YAML definitions that are **separate from database records**:

- `config/alchemy/page_layouts.yml` - Page layout definitions
- `config/alchemy/elements.yml` - Element definitions with ingredient specs
- Definition classes (`PageDefinition`, `ElementDefinition`, `IngredientDefinition`) are read-only configuration objects

This separation allows runtime configuration changes without database migrations.

### Key Model Relationships

```
Site
└── languages (Alchemy::Language)
    └── pages (Alchemy::Page - nested set tree)
        └── versions (Alchemy::PageVersion)
            └── elements (Alchemy::Element - can be nested)
                └── ingredients (Alchemy::Ingredient - STI)
                    └── related_object (polymorphic: Picture, Page, etc.)
```

### Navigation System

**Nodes** (`Alchemy::Node`) represent menu/navigation structure separate from page hierarchy:
- Also uses nested set pattern per language
- Can attach to pages or link to external URLs
- Referenced by elements via `Ingredients::Node`

### Service Objects

Complex business logic is extracted into service classes in `app/services/alchemy/`:

- `CopyPage` - Deep copies pages with all elements and ingredients

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlchemyCMS/alchemy_cms](https://github.com/AlchemyCMS/alchemy_cms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
