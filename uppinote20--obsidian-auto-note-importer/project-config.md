---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
npm run build    # TypeScript check + production build
npm run dev      # Development mode with watch
npm run lint     # ESLint for src/*.ts files
npm test         # Vitest unit tests (watch)
npm run test:run # Vitest unit tests (one-shot)

# E2E (require Obsidian running with --remote-debugging-port=9222)
npm run test:e2e            # Airtable sync e2e
npm run test:e2e:settings   # Airtable settings UI e2e
npm run test:e2e:full       # Build + deploy + run Airtable sync e2e
npm run test:e2e:seatable          # SeaTable sync e2e (.env required)
npm run test:e2e:seatable:settings # SeaTable settings UI e2e
npm run test:e2e:seatable:full     # Build + deploy + run SeaTable e2e
```

## Architecture Overview

This is an Obsidian plugin that syncs notes bidirectionally between **remote databases (Airtable, SeaTable; Supabase / Notion / Custom API tracked in epic #11)** and your Obsidian vault. Higher layers operate on the provider-agnostic `DatabaseProvider` interface — adding a new provider is documented in handbook §4.4.

### Module Structure (`src/`)

```
src/
├── main.ts                          # Plugin entry point, service orchestration
├── types/                           # Type definitions
│   ├── settings.types.ts                # AutoNoteImporterSettings, LegacySettings, DEFAULT_SETTINGS
│   ├── config.types.ts                  # ConfigEntry (per-config sync settings), SharedServices
│   ├── credential.types.ts              # CredentialType union + AirtableCredential / SeaTableCredential / etc.
│   ├── database.types.ts                # DatabaseProvider interface, RemoteNote, SyncResult, ProviderCapabilities
│   ├── field-types.types.ts             # StandardFieldType + FieldTypeMapper
│   ├── provider-settings.types.ts       # CredentialFormRenderer (settings-tab plugin point)
│   ├── airtable.types.ts                # Airtable-specific (AirtableField, AirtableBase, AirtableTable, AirtableView)
│   └── sync.types.ts                    # SyncMode, SyncScope, SyncRequest
├── constants/                       # Constants
│   ├── api.ts                           # AIRTABLE_* / SEATABLE_* / RATE_LIMIT_INTERVAL_MS / retry knobs
│   └── system-fields.ts                 # SYSTEM_FIELDS, isSystemField (frontmatter-reserved)
├── services/                        # External service integration
│   ├── airtable-client.ts               # Airtable DatabaseProvider impl (REST API)
│   ├── airtable-field-mapper.ts         # Airtable type → StandardFieldType
│   ├── airtable-credential-form.ts      # Airtable settings form + connection test
│   ├── seatable-client.ts               # SeaTable DatabaseProvider impl (API Gateway v2 + Base-Token caching)
│   ├── seatable-field-mapper.ts         # SeaTable type → StandardFieldType
│   ├── seatable-credential-form.ts      # SeaTable settings form + connection test
│   ├── provider-registry.ts             # Factory + mapper + form-renderer registry by CredentialType
│   ├── rate-limiter.ts                  # Per-credential request throttling + 429 retry + transient retry
│   └── field-cache.ts                   # Airtable field metadata cache (Meta API)
├── core/                            # Business logic
│   ├── sync-orchestrator.ts             # processSyncRequest (pull / push / bidirectional)
│   ├── conflict-resolver.ts             # detectConflicts + resolve by mode
│   ├── sync-queue.ts                    # Queue-based sync (dedup + merge)
│   ├── config-instance.ts               # Per-config service stack
│   └── config-manager.ts                # ConfigInstance lifecycle
├── builders/                        # Content generation
│   ├── note-builder.ts                  # Template parsing, markdown generation
│   └── bases-file-generator.ts          # Obsidian Bases (.base) generator
├── file-operations/                 # File system operations
│   ├── file-watcher.ts                  # Change detection (debounced)
│   └── frontmatter-parser.ts            # YAML frontmatter (read/inject + read-only filter)
├── ui/                              # UI components
│   ├── settings-tab.ts                  # Settings panel (multi-config, provider-aware cards)
│   └── suggest/                         # Folder/file autocomplete
└── utils/                           # Utilities
    ├── sanitizers.ts                    # File/folder name sanitization
    ├── yaml-formatter.ts                # YAML value formatting (Bases-aware)
    ├── object-utils.ts                  # Deep equality, generateId
    ├── settings-bridge.ts               # ConfigEntry + Credential → LegacySettings
    ├── migration.ts                     # Settings v1 → v3 migration
    ├── validation.ts                    # Folder overlap validation
    └── api-errors.ts                    # Cross-provider API error extraction + URL normalization
```

### Key Classes

- **`DatabaseProvider`** — Provider-agnostic interface for remote databases (handbook §4.4)
- **`AirtableClient` / `SeaTableClient`** — Concrete `DatabaseProvider` impls

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uppinote20/obsidian-auto-note-importer](https://github.com/uppinote20/obsidian-auto-note-importer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
