---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Antimony**

Antimony is a personal-project compiled programming language with a multi-backend architecture (JS, C, QBE, LLVM, x86). The immediate goal is to mature the QBE backend until two milestones are reached: a bootstrapped compiler (the Antimony compiler written in Antimony and compiled via QBE) and a Doom port written in Antimony.

**Core Value:** The QBE backend must become capable enough that real systems programs — including the compiler itself — can be written in Antimony and compiled correctly.

### Constraints

- **Tech Stack**: QBE as the primary backend — all systems-level work must target QBE
- **Bootstrap**: The bootstrapped compiler must be a full rewrite (not a subset), compiled via QBE
- **Personal project**: No team, no deadlines — prioritize learning and correctness over velocity
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Rust 1.93 - Core compiler implementation, lexer, parser, code generators, CLI
- Python 3.8 - Documentation build tooling
- C - Backend/intermediate language target for compilation
- JavaScript - Backend/intermediate language target for compilation
- SSA (QBE IL) - Intermediate representation for code generation
- x86 Assembly - Backend/intermediate language target (partial)
## Runtime
- Linux (alpine:3.13 for Docker)
- Targets: JavaScript (Node.js), C, QBE, x86 assembly
- Cargo (Rust package manager)
- Lockfile: `Cargo.lock` (present)
## Frameworks
- clap 4.6.0 - CLI argument parsing with derive macros
- rust-embed 8.11.0 - Embed static files (library and builtin code) at compile time
- Cargo - Build system and project management
- mdBook - Documentation generation (with Markdown source)
- mdbook 2.0+ - Book generator for documentation
- qbe 3.0.0 - QBE (QBE Backend Equator) for intermediate code generation
- inkwell 0.7.1 (optional feature: llvm10-0) - LLVM bindings (optional, behind `llvm` feature flag)
- rustfmt - Code formatting (enforced in CI)
- clippy - Linting tool (enforced in CI with `-D warnings`)
## Key Dependencies
- qbe 3.0.0 - QBE backend for SSA/IR generation, required for compilation
- clap 4.6.0 - CLI interface, essential for compiler invocation
- rust-embed 8.11.0 - Embeds builtin libraries and standard library at compile time, required for runtime
- lazy_static 1.5.0 - Lazy static initialization
- llvm-sys 100.2.4 - Low-level LLVM FFI (used only when `llvm` feature enabled)
- libc - C library bindings (required by llvm-sys)
## Configuration
- No environment variables required for basic operation
- Dockerfile uses static linking via musl target for portability
- `Cargo.toml` - Rust manifest with dependencies and metadata
- `rust-toolchain.toml` - Specifies Rust 1.93 with minimal profile
- `.github/workflows/ci.yml` - GitHub Actions CI pipeline
- `book.toml` - mdBook documentation configuration
## Platform Requirements
- Rust 1.93+ (managed via `rust-toolchain.toml`)
- LLVM 10 (required for LLVM backend compilation tests, installed in CI)
- QBE 1.2+ (downloaded and compiled from source in CI)
- Python 3.8+ (for documentation building)
- Standard POSIX tools (git, shell, etc.)
- No external runtime dependencies beyond standard C library
- Docker deployment uses alpine:3.13 base
- Compiled binary can be statically linked with x86_64-unknown-linux-musl target
## Compile Targets
- `.c` - C language output (backend)
- `.js` - JavaScript output (backend)
- `.ssa` - QBE SSA intermediate representation (backend)
- `.s` - x86-64 assembly output (backend, partial)
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Snake case for module files: `string_util.rs`, `parser.rs`, `lexer.rs`
- Test files placed inline with `mod tests;` declarations or in separate `tests/` directory
- Generator implementation files named by target: `js.rs`, `c.rs`, `qbe.rs`, `x86.rs`, `llvm.rs`
- Snake case for all functions: `test_basic_tokenizing()`, `parse_module()`, `generate_function()`, `match_token()`
- Test functions prefixed with `test_`: `test_empty_main()`, `test_parse_function_with_return()`, `test_generate_block_empty()`
- Helper functions in tests prefixed with descriptive names: `builtins()`, `user_code()`, `block()`, `var()`, `module()`, `func()`
- Private helper functions use leading underscore if needed for clarity
- Snake case for all variables: `in_file`, `out_file`, `token_kind`, `var_types`, `dir_out`
- Type-aliased collections are descriptive: `SymbolTable = HashMap<String, Option<Type>>`
- PascalCase for structs: `Token`, `Parser`, `Module`, `Function`, `Variable`, `Statement`
- PascalCase for enums: `TokenKind`, `Keyword`, `Expression`, `BinOp`, `Target`
- PascalCase for trait names: `Generator`
- UPPERCASE for constants
- Type aliases in snake_case when they're type definitions: `pub type GeneratorResult<T> = Result<T, String>;`
## Code Style
- Rust standard formatting (inferred from codebase)
- 4-space indentation observed throughout
- No `.rustfmt.toml` file in repository; uses default Rust formatting
- Clippy is integrated - code uses `#[allow(clippy::...)]` annotations where needed
- Example: `#[allow(clippy::needless_collect)]` in `src/parser/parser.rs:33`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [antimony-lang/antimony](https://github.com/antimony-lang/antimony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
