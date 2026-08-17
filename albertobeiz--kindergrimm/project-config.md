---
trigger: always_on
description: **Read `ARCHITECTURE.md` first.** It defines the part contract, the two
---

# Working on drawai

**Read `ARCHITECTURE.md` first.** It defines the part contract, the two
coordinate systems and the rules that keep everything looking like one
drawing. Most tasks here are "add a part type" or "add a variant", and
that document is written to make those mechanical.

## Run it

```bash
python3 serve.py
```

`index.html` is the **menu** and the only place the scenes are linked
from — a scene never links to another, only back to the menu. There are
TWO games and they get the SAME card on it: same size, same dark ink,
same weight. The menu is not allowed to say which one is the real one.
`orla.html` is **the class photo**, scored like a poker
hand — pick five of ten children. The five in the photo LEAVE FOR
GOOD and five strangers replace them, so the shelf is always half
faces you have weighed up and half new ones; an object is drafted
after every photo. A persistent class with veteran bonuses was tried
and removed — a bonus that grows on use makes picking the same five
optimal, and it did. It is a FLAT page like the editor and the crowd, and its whole
rule set is one file (`src/orla.js`). `game.html` is **Kindergrimm**:
an endless dark floor, one verb (tap where the class walks), children
who stop to fight whatever they can SEE, and a lamp that competes with
a weapon for the same hand. Its rules are in ARCHITECTURE.md §6b and
they are not the ones an older version of this file described — no
beds, no toys, no play, and light no longer slows anything.
`editor.html` is the face editor,
`crowd.html` a 7×5 page of live characters, `items.html`
the object contact sheet. `voxel.html` is the **voxel lab** — the same
recipe idea built out of cubes instead of graphite, with its own hand,
layout, parts and animator under `src/voxel/`; it shares nothing with
the drawn generator but `rng.js`, and ARCHITECTURE.md §11 is its
contract. `voxelcrowd.html` is the voxel crowd: twenty of them on a
midnight platform, the ONE scene in the project with real (moving,
shadow-casting) lights, and characters that assemble voxel by voxel
via `setDrawRange` — its rules are at the end of §11. `serve.py` sends `no-store` on purpose:
browsers cache ES modules by URL, and a stale module makes an edited
file look like a phantom `SyntaxError`.

## The short rules

- Adding a part = one file in `src/parts/` + one line in
  `src/parts/index.js`. Do not edit `rig.js` for this.
- Adding a **species** = one entry of weights in `src/species.js`.
  A species loads the dice at generation time; it never reaches into
  `draw()`.
- But weights alone give you *a kid in a costume*. A species that
  needs a different HEAD gets a skull param the profile sets (see
  `muzzle`), and a species that needs a shape nobody else could have
  gets its own part with `species: ['nightmare']`. Prefer the cheaper
  lever: a snout is a param, wings are a part.
- Draw through `F.media.tone / skin / edge`. Never call `pencilFill`,
  `washFill`, `oilFill` etc. from a part.
- Size from `F.s`, `F.w`, `F.B.*`. No raw pixel constants.
- `bones()` and `size()` are in world units (`px / U`); `draw()` is in
  pixels with y down and the origin at the head's centre.
- Choices that must hold still go in `gen()`. Randomness used inside
  `draw()` is re-rolled every redraw — that is the line boil, and it
  will shimmer.
- Anything two parts must agree on belongs in `src/layout.js`.
- `game.html` (Kindergrimm) is the only **3D** scene: floor on XZ,
  orbiting ortho camera, yaw-only billboards. It does NOT use
  `addPaper()` — those are camera-facing quads and would go edge-on.
  See ARCHITECTURE.md §6b before touching it. The editor and the
  crowd are still flat pages and must stay working.
- In `game.html` the one rule that everything else hangs off is
  **light is sight, and nothing else**. It does not slow a nightmare,
  it does not shelter a child, and standing in it is free — but a
  child cannot fight what it cannot see. Give light a second job and
  the lamp-or-weapon choice in the `held` slot collapses, which is the
  only decision the game has.
- Adding an **object** = one file in `src/items/` + one line in
  `src/items/index.js`. **The stats ARE the drawing**: the same rolled
  params feed `draw()` and `statsOf()`, so a long blade is drawn long
  AND reaches further. Never add a stat you cannot see; never draw a
  feature that means nothing.
- The draft deals a fixed HAND: **one lamp and four kit**, and every
  card is something a child CARRIES. The light group is
  `kind: 'light'` minus `floor`, so it only ever deals the held
  `Lamp`; the kit group takes everything else that is not `floor`.
  Floor lanterns are not dealt — `placeLantern()` scatters them in the
  dark for you to walk into. `Toy` and `Bed` still exist and are still
  on `items.html`, but nothing deals them any more.
- An item is authored ONCE, in `REF` space with the origin at its
  anchor, and `stamp()` puts it on the card, on the floor and in a
  child's fist. Scale through `ctx.scale`, never by multiplying your
  own numbers — `Sketch` decides granulation and resampling in user
  units, so hand-scaling silently gives you a different item.
- Close every item shape through `finish()`, never `paperFill`/
  `stroke` directly. That is what `F.media.*` is for parts: it owns

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [albertobeiz/kindergrimm](https://github.com/albertobeiz/kindergrimm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
