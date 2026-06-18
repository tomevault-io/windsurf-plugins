---
trigger: always_on
description: This file provides guidance to Claude Code and other AI agents working with this repository.
---

# Agent Instructions

This file provides guidance to Claude Code and other AI agents working with this repository.
See [API_DESIGN_GUIDELINES.md](API_DESIGN_GUIDELINES.md) for CLI command naming, flag conventions, and output format rules.

## Commands

```bash
# Build
cargo build           # debug
cargo build --release # production binary (~4MB)

# Run
cargo run -- <args>   # e.g. cargo run -- ls -o json
./target/release/lox <args>

# Check / Lint
cargo check
cargo clippy

# Test
cargo test
cargo test <test_name>  # run a single test
```

## Pre-push CI Checklist

**ALWAYS run these four checks before committing/pushing.** They mirror `.github/workflows/ci.yml` (ubuntu + macos):

```bash
cargo fmt --check            # formatting
cargo clippy -- -D warnings  # lints (warnings are errors)
cargo build --release        # release build
cargo test                   # all tests
```

Do not push until all four pass. Fix issues and re-run before committing.

## Architecture

Single Rust binary. CLI commands use reqwest blocking; token auth uses tokio + WebSocket for the RSA/AES key exchange.

### Source files

| File | Purpose |
|------|---------|
| `src/main.rs` | All CLI commands (60+), clap argument parsing, helper functions (RGB→HSV, weather/stats binary parsing) |
| `src/client.rs` | `LoxClient` (HTTP) — control resolution, structure cache, categories, global states, operating modes |
| `src/config.rs` | `Config` + `GlobalConfig` — loads/saves config (flat or multi-context), context resolution, project-local `.lox/` discovery |
| `src/config_edit.rs` | `ConfigEditor` — DOM-based XML editing engine: properties, attributes, room moves, wiring, element add/remove, BOM-aware write-back |
| `src/errors.rs` | Rich error types with Levenshtein fuzzy matching, "did you mean?" suggestions, doc links |
| `src/loxcc.rs` | LoxCC compress/decompress — LZ4-style algorithm with CRC32 checksums for Miniserver config archives |
| `src/loxone_xml.rs` | XML parsing: rooms, controls, users, devices, config summary, diff |
| `src/commands/config_cmd.rs` | Config editing commands: rooms, controls, MQTT, wiring, validate, push, add, user CRUD |
| `src/commands/ctx.rs` | `lox ctx` commands — add/use/list/remove/rename/init/migrate contexts |
| `src/gitops.rs` | Git-based config versioning — init, pull (FTP→LoxCC→diff→commit), log, restore workflows |
| `src/stream.rs` | WebSocket binary protocol: state events, weather, keepalive, AES-256-CBC encrypted commands |
| `src/scene.rs` | Scene loading/listing from `~/.lox/scenes/*.yaml` |
| `src/ws.rs` | `LoxWsClient` — async WebSocket connection used by token auth (RSA+AES key exchange handshake) |
| `src/token.rs` | Token auth flow: RSA key exchange, AES-encrypted credential exchange, token storage, HMAC token hashing |

### Key design points

**Control resolution** (`LoxClient::resolve_with_room`): Names are matched against the structure cache using fuzzy substring matching. Resolution order: alias → exact UUID → bracket room qualifier (`"Name [Room]"`) → `--room` flag → fuzzy substring. Ambiguous matches are an error.

**Multi-context config**: `~/.lox/config.yaml` supports both flat (single-Miniserver, backward compatible) and multi-context format. `Config::load()` resolution: `LOX_CONFIG` env → project-local `.lox/` (walk up from cwd) → global config → `--ctx` flag override. Each context gets isolated data under `~/.lox/contexts/<name>/`.

**Structure cache**: `LoxApp3.json` (~150KB) is cached per-context (e.g. `~/.lox/contexts/<name>/cache/structure.json`) with a 24h TTL. All commands that need control UUIDs load this cache first; `lox cache refresh` forces a re-fetch.

**Mixed sync/async**: The CLI commands use `reqwest::blocking`. `main.rs` uses `#[tokio::main]` because `lox token fetch` needs async (WebSocket for the key exchange). The blocking reqwest client spawns its own thread pool so both modes coexist.

**Token auth**: RSA public key fetched from Miniserver → encrypt AES session key → send encrypted credentials via WebSocket → receive token. Token stored per-context (e.g. `~/.lox/contexts/<name>/token.json`), valid ~20 days.

**Config editing** (`ConfigEditor`): DOM-based XML editing via `xmltree` crate. Element selector syntax: `"Title"` (fuzzy), `"Type:WeatherData"`, `"uuid:abc-123"`, `"gid:Mqtt"`. Write-back preserves UTF-8 BOM and line endings via post-processing. LoxCC compression uses literals-only LZ4 with CRC32 checksums (required for the Miniserver to trust password fields).

**LoxCC format**: Binary config archive with 16-byte header (magic `0xAABBCCEE`, compressed size, uncompressed size, CRC32). The CRC32 at offset 12 is critical — zero CRC causes the Miniserver to ignore encrypted `t="15"` password fields. Plaintext `t="11"` passwords are accepted by the Miniserver directly.

### Pipeline scripts

| Script | Purpose | When to run |
|--------|---------|-------------|
| `scripts/scrape-docs.py` | Fetch loxone.com/enen/kb/ articles → markdown in `docs/kb/` | New Loxone Config version |
| `scripts/extract-strings.py` | Parse `LoxoneConfigres_*.dll` → `i18n/<locale>.json` | New Loxone Config version |
| `scripts/build-templates.py` | Generate type schemas from `.Loxone` XML → `docs/templates.json` | After config changes |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eisber/lox-cli](https://github.com/eisber/lox-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
