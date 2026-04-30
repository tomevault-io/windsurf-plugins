---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MCP Tools for Elementor Plugin — a WordPress plugin that extends the official WordPress MCP Adapter to expose Elementor data, widgets, structures, and methods as MCP (Model Context Protocol) tools. This enables AI tools (Claude, Cursor, etc.) to create and manipulate Elementor page designs programmatically via 97 MCP tools.

**Current status: All phases implemented (P0/P1/P2).** Foundation layer, 7 read-only query tools, page CRUD, layout, widget, template, global, composite tools, stock images, SVG icons, custom code tools, and full widget coverage are all complete (97 MCP tools total). See `PLAN.md` for the full architectural specification.

## Dependencies & Requirements

- WordPress >= 6.8
- Elementor >= 3.20 (container support required)
- WordPress Abilities API (bundled in WP 6.9+, or via composer)
- WordPress MCP Adapter (`wordpress/mcp-adapter` via composer)
- PHP 7.4+

## Build & Development Commands

No external dependencies. The plugin uses WordPress core, Elementor, MCP Adapter, and the Abilities API (all loaded as separate plugins or WP core).

For plugin review tooling, the `.claude/skills/wp-plugin-review/scripts/setup_tools.sh` script installs PHPCS, WPCS, PHPStan, and PHPUnit.

## Architecture

### MCP Server Registration

The plugin registers a dedicated MCP server `elementor-mcp-server` at `/wp-json/mcp/elementor-mcp-server`. All abilities use the `elementor-mcp/` namespace.

### Directory Structure

```
elementor-mcp/
├── elementor-mcp.php                          # Bootstrap: plugin header, constants, dependency checks, require_once, singleton init
├── includes/
│   ├── class-plugin.php                       # Singleton orchestrator — hooks into wp_abilities_api_categories_init, wp_abilities_api_init, mcp_adapter_init
│   ├── class-elementor-data.php               # Data access layer wrapping Elementor documents, widgets, element tree
│   ├── class-element-factory.php              # Builds valid Elementor JSON element structures (container, widget, section, column)
│   ├── class-id-generator.php                 # 7-char hex unique IDs via random_bytes()
│   ├── class-openverse-client.php             # HTTP client for Openverse image search API
│   ├── abilities/
│   │   ├── class-ability-registrar.php        # Coordinates registration of all ability groups across all phases
│   │   ├── class-query-abilities.php          # P0: 7 read-only tools (list-widgets, get-widget-schema, get-page-structure, etc.)
│   │   ├── class-page-abilities.php           # P1: 5 page CRUD tools (create-page, update-page-settings, delete-page-content, import-template, export-page)
│   │   ├── class-layout-abilities.php         # P1: 4 layout tools (add-container, move-element, remove-element, duplicate-element)
│   │   ├── class-widget-abilities.php         # P1/P2: 2 universal + 9 core + 6 Pro convenience widget tools
│   │   ├── class-template-abilities.php       # P2: 2 template tools (save-as-template, apply-template)
│   │   ├── class-global-abilities.php         # P2: 2 global tools (update-global-colors, update-global-typography)
│   │   ├── class-composite-abilities.php      # P2: 1 composite tool (build-page)
│   │   ├── class-stock-image-abilities.php    # 3 stock image tools (search-images, sideload-image, add-stock-image)
│   │   └── class-custom-code-abilities.php   # 4 custom code tools (add-custom-css, add-custom-js, add-code-snippet, list-code-snippets)
│   ├── admin/
│   │   ├── class-admin.php                    # Admin settings page: 3 tabs (Tools, Connection, Prompts), stats bar, header
│   │   └── views/
│   │       ├── page-tools.php                 # Tools tab: category-grouped tool toggles with bulk actions
│   │       ├── page-connection.php            # Connection tab: status cards, credential form, MCP client configs
│   │       └── page-prompts.php               # Prompts tab: sample prompt cards with one-click copy, CTA banner
│   ├── schemas/
│   │   ├── class-schema-generator.php         # Generates JSON Schema from Elementor widget controls
│   │   └── class-control-mapper.php           # Maps individual Elementor control types → JSON Schema fragments
│   └── validators/
│       ├── class-element-validator.php         # Validates element structure (id, elType, widgetType)
│       └── class-settings-validator.php        # Validates widget settings against generated schema
├── prompts/                                    # Sample landing page prompt blueprints (Markdown)
│   ├── LOCAL_BUSINESS.md
│   ├── DENTAL_CLINIC.md
│   ├── WEB_DEVELOPER_PORTFOLIO.md
│   ├── HAIR_SALON.md
│   └── CAR_WASH.md
└── tests/                                      # PHPUnit tests (not yet created)
```

### Hook Registration Flow

The plugin integrates via three WordPress hooks (in execution order):
1. **`wp_abilities_api_categories_init`** → Registers the `elementor-mcp` ability category
2. **`wp_abilities_api_init`** → Registers all abilities via `wp_register_ability()` (ability names must match `[a-z0-9-]+/[a-z0-9-]+`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [msrbuilds/elementor-mcp](https://github.com/msrbuilds/elementor-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
