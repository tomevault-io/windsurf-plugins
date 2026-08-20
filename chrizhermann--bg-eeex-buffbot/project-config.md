---
trigger: always_on
description: BuffBot is a mod for Baldur's Gate: Enhanced Edition (BG:EE) and BG2:EE that provides in-game configurable buff automation. It depends on [EEex](https://github.com/Bubb13/EEex) for Lua access to engine internals. Inspired by [Bubble Buffs (BUBBLES)](https://github.com/factubsio/BubbleBuffs) from Pathfinder: Wrath of the Righteous.
---

# BuffBot — Project Context for Claude Code

## What This Is

BuffBot is a mod for Baldur's Gate: Enhanced Edition (BG:EE) and BG2:EE that provides in-game configurable buff automation. It depends on [EEex](https://github.com/Bubb13/EEex) for Lua access to engine internals. Inspired by [Bubble Buffs (BUBBLES)](https://github.com/factubsio/BubbleBuffs) from Pathfinder: Wrath of the Righteous.

## Current Phase

Alpha — all MVP features implemented and verified in-game. See `CHANGELOG.md` for the release timeline and `gh issue list --repo Chrizhermann/bg-eeex-buffbot` for open work.

### Modules

- `M_BfBot.lua` — entry point loaded by WeiDU via M_ prefix
- `BfBotCor.lua` — namespace, version, logging utilities
- `BfBotCls.lua` — spell classifier (scoring, targeting, duration, variants, overrides)
- `BfBotScn.lua` — per-sprite spellbook scan (iterator-based, cached)
- `BfBotExe.lua` — parallel per-caster execution, skip detection, combat safety
- `BfBotPer.lua` — persistence (UDAux marshal, presets, INI, export/import)
- `BfBotInn.lua` — per-preset F12 innate abilities (runtime SPL generation)
- `BfBotUI.lua` + `BuffBot.menu` — in-game config panel (movable, resizable, dynamic ~80% screen)
- `BfBotTst.lua` — in-game test suite (`BfBot.Test.RunAll()` in in-game console)
- `setup-buffbot.tp2` — WeiDU installer (under `buffbot/`)
- `tools/deploy.sh` — dev deploy (copies to `<game>/override/`, patches TLK)
- `tools/patch_tlk.py` — appends BuffBot innate tooltip names to `dialog.tlk`

## Design Decisions (do not second-guess without asking)

- **Per-character tabs** are the primary axis — BG spell lists barely overlap across party members (unlike PF:WoTR). Presets are the secondary axis.
- **Dynamic spellbook scan, no hardcoded spell lists** — the mod works with SCS / Spell Revisions / kit mods by reading spell data at runtime.
- **Skip active buffs by default** via SPLSTATE fast-negative + effect-list fallback.
- **Config lives in save games** (per-character, via UDAux marshal). Global prefs in `baldur.ini` section `[BuffBot]`. External export/import via `override/bfbot_presets/*.lua`.
- **Normal mode is engine-paced**; Quick Cast / Cheat Mode is a per-preset 3-state toggle (Off / Long only / All) that applies runtime `BFBTCH.SPL` (Improved Alacrity + casting-speed reduction).
- **User spell priority is always respected** — no automatic reordering by duration category.
- **Must support both BG1EE and BG2EE** — BG1 caps at spell level 5, BG2 reaches level 9 + HLAs. Aura cooldown is 6s; let the engine pace it.
- **Dynamic buff-source scan** — party presets include spellbooks, activated equipped items, quickitems, and inventory potions. Scrolls, wands, containers, and sequencers/contingencies remain deferred.

## Project-Specific Invariants & Gotchas

Generic IE / EEex / .menu gotchas (Lua 0-truthy, opcode 188/189 params, marshal-no-booleans, PlayerN join order, button-in-list, `text color lua` + BAM, `rowNumber` stale in list callbacks, `countAllOfTypeStringInRange` SPAWN_RANGE=400, `.menu` `enabled` as periodic tick, etc.) live in `~/.claude/skills/bg-modding/references/` — invoke the `bg-modding` skill to pull them in. Project-specific rules:

- **Marshal handler name**: `"BuffBot"`, registered in `BfBot.Persist.Init()`. Do NOT rename — every existing save breaks.
- **Config schema v10**: `{v=10, ap, presets=[{name,cat,qc,spells={[resref]={kind,on,tgt,pri,rep,tgtUnlock,lock,var}}}], opts={skip}, ovr={[resref]=1|-1}, summons={[identity]={presets={[n]={qc,spells={[resref]={on,tgt,pri,rep,var}}}}}}}`. Party `kind` is `"spl"|"itm"`; summon entries remain spell-only and deliberately have no `kind`. `var` is optional. `rep` must be a finite integer from `1` through `BfBot.MAX_SPELL_REPEATS` (currently 5); missing or invalid values become `1`. `tgt` = `"s"` / `"p"` / character name / ordered table of names. Legacy slot strings (`"1"`-`"6"`) lazily convert to names in `_Refresh`. Only the protagonist's `summons` subtree is read/written. Older saves migrate lazily; v8→v9 initializes `rep` in both party and summon presets, and v9→v10 stamps missing party kinds as `"spl"`.
- **Items are party-only**: summon discovery, summon presets, and clone seeding must never admit `kind="itm"`. Items execute through `UseItem(resref,target)`, bypass Quick Cast, and only ability 0 is admissible until #53 is solved. Scrolls and wands are explicitly excluded even in quickslots.
- **Summon caster identity and filtering**: `BfBot.Scan.GetAlliedSummons()` returns alive, non-party sprites with allied EA 2–30 and at least one castable spell, keyed stably as clone-owner / script name / CRE resref / display-name fallback. Returned cache tables are read-only; copy before sorting. Re-resolve by object ID + name and revalidate allegiance before acting.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Chrizhermann/bg-eeex-buffbot](https://github.com/Chrizhermann/bg-eeex-buffbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
