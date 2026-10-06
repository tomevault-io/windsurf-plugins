---
trigger: always_on
description: Essential commands, structure, and patterns for AI agents.
---

# Agent Guide for pgconfig

Essential commands, structure, and patterns for AI agents.

## Essential Commands

```bash
just web          # Build the web app, which the server embeds
just test         # Web tests, then cargo test (goldens included)
just lint         # cargo fmt --check and cargo clippy -D warnings
just run          # Serve the API, the web app, and MCP on :3000
just check-conf   # Load a generated config in PostgreSQL (needs Docker)
just release-build  # Rehearse the release build for Linux and macOS
```

`mise.toml` lists every tool the repository uses: Rust, Node, just, and the
release tooling. `mise install` sets them up. Do not
add another tool manager, such as Nix. Run commands through `mise exec --`
when the tools are not on the `PATH`.

## Project Structure

```
.
├── crates/pgconfig/         # Tuning engine, no I/O. `tune` is the entry point
├── crates/pgconfig-server/  # axum: REST v1, OpenAPI at /docs, MCP at /mcp, web app
├── crates/pgconfigctl/      # clap CLI
├── crates/golden/           # Records and replays tests/golden
├── crates/parameter-docs/   # Extracts parameters/ from the PostgreSQL source
├── web/                     # React 19, Vite, Kiso: comparison, export, guide
├── tests/golden/            # Recorded REST v1 and CLI outputs
├── skills/                  # Agent skills of this repository
├── parameters/              # The manual's entry for each parameter, per version
├── rules.yml                # Rule metadata (abstracts, recommendations)
└── pg-docs.yml              # REST v1 `show_doc` text, frozen (ADR 0003)
```

## Code Patterns

- **Rust**, edition 2024, one Cargo workspace. `rustfmt.toml` sets
  `max_width = 100`.
- **axum** for HTTP, **clap** for the CLI, **utoipa** for OpenAPI, **rmcp** for
  MCP.
- **English Language**: All code comments, documentation, and variable names must be in English.
- `crates/pgconfig/src/rules.rs` is the one place values are computed. `tune`
  and the `v1` module both start from it.
- `tune` takes a Tuning Request and returns recommendations with reasons,
  assumptions, and warnings. Use the domain terms: Tuning Request, Tuning
  Recommendation, Tuning Assumption, PostgreSQL Version, PostgreSQL Major
  Version.
- The `v1` module reproduces REST v1 and `pgconfigctl`, known defects included.
  Read `docs/adr/0001-rust-engine-with-v1-frozen-by-goldens.md` before
  changing it.
- `rules.yml`, `pg-docs.yml`, and `parameters/` are compiled into the crate by
  its `build.rs`. No YAML is parsed at run time.
- Three v1 output formats besides JSON: `conf`, `alter_system`, `stackgres`.

## Testing

- `cargo test` runs the unit tests and replays every golden against the Rust
  server and CLI. `tests/golden/README.md` explains the goldens.
- A change to a rule must come with re-recorded goldens. Never edit a golden
  by hand, and never weaken one to make a test pass.
- The server tests need the web bundle: run `just web` first. `cargo test`
  fails with `the web bundle is missing` when you have not.
- `crates/pgconfig/tests/snapshots` holds `insta` snapshots of full results.
- CI: `cover.yml` (Verify), `integration.yml` (the generated config loads in
  PostgreSQL 9.5 to 18), `mcp-conformance.yml`.

## Adding a New Rule

1. Add the calculation to `compute` in `crates/pgconfig/src/rules.rs`, and the
   setting to `Computed` and `Computed::groups`. A setting the release lacks
   is `None`, never zero.
2. Add its reason in `crates/pgconfig/src/reasons.rs`.
3. Write the test first, in `crates/pgconfig/tests/tuning.rs`.
4. Update `rules.yml` if the rule needs metadata. `pg-docs.yml` is frozen, so
   a parameter it lacks gets an empty `show_doc` entry in REST v1: raise that
   with the user before adding one.
5. Record the goldens again and review the diff. A rule change alters REST v1
   output, so it needs a changeset.

## MCP

`docs/mcp.md` is the public contract of `/mcp`. Change it with the code, and
keep the server stateless and read-only.

## CI/CD

- **cover.yml**: release config checks, the web and Rust test suite, and a
  build on macOS and Windows
- **integration.yml**: loads the generated config in PostgreSQL 9.5 to 18
- **mcp-conformance.yml**: the official MCP conformance suite, pinned
- **changesets.yml**: on `main`, opens or updates the Changesets version PR;
  merging it tags `v<version>` and calls `release.yml`
- **release.yml**: builds every target, then publishes with GoReleaser:
  binaries, deb and rpm packages, and Docker images, using the tag's
  `CHANGELOG.md` entry as release notes
- **pr-title.yml**: validates pull request titles as Conventional Commits

## Commit Conventions

Follow commit conventions from `~/.claude/pgconfig.md`:

```
<type>: <subject line (max 50 chars)>

<body wrapped at 80 cols, focus on WHY not WHAT>
```

Types: `feat`, `fix`, `refactor`, `docs`, `chore`, `test`, `style`, `ci`,
`build`, `perf`, `revert`

Rules:
- Title ≤50 chars, imperative mood ("fix" not "fixed").
- Body wrapped at 80 cols, focus on WHY.
- Use `feat` only for user-facing product capabilities. Release automation,
  workflows, and other CI/CD infrastructure must use `ci`.
- Add a changeset (`npm run changeset`) for each user-visible change and commit
  it with the change. CI and docs-only changes need none. See

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [momoi-labs/pgconfig](https://github.com/momoi-labs/pgconfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
