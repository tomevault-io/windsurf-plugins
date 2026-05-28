---
trigger: always_on
description: Arkade Compiler compiles `.ark` contracts into JSON ABI + script assembly via Rust library, CLI (`arkadec`), and optional WASM bindings.
---

<identity>
Arkade Compiler compiles `.ark` contracts into JSON ABI + script assembly via Rust library, CLI (`arkadec`), and optional WASM bindings.
</identity>

<stack>
| Layer | Technology | Version | Notes |
|---|---|---|---|
| Runtime | Rust toolchain | stable [verify] | Local validation used `rustc 1.91.1` on 2026-03-02 |
| Language | Rust | Edition 2021 | Set in `Cargo.toml` |
| Parser | `pest`, `pest_derive` | `^2.7.8` (`2.8.6` resolved [verify]) | Grammar in `src/parser/grammar.pest` |
| CLI | `clap` | `^4.5.3` | Entry point `src/main.rs` |
| Serialization | `serde`, `serde_json` | `^1.0` | ABI/output models in `src/models/mod.rs` |
| Time metadata | `chrono` | `^0.4.34` | Generates `updatedAt` |
| WASM bridge | `wasm-bindgen` | `0.2` (optional) | Enabled by `--features wasm` |
| Package manager | Cargo | bundled | No workspace; single crate |
| Test framework | Cargo integration tests | n/a | Test suite in `tests/*.rs` |
| Playground runtime | Browser JS + Node + Python | [verify] | Node used by `playground/generate_contracts.sh`; Python serves static files |
| CI/CD | GitHub Actions | [verify] | Build/test + GitHub Pages deploy on `master` |
</stack>

<structure>
project-root/
- `src/` # Compiler crate source [agent: autonomous except gated files]
- `src/main.rs` # CLI entrypoint (`arkadec`) [autonomous]
- `src/lib.rs` # Public library API (`compile`) [autonomous]
- `src/parser/` # Pest grammar + AST parser [autonomous except `grammar.pest` gated]
- `src/compiler/mod.rs` # AST -> ABI/ASM generation [gated]
- `src/models/mod.rs` # AST and JSON ABI data model [gated]
- `src/opcodes/mod.rs` # Opcode constants [autonomous]
- `src/wasm.rs` # WASM exports behind feature flag [autonomous]
- `tests/` # Integration tests and CLI parity checks [autonomous]
- `examples/` # `.ark` fixtures and generated `.json`/`.hack` artifacts [autonomous]
- `docs/` # Language/design references [autonomous]
- `playground/` # Static web playground + build scripts [autonomous]
- `scripts/pre-commit` # Local formatting hook [gated]
- `.github/workflows/` # CI and pages deploy [gated]
- `Cargo.toml` # crate metadata + dependency constraints [gated]
- `README.md` # user-facing docs (can drift; verify against source) [autonomous]
- `.codex/skills/` # project skill catalog [forbidden without explicit instruction]
- `CLAUDE.md`, `agents.md` # agent context configs [forbidden without explicit instruction]
</structure>

<commands>
| Task | Command | Notes |
|---|---|---|
| Fetch dependencies | `cargo fetch` | Use after dependency edits |
| Build compiler | `cargo build` | Primary compile validation |
| Run all tests | `cargo test` | Includes integration and CLI tests |
| Run one test file | `cargo test --test htlc_test` | Fast iteration on one contract scenario |
| Format check | `cargo fmt --check` | Required by CI |
| Format fix | `cargo fmt` | Also used by `scripts/pre-commit` |
| Run CLI | `cargo run -- examples/htlc.ark -o /tmp/htlc.json` | Real args are only `<file>` and optional `-o/--output` |
| Build WASM package | `wasm-pack build --target web --out-dir playground/pkg --features wasm` | Requires `wasm-pack` + wasm target |
| Generate playground contracts | `./playground/generate_contracts.sh` | Regenerates `playground/contracts.js` from `examples/*.ark` |
| Full playground build | `./playground/build.sh` | Generate contracts + wasm-pack + cleanup |
| Serve playground | `./playground/serve.sh 8080` | Uses Python HTTP server |
</commands>

<conventions>
  <code_style>
    Rust naming: `snake_case` for functions/variables, `PascalCase` for enums/structs, `SCREAMING_SNAKE_CASE` for opcode constants.
    Keep module boundaries strict: grammar in `src/parser/grammar.pest`, parse logic in `src/parser/mod.rs`, emit logic in `src/compiler/mod.rs`.
    Prefer explicit `Result<_, String>` in parser/compiler internals; map to richer error in public API (`src/lib.rs`) and CLI (`src/main.rs`).
    Always run `cargo fmt` after source edits.
  </code_style>

  <patterns>
    <do>
      - Update `models` + `parser` + `compiler` together for any language feature change.
      - Add integration tests in `tests/` for every new syntax/opcode path.
      - Validate both function variants (`serverVariant=true/false`) for non-internal functions.
      - Strip or ignore `updatedAt` when comparing expected vs actual JSON in tests.
      - Keep placeholder format `<name>` in emitted ASM.
      - ALL contracts: use `options { server = server; exit = exit; }` (plus `renew = renew;` if needed). Add `int exit` (and `int renew`) as constructor parameters so the playground can set them.
      - Taproot dust: use 330 sats for all minimum output value checks (`>= 330` for viable outputs, `> 330` for dust branch decisions).
    </do>
    <dont>
      - Do not trust README CLI flags blindly; source of truth is `src/main.rs` clap args.
      - Do not edit generated playground artifacts manually (`playground/contracts.js`, `playground/pkg/*`); regenerate.
      - Do not change grammar ordering casually; PEG alternative order changes parse behavior.
      - Do not add new Expression/Requirement variants without compiler emission and tests.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkade-os/compiler](https://github.com/arkade-os/compiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
