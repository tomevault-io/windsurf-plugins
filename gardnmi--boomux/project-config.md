---
trigger: always_on
description: Use `DEVELOPMENT.md` for the human development lifecycle and local build loop.
---

# Repository Workflow

## Start Here

Use `DEVELOPMENT.md` for the human development lifecycle and local build loop.
Use product documentation in this order:

1. `CONTEXT.md` defines canonical product terms and distinctions. Preserve those
   semantics when names in code are less precise.
2. `docs/architecture.md` describes the current implementation boundaries and
   cross-cutting invariants.
3. Contract documents such as `docs/cli-json.md`, `docs/event-stream.md`, and
   `docs/live-pty-handoff.md` govern their
   named interfaces and guarantees.
4. `docs/adr/` records accepted decisions and rationale.
5. `docs/lifecycle-validation.md` records compatibility evidence from specific
   host versions; it is evidence, not a general specification.
6. `docs/roadmap.md` is non-authoritative future intent. Documents marked as
   historical explain how the design was reached but do not override current
   architecture, source, or tests.

For exact protocol and persistence versions, source and compatibility tests are
authoritative. Start a change in the owning module listed in the architecture
module map, then read its colocated tests and the relevant contract document.

## Local Build Cache

- Use kache for local Rust builds. `.cargo/config.toml` selects
  `scripts/rustc-cache.sh`, so ordinary `cargo check`, `build`, `test`, and
  `clippy` commands use it automatically when `kache` is on PATH.
- Manage project development tools with **mise**; keep version pins in
  `mise.toml`. Run `mise install` when setting up a checkout. Kache **0.27.0**
  is installed from its official GitHub release through mise, alongside Zig.
  Do not install a separate kache binary with Cargo or a manual download.
- Use `mise exec -- cargo ...` in agent/non-interactive shells so the pinned
  tools are on PATH; an activated mise shell may use ordinary Cargo commands.
  Check `mise exec -- kache --version` before the first build on a new machine.
  If unavailable, report that caching is disabled; the wrapper permits an
  uncached build. Do not run `kache init`:
  project configuration already enables it without editing global Cargo/shell
  settings or installing a login service.
- `.kache.toml` selects local-only caching, automatic garbage collection, and a
  **20 GiB store budget**. Adaptive/forced incremental caching is disabled to
  avoid accumulating per-worktree incremental state. Native build-script runs
  (including Ghostty's Zig build) remain uncached; Rust compiler outputs are
  cached. Do not enable remote uploads or broaden native caching implicitly.
- Keep each worktree's own `target/` directory. Kache shares reusable artifacts;
  do not point concurrent worktrees at one Cargo target directory.
- Inspect reuse with `kache report --last-build --root "$PWD"` and `kache stats`.
  A Cargo-fresh build may invoke no compiler and produce no new cache events.
  Do not claim a hit rate or speedup without checking the actual report.
  `kache doctor` may flag our delegating wrapper because it expects a direct
  kache wrapper; do not replace project configuration with `doctor --fix`.
- Use `KACHE_DISABLED=1 cargo ...` for a temporary uncached comparison. CI skips
  this wrapper's cache and retains its existing caching workflow. Preserve exact
  compiler arguments and exit status; never retry a failed cached compile
  automatically as an uncached build.
- Cache GC does not cap all project disk usage: target files can retain shared
  blocks. Inspect `kache targets` and a cleanup dry run before removing stale
  outputs. **Never broadly clean `target/`** without preserving
  `target/desktop-dev/`, which contains development session/configuration data.
  Follow the existing worktree-removal rules; caches do not authorize deleting
  worktrees or user work.

See [the local build-cache guide](DEVELOPMENT.md#local-build-cache).

## Validation

Use focused local checks during development. PR CI owns the complete validation
selected by `docs/ci.md`; do not run the full root/Desktop suites or a release
build locally as a routine prerequisite to opening or updating a PR. Require all
selected CI checks to pass before merging.

Do not over-apply validation. Choose the smallest check that answers a concrete
question about the changed behavior, then stop when it passes. Do not stack
`cargo check`, Clippy, full tests, and release builds "just to be safe," or treat
the command lists below as mandatory local gates. Documentation-only edits need
no Rust checks. Small UI edits do not automatically justify a full test build.
Do not repeat successful checks unless subsequent changes affect what they
validated. Report any unverified behavior plainly and leave comprehensive
coverage to PR CI; do not delay reviewable work to duplicate CI locally.

| Change | Local development validation |
| --- | --- |
| Unpackaged Markdown/guidance only | Review links and claims; `git diff --check` |
| Desktop Rust only, optionally with guidance | Formatting and `cargo check -p boomux-desktop --locked`; focused tests or a manual UI check for the changed behavior |
| CI/workflow or packaging scripts without Rust/dependency changes | Relevant Python/Bun/shell fixtures and shell syntax; actionlint for workflow changes |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gardnmi/boomux](https://github.com/gardnmi/boomux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
