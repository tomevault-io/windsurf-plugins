---
trigger: always_on
description: enableSorting: true,
---

# AGENTS.md - AI Assistant Context

This is a WordPress plugin ClawPress. When working with this codebase, follow these patterns and conventions.

## Project Overview

The plugin includes:
- React-based admin UI using DataViews
- REST API with namespaced endpoints
- Floating wp-admin panel UI (`src/panel`)
- wp-scripts build system

## Architecture Patterns

### PHP Structure

- **Main file** (`clawpress.php`): Only defines constants and requires modules
- **Feature modules** (`includes/class-*.php`): Each feature isolated in its own namespaced class file
- **Strict types**: All PHP files use `declare( strict_types=1 )`
- **Namespace convention**: `ClawPress\FeatureName`
- **Coding standards**: WordPress Coding Standards (WPCS) - use tabs, spaces inside parentheses
- **Linting config**: `phpcs.xml.dist` - run `npm run lint:php` to check

### JavaScript Structure

- **Entry point**: `src/js/admin/index.js` - mounts React app on plugin admin page
- **Panel entry point**: `src/panel/index.jsx` - mounts floating admin panel
- **Components**: `src/js/admin/components/` - React components
- **Hooks**: `src/js/admin/hooks/` - Custom hooks (data fetching, state)
- **Config**: `src/js/admin/config/` - Field definitions, actions, settings

### JavaScript Formatting Rules

Apply these rules consistently to avoid large lint/fix churn:

- Use tabs for indentation in JS/JSX files.
- Use single quotes for JS strings/imports (unless escaping/template use makes that inappropriate).
- Let Prettier collapse JSX/expressions to single-line when short; do not force manual multi-line wrapping.
- Do not manually align tokens/spaces for visual columns; keep standard Prettier spacing.
- Keep object/array/function formatting Prettier-first; avoid hand-formatted stylistic layouts.
- Avoid nested ternaries; expand to clear `if`/`else` when logic branches.
- Keep `ideas/` out of JS lint scope (ignored via `.eslintignore` and changed-file lint script).
- Before committing JS changes, run `npm run lint:js -- --fix` (or scoped equivalent) and then `npm run lint:js`.

### REST API Pattern

Located in `includes/class-rest-api.php`:
- Namespace: `clawpress/v1`
- Always include `permission_callback`
- Use `sanitize_callback` and `validate_callback` for args
- Return `WP_REST_Response` with appropriate status codes

**Note**: The DataViews demo uses `@wordpress/core-data` which handles REST calls for post types internally. Custom endpoints are only needed for operations not covered by WordPress core (custom business logic, aggregations, etc.).

### DataViews Pattern

The admin UI uses `@wordpress/dataviews` with the WordPress Data Layer:
- **fields**: Define columns in `config/itemConfig.js`
- **actions**: Define row actions (view, edit, trash) in same file
- **useItems hook**: Uses `useEntityRecords` from `@wordpress/core-data`

To switch post types, change `'page'` to your CPT slug in `useItems.js`.

## Key Files to Modify

When adding features:

| Task | Files to Edit |
|------|---------------|
| Add custom post type | `includes/class-post-types.php` |
| Add REST endpoint | `includes/class-rest-api.php` |
| Add admin component | `src/js/admin/components/`, import in `App.js` |
| Update floating panel UI | `src/panel/` and `includes/class-panel.php` |
| Add PHP heartbeat/scheduler hook | `includes/class-heartbeat.php` |

## Helper Class Reference

Use helpers in `includes/helpers/` as the primary integration surface for shared logic.

| Helper | File | Responsibility | Usage Rule |
|------|------|------|------|
| `Action_Log_Helper` | `includes/helpers/class-action-log-helper.php` | Action/event log persistence and retrieval in `clawpress_action_logs` | Route action/event log writes and reads through this helper. |
| `Agent_File_Helper` | `includes/helpers/class-agent-file-helper.php` | Bootstrap and resolve agent files from `clawpress_agent_file` | Resolve logical files through this helper; do not bypass with ad-hoc lookups. |
| `Chat_Helper` | `includes/helpers/class-chat-helper.php` | Online/offline reply orchestration and AI-call integration | Route model replies through this helper from chat flows. |
| `Chat_History_Helper` | `includes/helpers/class-chat-history-helper.php` | Per-user chat transcript persistence | Use for chat history reads/writes and clears. |
| `Context_Helper` | `includes/helpers/class-context-helper.php` | Builds system prompt + message arrays from bootstrap/memory/skills/history | Use this as the central context builder for LLM calls. |
| `Memory_Helper` | `includes/helpers/class-memory-helper.php` | Memory persistence/retrieval in `clawpress_agent_mem` (`memory.md` + `memory-ddmmyyyy.md`) | All memory save/read/clear operations must use this helper only. |
| `Model_Helper` | `includes/helpers/class-model-helper.php` | Provider-specific model option lists and defaults | Use for model option resolution in UI/commands. |
| `Panel_Helper` | `includes/helpers/class-panel-helper.php` | Floating panel state normalization and defaults | Use for panel state storage and hydration. |
| `Provider_Helper` | `includes/helpers/class-provider-helper.php` | Provider availability/configuration/model resolution | Use for provider detection and configured-provider fallback logic. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradvin/clawpress](https://github.com/bradvin/clawpress) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
