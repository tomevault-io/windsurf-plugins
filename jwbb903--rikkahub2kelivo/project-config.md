---
trigger: always_on
description: A Go CLI tool and web server for converting backup files between two AI chat applications: [Kelivo](https://github.com/nicepkg/kelivo) (Flutter) and [RikkaHub](https://github.com/rikka/rikkahub) (Android/Kotlin). Converts providers, assistants, MCP servers, world books/lorebooks, quick messages, instruction injections, and full chat history with branching/versions.
---

# AGENTS.md

## Project Overview

A Go CLI tool and web server for converting backup files between two AI chat applications: [Kelivo](https://github.com/nicepkg/kelivo) (Flutter) and [RikkaHub](https://github.com/rikka/rikkahub) (Android/Kotlin). Converts providers, assistants, MCP servers, world books/lorebooks, quick messages, instruction injections, and full chat history with branching/versions.

The UI and error messages are in Chinese (中文). Keep them in Chinese when modifying.

## Build & Run

```bash
# Build (requires Go 1.21+, CGO needed for SQLite)
go build -o backup-converter .

# CLI usage
./backup-converter k2r -i kelivo_backup.zip -o output.zip      # Kelivo → RikkaHub
./backup-converter r2k -i rikkahub_backup.zip -o output.zip    # RikkaHub → Kelivo
./backup-converter info -i backup.zip                           # Inspect backup
./backup-converter serve -p 8080                                # Web server mode

# Interactive shell script (builds if needed)
./run.sh
```

The CI workflow (`.github/workflows/build.yml`) cross-compiles with `CGO_ENABLED=0` and `go build -ldflags="-s -w"` for 6 OS/arch combos. It triggers on `v*` tags and auto-creates a GitHub Release with binaries.

## Architecture

```
main.go          → CLI entry, arg parsing, command dispatch
server.go        → HTTP server (embeds web/index.html via go:embed)
models/          → Data structures for both formats (no logic)
  kelivo.go      → Kelivo types (settings, providers, assistants, messages...)
  rikkahub.go    → RikkaHub types (settings, providers, assistants, message nodes...)
parser/          → Deserialize backup zips into models
  kelivo_parser.go   → Reads settings.json + chats.json from zip
  rikkahub_parser.go → Reads settings.json + SQLite DB from zip (with WAL support)
converter/       → Bidirectional conversion logic (pure data transformation, no I/O)
  kelivo_to_rikkahub.go  → K→R conversion
  rikkahub_to_kelivo.go  → R→K conversion
  builtins.go            → Built-in provider ID mappings between the two apps
writer/          → Serialize models back to backup zips
  kelivo_writer.go   → Writes settings.json + chats.json to zip
  rikkahub_writer.go → Creates fresh SQLite DB + writes to zip
```

### Data Flow

**CLI path:** `parser.Parse*Backup(zip) → converter.Convert*(backup) → writer.Write*Backup(result, zip)`

**Web server path:** Upload → parse to `models.*Backup` → store in `Server` struct (mutex-protected) → API endpoints read/transform → convert + marshal on demand → download as zip.

### Key Architectural Differences Between Formats

- **Kelivo** stores chats as flat JSON: `chats.json` with `conversations[]` and `messages[]`, using `groupId` + `version` fields for branching.
- **RikkaHub** stores chats in SQLite (`rikka_hub.db`): `ConversationEntity` and `message_node` tables. Each node contains an array of messages (versions). Messages within nodes have a `select_index` for the active version.
- **Kelivo settings** uses double-serialization: complex fields (providers, assistants, MCP servers, etc.) are JSON strings stored as top-level string fields (e.g., `provider_configs_v1`). The parser deserializes these into the `json:"-"` tagged struct fields. The writer re-serializes them back.
- **RikkaHub settings** uses normal nested JSON structures.

## Critical Gotchas

### Built-in Provider Mapping (`converter/builtins.go`)

Both apps ship with preset providers (OpenAI, DeepSeek, Gemini, etc.) that share API endpoints but have different internal IDs. During conversion:

- **K→R direction**: Matched by `(baseUrl, providerType)`. Only apiKey and user-added models are transferred; the built-in provider's RikkaHub UUID is used.
- **R→K direction**: Matched by RikkaHub provider UUID. Maps to the corresponding Kelivo built-in provider key.

When adding new built-in providers, add entries to the `builtinProviderMappings` slice in `builtins.go`.

### Model ID Mapping

Models are identified differently in each app:
- **Kelivo**: `"ProviderID::modelId"` format (e.g., `"OpenAI::gpt-4o"`)
- **RikkaHub**: Each model gets a UUID. A `modelIDMap` is built during conversion to translate between formats.

The mapping resolution is fuzzy — falls back to matching by model ID suffix if the full key isn't found.

### Kelivo's Double-Serialized Settings

`models/kelivo.go` has paired fields: a `string` "raw" field (e.g., `ProviderConfigsRaw`) that holds JSON text deserialized from `settings.json`, and a `json:"-"` parsed field (e.g., `ProviderConfigs`) populated by the parser. The writer reverses this. When constructing `KelivoSettings` in converter code, you must populate **both** the raw and parsed fields (or the writer will handle the re-serialization if only parsed fields are set).

### SQLite for RikkaHub


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jwbb903/Rikkahub2Kelivo](https://github.com/jwbb903/Rikkahub2Kelivo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
