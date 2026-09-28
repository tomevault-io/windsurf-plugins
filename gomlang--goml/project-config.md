---
trigger: always_on
description: goml is a statically typed, garbage-collected language with Rust-like syntax that compiles to Go. Sources use `.gom`; there is no ownership system or lifetime syntax. The compiler monomorphizes generics and lambda-lifts GoML closures.
---

# Repository Guidelines

goml is a statically typed, garbage-collected language with Rust-like syntax that compiles to Go. Sources use `.gom`; there is no ownership system or lifetime syntax. The compiler monomorphizes generics and lambda-lifts GoML closures.

## Documentation and Source Map

Read the documentation relevant to the change:

- [Language guide](docs/goml.md): canonical syntax, semantics, packages, builtins, standard-library APIs, examples, and grammar.
- [Formatting](docs/formatting.md): formatter behavior and CLI.
- [Releases](docs/releasing.md): publishing, toolchain installation, and stage0 advancement.
- [Compile-time evaluation](docs/comptime.md), [Go FFI](docs/ffi/bind-go.md), and [FFI protocol](docs/ffi/protocol-v1.md): specialized workflows and the Go metadata boundary.
- [gomlgo](gomlgo/README.md): independent Go frontend, interpreter, compatibility target, and differential tests.
- [VS Code extension](editors/vscode/README.md): editor setup, behavior, and configuration.

| Path | Responsibility |
| --- | --- |
| `gomlc/` | Self-hosted compiler, query engine, LSP, resource loader, and compiler tests |
| `goml/` | Self-hosted project driver, registry client, dependency resolver, and CLI tests |
| `gomlgo/` | Independent Go frontend and interpreter implemented in GoML |
| `lib/builtin/` | Hidden compiler/runtime contract, runtime hooks, and language items |
| `lib/prelude/` | Independent package defining the automatically scoped public API |
| `lib/std/` | Standard-library packages loaded through explicit imports |
| `bootstrap/` | Bootstrap scripts and checksum-pinned released stage0 metadata |
| `tools/` | Build, library packaging, release, and Go metadata tooling |
| `editors/vscode/` | VS Code extension |
| `gomlc/testdata/` | Compiler regression fixtures and generated golden files |

The current bootstrap uses toolchain prefixes under `stage0`, `stage2`, and `stage3`; use `stage2/bin` for local development. Each loads resources from its executable-relative `lib/`, including its finalized compiler world under `lib/compiler/`. Build outputs belong under `_bootstrap/`, `_artifact/`, or module-configured target directories.

## Development Workflow

Requirements: Linux amd64, Go 1.26+, a C compiler for race-detector tests, Node 20+, npm, `just`, Bash, curl, tar, sha256sum, and jq. Generated Go code and Go FFI target Go 1.26. The gomlgo interpreter and differential tests require Go 1.26.x; see its README for details.

Run recipes from the repository root; [.justfile](.justfile) is the command reference.

| Command | Purpose |
| --- | --- |
| `just make` | Incrementally build stage2 from pinned stage0 |
| `just test` / `just all` | Build and run compiler, driver, and Go metadata tests |
| `just ci` | Full CI, including bootstrap fixed point and packaging |
| `just bootstrap` | Clean bootstrap and fixed-point verification |
| `just verify-golden` / `just update-golden` | Verify / regenerate snapshots through self-hosted tests |
| `just vscode-ext` | Build the LSP and compile the extension |
| `just gomlgo-test` | Run the independent gomlgo test suite |
| `just clean` | Remove root and compiler/driver build caches and generated development stages; retain stage0 |

- After editing `.gom` files, run `goml fmt` from every affected module before tests or commits. Modules include `gomlc/`, `goml/`, `gomlgo/`, and the separate library projects under `lib/`.
- Use the repository formatter, for example `cd gomlc && ../stage2/bin/goml fmt`; `fmt --check` verifies formatting.
- Run a focused fixture with `stage2/bin/gomlc run-single <file.gom>`. Add `--dump-ast`, `--dump-expanded-ast`, `--dump-hir`, `--dump-tast`, `--dump-ctir`, `--dump-core`, `--dump-mono`, `--dump-lift`, `--dump-anf`, or `--dump-go` to inspect lowering.
- `goml check`, `goml build`, and `goml test` discover the enclosing `goml.toml` and operate on the complete module, without package targets. `--dry-run` prints planned commands.
- The driver finds `gomlc` through `--compiler`, `GOMLC`, a sibling binary, `GOML_HOME/bin`, then `PATH`, and verifies the driver protocol.
- Run checks relevant to the change. Run `just ci` locally for changes affecting bootstrap compatibility, toolchain construction, or packaging, for release preparation, or when explicitly requested. Read-only reviews and documentation-only changes require only applicable checks.
- Changes to `gomlgo/` behavior also need its separate tests; consult its README for focused differential checks. After checks pass, rerun or broaden them only when new changes, failures, or unresolved concerns warrant it.

## Coding and Architecture Rules

- Do not add code comments; write clear, self-explanatory code.
- GoML uses four-space indentation, snake_case functions/packages, CamelCase types, and explicit top-level function signatures. Generics use square brackets; local closures have one concrete type and cannot declare generics or use let-generalization.
- TypeScript uses two-space indentation and PascalCase components; prefer named exports.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gomlang/goml](https://github.com/gomlang/goml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
