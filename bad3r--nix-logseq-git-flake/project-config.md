---
trigger: always_on
description: This file provides guidance to coding agents when working with this repository.
---

# AGENTS.md / CLAUDE.md

This file provides guidance to coding agents when working with this repository.

`CLAUDE.md` is a symlink to this file for Claude Code compatibility; keep the content tool-agnostic.

## Scope

- This repo packages Logseq nightly builds as a Nix flake for `x86_64-linux`, `aarch64-linux`, and `aarch64-darwin`.
- Main outputs are `logseq` (desktop), `logseq-cli` (CLI), and `default` (both).
- Most implementation work happens in `modules/`, `lib/loadManifest.nix`, `lib/runtime-libs.nix`, `scripts/update-nightly.sh`, `scripts/render-nightly-release-notes.sh`, and `.github/workflows/nightly.yml`.

## Repo Map

- `flake.nix` is the small flake-parts/import-tree entrypoint.
- `modules/` contains auto-imported flake-parts modules for packages, checks, formatter, dev shells, hooks, overlays, and supported systems.
- `modules/_packages/`, `_checks/`, and `_hooks/` are helper trees ignored by import-tree because their paths include `/_`.
- `modules/_packages/logseq-nightly.nix` assembles the private package set exposed to flake modules as `logseqNightly`.
- `modules/_packages/desktop/assembly.nix` wires the desktop payload, upstream source, and OS-specific desktop derivation. Linux uses the FHS wrapper path; Darwin uses the `.app` bundle package path.
- `modules/_packages/desktop/package.nix` is the actual desktop derivation.
- `modules/_packages/desktop/package-darwin.nix` installs and re-signs the Darwin `.app` bundle.
- `data/logseq-nightly.json` is a validated manifest, not loose config.
- `lib/` contains generic helpers shared by flake modules, including manifest validation and runtime library lists.
- `lib/loadManifest.nix` enforces required keys and `sha256-` SRI hashes.
- `lib/runtime-libs.nix` feeds the Linux desktop FHS wrapper; `overlays/default.nix` stays intentionally small.
- `modules/_packages/logseq-cli/` builds the upstream CLI from the Logseq monorepo. Since `logseq/logseq#dbd220c95d` the CLI front-end is OCaml compiled via Melange and bundled with Vite (`cli/`, `dune build @bundle`); its OCaml/melange/humanize closure is resolved by opam-nix (`opam-deps.nix`) on OCaml 5.4.0. The db-worker-node it spawns at runtime is still a shadow-cljs `:node-script` release. Build inputs: root offline pnpm deps (`pnpm-deps.nix`), the separate `cli/` pnpm workspace for Vite + transit-js (`cli-pnpm-deps.nix`), and the offline Clojure dependency tree for the db-worker (Maven jars plus tools.gitlibs git checkouts, `clj-deps.nix`).
- `scripts/update-nightly.sh` regenerates manifest fields and the CLI source, root-pnpm, cli-bundle-pnpm, and Clojure-deps hashes.
- `scripts/render-nightly-release-notes.sh` renders release notes from the cloned upstream Logseq repo.
- `scripts/render-pr-build-report.sh` renders the per-arch result table posted as a PR comment by `pr-build.yml`.
- `patches/` carries temporary fixes for upstream Logseq source; each patch header documents the bug and its removal condition.
- `.github/actions/resolve-build-metadata/` is a composite action shared by `test-build.yml` and `pr-build.yml`: maps downloaded build artifacts to env vars and outputs (tarball paths, URLs, SRI hashes, tag). Input defaults cover the test flow (all systems, current repo, tag `test-<datestring>-<run_id>`); it exports `RELEASE_TAG`, `RELEASE_NOTES`, and `RELEASE_ASSETS` for the publish composite.
- `.github/actions/publish-release-assets/` is a composite action shared by `test-build.yml` and `pr-build.yml`: idempotently creates a GitHub release and attaches assets. All inputs are optional; tag, notes, and assets fall back to the `RELEASE_*` env handoff from `resolve-build-metadata`, target defaults to the current commit, and `prerelease`/`cleanup-tag` default to true (a future nightly caller would override these). It fails loudly when no asset paths resolve.
- `.actrc` is tracked local `act` configuration; `.act/` is runtime state and stays ignored.
- `.github/workflows/validate.yml` is the clearest snapshot of CI expectations.

## Architecture

### Nightly pipeline

The flake does **not** build Logseq Desktop from source inside Nix. It consumes per-system pre-packaged binary tarballs from a GitHub Release. Linux systems use the flat Electron payload wrapped in an FHS env with a desktop entry and icon. `aarch64-darwin` uses a top-level `Logseq.app` payload, installs it to `$out/Applications/Logseq.app`, re-signs the installed app with an ad-hoc signature, and exposes `$out/bin/logseq`. The CLI is built from source inside Nix on every supported system (OCaml/Melange front-end resolved by opam-nix; the db-worker-node it spawns is a shadow-cljs release).

End-to-end data flow:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Bad3r/nix-logseq-git-flake](https://github.com/Bad3r/nix-logseq-git-flake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
