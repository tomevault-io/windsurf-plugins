---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repository. This is a
---

# AGENTS.md

Guidance for AI agents (and humans) working in this repository. This is a
condensed orientation focused on the non-obvious conventions and invariants
that aren't immediately apparent from reading a single file. The authoritative
sources for the details behind each section are `README.md` and `CONTRIBUTING.md`.

Stella is a fast, BYOK ("bring your own key"), model-agnostic terminal coding
agent, written in Rust. Proving a task **done** with a **witness test** — one
that fails on the old code and passes on the new — is what "verified done,
not claimed done" means here, and that guarantee is **a property of the path
that produced the evidence, not of the binary**. The built-in staged
pipeline that used to run the check itself and watch the fail→pass flip has
been deleted from this workspace (#3865; `docs/spec/pipeline-as-plugins.md`
§7 names it "the last slice" of the extraction plan, and it has now landed) —
`stella run --pipeline classic` is refused outright, naming `stella plugin
install` as the remedy. Host-run verification no longer exists in this
workspace at all: the only verification path left is an **installed
verification plugin** — `stella run --pipeline <plugin-id>` hands the turn to
that plugin, whose evidence is self-reported. Stella evaluates that evidence
against the plugin's declared rule and never re-runs or re-checks it itself
(#3511; `doc:pipeline-as-plugins` is the extraction plan; `plugins/stella-witness`
ships here as the open reference plugin, and Vera is the paid superset).
Neither path runs by default — a plain `stella run` is the raw step-loop
with no verification stage over it. It is the open-source reference
implementation of
Oxagen's *Engineering Deterministic AI Coding Agents* field manual.

---

## Essential commands

The repo is a Cargo workspace. Rust is **pinned to a concrete version**
(currently 1.97.0) via `rust-toolchain.toml` (rustup fetches it automatically).
Floating on `channel = "stable"` was tried and reverted — each new stable
release ships a slightly different rustfmt, which silently reformats
previously-clean files and turns the CI fmt gate red with zero code changes.
When bumping the pin for a new Rust release, do it as one dedicated PR that
updates the version in `rust-toolchain.toml` and runs `cargo fmt --all` in the
same commit (or the next one) so drift never accumulates. A **`Makefile`**
wraps the common commands with the correct flags — run `make help` for the
full list.

```bash
make build               # cargo build --workspace
make test                # cargo test --workspace
make format              # cargo fmt
make lint                # cargo clippy --workspace --all-targets -- -D warnings
make smoke               # compile check — runs `stella models` (no API key needed)
make help                # list every target
```

**Iterate on a single crate** (much faster than the whole workspace):

```bash
make test-core           # or: cargo test -p stella-core
make test-model          # or: cargo test -p stella-model
make test-tools          # or: cargo test -p stella-tools
```

**Watch mode** (requires `cargo install cargo-watch`):

```bash
make watch               # re-run workspace tests on every save
make watch-core          # re-test stella-core only (fastest loop)
make watch-lint          # re-run clippy on every save
```

**Rustdoc, scoped to the crate you're editing.** `make gate`'s `doc-warnings`
step runs `cargo doc -D warnings --document-private-items` over the whole
workspace, which is what catches a `rustdoc::private_intra_doc_links` error —
a doc comment linking a crate-private item. Neither `make guards-fast` nor a
scoped `cargo clippy` builds docs, so neither one catches it. `CARGO_SCOPE`
narrows the same check to the crate at hand, the same way it narrows `lint`
and `test` above:

```bash
make doc-warnings CARGO_SCOPE="-p stella-core"
# equivalent to:
RUSTDOCFLAGS="-D warnings" cargo doc -p stella-core --no-deps --document-private-items
```

It compiles, so it stays a `make gate` step rather than moving into
`make guards-fast` — CARGO_SCOPE is what makes it seconds instead of the
full workspace.

### The gate — what every push is held to

A red gate is an automatic "not yet". CI is where it runs: on the
maintainer's laptop an agent session does not run `make gate`, a workspace
build, or the workspace test suite — it pushes and reads the run
(CLAUDE.md, "CI builds and tests; this laptop does not"). The list below is
the contract CI enforces and the command a contributor with their own
machine runs before pushing:

```bash
make gate                # = no-scratch + no-secrets + design-refs
                         #   + action-pins + cargo-install-pins
                         #   + untrusted-checkout (no workflow_run job
                         #     checks out a ref its trigger does not
                         #     vouch for)
                         #   + license-allowlist-parity + repro-wiring
                         #   + shellcheck + invariants + doc-links
                         #   + adr-numbering
                         #   + command-docs + brand-case + file-size
                         #   + website-inputs (a Rust test's website/ inputs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [macanderson/stella](https://github.com/macanderson/stella) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
