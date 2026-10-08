---
trigger: always_on
description: Guidelines for AI coding agents working in this repository. Claude-compatible
---

# AGENTS.md — FrankenSonos

Guidelines for AI coding agents working in this repository. Claude-compatible
tools also read `CLAUDE.md`, which points here.

## Rule 0 — Direct Instructions

Follow Jeffrey's direct instructions. These rules encode standing preferences;
they do not overrule the user. Finish authorized work and report concrete
results, with the checks that support them. Do not claim runtime or integration
success from documentation or a passing `cargo check` alone.

## Rule 1 — Scope boundary is law

Read [`docs/SCOPE.md`](docs/SCOPE.md) before any task. FrankenSonos is a **local
interoperability controller for the owner's own Sonos hardware on the owner's
own LAN** — the same category as SoCo and Home Assistant. Agents build:
discovery, UPnP/SOAP control, GENA events, DIDL/URI construction, Spotify **Web
API library reads** (owner's own account), the daemon, API, MCP server, CLI, DJ,
and the Tailscale deployment.

A reverse-engineering lane **is authorized** by the owner (see the exception in
[`docs/SCOPE.md`](docs/SCOPE.md)): agents may document the protocols, do static
analysis of Sonos firmware/binaries for interoperability understanding, passively
capture the owner's **own** LAN traffic, probe the owner's **own** devices, and
use parameters observed on the owner's own system at runtime.

The hard rules that still bind every lane, including RE:
- **Never commit real site data or secrets to this public repo** — no real IPs,
  MAC addresses, serial numbers, household IDs, tokens, keys, certificates,
  firmware images, or packet captures. Public docs/fixtures use placeholders and
  public protocol constants only; real captures and observed parameters live in
  a local, git-ignored area and/or are learned at runtime.
- **No doxing** the owner or anyone else.
- **Firmware *study* is in scope; *writing*/flashing/patching a device is not** —
  brick risk on hardware the owner relies on, so it stays human-led with one
  explicit owner approval per operation.
- Do not target anything the owner does not control, and do not defeat DRM or
  access controls on third-party services.

If a task reads as "handle someone's secrets," "dox the owner," or "write to a
device," stop and ask.

## No Deletion Or Destructive Git

Do not delete files or directories without explicit written permission,
including files you created. Do not run `git reset --hard`, `git clean -fd`,
`rm -rf`, force pushes, forced branch moves/deletes (`git branch -M/-D`,
`git push --force`), or equivalent, without explicit authorization for the exact
operation. Inspect before changing. Preserve work you did not create; never
stash, revert, or blanket-stage another agent's changes — stage explicit owned
paths. Work on `main`; create a branch only when asked.

## Project Mission And Reading Order

FrankenSonos is a memory-safe Rust end-run around Sonos's own software: control
the owner's Sonos gear reliably from a CLI, an HTTP API, and an MCP server, and
act as a tasteful classical-music DJ from the owner's own Spotify library,
including off-LAN over Tailscale.

```text
SSDP discover + UPnP/SOAP control + GENA events  (fsonos-proto)
  -> live inventory/topology + coordinator-addressed control + durable store (fsonos-core)
  -> Spotify library reads + DJ selection (fsonos-spotify)
  -> HTTP API (fsonos-api) + MCP tools (fsonos-mcp) + `fsonos` CLI/daemon (fsonos-cli)
```

Read before substantive work:

1. [`COMPREHENSIVE_PLAN_FOR_FRANKENSONOS.md`](COMPREHENSIVE_PLAN_FOR_FRANKENSONOS.md)
   — the self-contained source of truth: goals, architecture, lanes, protocols,
   the dependency recipe, data model, testing, milestones.
2. [`docs/SCOPE.md`](docs/SCOPE.md) — the in-scope / out-of-scope boundary.
3. [`README.md`](README.md) — the product and command contract.
4. Actual source, the committed `Cargo.lock`, fixtures, and live `br` task state.

When code and prose disagree, current source + executed checks establish
behavior; reconcile at the boundary and update the docs.

## Architecture Doctrine

- One Cargo workspace; the lanes own disjoint crates (`fsonos-{types,proto,
  core,spotify,api,mcp,cli}`). Do not reach into another lane's crate files;
  coordinate at the public API boundary.
- **Pure core, I/O at the edges.** Protocol encode/decode (SSDP, SOAP, GENA,
  DIDL) and DJ selection are pure, unit-tested without a network. I/O sits
  behind the `Transport` and `Store` traits; the franken async stack is wired in
  one place (bead `FND-DEPS`).
- **Coordinator-addressed control.** Group-wide commands target the group's
  coordinator; `fsonos-core` resolves it. **Event-driven state** via GENA, not
  polling storms. **Two households** (S1/S2) handled explicitly where protocols
  differ (music-service linkage, Queue, SonosNet vs WiFi).
- `#![forbid(unsafe_code)]` at every crate root (the workspace lints enforce it).
  This workload has no reason for unsafe, SIMD islands, or unsafe parsers.
- Rust 2024, toolchain pinned in `rust-toolchain.toml` (`nightly-2026-08-31`),
  committed `Cargo.lock` (this is an application — the lock IS tracked).

## Dependency Recipe (the franken stack) — highest integration risk

Wired by bead `FND-DEPS`; the workspace `Cargo.toml` holds the exact lines, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankensonos](https://github.com/Dicklesworthstone/frankensonos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
