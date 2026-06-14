---
trigger: always_on
description: Notes for anyone (human or AI agent) extending this editor. The reverse-engineered
---

# AGENTS.md — working notes for the Gothic 1 Remake save editor

Notes for anyone (human or AI agent) extending this editor. The reverse-engineered
save format is mostly self-describing, **except** for one problem: a *fresh* save
only contains the things the hero already has. You cannot learn what skills/items
exist by scanning one save. So we keep **catalogs** of everything the game defines,
and this file explains them and how to grow them.

> Golden rule: **when you discover something new in a save that the editor can't
> offer yet, add it to the catalog** (and note it here if it's a new pattern).

---

## The save format in one paragraph

A `.sav` is an Oodle/Kraken container (`g1r.py: Container`, `decompress`, `rebuild`).
The decompressed **payload** is GVAS: a tree of tagged properties, each carrying a
byte-size field. Every length-changing edit must update **all enclosing size fields**
and re-validate — `g1r.validate()` is the structural oracle (a valid payload parses
to exactly `len - 4`). Never ship an edit that doesn't re-validate. The generic
length-changing editor is `g1r.apply_ops()` (FString replace / array-element delete);
array-element **insert** is done by hand in `learn_skill` / `add_item`.

---

## Skills catalog — `app/catalog.py`

Skills are `GameplayEffectSpec`s in the hero's `ActiveEffects` array. A learned
skill is one effect whose GE class is `…/GE_Skill_<base>[_<tier>]`. The tier (and
sometimes the whole "is it learned") lives in the **class name**.

`catalog.SKILLS` is the authoritative list: `base → {label, category, kind, …}`.

### `kind` decides how the GE class name is built

| kind       | example base       | learned GE class                  | tiers offered |
|------------|--------------------|-----------------------------------|---------------|
| `ladder`   | `Melee_OneHanded`  | `GE_Skill_Melee_OneHanded_Master` | `ladder` list |
| `circle`   | `Mage_Circle`      | `GE_Skill_Mage_Circle_3`          | Amateur, 1..6 |
| `hunting`  | `Hunting_Teeth`    | `GE_Skill_Hunting_Teeth_Trained`  | on/off (Trained) |
| `binary`   | `Acrobatics`       | `GE_Skill_Acrobatics` (no suffix) | on/off |
| `language` | `Orcish`           | `GE_Skill_Orcish_Untrained`       | on/off |

`catalog.skill_class(base, value)` returns the full class path for any kind — use
it everywhere instead of string-concatenating `base + "_" + tier` (binary skills
have **no** suffix; `language`/`hunting` have fixed suffixes that differ from the UI
value).

### Other fields

- `category` — UI grouping (Combat / Thievery / Hunting / Movement / Crafting / Magic / Language).
- `ladder` — (ladder/circle only) ordered tiers above Untrained.
- `has_untrained` — `True` if an `_Untrained` GE class exists, so lowering to
  Untrained is a **rename**; otherwise "Untrained" means **unlearn/delete**.
- `tier_labels` — *(optional)* `{tier: hint}` extra UI text per tier, e.g.
  `Crafting_Blacksmith` shows Trained → "1H weapons", Master → "2H weapons".

`g1r.py` consumes derived views (`catalog.LABELS / CATEGORY / TIER_LADDER /
HAS_UNTRAINED / LEARNABLE`). `LEARNABLE` is what makes a **fresh hero** show every
skill in the roster, not just the ones already in the save.

### How learning works on a fresh hero (no donor to clone)

`learn_skill()` normally clones an existing learned effect and retargets its class.
With nothing to clone, it falls back to `_learn_from_template()`:

1. **Locate the hero's array even when empty.** `_hero_ae_array()` finds the unique
   `Hero` map-key immediately followed by the `ActiveEffects` ArrayProperty
   (`_HERO_AE` regex). Contents-based detection (`_player_skill_span`, the hunting
   anchor) can't do this when the hero has no skills yet.
2. **Append a captured donor element** (`app/skill_donor.py`, base64) to the array:
   `count++`, grow every enclosing size field, splice the bytes, re-validate.
3. **Retarget** the appended element's GE reference to `catalog.skill_class(...)`.

Caveat (same as the pre-existing clone path): only the GE **class** is retargeted;
the effect's internal tags still read like the donor until the game re-derives them
from the class on load. This is why the feature is labelled *experimental*.

---

## How to extend the skill catalog

### 1. Rescan a save for every skill class it contains

```bash
cd app && python3 -c "import g1r,collections; \
  b=open('../../local-test/G1R-012.payload.bin','rb').read(); \
  c=collections.defaultdict(set); \
  [c[g1r._skill_split(m.group(1).decode())[0]].add(m.group(1).decode()) \
   for m in g1r._SKILL_REF.finditer(b)]; \
  [print(k, sorted(v)) for k,v in sorted(c.items())]"
```

(Point it at a *different* save to discover skills `G1R-012` doesn't have, e.g.
`Crafting_Smithing`, `Diving` — these are intentionally **not** in the catalog yet
because no save we have proves their exact GE class name.)

### 2. Add the entry to `catalog.SKILLS`

Pick the right `kind` from the table above, set `label`/`category`, and `ladder` /
`has_untrained` if relevant. That's it — the roster, dropdowns and class paths all
derive from it. Add a quick check in the verify step below.

### 3. (Rarely) refresh the donor template

`app/skill_donor.py` is one real `ActiveGameplayEffect` element captured from

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xetoxyc/gothic-remake-savegame-editor](https://github.com/Xetoxyc/gothic-remake-savegame-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-14 -->
