---
trigger: always_on
description: This file guides Claude Code (claude.ai/code) when working in this repository. It documents what
---

# CLAUDE.md

This file guides Claude Code (claude.ai/code) when working in this repository. It documents what
**exists today**; the forward plan lives in `OS_PLAN.md`. When this file and the plan disagree
about how the code *currently works*, this file wins and the plan is stale — fix it.

## Project Overview

gatOS is a **standalone KSA mod** that runs a real, minimal **Alpine Linux** inside a **QEMU
microVM** subprocess. Players open terminal sessions into the guest through **purrTTY** — which
stays an unmodified terminal emulator, consumed only via its published `purrTTY.CustomShellContract`
extension point — over SSH. The whole point is to stop hand-writing terminal userland: real `apk`,
real shells, real pipes/jobs/pagers/editors, zero custom guest binaries.

Live KSA vehicle telemetry is exposed to the guest **as a filesystem**: a C#-implemented
**9P2000.L server** that the guest mounts at `/sim`, so the entire unix toolbox (`cat`, `watch`,
`tail -f`, `jq`, awk pipelines) becomes the game API surface. Persistence is qcow2 overlays, one
per save profile, on top of a pristine shipped base image.

> **`/sim` is a published API. Its complete catalog is [`SPEC_9P_FILESYSTEM.md`](SPEC_9P_FILESYSTEM.md)**
> — every path, value format, unit, read/write semantic, command action key, errno, and HTTP `/v1` /
> MQTT mirror. It is the reference for anyone (player, modder, or the `gatos` skill) writing programs
> against the sim. **See the binding constitution in "The `/sim` API contract" below: the SPEC must be
> updated in lockstep with any change to the `/sim` surface.**

The architecture and the research behind it are fixed in **`OS_ANALYSIS.md`** (options considered,
why QEMU won); **`OS_PLAN.md`** is the execution plan (milestones M0–M12, fine-grained tasks). Read
`OS_PLAN.md` Part 0 before starting any task — it defines the execution model, repo conventions,
and the decisions locked in (Part 1).

> **Sibling repo:** `../purrtty` is the structural reference (csproj/slnx/CI patterns) and the
> source of truth for the vendored contract assembly. KSA decompiled sources are under
> `thirdparty/ksa/`; the `ksa` skill documents the mod lifecycle and telemetry APIs.

## Current status (what is actually built)

> Full per-milestone detail, class names, and as-built notes → **[`docs/MILESTONES.md`](docs/MILESTONES.md)**

**All milestones through M9, plus G1–G7 (HTTP/serial/TypeScript SDK), the embedded MQTT
transport, host folder mounts (`/mnt/<name>`), the welds / `always_render_iva` / parts-listing
cheats ported from `unscience`, and the `/sim/audio` userland playback feature, are
code-complete.** The only pending work is a set of in-game passes (T6.6/T9.3/G1–G4, plus the
welds/IVA/parts, thug_life, per-vessel scale/always_render, debug impulse, `ctl/translate` and
`/sim/audio` checklists) that require a live KSA flight; checklists are in
[`docs/VALIDATION.md`](docs/VALIDATION.md). The purrTTY tip release is now cut.

> **KSA baseline: `2026.7.6.4939`** (upgrade-ksa playbook pass 2026-07-16, from 4892): **clean — no
> code changes needed**; build + tests green, full decomp/Content diff (gapless changelog, first time)
> found no bound-member drift. Behavior notes (the new fuel-line/tank-transfer/propellant-use system is
> additive on `Tank` but changes *when* engines see fuel — the propellant reads report it truthfully;
> the rev 4914 control-module lockout is **UI-only** — the module methods gatOS binds stay ungated, so
> `/sim` writes still actuate control-less vessels; animating parts now update colliders and force
> off-rails — `animation.goal` on landing legs has real physics; rev 4915 removes the old
> service-module parts, **save-breaking upstream**) + the pass record live in
> [`scope/FULL_SCOPE.md`](scope/FULL_SCOPE.md) §0 / the scope pages; live re-check items appended to
> [`docs/VALIDATION.md`](docs/VALIDATION.md).

> **Whole-mod perf pass (2026-07-02):** all seven plans of
> [`plans/GREENFIELD_PERFORMANCE_IMPROVEMENT_PLANS.md`](plans/GREENFIELD_PERFORMANCE_IMPROVEMENT_PLANS.md)
> (GP1–GP7) are landed — zero-steady-state-alloc sampler (+"Sample alloc" tripwire in the status
> window), memoized `/sim` read surface, subscription-gated/paced MQTT, 9p zero-copy reads + inline
> dispatch, zero-alloc SSH pump, display static-frame suppression, HTTP keep-alive. The formal
> before/after measurement pass (plan §6) is still open.

| Milestone | Status | Key entry points |
|---|---|---|
| M0 — scaffold | DONE | `gatos.slnx`, `Directory.Build.props`, `GatOsPaths` |
| M1 — spike | DONE | `spike/NOTES.md` (**required reading** before M3/M4/M7/M8) |
| M2 — guest image | DONE | `guest/build-image.sh`, `guest/fetch-guest.*`, `GUEST_VERSION`=17 |
| M3 — gatOS.Vm | DONE | `VmHost.cs`, `QemuCommandBuilder`, `DiskManager`, `PortAllocator` |
| M4 — gatOS.Ssh | DONE | `SshShellSession.cs`, `VmConnectionBroker.cs` |
| M5 — purrTTY upstream | DONE (tip release cut) | purrtty commits `9fb5e13`/`a56966a` |
| M6 — gatOS.GameMod | Code DONE; T6.6 pending | `Mod.cs`, `Game/Mod.Game.cs`, `Game/TelemetrySampler.cs` |
| M7 — gatOS.NineP | DONE | `NineP/Server/Session.cs`, `NineP/Vfs/`, `Protocol/` |
| M8 — gatOS.SimFs | DONE | `SimFsTree.cs`, `SnapshotStore`, `StreamFile`, `EventsFile` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meow-sci/gatOS](https://github.com/meow-sci/gatOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
