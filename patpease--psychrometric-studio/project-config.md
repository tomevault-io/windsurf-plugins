---
trigger: always_on
description: A psychrometric chart that solves an air-handling chain, checks it against
---

# Psychrometric Studio

A psychrometric chart that solves an air-handling chain, checks it against
ASHRAE 55 comfort, and counts a year of weather against it. Everything runs in
the browser: no account, no upload, nothing kept. Live at
`psychrometric-studio.patpease0.workers.dev`; source is MIT.

It is a **Cloudflare Worker, not a Pages site.** They are different products and
the difference has broken a deploy: a `functions/` directory is ignored here,
and a route that should 404 instead returns the SPA shell with a 200. The Worker
entry point is `web/worker/index.ts`.

**Read this file first, then only what you need.** `PLAN.md` is 1,400 lines of
build history — a record of *why*, not an orientation. Do not read it whole.

## Layout

```
web/src/psych/       unit-aware state engine over vendored PsychroLib
web/src/chart/       scales, line families, SVG renderer, overlays
web/src/processes/   17 process models, chain solver, duty accounting
web/src/comfort/     ASHRAE 55 PMV/PPD, comfort polygon, adaptive model
web/src/weather/     EPW parsing, density binning, hours-in-zone
web/src/education/   equipment + concept content, live design checks, walkthrough
web/src/io/          project files, share links, CSV, SVG/PNG, report client
web/src/icons/       60 equipment SVGs + build-time generator
web/worker/          the Worker entry point and the weather relay route
api/                 FastAPI PDF report service. Optional; not deployed.
shared/schema/       project.schema.json — authoritative project file format
```

## The shape of a project

A project holds **operating cases** — normally two, a cooling one and a heating
one, turned between by the folded corner on the chart. This sits *above*
airstreams and the distinction matters: an airstream is a parallel duct within
one case and stage couplings resolve across them **by id, scoped per case**. Two
cases both having a `return` stream is ordinary; a coupling in one must never
resolve to the other's duct. `solveSystem` therefore takes one case, not a
project.

Shared across cases: units, site pressure, comfort, the weather file, and
`meta`. Per case: the chain, the chart view, the hour filter, and its own notes.
`meta` deliberately does not fork — a client name in two places is one that will
disagree with itself.

The schema is at version 2 and `MIGRATIONS` is no longer empty; a v1 file opens
as a single cooling case.

## Verifying a change

```bash
cd web && npm run typecheck && npm test && npm run build
```

**A green suite is not evidence the browser works.** This project has been
bitten by exactly that (see `docs/adr/0003-umd-interop.md`): tests passed while
the app failed to boot, because vitest did CJS interop that Vite's ESM pipeline
would not. For anything user-visible, open it. `npm run dev` serves on 5183.

**A green build is not evidence the deploy works either.** Two deploys have
failed with everything passing — once on the root directory, once on
Pages-versus-Workers. `npm run preview:worker` serves the built site and the
relay together in the real Workers runtime, and would have caught both.

**Measure performance on the production build, never the dev one.** React's
development build is several times slower. A page turn measured a 100 ms hitch
in dev and 18 ms in production; optimising against the dev number would have
been chasing something no user has. `npm run preview` serves the built app on
4183.

## Rules the tests enforce

1. **Never import `web/vendor/psychrolib.js` directly.** Go through
   `src/psych/psychrolib.ts` and `lib(units)`. Nothing may call
   `SetUnitSystem` — there are two pinned instances (ADR 0002).
2. **Store canonical, convert at the edge.** Humidity ratio in lb/lb or kg/kg;
   enthalpy in Btu/lb or **J/kg**. Display conversion lives in `units.ts`.
3. **Do not assert precision finer than `CONVERGENCE_TOLERANCE`.** Wet bulb is
   iterative and good to ±0.001 °C.
4. **Enthalpy is not comparable across unit systems.** IP measures from 0 °F,
   SI from 0 °C. Only *differences* convert.
5. **The API lays out; it never calculates.** Every number in a report is solved
   in the browser and sent ready to typeset.
6. **A design check must stay silent on a good design**, in both unit systems.
   Thresholds are declared in kelvin and converted; never compare a Fahrenheit
   delta against a Celsius limit.

## Things that look right and are not

Each of these shipped, compiled, and passed tests before being caught. They are
the failure shapes this codebase produces.

- **Reading computed styles from the source element instead of the export
  clone.** Produces a valid SVG in the wrong theme. See ADR 0004.
- **Stripping a parent's class before reading its children.** Kills every
  descendant CSS rule below it; the comfort zone exported as a solid black box.
- **Effect cleanup that clears state the crash screen needs.** React unmounts
  the tree when an error boundary catches, so `setRescue(null)` on unmount wipes
  the rescue exactly when it is wanted. `web/src/io/rescue.ts`.
- **`JSON.stringify(NaN)` prints `null`.** jsthermalcomfort returns NaN, not
  null, out of range. A `=== null` guard never fires and "NaN °F" reaches the
  UI. Test finiteness.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patpease/psychrometric-studio](https://github.com/patpease/psychrometric-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
