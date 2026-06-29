---
trigger: always_on
description: Guidelines for AI agents working on this project. `CLAUDE.md` is a symlink to this file.
---

# AGENTS.md

Guidelines for AI agents working on this project. `CLAUDE.md` is a symlink to this file.

## Project Goals

This project aims to create a complete port of the official Svelte compiler in Rust.

1. **100% Test Compatibility** - Pass all tests from `svelte/compiler` test suite
2. **100x Performance** - Achieve 100x speed using Rust optimizations and parallelism
3. **Drop-in Replacement** - Provide N-API bindings compatible with existing tools (Vite, etc.)
4. **OXC Integration** - Design for integration into [oxc](https://oxc.rs/) ecosystem

## Architecture

Directory structure mirrors the official Svelte compiler at `submodules/svelte/packages/svelte/src/compiler/`.

```
crates/rsvelte_core/src/compiler/phases/
├── 1_parse/     # Parsing (Svelte syntax → AST)
├── 2_analyze/   # Analysis (scope tree, bindings)
└── 3_transform/ # Code generation (AST → JS/CSS)
```

Upstream reference repos live under `submodules/`:

```
submodules/
├── svelte/                  # Svelte 5 compiler (mirror target)
├── language-tools/          # svelte2tsx, language-server, svelte-check, typescript-plugin, svelte-vscode
└── typescript-go/           # tsgo — type-check backend for Wave 2 svelte-check
```

The `@rsvelte/vite-plugin-svelte` Vite plugin (a fork of `@sveltejs/vite-plugin-svelte`)
is vendored as a workspace package at `apps/npm/vite-plugin-svelte`, not a submodule.

See `docs/ecosystem-implementation-plan.md` for the multi-wave plan to port
the Svelte ecosystem (svelte2tsx, svelte-check, vite-plugin-svelte) on top
of the rsvelte compiler.

### Corpus output-equality pipeline (`scripts/compat-corpus/`)

Every `.svelte` / `.svelte.(js|ts)` source (including markdown code blocks)
from every corpus source repository — sveltejs/svelte, sveltejs/svelte.dev, and
the real-world projects bits-ui / flowbite-svelte / melt-ui / shadcn-svelte, all
pinned as submodules and listed in `scripts/compat-corpus/corpus-sources.json`
(~12,000 entries) — is compiled with both the official compiler and rsvelte for
CSR **and** SSR, and the outputs must be byte-identical after comparison-side
normalization (oxfmt + blank-line stripping — never compiler post-passes). It is
one unified corpus (no separate "ecosystem" track); to grow it, add a submodule
+ a line to `corpus-sources.json` (see the README). CI ratchet:
`compat/corpus/known-failures.json` may only shrink. See
[scripts/compat-corpus/README.md](scripts/compat-corpus/README.md),
[docs/corpus-remaining-work.md](docs/corpus-remaining-work.md) (burn-down
playbook for the remaining entries), and
[docs/phase3-server-ast-remaining-work.md](docs/phase3-server-ast-remaining-work.md)
(Phase-3 AST refactor status + handoff). **Phase-3 OUTPUT codegen is now AST:**
server SSR is pure-AST (legacy text generator, ~32k lines, deleted) and client CSR
defaults to `js_ast::to_oxc` → `rsvelte_esrap` (the 3324-line text printer is now a
~6% fallback for comment-bearing / unsupported-node programs). Remaining text
processing is the INTERNAL IR construction (client visitors building `Raw` strings
that `to_oxc` re-parses; `shared/async_body.rs` ~3100 lines of string-based async
splitting; the `.svelte.js` module path) — output is unchanged, so it is an
elegance/maintainability cleanup tracked in the handoff doc.

**Key Design Decisions:**

- Memory-efficient layout (u32 positions, compact_str)
- Thread-safe parser with rayon parallelism
- Direct AST passing (no re-parsing between phases)
- No backward compatibility for internal APIs (refactor freely)

## Implementation Principles

**CRITICAL**: All implementations must follow the official Svelte compiler implementation.

1. **Reference Implementation** - Always check `submodules/svelte/packages/svelte/src/compiler/` before implementing
2. **Structural Consistency** - Mirror directory structure, module organization, and naming
3. **Exact Output** - Output must match the official compiler exactly (verified by tests)
4. **Test-Driven** - Verify all changes against the official Svelte test suite

When implementing, reference the corresponding file in `submodules/svelte/packages/svelte/src/compiler/` and use the same algorithms and logic.

## Development Workflow

### Setup

```bash
git submodule update --init --recursive
git config core.hooksPath .githooks
pnpm install
pnpm run generate-fixtures  # Required before running tests
```

### Build & Test

```bash
cargo build                                          # Build
cargo test                                           # Run all tests
cargo test --release                                 # Release mode (recommended for full runs)
cargo test --test parser_fixtures -- --nocapture     # Run a single suite
pnpm run compatibility-report                        # Generate compatibility report JSON
pnpm run test-and-update                             # Refresh report + docs
```

Pre-commit hooks run `cargo fmt` and `cargo clippy` automatically (`.githooks/pre-commit`).

### Docker (optional)

A `Dockerfile` and `docker-compose.yml` are provided for a reproducible toolchain (Rust nightly + Node 22 + pnpm). There is no `docker-dev.sh` wrapper — invoke Compose directly:

```bash
docker compose up -d            # Start dev container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [baseballyama/rsvelte](https://github.com/baseballyama/rsvelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
