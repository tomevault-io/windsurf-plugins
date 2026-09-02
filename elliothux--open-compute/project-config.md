---
trigger: always_on
description: - Do not send optional commentary.
---

# AGENTS – open-compute

## Operating Contract

- Do not send optional commentary.
- For answer, explanation, review, diagnosis, or planning requests, inspect the relevant evidence and report the result; do not implement changes unless requested.
- For change, build, or fix requests, make the requested in-scope local changes and run relevant non-destructive validation without asking first.
- Reading files, inspecting logs and Git state, editing requested local files, and running non-destructive checks are authorized local actions.
- Ask before external writes, destructive actions, privileged commands, runtime downloads, release packaging, or a material scope expansion.

## Day1 Architecture

- **open-compute is still in development and has not been deployed to production. All logic and architecture must follow Day1 design for current requirements.** This applies to Rust, TypeScript, runtime assets, persistence, APIs, CLI, configuration, and tooling.
- Do not design for compatibility with earlier open-compute revisions, historical development data, old APIs/configuration, schemas, snapshots, artifacts, module paths, or byte/hash formats. Do not introduce or retain compatibility shims, legacy implementations, version-selection branches, dual read/write paths, aliases, defaults, or backfills to preserve those earlier states.
- The only compatibility exception is observable behavior required by an official Cloudflare API contract within this project's declared support scope, including applicable compatibility dates and flags. Before retaining a compatibility branch, document the official source, the exact affected behavior, its supported date/flag or version range, the pinned workerd constraints where relevant, and regression coverage. Implement that contract directly; it does not justify preserving obsolete open-compute engines, private protocols, schemas, configurations, snapshots, or byte/hash formats.
- Change the current model directly and keep one authoritative implementation. Update affected producers, consumers, schema definitions, fixtures, tests, build paths, and documentation together within the requested scope; remove superseded code in the same change. Existing code, version labels, tests, and historical design documents do not create compatibility obligations.
- Continue to enforce current dependency/protocol contracts, the pinned workerd requirements, security boundaries, data integrity, immutable deployment semantics, and restart/crash recovery within the current implementation. Development status does not permit weaker validation, skipped Gates, or silent repair of corrupt state.
- Day1 design does not authorize deleting or resetting existing local data, rewriting Git history, or removing retained evidence. The Operating Contract's approval requirements and evidence-preservation rules still apply.
- The completed removals and their acceptance evidence are archived in [Day1 architecture cleanup](docs/implemented/day1-architecture-cleanup.md). Historical stage documents do not recreate compatibility obligations.

## Repository Scope

- This file applies to the entire repository. Do not add nested `AGENTS.md` files.
- Treat this repository as the source of truth for `open-compute`; do not edit the parent Lynx OS project unless the user explicitly includes it in scope.
- The authoritative project and documentation origin is `https://open-compute.dev`; reverse-DNS service identifiers use the `dev.open-compute` prefix.
- `crates/**`, `packages/**`, and `share/**` own production sources, tooling, and assets; `test/**` owns repository-level test/Gate scripts, fixtures, and fuzz tooling; `scripts/**` and `examples/**` are operator/release surfaces. Keep crate-local and package-local tests beside their owning code. The TypeScript runtime lives in `packages/runtime/`; its generated `dist/` is not tracked. See [Runtime and test layout](docs/implemented/runtime-and-test-layout.md) for scope and acceptance evidence.
- Completed POC upstream-capability probes are disposable one-time code, not recurring acceptance Gates. Delete them in the scoped cleanup; move surviving product regression tests and their required harness/fixtures from `poc/` into `test/`, consolidating equivalent coverage instead of retaining a second prototype implementation. Do not keep an old `poc/g0` compatibility entry point.
- Preserve `docs/implemented/g0-results.md` and other historical evidence as records of their actual runs. Do not hand-edit them, rewrite them to claim new acceptance, or require their old generator to survive the POC cleanup. Deleting probe code does not authorize deleting retained run evidence or data.
- Put all repository-local disposable caches, temporary run directories, diagnostic logs, and retained failure evidence under the root `.temp/<purpose>/` directory. Examples: `.temp/ruff-cache/`, `.temp/cargo-home/`, `.temp/runtime-cache/`, `.temp/g0-run/`, and `.temp/single-binary-run/`. Configure each tool or test producer to use that location; do not add a new `.gitignore` entry for each cache or run directory. The single `/.temp/` rule covers them all.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elliothux/open-compute](https://github.com/elliothux/open-compute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
