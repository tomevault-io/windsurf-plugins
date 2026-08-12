---
trigger: always_on
description: This is the v2 rewrite. The v1 fork (`../kunnus-scanner/`) is osv-scanner with a
---

# CLAUDE.md — kunnus-scanner project notes

## Project context

This is the v2 rewrite. The v1 fork (`../kunnus-scanner/`) is osv-scanner with a
`cmd/kunnus/` subcommand bolted on; ~300 files of inherited surface for ~2000 LoC
of actual kunnus logic, plus flaky tests for features we never ship.

v2 depends on **osv-scalibr** directly. We bring our own CLI shell and SBOM
encoder; the scanner library does the extraction work.

## Architectural rules — enforced in code review

1. **Host introspection lives in `internal/detect/`. Target introspection lives
   next to its plugin mapping.** `detect/` answers *where am I running?* and
   stays scalibr-free. Detecting what's at the scan root belongs in the
   registry package that owns the corresponding scalibr plugin names —
   `internal/ecosystem/` for language ecosystems, `internal/osfamily/` for
   Linux distros — so that detection metadata and plugin selection cannot
   drift apart.
2. `internal/scan/` is the **only** package that calls a scalibr scan —
   `scalibr.New().Scan()` (filesystem) and `.ScanContainer()` (container image).
   Every other package operates on `scan.Result` instead.
3. `internal/command/` is the **only** package that imports `urfave/cli/v3`.
   Modes don't know they're being invoked from a CLI.
4. Each `internal/mode/<x>/` package builds a `*scalibr.ScanConfig` from a path
   plus `mode.Overrides`. Its only I/O is calls into `detect`, `ecosystem`, or
   `osfamily` — no raw filesystem reads of its own.
5. **Every scan flavour is a `mode.Mode`; the runner dispatches on the plan.**
   `internal/mode/container/` implements `mode.Mode` like repo and os — its
   `Plan` just takes an image reference instead of a filesystem path, opens the
   image (pulling it for a remote reference), and builds the union of every
   ecosystem and Linux OS-family plugin filtered to Linux capabilities. It
   signals a container scan by setting `Plan.Image`; the shared `runScan` calls
   `scan.RunContainer` when `Plan.Image` is non-nil and `scan.Run` otherwise, so
   all three subcommands are the same `runScan(ctx, cmd, mode, target, ov)`
   one-liner and `internal/scan` stays free of mode types (the dispatch lives in
   `command`, which may know both). Plugin selection skips detection: the union
   is enabled and scalibr's per-extractor `FileRequired` decides what the image
   matches.
   Digests that are only knowable after the scan ride on `Plan.PostScanHashes`,
   a hook the runner invokes with the resulting inventory and merges into
   `Plan.Hashes` — for modes (like container) whose digests key off the scanned
   packages rather than being harvestable during planning.

## Cohesion summary

| Package | Knows about | Does NOT know about |
|---|---|---|
| `command` | flags, modes, scan, sbom, upload | scalibr internals |
| `mode` | detect, ecosystem, osfamily, scalibr plugin names + capabilities | encoding, uploading, CLI flags |
| `mode/container` | image sources (registry/tarball/docker), the installed-state extractors + OS families, scalibr image opening | encoding, uploading, CLI flags |
| `detect` | runtime.GOOS — host introspection only | scalibr, modes, scan-root inspection |
| `ecosystem` | language markers, lockfile hash + licence + dependency-graph parsers, scalibr plugin names (as strings), the `NativeExtractor` flag for ecosystems with no scalibr plugin, the `Supersedes` rules a resolved lockfile makes redundant | scalibr APIs, modes, CLI |
| `osfamily` | distro fingerprints + scalibr plugin imports for each family | modes, CLI, ecosystems |
| `binclass` | filename globs + version-string regexes for non-packaged ELF binaries (ported from syft, Apache-2.0) | modes, CLI, encoding, OS package managers |
| `modustoolbox` | `.mtb` manifest parsing (Infineon/Cypress embedded firmware) → `pkg:github` components | modes, CLI, encoding, ecosystem registry |
| `vcpkg` | `vcpkg.json` manifest parsing (dependencies + overrides + `version>=` floors) → `pkg:vcpkg` components | modes, CLI, encoding, ecosystem registry |
| `gitsubmodule` | `.gitmodules` stanza parsing + `.git/index` gitlink SHAs → `pkg:github`/`pkg:generic` components | modes, CLI, encoding, ecosystem registry |
| `platformio` | `platformio.ini` `lib_deps` parsing (registry specs + VCS URLs) → `pkg:generic`/`pkg:github` components | modes, CLI, encoding, ecosystem registry |
| `espidf` | `dependencies.lock` + `idf_component.yml` parsing (lock preferred) → `pkg:generic`/`pkg:github` components | modes, CLI, encoding, ecosystem registry |
| `zephyr` | `west.yml` manifest resolution (remotes + defaults + repo-path) → `pkg:github`/`pkg:generic` components | modes, CLI, encoding, ecosystem registry |
| `cmakedecl` | FetchContent/ExternalProject/CPM declare grammar in CMake source (pure: stdlib + hashes only) | scalibr, modes, CLI, encoding |
| `arduino` | `library.properties` (vendored lib metadata) + `sketch.yaml` profile pins → `pkg:generic` components | modes, CLI, encoding, ecosystem registry |
| `cmsis` | `*.csolution.yml` `solution.packs` specs → vendor-namespaced `pkg:generic` components | modes, CLI, encoding, ecosystem registry |
| `cmake` | thin `filesystem.Extractor` shell over `cmakedecl` | grammar details (owned by cmakedecl), modes, CLI, encoding |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [think-ahead-technologies/kunnus-scanner](https://github.com/think-ahead-technologies/kunnus-scanner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
