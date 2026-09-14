---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Kenat (ቀናት) is a TypeScript library for the Ethiopian calendar: bidirectional Ethiopian↔Gregorian date conversion, formatting (Amharic/English, Geez numerals), date arithmetic, calendar grid generation, and a full Bahire Hasab (ባሕረ ሃሳብ) implementation for computing movable religious holidays and fasting periods. ESM source (`"type": "module"`), no runtime dependencies. Written in TypeScript (`src/**/*.ts`) with strict mode on — types live inline in the source, not in a hand-maintained mirror. `tsup` bundles `src/index.ts` into `dist/` (ESM + CJS + a minified browser IIFE global) with `.d.ts`/`.d.cts` declarations; that `dist/` output is what's published to npm and what consumers actually import.

## Commands

- `npm test` — run the full Jest suite via `ts-jest` (`node --experimental-vm-modules ... jest`, required because the codebase is ESM)
- `npx jest tests/bahireHasab.test.ts` — run a single test file
- `npx jest -t "some test name"` — run tests matching a name pattern
- `npm run typecheck` — `tsc --noEmit`, the fast type-correctness check (no build output)
- `npm run build` — `tsup`, produces `dist/index.{js,cjs,global.js}` + `.d.ts`/`.d.cts`; run this before opening anything in `examples/`, which imports from `../../dist/index.js`, not `src/`
- `npm run docs` — generate API docs into `docs/` via `typedoc` (reads the TS source directly, not JSDoc comments)
- `npm run prepack` — `typecheck && build`, runs automatically before `npm pack`/`npm publish`
- Releasing: `npm run release:patch|minor|major` only bumps the version and pushes the tag (`git push --follow-tags`) — it does **not** publish. Pushing a `v*` tag triggers `.github/workflows/release.yml`, which re-typechecks/re-tests/re-builds and publishes with `npm publish --provenance`. Publishing from a local machine is not the supported flow anymore.

There is no lint script in the repo.

## Architecture

**Entry point:** `src/index.ts` re-exports everything consumers use — the `Kenat` class as default export, named utility exports (`toEC`, `toGC`, `toGeez`, holiday/fasting functions, `MonthGrid`, `Time`, constants), and `export type` re-exports of the public interfaces (`FormatOptions`, `MonthGridConfig`, `DiffBreakdown`, etc., defined alongside their owning module or in `src/types.ts` for cross-cutting domain types like `EthiopianDate`/`GregorianDate`/`Holiday`). When adding a new public function, class, or type, it must be wired into this file to be usable by library consumers.

**Core module dependency shape** (roughly bottom-up):
- `constants.ts` — static data: month/weekday names (English/Amharic), `HolidayTags`, `HolidayNames`, evangelist names, and the Bahire Hasab lookup tables (`movableHolidayTewsak`, `keyToTewsakMap`, `holidayInfo`, `movableHolidays`). Adding or changing a holiday starts here.
- `utils.ts` — low-level validation (`validateNumericInputs`, `validateEthiopianDateObject`) and pure date-math helpers (day-of-year, leap-year checks, weekday calculation) shared across the codebase.
- `conversions.ts` — the Ethiopian↔Gregorian conversion core (`toEC`, `toGC`, `toGCDate`, `fromDateToEC`), plus Hijri/Islamic-calendar conversion (`getHijriYear`, `hijriToGregorian`, via `Intl.DateTimeFormat` with the `islamic` calendar). All date validity checks happen here before conversion math runs.
- `dayArithmetic.ts` — `addDays`/`addMonths`/`addYears` and diff/`diffBreakdown` functions operating on plain `{year,month,day}` Ethiopian date objects (not `Kenat` instances).
- `bahireHasab.ts` — implements the traditional Bahire Hasab algorithm (`_calculateBahireHasabBase` is the single source of truth for `ameteAlem`, `medeb`, `wenber`, `metqi`, `ninevehDate`, etc.). Movable holiday dates are computed as offsets (`tewsak`) from the Nineveh date via `constants.ts`'s `movableHolidayTewsak` table, then converted to Gregorian for the output.
- `holidays.ts` — combines fixed-date holidays with `bahireHasab.ts`'s movable feasts to answer "holidays in year/month" queries; supports tag-based filtering (`HolidayTags`).
- `fasting.ts` — fasting period calculations (Lent, Ramadan, etc.) built on top of `bahireHasab.ts` and Hijri conversion.
- `formatting.ts` — pure formatting functions (standard, Geez/Amharic, with-weekday, ISO string) that take Ethiopian date objects and language/numeral options.
- `geezConverter.ts` — Arabic numeral ↔ Geez numeral conversion (`toGeez`, `toArabic`), used by formatting and directly exported.
- `Time.ts` — models Ethiopian 12-hour day/night time, with conversion to/from Gregorian 24-hour time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MelakuDemeke/kenat](https://github.com/MelakuDemeke/kenat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
