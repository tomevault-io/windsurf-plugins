---
trigger: always_on
description: > Project brief for Claude Code. Read this first, every session, before touching code.
---

# Aegis_SBR — CLAUDE.md

> Project brief for Claude Code. Read this first, every session, before touching code.

## Project Overview (WHY)
**Aegis: Single Button Rotation** (repo/folder: `Aegis_SBR`, formerly "AutoRota") is a
one-button rotation-engine addon for **Turtle WoW** — a private Vanilla+ server running a
custom **1.12 client, patch 1.18.1**. It executes exactly **one primary ability per key
press** using strict single-cast priority lists, to avoid global-cooldown clipping. It
reads combat state (mana/rage/energy, procs, debuff windows) and fires the highest-priority
ability for the player's class/spec/context. Users tune priorities in an in-game config UI
(flat-dark theme, spec tab rails, per-class ability toggles) with per-profile management.

Author tag: "Mercaius & Subtilizer (Torchlite)".

## ⛔ CRITICAL RULES (read first, never violate)
1. **NEVER change rotation or ability-priority logic without explicit user approval first.**
   The existing per-class priority lists are hand-tuned. When the research in
   `docs/rotations.md` disagrees with what a module actually does, your job is to **REPORT
   the discrepancy and ask** — produce a written diff (what the code does vs. what the
   research says, with the source/confidence tag) and WAIT for the user to decide, per
   class. Do not "fix" rotations proactively, even if you're confident. Non-rotation work
   (rebrand, UI, tooling, bug fixes that don't alter priority) does not need this gate, but
   anything that changes WHICH ability fires or in WHAT ORDER does.
2. **The Phase 0 rebrand to Aegis_SBR is DONE (v0.14.0)** — folder/.toc/files renamed,
   `/sbr` primary (+ `/aegis`, legacy `/ar`), `AutoRotaDB` → `AegisDB` migration shim in
   place. Do not reintroduce the old names; keep the shim + toc backup line until the
   deprecation window closes (see `docs/roadmap.md` Phase 0).
3. Run `python3 scripts/verify.py --all` after every edit; never hand off a failing file.

## Current State / Next Task
The rebrand shipped as **v0.14.0** (pending the user's in-game verification: profile
migration, `/sbr` + `/ar`, zero load errors). The addon is **Aegis_SBR** throughout:
core global `Aegis_SBR`, UI `Aegis_SBR_UI`, layout `Aegis_SBR_Layout`, minimap
`Aegis_SBR_Minimap`, frames `Aegis_SBR_*`/`AegisUI_*`, saved variable `AegisDB` (old
`AutoRotaDB` still toc-listed as a rollback backup — drop it + clear on PLAYER_LOGOUT a
few versions from now). **The Phase 1 audit-and-report is DELIVERED (v0.14.1)**: see
`docs/audit-phase1-rotations.md` — a per-class discrepancy report (all 9 classes) with
source/confidence/action/risk per finding, plus the roadmap-pre-authorized Hunter
sting-detection fix (the only code change; no priorities touched). **Next: the user
signs off findings per class; approved items are then implemented as their own gated,
verified batches** (Critical Rule #1 still applies to every one of them).

**Logos:** the user will provide raw logo image files LATER. They need converting to TGA
(power-of-two dimensions, 32-bit, GIMP/uncompressed export — see `docs/roadmap.md` Phase 0
step 6 and `docs/architecture.md`). The header stub is already wired: it tries
`Interface\\AddOns\\Aegis_SBR\\logo` and falls back to the sigil + wordmark while the
file is absent (1.12 `SetTexture` returns nil for a missing file). Drop the TGA in the
addon root as `logo.tga` and do a **full relog** to see it.

All 9 class panels use a unified single-row config layout; all four healer specs have
config panels; a Shaman totem system maintains totems across every spec via SuperWoW's
`UNIT_CASTEVENT`. **Phase 2 is underway:** a shared weapon-enchant detection helper
(`Aegis_SBR:WeaponEnchant`/`WeaponEnchantId`, `GetWeaponEnchantInfo`-based, v0.15.0) backs
Shaman main-hand imbue upkeep and a Rogue poison reminder — the latter was superseded in
v0.16.0 by the **BuffUp integration** (`Aegis_SBR_BuffUp.lua`): an optional buff-watch
monitor (any class, clickable rebuff buttons) and a rogue poison Quick Bar (up to 4
presets, click-applied, never cast from the macro). Also since Phase 2 began: Warrior
Battle Shout + Demoralizing Shout upkeep (v0.15.3, audit items W1/W4), an opt-in Rogue
execute finisher and a Paladin double-heal fix (v0.16.0), and an opt-in Warrior Master
Strike (v0.16.2, Arms talent, off by default). Off-hand imbue, poison auto-apply beyond
the Quick Bar, and Shaman totem-destruction detection remain open Phase 2 items.
v1.1.0 was a docs-only cut (README overhaul + a `docs/research-classicapi.md` deep-dive);
no rotation/engine code changed in it. v1.1.3 was the first code
cut after it, folding in four merges: Rogue buff-renew slider + opt-in Cold Blood and the
`/sbr log` press log (#30), a Paladin Consecration mana-recovery opt-out + creature-type
cache (#31), the sub-level-20 +healing penalty applied to downranked Holy Light (#33), and
the Warlock filler upgrades — Drain Soul as a main filler and a configurable Dark Harvest
gap filler (#34). No priority ORDER changed in any of them. PR **#32** (a `holyLightPct`
health gate for the same Flash of Light problem) was **closed unmerged**, superseded by #33
— don't treat it as shipped; whether a slider is wanted, and which way it points, is open.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Torchlite-bit/Aegis_SBR](https://github.com/Torchlite-bit/Aegis_SBR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
