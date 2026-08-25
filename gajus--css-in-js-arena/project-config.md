---
trigger: always_on
description: Benchmark harness comparing **compile-time CSS engines**. One app per engine under `apps/`, all
---

# CLAUDE.md

Benchmark harness comparing **compile-time CSS engines**. One app per engine under `apps/`, all
rendering the same six-page console; `tools/` holds the parity and measurement harness.

Three documents, three audiences — keep them apart. `README.md` is for a reader deciding which engine
to use: results, methodology and caveats, no operational detail. `RUNNING.md` is for someone
reproducing the numbers: ports, commands, probe hygiene. This file is for the agent. Setup steps do
not belong in `README.md`; when the harness changes, `RUNNING.md` is what needs updating.

`README.md` runs answer-first: intro, engines, summary table, verdict, then `## Full results` (the
detail table), `## Where the main table doesn't generalise` (the scale and theming scenarios, which
exist because the main table is one app in one configuration), `## What's measured` (ground rules and
the page inventory), `## Reproducing this`, `## FAQ`. Methodology asides that answer an anticipated
question go in the FAQ, one `<details>` per question, not inline next to the tables. The verdict is
stated once, at the top — do not add a second summary after the detail table.

**`README.md` captures one point in time, never a trend.** No release-over-release narrative, no
"this used to be", no "an earlier revision said". A version bump rewrites the affected cells and the
prose around them; it does not append to a history. Findings about *how a version behaved* are
interesting during a run and go in the reply to the user — not into the document. If a number needs a
caveat, state the caveat as a present fact.

`apps/bamboo` is the **reference app**. Every other app is diffed against it, so all apps match each
other transitively.

## The one rule

**Every comparison run must end by updating the results tables in `README.md`.**

That table is the deliverable — the repo exists to keep it current. A run that produces numbers and
does not write them back is incomplete. When updating it:

- Replace the whole table, do not patch individual cells — partial updates mix numbers from
  different runs and silently drift.
- **Versions are stated once**, in the engine table at the top. Update them there and nowhere else:
  the stamp below it carries only the date and platform, and every other table labels its
  columns `Bamboo` / `StyleX` / `Panda` with no version. The one exception is the release-history
  prose, where a version number *is* the subject (`1.44.0 → 1.45.0`) — those stay.
- Every number must come from **one contiguous measurement session** on the same machine with
  nothing else heavy running. Never stitch a build time from one run into a byte count from another.
- One column per engine and nothing else. **No `Margin` column** — a percentage gap is derivable from
  the row and reading it off is the reader's job. Where a cell needs a caveat that is *not* derivable
  ("one unreferenced", "rest computed in the browser"), put it in that engine's own cell, short.
- Mark every winning cell with `🏆` and bold. Ties under ~2% spread get a trophy on each tied engine,
  or none at all if the axis is not a quality signal — for an unscored row, say why in the `Axis`
  cell in italics (`— *not a quality axis*`, `— *tie, spread 0.3%*`), since a row with no trophy
  anywhere is otherwise indistinguishable from an oversight.
- Put `🏆` on the overall winner's column header, and end the table with a `**Rows won**` tally
  row carrying the count of scored rows in its label. Recount it from the rendered table, do not
  carry it over.
- Keep the **summary table** (top of the document, under the stamp) in sync: one row per engine,
  one column per category (`Shipped bytes`, `Build & dev`, `Authoring`, `Correctness & maintenance`)
  plus the total, each as `won / scored`. Recount it from the rendered detail table too — the two
  disagreeing is the single easiest way for this file to start lying. Both counts have been wrong
  before, so verify by script rather than by eye.
- Say in the verdict that the tally is a scanning aid, not the judgement — axes are not equally
  weighted and some are unscored.

## Running the comparison

Run in this order. Steps 1–3 gate the rest: **if parity fails, the numbers are meaningless** and the
divergence has to be fixed before measuring anything.

### 1. Build every app

```bash
for d in apps/*; do (cd "$d" && npm run build && npm run typecheck); done
```

Every app must build with **0 engine warnings and 0 type errors**. For Bamboo specifically, a
`🎋 warn [utility]` line means a token does not resolve and the browser will drop the declaration —
a real bug even when screenshots look fine. Since 1.43.0 a misspelled token in a token category
fails the build outright.

### 2. Serve every app

Each on its assigned port (`300N` by engine index):

```bash
cd apps/bamboo && PORT=3001 npm start &
cd apps/stylex && PORT=3002 npm start &
cd apps/panda  && PORT=3003 npm start &
```

### 3. Verify parity — gate

```bash
cd tools
for r in / /projects /settings /pricing /docs /lab; do
  for c in stylex panda; do node layout-diff.mjs "$r" 2 "$c"; done
done
node compare.mjs
```

Note the fourth argument: `layout-diff.mjs` defaults to `stylex`, so a loop without it never
geometry-checks Panda at all.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gajus/css-in-js-arena](https://github.com/gajus/css-in-js-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
