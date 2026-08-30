---
trigger: always_on
description: Indonesian Throughflow visualisation: particle advection over NASA OSCAR surface currents, with bathymetry as the mechanism and mooring-derived transports at the gates. Static site, GitHub Pages, no backend, no runtime network.
---

# CLAUDE.md — Arus Lintas

Indonesian Throughflow visualisation: particle advection over NASA OSCAR surface currents, with bathymetry as the mechanism and mooring-derived transports at the gates. Static site, GitHub Pages, no backend, no runtime network.

Read `PRD.md` before starting any task, and **`DESIGN.md` before writing any UI** — it is authoritative for visuals and it opens with the shared house layer used across these projects.

**Four things shape everything:**

1. **The chokepoints are below the grid.** OSCAR is 0.25° — roughly 27 km. Labani is 45 km wide, Lombok 35 km. The most important passages are one or two cells across, and published assessments show low-resolution products underestimate net flow there specifically. **Every gate narrower than two cells renders with a visible under-resolved flag**, and the numeric claims come from moorings, not from the grid.
2. **OSCAR is derived, and surface only.** Computed from altimetry, winds and SST through a simplified dynamical model, averaged over the top 30 m. Never present it as measurement, never as the full-depth throughflow.
3. **Particles never cross land.** A particle traversing an island is both a bug and a physical impossibility. Land is masked from advection; culls are matched by respawns.
4. **The advection is the product.** One orchestrated moment, per the house layer. Nothing else animates.

---

## Stack

- Next.js 14, App Router, `output: 'export'` — static only
- TypeScript, `strict: true`
- Tailwind CSS, tokens from `DESIGN.md`
- WebGL2 for particle state, canvas fallback
- Zod for manifest validation
- Vitest
- pnpm
- **No mapping library with a tile dependency, no charting library, no particle library.** The advection is the project.
- Fonts via `next/font`, self-hosted.

## Commands

```bash
pnpm dev
pnpm build                  # static export; runs data:validate first
pnpm preview                # serve ./out under the production basePath
pnpm test                   # vitest watch
pnpm test:run               # vitest once — before every commit
pnpm test:field             # decode integrity, sampling, quantisation round-trip
pnpm test:advect            # domain containment, land masking, population, determinism
pnpm test:gates             # citation completeness, under-resolved flags
pnpm data:fetch             # DEV/CI — pull OSCAR + bathymetry
pnpm data:build             # crop, quantise, PNG-pack, emit manifest
pnpm data:validate          # manifest, DOI, version, size budget
pnpm bench:particles        # particle budget + frame rate on a mid-range profile
pnpm typecheck
pnpm lint
```

`pnpm test:advect`, `pnpm test:gates` and `pnpm data:validate` gate the build and CI.

## Layout

```
app/
  [locale]/                 # id (default), en
    arus/                   # the field + controls + stops
    gerbang/[slug]/         # gate detail + section
    metode/                 # dataset, DOI, limitations, citations
components/
  field/                    # bathymetry + particle canvas
  dial/                     # month dial — cyclical
  enso/                     # three-state switch — categorical
  gate/                     # marker, flag, transport panel
  section/                  # cross-channel depth profile
  legend/                   # the honesty contract
  table/                    # text equivalent of the map
lib/
  field/                    # THE CORE. Pure. Runs in Node.
    decode.ts               # PNG-packed Uint8 → u,v via stated scale
    sample.ts               # bilinear interpolation
    mask.ts                 # land mask lookup
    streamline.ts           # integration, used for reduced-motion fallback
  advect/                   # particle state, seeding, culling, respawn
  gates/                    # passage definitions, mooring figures, citations
scripts/
  build-data.ts             # DEV/CI — netCDF + bathymetry → packed layers
data/
  field/                    # packed u/v layers, month × ENSO, + manifest
  bathymetry/               # packed depth grid + land mask
  gates/                    # five passages: width, sill, transport, citation
tests/
  field/  advect/  gates/
```

## Invariants

1. **`lib/field` is pure.** Typed arrays in, values out. No DOM, no React, no clock, no network, no module-level mutable state. Testable in Node.

2. **Land is masked from advection.** Every advection step checks the mask. A particle entering land is culled and respawned. **Never let a trail cross an island** — asserted by test.

3. **Particle population is conserved.** Every cull is matched by a respawn. Population drift is a bug.

4. **Advection is deterministic given a seed.** Seeded PRNG carried in state, never `Math.random`. Same seed, identical trajectory set.

5. **Particles never leave the domain.** Boundary handling is cull-and-respawn, never clamp — clamping produces particles piling up at the edge, which reads as a current that does not exist.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andifathulms/indonesian-throughflow](https://github.com/andifathulms/indonesian-throughflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
