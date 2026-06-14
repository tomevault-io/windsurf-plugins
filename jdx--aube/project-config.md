---
trigger: always_on
description: kind: Node.js package manager, Rust workspace
---

project:
  name: aube
  kind: Node.js package manager, Rust workspace
  binary_surface: drop-in for pnpm CLI
  language: Rust 2024
  msrv: 1.93
  async_runtime: tokio (multi-thread for install, current-thread for nested islands)
  errors: miette (user-facing, top-level), thiserror (per-library crate)
  error_codes: every error/warning carries a stable ERR_AUBE_*/WARN_AUBE_* identifier from aube-codes crate; bespoke exit codes for a curated subset; see error_codes section below for conventions

layout:
  global_store: $XDG_DATA_HOME/aube/store/v1/ (contains files/ for CAS shards + index/ for cached package indexes; falls back to ~/.local/share/aube/store/v1/). `aube store path` prints this v1/ dir, matching `pnpm store path` granularity. The legacy index location at $XDG_CACHE_HOME/aube/index/ is migrated in-place on the first store open after upgrade.
  store_hashing: BLAKE3 content addressed, 2-char sharding
  virtual_store: node_modules/.aube/ (isolated symlink layout)
  cache_packuments: $XDG_CACHE_HOME/aube/packuments-{v1,full-v1}/ (regenerable from registry; not part of `aube store path`)
  state: node_modules/.aube-state (install freshness hashes)
  scripts_default: skipped for security, allow-builds allowlist opts in (wired via aube-scripts policy + aube approve-builds)
  coexistence: never touches .pnpm/ or ~/.pnpm-store/ or foreign node_modules (by construction, not enforced)

lockfile:
  canonical: aube-lock.yaml
  preserves_existing: true (detect_existing_lockfile_kind)
  precedence[6]: aube-lock.yaml, pnpm-lock.yaml, bun.lock, yarn.lock, npm-shrinkwrap.json, package-lock.json
  frozen_fast_path: install/frozen.rs short-circuits when lockfile + install state match

crates[12]{name,role}:
  aube,CLI binary, clap dispatch, all command impls, progress UI, state tracking
  aube-codes,stable ERR_AUBE_*/WARN_AUBE_* code constants + EXIT_TABLE, no runtime deps, every other crate depends on it
  aube-settings,build.rs codegen from settings.toml, typed accessors, ResolveCtx (cli > env > npmrc > workspace > default)
  aube-resolver,BFS semver resolver, packument cache (5-min TTL + ETag revalidation), peer-context pass, overrides, catalog protocol
  aube-registry,HTTP client for npm abbreviated packuments + tarballs, JSR registry support
  aube-store,CAS at store root (BLAKE3 content), SHA-512 tarball integrity (registry format)
  aube-linker,isolated mode under .aube/<dep_path>/node_modules/<name>, hoisted mode via hoisted.rs, rayon-parallel symlink pass
  aube-lockfile,parse/write for all 6 formats, peer-context aware, format-preserving merge
  aube-manifest,package.json parser, workspace glob support, extra: BTreeMap for upstream forward-compat
  aube-scripts,root lifecycle runner, BuildPolicy allowlist for dep scripts (default deny), side-effects cache
  aube-workspace,pnpm-workspace.yaml discovery, workspace: protocol resolution
  aube-runtime,Node version resolution/discovery/install (devEngines.runtime + .nvmrc/.node-version, mise delegation, nodejs.org downloads to $XDG_DATA_HOME/aube/nodejs/, AUBE_RUNTIME_DIR test override)

pipeline:
  install: cli -> resolver (+ lockfile read) -> registry (fetch tarballs) -> store (CAS import) -> linker (virtual store + symlinks) -> scripts (root only by default)
  tarball_integrity: SHA-512 (registry format, keep as-is)
  cas_hash: BLAKE3 (blake3::hash / blake3::Hasher)
  file_materialize: reflink (APFS/btrfs) -> hardlink (ext4) -> copy (fallback, probed via LinkStrategy::detect_strategy)
  auto_install_detection: BLAKE3 hash of lockfile + package.json + resolved install-shape settings, stored in node_modules/.aube-state

subsystems:
  catalogs: catalog: protocol, root pnpm-workspace.yaml catalog + named catalogs (evens, etc.), resolver rewrites to resolved spec
  workspace_protocol: workspace:*, workspace:^, workspace:~ resolve to local workspace package via aube-workspace
  peer_context: aube-resolver/src/peer_context.rs, fixed-point loop (max 16 iterations) computes nested peer suffixes for dep_path hashes
  overrides: aube-resolver/src/override_rule.rs, pnpm.overrides / overrides map rewrites versions during resolve
  patches: aube patch / patch-commit / patch-remove, diffs applied at materialize time, sidecar .aube-patches.json
  frozen_fast_path: install/frozen.rs, short-circuits install when state + lockfile + install-shape settings match
  hoisted_mode: aube-linker/src/hoisted.rs, alternative layout (node-linker=hoisted) for pnpm-compat
  side_effects_cache: aube/src/commands/install/side_effects_cache.rs, caches allowlisted build output (node-gyp rebuild etc.)
  jsr_registry: aube-registry/src/jsr.rs, JSR package registry alongside npm
  progress_ui: clx::progress wrapper in aube/src/progress/, load-bearing for all install output

support:
  issues_disabled: true
  channel: https://github.com/jdx/aube/discussions
  doc_phrasing: link Discussions, never "file an issue" or "open an issue"

github_interactions:
  comments: when posting comments on GitHub PRs or Discussions, include a note naming the AI coding assistant that generated the comment
  examples[2]:
    - "*This comment was generated by Codex.*"
    - "*This comment was generated by Claude.*"

error_codes:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdx/aube](https://github.com/jdx/aube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
