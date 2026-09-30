---
trigger: always_on
description: Repository-level guide for AI coding agents working on `maxrects-packer`.
---

# AGENTS.md

Repository-level guide for AI coding agents working on `maxrects-packer`.
Human contributors should start with [CONTRIBUTING.md](./CONTRIBUTING.md) instead.

## Project

`maxrects-packer` (v2.7.4, MIT, zero runtime dependencies) is a **MaxRects 2D bin packing** library
written in TypeScript. It aims to pack arbitrary `{width, height}` rectangles into as few bins as
possible, each staying within `maxWidth × maxHeight` (sprite sheets / texture atlases).

- **Packing is a heuristic, not an optimal solver — never claim a minimum bin count.** `addArray()`
  sorts the input, `add()` first-fits each rect into the existing bins in order, and inside a bin
  `findNode()` greedily takes the best-scoring free rectangle for `options.logic`. Nothing searches
  for a global optimum, so a change may only ever claim "fewer bins than before **on these inputs**",
  measured — never "the fewest possible".
- Geometry only: it never reads or writes images or files and depends on no DOM/Node API, so it runs
  in the browser as well.
- The target use case is WebGL/atlases: opening another bin is preferred over emitting a single
  gigantic image.
- It is a rewrite of [Multi-Bin-Packer](https://github.com/marekventur/multi-bin-packer) and the
  public API is kept compatible, so do not change signatures casually.

## Layout

| File | Responsibility |
| --- | --- |
| `src/index.ts` | The only barrel export. Runtime values: `Rectangle / MaxRectsPacker / PACKING_LOGIC / Bin / MaxRectsBin / OversizedElementBin`; types: `IRectangle / IOption / IBin`. The surface is pinned by `test/index.spec.js` — adding or renaming a public value means updating that list in the same commit — and what consumers can actually import is checked by the type fixture in `npm run verify:package` against `package.json`'s `types` entry |
| `src/types.ts` | `IOption`, `PACKING_LOGIC` (MAX_AREA/MAX_EDGE/FILL_WIDTH), `EDGE_MAX_VALUE=4096`, `EDGE_MIN_VALUE=128` (**never used inside the library, but re-exported by `src/maxrects-packer.ts` and part of the public `.d.ts` — not dead code, do not delete**) |
| `src/geom/Rectangle.ts` | `IRectangle` interface + `Rectangle`: `width/height/x/y/rot/data/allowRotation` all go through getters/setters and bump `_dirty` on every mutation; the `rot` setter swaps width/height, the `data` setter syncs `data.allowRotation`; the `Clone` static copies a rect for the bins' `clone()` — prototype and own properties, shallow, or the rect's own `clone()` when its class has one |
| `src/abstract-bin.ts` | `IBin` / abstract `Bin<T>`: the `dirty` semantics and `setDirty()`; `add/reset/repack/clone` are left to subclasses |
| `src/maxrects-bin.ts` | Core single-bin algorithm: `place → findNode(scoring) → updateBinSize(expand) → splitNode(split) → pruneFreeList` |
| `src/oversized-element-bin.ts` | Placeholder bin for one oversized element (`rect.oversized = true`, `add()` always returns `undefined`) |
| `src/maxrects-packer.ts` | Multi-bin scheduling: `add / addArray / next / repack / save / load / sort / rects` |

Data flow:

```
caller's rects
  → addArray: sort (MAX_EDGE|MAX_AREA, ties broken by hash; group by tag first when tags are not exclusive)
  → add: out of bounds? → OversizedElementBin; otherwise take the first bin from bins[_currentBinIndex..] that fits
  → MaxRectsBin.place: findNode scores free rects by logic → updateBinSize grows the bin (pot/square) → splitNode splits → pruneFreeList
  → write x/y/rot back onto that very rect in place and push it into bin.rects
```

## Invariants you must preserve

1. **In-place mutation**: `add()/addArray()` write `x/y/rot/oversized` directly onto the object passed
   in and return that same object (only the multi-argument overloads construct an internal
   `new Rectangle`).
2. **Tag grouping**: with `exclusiveTag: true` (the default) a tagged rect may only enter a bin with
   the same tag, and an untagged bin refuses tagged rects. The check lives in `MaxRectsBin.place()`,
   which every path reaches — `MaxRectsBin.add()` deliberately does not pre-check, so a refused rect
   is simply the `undefined` `place()` returns. `exclusiveTag: false` takes the grouping + recursion
   path inside `addArray()`, which probes a candidate bin by **copying** it: a bin whose `clone()`
   throws (invariant 8) counts as one the group does not fit, so a single uncopyable bin can never
   fail the whole `addArray()` call.
3. **`next()` only affects what comes after**: it sets `_currentBinIndex = bins.length`, so earlier
   bins stop accepting new elements and every lookup starts at that index.
4. **Dirty propagation**: mutating a `Rectangle` property increments `_dirty`; `Bin.dirty` is true
   when its own `_dirty > 0` or any of its rects is dirty; `repack(quick=true)` only re-packs dirty
   bins. Changing setter semantics breaks incremental repack.
5. **save/load store free space only**: bins produced by `save()` always carry an empty `rects` array
   — dimensions/options/tag/freeRects only; `load()` restores free rectangles only. Already-placed
   rects have to be re-`add`ed by the caller (tests lock this behavior in — do not "helpfully" fill
   it in).
6. **Size accounting**: the initial freeRect is `maxWidth + padding - border*2`; placement probes with

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soimy/maxrects-packer](https://github.com/soimy/maxrects-packer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
