---
trigger: always_on
description: - Margarine continues compilation after diagnostics. Compiler errors alone must
---

# Margarine compiler guidance

## Non-negotiable rules

- Margarine continues compilation after diagnostics. Compiler errors alone must
  never cause an early exit, abort, or code-generation guard.
- Do not run `cargo fmt`; preserve the repository's existing formatting style.
  For multi-line right-hand-side expressions, put the binding or operator on
  its own line and begin the expression on the next line.
- Read `CONTRIBUTING.md` before committing. Commit messages use
  `<type>(<scope>): <summary>` with a lowercase, imperative or noun summary.

## Required verification

After every change to a project or repository file, run the complete,
unfiltered core language suite from the repository root:

```sh
cargo run -p margarine -- test tests/core.mar
```

Report every failure and its output. Compare against a clean checkout before
attributing an existing failure to the change. Prefer `.mar` tests under
`tests/` for observable language and compiler behavior; use Rust tests for
parser/AST contracts, low-level contracts, and diagnostics or compile failures
that cannot run through the language test runner.
- Release, version, managed-layout, and updater behavior tests live under
  `margarine/tests/`; test them through the public API or CLI instead of adding
  source-embedded tests to those production modules. The updater integration
  suite owns its fake release-server and managed-installation fixtures.

## Compiler and CLI architecture

- `margarine/src/cli/mod.rs` owns the clap command surface, dispatch, typed
  `CliError`, and exit-code rendering. `compile.rs`, `test.rs`,
  `artifacts.rs`, `distribution.rs`, `installation.rs`, `update.rs`, and
  `toolchain.rs` own their named responsibilities.
  `margarine/src/main.rs` is the process boundary and the only
  `std::process::exit` callsite.
- Command implementations return `CliResult`. Exit codes are 0 for success,
  1 for compilation or test failures, 2 for clap misuse, 3 for link/toolchain
  failures, and 4 when `run` cannot represent a child exit status.
- `check` tokenizes, parses, and performs semantic analysis only. `build`,
  `run`, and `test` still codegen after diagnostics. The public pipeline is
  `Compiler` + `CompilationResult`: register the entry with `FileData::open`,
  call `run`, then `check`, then `codegen` as appropriate.
- `build --shared` selects `BuildMode::Shared` through the single public
  `build` API. It links native output as `.dylib` on macOS, `.so` on Linux,
  or `.wasm` for Wasm, omits the executable entry wrapper, and shares the
  native linker path with `test`.

## Targets, linking, and toolchains

Supported compilation targets are:

- `arm64-apple-darwin`
- `x86_64-unknown-linux-gnu`
- `aarch64-unknown-linux-gnu`
- `wasm32-unknown-unknown`

Native Linux executables link with `clang` and `libstdc++`; native shared
libraries and test libraries use `.so`. Native macOS shared libraries use
`.dylib`. Shared Wasm modules use `wasm-ld --shared` without an executable
entry or exported linear memory. The compiler links system LLVM 18.1 through
`llvm-sys` and vendors libgit2/OpenSSL while using Rustls for HTTP. Native
non-Wasm external ABIs use indirect returns for structs larger than 16 bytes.
macOS release compiler builds statically embed zstd, resolve unwind through
Apple's SDK and `libSystem`, and reject every non-system dynamic dependency.
Apple system libraries and frameworks remain dynamic.

`core` and `std` runtime archives are built from the C sources under
`margarine/runtime/core/` and `margarine/runtime/std/` by
`scripts/build-toolchains.sh`. The Wasm `libcore.a` also bundles the vendored
dlmalloc implementation through its linear-memory growth adapter; it does not
depend on WASI or a host-provided libc allocator. Release toolchains place the
resulting archives under `toolchains/<target>/libs/`;
`MARGARINE_TOOLCHAIN_DIR` overrides the root. Every native and Wasm link
searches that directory and passes each regular file there as an explicit
input. Source packages are fetched from the published CDN `share/` tree by
default; `MARGARINE_DEFAULT_URL` overrides the base for ordinary `pkg:`
imports, and `MARGARINE_PRELUDE` overrides preludes. The `git2` dependency
must keep its `https` feature enabled because those package URLs are cloned
over HTTPS; `reqwest` remains the Rustls-only HTTP client.

Ordinary compiler commands work from unmanaged or source-built binaries.
`update` and `toolchain add` require a managed installation. Initial release
installation is owned exclusively by `scripts/install.sh`; there is no
`margarine install` command. The script downloads and checksum-validates the
complete compiler and host-toolchain archives, rejects unsafe archive entries,
stages both under the versioned layout, verifies the compiler before and after
activation, publishes atomically, and rolls back on failure. The updater uses
the same versioned layout and complete compiler archives. Versioned
installation directories and release tags use full SemVer, including
prerelease and build metadata; update comparisons use SemVer precedence.

Native generated entry points use the C `main(int, char **)` ABI and forward
`argc`/`argv` to the standard-library runtime before startup functions run;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [todaymare/margarine](https://github.com/todaymare/margarine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
