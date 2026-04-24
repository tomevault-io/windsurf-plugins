---
trigger: always_on
description: Rust drop-in replacement for `svelte-check` (**Svelte 5+ only**).
---

# svelte-check-rs

Rust drop-in replacement for `svelte-check` (**Svelte 5+ only**).

**Rust**: Edition 2021, MSRV 1.75

## Project Structure

- `crates/source-map`: Source position tracking and mapping.
- `crates/svelte-parser`: Svelte 5 syntax parser (lexer, AST, parser).
- `crates/svelte-transformer`: Svelte-to-TypeScript transformation for type-checking.
- `crates/svelte-diagnostics`: Svelte-specific diagnostics (a11y, component).
- `crates/tsgo-runner`: Bridge to `tsgo` for TypeScript type-checking.
- `crates/bun-runner`: Bridge to bun-managed Svelte compiler diagnostics.
- `crates/svelte-check-rs`: Main CLI and orchestration logic.
- `test-fixtures/`: Svelte component fixtures for testing.
- `node_modules/.cache/svelte-check-rs/`: Per-project cache (transformed files, tsgo incremental build info, bun runner script).

## Architecture

**Pipeline** (orchestrator.rs): File discovery → Parse → Svelte diagnostics → Svelte compiler diagnostics (bun) → Transform to TS → tsgo type-check

1. **Discovery**: Walk workspace, filter by `.svelte`/`.svelte.ts`/`.svelte.js`, respect `tsconfig.json` excludes
2. **Parse**: `svelte_parser::parse()` → AST + parse errors (parallel via `rayon`)
3. **Diagnostics**: A11y + component checks on AST
4. **Compiler**: Run Svelte compiler diagnostics via bun on original sources
5. **Transform**: `svelte_transformer::transform()` → TypeScript with source maps
6. **Type-check**: Send all transformed files to `tsgo` subprocess, map errors back via source maps

**tsgo Integration** (tsgo-runner crate):
- External TypeScript type-checker (Go-based, faster than tsc)
- Resolved from workspace `node_modules/.bin` (walks up from `--workspace`)
- Requires `@typescript/native-preview` to be installed in the workspace (peer dependency range: `>=7.0.0-dev.0`)
- Communication: JSON over stdin/stdout
- Incremental builds via `node_modules/.cache/svelte-check-rs/tsgo.tsbuildinfo`

**Svelte Compiler Integration** (bun-runner crate):
- bun-managed persistent workers that call `svelte/compiler`
- Auto-installed to cache dir on first run if not found
- Diagnostics reported against original `.svelte` sources (no extra source maps)

**SvelteKit Support**:
- Detects route files (`+page.svelte`, `+layout.svelte`, etc.) for proper prop types
- Runs `svelte-kit sync` before type-checking to generate `.svelte-kit/` types
- Resolves `$lib` and other aliases from `svelte.config.js`

## Commands

```bash
cargo build                    # Build all crates
cargo test                     # Run all tests
cargo clippy --all-targets -- -D warnings  # Lint (warnings as errors)
cargo fmt                      # Format (always run before committing)
cargo run -p svelte-check-rs -- --workspace ./path/to/project [--emit-ts]
```

**Useful CLI flags for development**:
- `--emit-ts`: Print transformed TypeScript (debugging transforms)
- `--emit-ast`: Print parsed AST for each file (debugging parser)
- `--emit-source-map`: Print source map mappings (debugging position mapping)
- `--output json`: Machine-readable diagnostics with exact locations
- `--timings`: Show parse/transform/check timing breakdown
- `--cache-stats`: Show cache statistics (files written/skipped)
- `--watch`: File watcher mode (uses `notify` crate, 1s polling)
- `--debug-paths`: Show resolved tsgo, bun, package manager, svelte-kit paths
- `--show-config`: Print resolved configuration (tsconfig, svelte.config.js, excludes)
- `--list-files`: List files that would be checked, then exit
- `--single-file <path>`: Process only a single file (isolate issues)
- `--skip-tsgo`: Skip TypeScript type-checking, only run Svelte diagnostics (faster iteration)
- `--bun-version`: Show installed bun version/path
- `--bun-update[=<ver>]`: Update bun to latest or specific version

## Testing

**Fixtures** (`test-fixtures/`):

- `valid/` - Components that must parse without errors (includes `a11y/`, `parser/` subdirs)
- `invalid/` - Components with semantic issues (a11y, type errors) - must not panic
- `projects/` - Full SvelteKit projects for integration tests (`sveltekit-bundler/`, etc.)

**Test Types**:

| Test | Command | Purpose |
|------|---------|---------|
| Snapshots | `cargo test --test snapshots` | Parser/transformer output verification |
| Corpus | `cargo test --test corpus_test` | All fixtures parse without panics |
| Integration | `cargo test --test integration_*` | Full CLI workflow with project fixtures |
| Unit | `cargo test -p <crate>` | In-source `mod tests` blocks |

**Snapshots** (uses `insta`): Located in `crates/*/tests/snapshots/`.

```rust
// Parser snapshot - outputs source, errors, AST
parse_snapshot("my_feature", "<div>{value}</div>");

// Transformer snapshot - outputs source, TSX, source map count
transform_snapshot("my_rune", r#"<script>let x = $state(0);</script>"#);
```

```bash
# Accept new snapshots
find crates -name "*.snap.new" -exec sh -c 'mv "$1" "${1%.new}"' _ {} \;
# Or interactively
cargo insta review
```

**Adding Tests**:

- **New fixture**: Add `.svelte` file to `test-fixtures/valid/` or `invalid/` - automatically picked up by corpus tests
- **New snapshot**: Add test fn in `crates/*/tests/snapshots.rs` using `parse_snapshot()` or `transform_snapshot()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pheuter/svelte-check-rs](https://github.com/pheuter/svelte-check-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
