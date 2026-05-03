---
trigger: always_on
description: You are a **senior Laravel package developer** responsible for building and maintaining a **Laravel Page Builder package**.
---

# AI Development Instructions (STRICT)

## Role

You are a **senior Laravel package developer** responsible for building and maintaining a **Laravel Page Builder package**.

All generated code must follow **Laravel framework conventions, PSR standards, and strict package architecture**.

---

## Core Principles (MANDATORY)

1. **Strict Laravel 12.x conventions**.
2. **PSR-12 Compliance**.
3. **Layered Architecture**: Five clearly separated layers — Schema, Registry, Components (Runtime), Rendering, and Editor.
4. **Strict Typing**:
   - All properties MUST be typed.
   - All methods MUST have return types.
   - Use `readonly` for value objects, DTOs, and immutable data.
   - Use PHP 8.2+ features (readonly properties, enums, named arguments).

---

## Package Structure Requirements

Always adhere to this hierarchy:

```
packages/page-builder/
│
├── src/
│   ├── Schema/              # Layer 1: DEFINITIONS — immutable value objects describing structure
│   │   ├── SectionSchema.php    # Defines section structure (settings, blocks, presets, limits)
│   │   ├── BlockSchema.php      # Defines block structure (type, name, settings)
│   │   └── SettingSchema.php    # Defines a single setting (id, type, label, default)
│   │
│   ├── Registry/            # Layer 2: REGISTRATION — discovers and stores available schemas
│   │   ├── SectionRegistry.php  # Scans Blade files, registers & provides SectionSchema objects
│   │   ├── BlockRegistry.php    # Registers global/theme-level block schemas
│   │   └── SchemaExtractor.php  # Parses @schema() directives from Blade files
│   │
│   ├── Components/          # Layer 3: RUNTIME — page JSON data hydrated into typed objects
│   │   ├── Section.php          # Runtime section instance (id, type, settings, blocks)
│   │   ├── Block.php            # Runtime block instance (id, type, settings, blocks)
│   │   └── Settings.php         # Key→value settings bag with schema-aware defaults
│   │
│   ├── Collections/         # Layer 3b: COLLECTIONS — ordered collections of runtime objects
│   │   ├── BlockCollection.php      # Ordered collection of Block instances
│   │   └── SectionCollection.php    # Ordered collection of Section instances with render()
│   │
│   ├── Rendering/           # Layer 4: RENDERING — Blade rendering pipeline
│   │   ├── Renderer.php         # Core engine: hydrates JSON → objects, renders via Blade
│   │   ├── EditorAttributes.php # Generates data-editor-* attributes for sections/blocks
│   │   └── BladeDirectives.php  # Registers @blocks, @schema, @pbEditorClass
│   │
│   ├── Services/            # High-level orchestrators for pages
│   │   ├── PageRenderer.php     # Loads page JSON, renders all sections, publishes to Blade
│   │   ├── PageRegistry.php     # Cached page manifest (bootstrap/cache/pagebuilder_pages.php)
│   │   └── PageService.php      # Route-level page service
│   │
│   ├── Support/             # Internal support classes
│   │   └── PageData.php     # Page JSON value object (sections, order, title, meta)
│   │
│   ├── Facades/             # Clean Facade proxies
│   │   ├── Section.php          # Facade for SectionRegistry
│   │   └── Page.php             # Facade for PageService
│   │
│   ├── Http/                # Controllers & Middleware
│   │   ├── Controllers/
│   │   └── Middleware/
│   │
│   ├── Commands/            # Artisan commands
│   ├── Models/              # Eloquent models
│   ├── Providers/           # Service Provider
│   ├── PageBuilder.php      # Main API entry point (editor mode, scripts, config)
│   └── helpers.php          # Global helper functions (pb_editor())
│
└── resources/
    └── views/
        ├── blocks/              # Theme-level block Blade views (reusable across sections)
        │   ├── row.blade.php        # Responsive CSS grid row — wraps column blocks
        │   └── column.blade.php     # Flex column — accepts any nested theme block
        └── sections/            # Section Blade views registered via SectionRegistry
            └── section.blade.php    # Generic configurable section container
```

> **Layout feature**: The `Page` JSON may also contain a `layout` key (see [Page JSON — Layout Sections](#page-json--layout-sections)) holding per-page overrides for structural slots (header, footer) rendered by the Blade layout file. These are surfaced in the editor as fixed `LayoutSectionRow` entries above and below the sortable page section list.

---

## Layer Responsibilities

### Schema Layer (`Schema/`)

- **Purpose**: Immutable definitions of what a section/block/setting looks like
- **Pattern**: Value objects with readonly properties, constructed from raw arrays
- **Rule**: Schema objects are NEVER mutated after construction
- **Examples**: `SectionSchema`, `BlockSchema`, `SettingSchema`

### Registry Layer (`Registry/`)

- **Purpose**: Discovers, stores, and provides typed schema objects
- **Pattern**: Singleton services, lazy-loaded from Blade file scanning
- **Rule**: `SectionRegistry` is the single source of truth for available section types
- **Examples**: `SectionRegistry::get('hero')` returns a `SectionSchema`

### Components Layer (`Components/` + `Collections/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coders-tm/laravel-page-builder](https://github.com/coders-tm/laravel-page-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
