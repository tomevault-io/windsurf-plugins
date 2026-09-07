---
trigger: always_on
description: Old World **XML-only mod that strips high-variance events from competitive multiplayer**.
---

# CLAUDE.md — owmptweaks

Old World **XML-only mod that strips high-variance events from competitive multiplayer**.
Read `README.md` first, then `WHATS-DIFFERENT.md` for the shipped list. This file holds the
grounding facts and the reasoning behind the shape.

## ► STATUS (2026-08-23) — v0.1 built, NOT yet GUI-verified

- ✅ 49 eventStory removals + 1 global + 1 event-option reweight, all generated from
  `tools/tweaks.py`, all verified against the shipped XML (`python3 tools/gen.py`).
- ✅ `./build.sh` deploys to `~/Library/Application Support/OldWorld/Mods/OwMpTweaks/`.
- ⏳ **Never launched.** Nobody has confirmed the game loads it, the `-change` files apply, or
  that a removed event actually stops firing. That is the next step and Claude cannot do it.
- ⏳ No `mod/modpicture.png`. Only needed to upload to the Workshop (≥256×256, <1MB).
- ⏳ `modplatform` is `Local` and `workshopFileID` is 0. **After the first upload the game
  writes the Workshop identity back into the installed copy — copy it into `mod/ModInfo.xml`
  by hand, or the next `build.sh` (`rm -rf "$DEST"`) severs the link and re-uploading creates
  a duplicate item.** Same trap as `../owtraitmod`.

### Verifying in game

Enable, restart, start a game, then check the removals took. Cheapest check: `INFERTILE_PROB`
is a global, so if the mod loaded at all the XML pipeline worked. For the events, the honest
test is a long hot-seat game, which is slow — consider borrowing `../owtraitmod/test/` (it
merges `mod/Infos/*.xml` into a symlinked copy of the base XML tree and boots the real
GameCore headless) and asserting that a removed zType has `miWeight == 0` after Infos loads.
That would catch a `-change` file silently not applying, which is the failure mode most
likely to go unnoticed.

## Design decisions (settled — don't relitigate)

- **One mod, preferences baked in, no game-option toggles.** An earlier draft proposed adding
  `GAMEOPTION_MPTWEAKS_*` entries and gating each story with `aeGameOptionInvalid`. Dropped
  deliberately: it prevents option hell, and splitting into per-tweak mods is worse still
  because **every player in an MP lobby has to enable the identical set** — one checkbox
  scales, eighteen do not. If people feel strongly, revisit then, not now.
- **Stressed and Unpopular are neutralised with `iRemoveProb 100`, not blocked outright.**
  Settled 2026-08-25: **do not** switch to `iMinAge 999`. The trait landing for a
  single turn and then clearing is the accepted behaviour; the knock-on effects of a hard block
  are not worth it. Full reasoning in the `TRAITS` comment in `tools/tweaks.py`. Don't re-offer
  this.
- **Generated, not hand-written.** `tools/tweaks.py` is the only file to edit; `tools/gen.py`
  emits the three XML files *and* `WHATS-DIFFERENT.md` from it, so the docs cannot drift.
  `gen.py --check` verifies without writing — wire it into CI if this ever gets one.

## Engine facts this is built on (verified in the shipped source)

Source of truth: the shipped `Old World/Reference/` tree (Source + XML) in your Steam
install. **Never edit it** — Steam-synced. `tools/ow.py` locates it automatically, or set
`OW_REFERENCE`. Installed build **1.0.84365 (2026-08-05)**,
Steam buildid 24569266.

### Disabling an event

`Player.cacheEventPools()` → local `canEverDoEvent()` (PlayerEvent.cs:12788) rejects a story
when **`miWeight <= 0`**, before it can enter a trigger pool. Every XML-driven story reaches a
player through a pool, so `<iWeight>0</iWeight>` is a complete disable.

Levers considered and rejected:
- `iMinTurns 200` — what the shipped Barbarian/Carthage mods use. Works, but expires in a long
  game and does not remove the story from the pool.
- `aeGameOptionInvalid` — also honoured by `canEverDoEvent`, and the right lever *if* toggles
  ever come back. Needs a matching `gameOption-add.xml`.
- `Player.isValidEventStory` (PlayerEvent.cs:730, `protected virtual`) — the C# chokepoint, but
  it is **only reached from the pool-selection path** (called once, at PlayerEvent.cs:13031).
  `doEventStory()` does not call it, so it is not a superset of the weight check.

### Mod XML load order (Infos.cs:840–890)

base → **`-add`** (`ADD`, `ADD_ALWAYS`) → **`-change`** → **append** (`AppendLists = true`) →
`-change` again. Two facts that matter:

- The suffix is **`-change`**, not `-modify`. 237 shipped examples under `Reference/XML/Mods/`.
  (`spmorelikemp1` in the local Mods folder uses `council-modify.xml`, which is very likely a
  silent no-op — worth telling the author.)
- **A `-change` REPLACES list fields.** `Infos.readTypes` / `readPairList` call `.Clear()`
  unless `ctx.AppendLists`, which only the append pass sets. So `eventOption-change.xml` must
  restate all ten `aiEventOptionProb` pairs, not just the one it lowers. `gen.py` verifies our
  pair list against the shipped one in both directions so a patch adding an eleventh outcome
  fails the build instead of silently deleting it.
- Scalars are safe: `readInt` assigns whatever parses, so an explicit `0` really is 0.

### Infertility is not a trait

`Character.mbInfertile`, set by `Character.checkMakeInfertile(bool bInit)` (Character.cs:7608),
which has three branches: the `bInit` roll on `INFERTILE_PROB` (10) for anyone created after

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alcaras/OldWorld-MPTweaks](https://github.com/alcaras/OldWorld-MPTweaks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
