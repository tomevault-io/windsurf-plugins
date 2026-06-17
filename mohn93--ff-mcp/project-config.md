---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

An MCP (Model Context Protocol) server that wraps the FlutterFlow Project API, enabling AI-assisted FlutterFlow development. Published to npm as `community-ff-mcp`. It exposes tools, resources, and prompts over stdio transport.

## Build & Run

```bash
npm run build          # TypeScript compile + chmod executable
npm run dev            # Watch mode (tsc --watch)
npm start              # Run the server (node build/index.js)
```

Tests use **vitest**. Run `npm test` to execute all tests. No linter is configured. Verify changes by running `npm run build` and `npm test`.

## Environment

Requires `FLUTTERFLOW_API_TOKEN` env var (Bearer token from FlutterFlow > Account Settings > API Token).

## Architecture

**Entry point:** `src/index.ts` — creates one `FlutterFlowClient` instance, registers all tools/resources/prompts on an MCP `StdioServerTransport`.

**API client:** `src/api/flutterflow.ts` — thin wrapper around `https://api.flutterflow.io/v2/`. All endpoints return `Promise<unknown>` — the tools handle response parsing.

**Critical encoding detail:** The `projectYamls` endpoint returns YAML as **base64-encoded ZIP**, not plain text. `src/utils/decode-yaml.ts` handles decoding. The `listPartitionedFileNames` endpoint returns file keys under `value.file_names` (snake_case, not camelCase).

### Registration Pattern

Every tool/resource/prompt follows the same pattern: a `register*` function that takes `(server, client?)` and calls `server.tool()`, `server.resource()`, or `server.prompt()`. New tools go in `src/tools/`, new prompts in `src/prompts/`, new resources in `src/resources/`. Register them in `src/index.ts`.

### Tools (25)

| Tool | File | Purpose |
|------|------|---------|
| `list_projects` | `tools/list-projects.ts` | List all FF projects |
| `list_project_files` | `tools/list-files.ts` | List YAML file keys (supports prefix filter) |
| `get_project_yaml` | `tools/get-yaml.ts` | Fetch + decode YAML by file key |
| `validate_yaml` | `tools/validate-yaml.ts` | Validate before pushing |
| `update_project_yaml` | `tools/update-yaml.ts` | Push YAML changes |
| `list_pages` | `tools/list-pages.ts` | Page index with names/folders (batched, 5 at a time) |
| `get_page_by_name` | `tools/get-page-by-name.ts` | Fetch page by human-readable name |
| `sync_project` | `tools/sync-project.ts` | Bulk download all YAML to local cache |
| `get_page_summary` | `tools/get-page-summary.ts` | Cache-based page summary (widget tree, actions, params) |
| `get_component_summary` | `tools/get-component-summary.ts` | Cache-based component summary |
| `find_component_usages` | `tools/find-component-usages.ts` | Find all pages/components using a component |
| `find_page_navigations` | `tools/find-page-navigations.ts` | Find all navigation actions targeting a page |
| `get_yaml_docs` | `tools/get-yaml-docs.ts` | Search/retrieve FF YAML reference docs by topic or file |
| `get_app_state` | `tools/get-app-state.ts` | Cache-based app state, constants, and environment settings |
| `get_api_endpoints` | `tools/get-api-endpoints.ts` | Cache-based API endpoint definitions (method, URL, variables, headers, response) |
| `get_data_models` | `tools/get-data-models.ts` | Cache-based data structs, enums, Firestore collections, Supabase tables |
| `get_custom_code` | `tools/get-custom-code.ts` | Custom actions, functions, widgets, agents, app-actions, custom-files |
| `get_general_settings` | `tools/get-general-settings.ts` | General: App Details, App Assets, Nav Bar & App Bar |
| `get_project_setup` | `tools/get-project-setup.ts` | Project Setup: Firebase, Languages, Platforms, Permissions, Dependencies, Dev Environments |
| `get_app_settings` | `tools/get-app-settings.ts` | App Settings: Authentication, Push Notifications, Mobile/Web Deployment |
| `get_in_app_purchases` | `tools/get-in-app-purchases.ts` | In-App Purchases: Stripe, Braintree, RevenueCat, Razorpay |
| `get_integrations` | `tools/get-integrations.ts` | Integrations: Supabase, SQLite, GitHub, Algolia, Google Maps, AdMob, etc. |
| `get_theme` | `tools/get-theme.ts` | Cache-based theme colors, typography, breakpoints, widget defaults |
| `get_editing_guide` | `tools/get-editing-guide.ts` | Workflow + doc guide for editing tasks (call before modifying YAML) |
| `search_project_files` | `tools/search-project-files.ts` | Search file keys by keyword/prefix/regex |

### Utilities

- `utils/decode-yaml.ts` — `decodeProjectYamlResponse()`: base64 → adm-zip → `Record<string, string>`
- `utils/parse-folders.ts` — `parseFolderMapping()`: regex-based extraction of scaffold→folder mapping from the `folders` YAML file
- `utils/cache.ts` — Local cache functions: `cacheRead`, `cacheWrite`, `cacheWriteBulk`, `cacheMeta`, `listCachedKeys`
- `utils/resolve-data-type.ts` — `resolveDataType()`: resolves FF data type objects to readable strings (e.g. `List<String>`, `DataStruct:MyStruct`)
- `utils/batch-process.ts` — `batchProcess()`: process items in parallel batches to avoid overwhelming the file system

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohn93/ff-mcp](https://github.com/mohn93/ff-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
