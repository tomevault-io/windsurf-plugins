---
trigger: always_on
description: We use the Just command runner for many common commands - look inside *.just files to see the
---

# Standard commands

We use the Just command runner for many common commands - look inside *.just files to see the
list of available commands. Some relevant ones are:

* `just build` - build the entire workspace
* `just package=many_cpus build` - build a single package (most commands accept a `package` parameter)
* `just test` - test the entire workspace; this does NOT run doctests, use `just test-docs` for that
* `just docs` - build API documentation

The `package` argument must be the first argument to any `just` command, if used. You can specify it on most commands to scope them down to a specific package instead of running them on the entire repo (which is slow).

Avoid running `just bench` (wall-clock Criterion benchmarks) without explicit confirmation: they
take a lot of time, and the numbers are also noisy and machine-dependent — running them on a
shared machine produces results that should not be acted on. `just test` already runs a single
iteration of every Criterion benchmark to validate that they still execute.

`just bench-cg` (Callgrind / Gungraun) is different: it runs each scenario once under Valgrind's
CPU simulator, so the instruction counts and simulated cache numbers are deterministic and
unaffected by other processes on the machine. It is safe to run `just bench-cg` (or
`just package=foo bench-cg`) any time without asking — including as a smoke test of a new
Callgrind benchmark. The same applies to the `bench-cycles` recipe in packages that still use it.

We generally prefer using Just commands over raw Cargo commands if there is a suitable Just command
defined in one of the *.just files.

Do not execute `just release` - this is a critical tool reserved for human use.

Do not use VS Code tasks, relying instead on `just` and, if necessary, `cargo` commands.

# Validating changes

Validate changes via `just validate-local`. This runs a number of different checks and will
uncover most issues. If you only touched a few packages, scope it to them via `package=foo`.

We operate under a "zero warnings allowed" requirement - fix all warnings that validation generates.

# Multiplatform codebase

This is a multiplatform codebase. In some packages you will find folders named `linux` and
`windows`, which contain platform-specific code. When modifying files of one platform, you
make the equivalent modifications in the other.

When running on a local PC, the operating systme is Windows. When running in GitHub, the operating
system is Linux.

On local PC, you can invoke any Linux commands using the syntax `wsl -e bash -l -c "command"`.
For example, to run the standard validation on both Windows and Linux, execute:

1. `just validate-local`
2. `wsl -e bash -l -c "just validate-local"`

# Facades and abstractions

Some packages like `many_cpus` use a platform abstraction layer (PAL), where an abstraction like
`trait Platform` defined in `packages/many_cpus/src/pal/abstractions/**` has multiple different
implementations:

1. A Windows implementation (`packages/many_cpus/src/pal/windows/**`)
2. A Linux implementation (`packages/many_cpus/src/pal/linux/**`)
3. A mock implementation (`packages/many_cpus/src/pal/mocks.rs`)

Logic code will consume this abstraction via facade types, which can either call into the real
implementation of the build target platform (Windows or Linux) or the mock implementation (only
when building in test mode). The facades are defined in `packages/many_cpus/src/pal/facade/**` and
only exist to be minimal pass-through layers to allow swapping in the mock implementation in tests.

When modifying the API of the PAL, you are expected to make the API changes in the
abstraction, facade and implementation types at the same time, as the API surface must match.

The same pattern may also be used elsewhere (e.g. inside the PAL implementations as a second layer
of abstraction, or in other packages).

# Filesystem structure

We prefer many smaller files over few large files, typically only packing implementation details
and unit tests into the same file but keeping separate API-visible types in separate files (even
if only API-visible inside the same crate).

We prefer to keep the public API relatively flat - even if we create separate Rust modules for
types, we re-export them all at the parent, so while we have modules like
`packages/many_cpus/src/hardware_tracker.rs` the type itself is exported at the crate root as
`many_cpus::HardwareTracker` instead of at the module as `many_cpus::hardware_tracker::HardwareTracker`.

`lib.rs` and `mod.rs` files should only contain API documentation and re-exports. Do not define
constants, helper functions, or other items in these files — move them to dedicated files (e.g.
`constants.rs`) for better filesystem organization.

Inline `mod` blocks defined directly in `lib.rs` / `mod.rs` count as "re-exports" for the purpose
of this rule as long as they only re-export items from other modules (no logic, no constants, no
new type definitions). This is common for `#[doc(hidden)] pub mod __private { pub use ...; }` or
similar grouping modules whose sole purpose is to gather and re-expose existing items under a
distinct namespace.

# Scripting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [folo-rs/folo](https://github.com/folo-rs/folo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
