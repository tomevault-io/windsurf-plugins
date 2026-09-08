---
trigger: always_on
description: > **Name: MinUI Zero** (chosen 2026-07-01) — "drive every unused rail to zero." A fork of
---

# CLAUDE.md — MinUI Zero: performant/efficient MinUI fork for the TrimUI Brick

> **Name: MinUI Zero** (chosen 2026-07-01) — "drive every unused rail to zero." A fork of
> `shauninman/MinUI`. The name lives in `README.md`; internal files/identifiers are **not** renamed
> for branding (keeps upstream merges clean) — the `ZERO_*` env flags are the only Zero-branded symbols.

## What this is
A fork of MinUI focused on **performance through efficiency** on the TrimUI Brick
(`tg5040` platform, Allwinner A133P). The thesis: run each emulated system at the **lowest
CPU clock that still holds its target frame rate**, so the device stays cool and sips power.
This is *not* a feature fork — it's the opposite. NextUI is the feature-rich/GL fork; this
one is the distilled, runs-cold one.

## Scope — **`tg5040` is the fork** (TrimUI Brick + TrimUI Smart Pro); `miyoomini` is a secondary port
The thesis is A133P-specific (cpufreq/OPP/thermal/PMIC), so `tg5040` is the only platform that
carries it and the only one that ships as a real release. `make` still defaults to
`PLATFORMS = tg5040`; a bare `make miyoomini` is rejected on purpose.

**`miyoomini` (Miyoo Mini Plus, SigmaStar SSD202D) is a second, experimental platform** — built,
running, and shipped as a supported (but newer, less-proven) platform, NOT a second first-class
target. Build it explicitly: `make PLATFORMS=miyoomini miyoomini`. Its artifacts are no longer
`-alpha`-stamped (dropped 2026-09-04: the ports fix more stock MinUI bugs than they lack, so the
label undersold them; per-device maturity nuance lives in the README sections instead).
MEASURED and worth knowing before spending effort there: **the efficiency thesis does not transfer**
(~1% — on that SoC the CPU is not where the power goes), so MMP work is parity/polish, not thesis
work. Deep sleep is proven impossible there. Anything that costs the Brick to serve the MMP loses.

Every other MinUI platform is frozen under `workspace/_unmaintained/` — present for history and
upstream merges, **not built or supported**. `workspace/macos/` stays as the zero-hardware dev/test
platform (launcher build + harnesses), not a device. Don't re-add a device without doing that
device's full bring-up (recon + per-SoC wiring) — the MMP port is what that costs.

## Multi-platform parity — features/settings/paks stay in sync unless divergence is EARNED
With more than one platform, drift is the default failure mode, and it happens silently: tg5040
grew systems/settings over months while the MMP kept the pak set from its port date, so 7 systems
sat unlaunchable with their cores already shipping (caught 2026-07-27). The rules:

- **Parity is the default.** A feature, setting, default.cfg value, pak, or core added to one
  platform gets added to the other in the same change — or gets an explicit written reason why not.
  "Didn't think about the other platform" is drift, not a decision.
- **Divergence must be earned by hardware and documented at the point of divergence** (comment in
  the pak/cfg, plus a D-log entry when it's a real decision). Current earned divergences:
  - *Scaling defaults* — MMP GB/GBC/FC ship `Native`, tg5040 ships `Aspect`: panel geometry
    (640×480 makes NES-at-Aspect 2.5× horizontal → scroll shimmer; 1024×768 gives a clean 4.0×).
  - *Audio architecture* — MMP uses the MI_AO daemon + `MINARCH_PRELOAD` shim; tg5040 does not.
  - *Deep sleep* — Brick/SP only (proven impossible on the MMP SoC). *GPU-dark menu* — Brick-only
    (SP panel scans the GLES layer). *Present-skip (`ZERO_DUP_SKIP`)* — qualified on the GLES path
    only; not enabled on the MMP fbdev path until measured there.
  - *Governor brackets* (`MINARCH_FMIN/FMAX`) — per-SoC receipts, never copied across platforms.
    An unmeasured bracket is labelled unmeasured in the launch.sh.
- **Sync by comparison, not memory.** `workspace/all/` is shared automatically — a change there
  hits every platform (verify both; regressing the Brick is the worst outcome). Everything under
  `skeleton/SYSTEM/<platform>/` and the per-platform makefile blocks/`cores/` is duplicated and
  WILL drift. When touching one platform's copy, diff the other: pak sets, default.cfgs, makefile
  core lists, core patches (equivalent modulo the platform block — hash pins must match).
- **Changes that span files ship atomically per platform.** A producer/consumer pair (e.g.
  MinUI.pak sets `MINARCH_PRELOAD`, all 8 emu paks consume it) deployed partially = a broken
  device that looks healthy. `tools/deploy-miyoomini.sh` syncs the whole payload by md5 for
  exactly this reason — never hand-pick files.
- **Config-default changes need the saved-cfg migration** (minarch `cfg_stale_keys`): saved cfgs
  carry every option, so a shipped-default change is silently defeated by old saves unless an
  entry migrates it — see the table's comment for the rules (per-entry version, only where the
  shipped chain resolves to the new value).

## North star / non-negotiables
- **Cool + efficient is the whole point.** Every change should serve "lowest clock that holds
  frame rate." If a change adds heat, idle power, or resident memory without earning it, don't.
- **Default to MinUI's lean software (RGB565) render path.** Fix tearing the cheap way first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zero-Labs-CFW/MinUI-Zero](https://github.com/Zero-Labs-CFW/MinUI-Zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
