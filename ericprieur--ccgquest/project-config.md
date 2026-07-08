---
trigger: always_on
description: This file is auto-loaded into Claude's context. Keep it short and focused on
---

# Project notes for Claude

This file is auto-loaded into Claude's context. Keep it short and focused on
things that aren't obvious from the code alone.

## Codex (debug `C` key)

The codex is mostly **auto-generated** from the same data the game uses at
runtime — adding a card / enemy / loot table to the right registry usually
makes it appear in the codex with no extra work. The cache is built lazily
on first codex access and stored in module-level `_codexSourceCache` in
`src/main.js`. A page reload refreshes it.

### What auto-discovers (do nothing extra)

| You added… | And it appears in the codex via… |
|---|---|
| A card creator in `CARD_REGISTRY` (main.js) | Player Cards tab + correct subtype filter |
| A card to a starter deck function (cards.js) | Starter source line + Decks tab section |
| A card to a class ability-choice list (cards.js) | Lost Shrine source line (if `tier === 1`) |
| An entry in `LOOT_TABLES` (main.js) | Loot Tables tab section + per-card `Loot: X (n%)` source line |
| An entry in `SHOP_INVENTORIES` (main.js) | `Shop: <Name> (<price>g)` source line |
| `lootCards: ['x']` on an encounter (encounter.js) | `Drop: <Encounter Name>` source line |
| `previewCreature: ...` on a card (cards.js) | Summons tab entry, side stamped `player` |
| `enemy.addCreature(new Creature(...))` in `setupEnemyForCombat` | Summons tab entry, side stamped `enemy` |
| A new card in an enemy's `enemy.deck.addCard(...)` calls | Enemy Cards tab + Decks tab section + `Enemy: X` source line |

### What needs a manual touch

When adding any of these, also update the listed file:

1. **A new enemy id** in `setupEnemyForCombat` (main.js ~2009) →
   - Add the id to `enemyIds = [...]` in `buildCodexSourceCache`
     (main.js ~12517) so the sandbox scan picks it up.
   - If you want a portrait under **Heroes & Monsters**, add the id to
     `getCodexMonsterIds()` (main.js ~12260) **and** ensure
     `assets/Cards/<EnemyArt>.jpg` is loaded as `creature_<id>` in
     `loadAssets()`.
2. **A new shop** in `SHOP_INVENTORIES` →
   - Add a friendly label to `shopLabels = {...}` in
     `buildCodexSourceCache` (main.js ~12480), otherwise the source line
     falls back to a title-cased id.
3. **A new loot table** in `LOOT_TABLES` →
   - Optional but recommended: add to `LOOT_TABLE_LABELS` and
     `LOOT_TABLE_NOTES` (main.js ~371) for a friendly title and the
     one-line description shown under the section header.
4. **A new non-persistent player card** (token-like, never in
   `CARD_REGISTRY`) → add the creator to `ALL_EXTRA_CARD_CREATORS`
   (main.js ~11492). Currently used for `Goodberry` and the four
   power-choice cards (`Fire`, `Ice`, `Feline Form`, `Bear Form`).
5. **A new power (player or enemy)** → add the creator to
   `ALL_POWER_CREATORS` (main.js, near the codex section). Without this
   the power gets *source lines* but no codex entry. Then:
   - If it's a **player class power**, also add the id to
     `PLAYER_POWER_IDS` so the codex tags it player-side (default is
     enemy).
6. **A sound fired imperatively from an event** (any
   `playSound('foo', ...)` call NOT routed through
   `CARD_SFX_OVERRIDES`/play/flesh/blocked/defense — e.g. a card's
   `playCardAmbient` special-case, an enemy power's start-of-turn
   hook, a fight-start splash, a death cue, an ambient layer) →
   - **The common case needs NO codex wiring:** a card's own cast / hit
     sound added via `CARD_SFX_OVERRIDES` (`play` / `flesh` / `blocked`)
     is surfaced in the codex Sounds tab automatically — the builder
     iterates `CARD_SFX_OVERRIDES` and lists the card under its sound.
     You ONLY have to add the alias to `SOUND_MAP`. The bullets below
     cover sounds that fall OUTSIDE that auto-surfaced path.
   - Make sure the alias is in `SOUND_MAP` (`src/sound.js`); if it's
     only in `SOUND_PACKS` it won't actually play.
   - If the cue is layered on top of a card cast, register it in
     `CARD_SFX_HINTS` (main.js, near `CARD_SFX_OVERRIDES`) so the
     codex Sounds tab shows that card under the file.
   - For per-enemy fight-start / fight-end cues, return the alias
     from `getFightStartSfxKey` / `getDeathSfxKey`. The codex
     Character panel reads those.
   - For looping `playAmbienceLayer(...)` beds, add the file path
     to `MUSIC_TAGS` with a `combat ambience: <fight name>` tag so
     the codex Sounds tab surfaces the wiring.
   - Per-creature swing sounds (Magma Mephit fire whoosh, Shark
     splash, etc.) belong in the creature-name branches of
     `getWeaponSfxKeys` (`flesh`/`blocked`/`play` keys).
7. **A new player summon that no card has `previewCreature` for**
   (e.g. spawned only by an effect handler) → add an explicit
   `addCreature(new Creature({...}), 'Summoned by: <source>')` block in
   `buildCodexSourceCache` (main.js, near the existing Restless Bone /
   Thorb additions). Set:
   - `_codexSide = 'player'` (Decks/Summons side filter)
   - `_sourceRarity = '<rarity of source card>'` (drives frame asset —
     `uncommon`+ uses the ornate frame)
   - `_sourceSubtype = '<subtype of source card>'` (drives frame tint —
     `'ability'` = purple, `'allies'` = brown, etc., via `SUBTYPE_COLORS`)

   For creatures from a card's `previewCreature`, all three fields are

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EricPrieur/ccgQuest](https://github.com/EricPrieur/ccgQuest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
