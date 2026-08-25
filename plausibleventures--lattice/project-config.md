---
trigger: always_on
description: **Lattice is a kit for building isometric, deterministic, zero-asset games in TypeScript.**
---

# Lattice — how to work in this repo

**Lattice is a kit for building isometric, deterministic, zero-asset games in TypeScript.**
Nine small libraries that compose, and a real game built from nothing but them.

This file is the constitution. It is written for agents first and humans second, because
agents outnumber humans here. Read it before you touch anything. It is short on purpose.

- **What each package is** → `.lattice/kit.json` (machine-readable), `README.md` § The nine packages,
  and each package's own `README.md`. The cross-package contracts are in `docs/SEAMS.md`
- **What to work on next** → `.lattice/tasks.json`
- **Where the build is** → `.lattice/state.json`
- **How one work cycle runs** → `docs/LOOP.md`

---

## The eleven non-negotiables

These are not preferences. A change that breaks one of these is reverted, not debated.

1. **Determinism is a feature, not an accident.** `Math.random()`, `Date.now()`, and
   `performance.now()` are banned inside every package's `src/`. Randomness comes from a
   seeded `Rng` passed in by the caller; time arrives as a parameter. This is enforced by
   `npm run lint`. A game built on Lattice must be able to replay a session from a seed and
   an input log and land on the same pixel.

   **And it has two tiers, because the language only promises so much.** ECMA-262 specifies
   `+ - * /`, `Math.sqrt`, `Math.imul` and the bitwise operators exactly. It explicitly does
   *not* require `sin`, `cos`, `pow`, `exp` or `log` to be correctly rounded, so two
   conforming engines may disagree in the last bit.

   | | arithmetic | promise | may reach |
   |---|---|---|---|
   | **Tier A** | `+ - * /`, `sqrt`, `imul`, bitwise | bit-identical on every engine | hashes, save files, replays, anything |
   | **Tier B** | `sin`, `cos`, `pow`, `exp`, `log`, … | correct to within an ulp or so | pixels only — never hashed, never persisted |

   Tier B is not banned; a cost curve is `b · r^k` and there is no honest way around that.
   It is required to **declare itself**: mark the site `@tier-b` and the linter is satisfied.
   That makes every one of them greppable, so an auditor can ask of each in turn whether it
   ever reaches a save file.

2. **`@latticekit/core` has no dependencies — and neither does anything else.** Not on npm, not
   on each other except along the layering below, not on the DOM unless the package name
   says so. The entire kit installs in one `npm i` with nothing transitive.

3. **The dependency graph is a DAG, and it points one way.**

   ```
   core ─┬─▶ iso ──┬─▶ draw ─┬─▶ ui
         ├─▶ loop  │         │
         ├─▶ sim   └─────────┤
         ├─▶ persist         │
         ├─▶ input ──────────┘
         └─▶ audio
   ```

   `core` imports nothing. Nothing imports `ui`. If you need an upward import, you have
   found a design error — say so in the task, do not add the edge.

4. **Pure where it can be, impure where it must be, and the two never mix in one file.**
   A module that touches `window`, `document`, `AudioContext` or `localStorage` says so in
   its first doc line. Everything else must run unchanged in Node with no shims.

5. **Every public symbol is documented with a `why`, not a `what`.** `/** Sets the zoom. */`
   on a method called `setZoom` is worse than no comment: it costs a line and teaches
   nothing. Say what breaks if you get it wrong. The prose in this kit is a load-bearing
   part of the product — an agent reading `camera.ts` should learn why pointer-anchored
   zoom exists, not just that it does.

6. **No public API without a test that would fail if it were deleted.** Coverage targets are
   in `.lattice/kit.json`; the current floor is 90% statements per package, 100% on
   anything in `core`. Tests are behavioural. A test that asserts an implementation detail
   is a future false alarm.

7. **The hot path allocates nothing.** Anything called per-frame or per-entity takes an
   output parameter or returns a primitive. `{ x, y }` returned sixty times a second times
   four hundred sprites is a garbage collector pause with a nice API. Benchmarks live in
   `*.bench.ts` and the numbers are in `docs/PERFORMANCE.md`.

8. **Zero assets.** No images, no audio files, no fonts, no binaries anywhere in a package.
   Art is procedural (`draw`), sound is synthesised (`audio`). This is what makes a Lattice
   game a few dozen kilobytes, recolourable at runtime, and diffable in review.

9. **Errors name the caller's mistake.** `throw new RangeError('camera.zoom: expected a
   finite number > 0, got -1')`. Never a bare `Error`, never a message that only makes
   sense with the source open beside it.

10. **Green is not evidence.** Every UX-affecting change ends with the demo game actually
    running (`npm run dev`) and someone — human or agent — looking at it. The kit is judged
    by whether a game can be built from it, not by whether its suite passes.

11. **An option a caller supplied is a value they can read back.** Every field of every
    `*Options` object is readable off the object it configured. No exceptions — a getter over
    private state costs three lines, has no policy attached, and cannot break an invariant.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [plausibleventures/lattice](https://github.com/plausibleventures/lattice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
