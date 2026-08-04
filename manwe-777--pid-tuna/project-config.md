---
trigger: always_on
description: Practical guide for anyone (human or AI) making changes to this repo. Keep
---

# Contributing to PIDTuna

Practical guide for anyone (human or AI) making changes to this repo. Keep
changes small, run the checks below before pushing, and follow the patterns
that already exist rather than inventing parallel ones.

## What this is

A fully client-side React + Vite app that parses Betaflight blackbox logs and
runs DSP on them. No backend, no auth, no database. Ships in three forms from
the same source: hosted web app (GitHub Pages), installable PWA (offline via
service worker), and a Tauri 2 desktop wrapper. See `README.md` for the user-
facing overview.

## Layout

```
src/
  App.tsx              the whole UI — section components, tab routing
  components/          small reusable components (TimeSeriesPlot, LogDotPlot, etc.)
  dsp/                 pure analysis code, one file per concern
    stepResponse.ts    Wiener deconvolution
    spectrogram.ts     PSDs / spectrograms (exports combinePsds for pooling)
    motorBalance.ts    motor stats + per-motor PSDs (exports combineMotorBalances)
    scorecard.ts       triage roll-up (uses combine* helpers when merging)
    latency.ts, battery.ts, gps*.ts, filters.ts, saturation.ts
  parser.ts            blackbox-log adapter, builds ParsedLog incl. `presence` flags
  sliceLog.ts          time-range slice of a ParsedLog
  logs.ts              LogSlot type, slot ID generator, multi-log colour palette
  types.ts             ParsedLog, DataPresence, axis aliases
  tuneView.ts          extracts PID config from the parsed setup headers
src-tauri/             Tauri 2 desktop wrapper (Rust + tauri.conf.json)
public/                static assets (icons, wordmark) — served at site root
.github/workflows/     ci.yml, build.yml (desktop releases), pages.yml
```

## Running the app

```bash
pnpm install
pnpm dev               # web dev server on :5173
pnpm tauri:dev         # desktop dev (Rust compile on first run ~2 min)
```

## Checks before pushing

```bash
pnpm typecheck         # tsc -b --noEmit, must be clean
pnpm build             # full Vite production build, must succeed
```

There is **no automated test suite right now**. Don't add one casually — if you
do, pick `vitest` (Vite-native, jsdom env) and wire it into `ci.yml` alongside
typecheck. For DSP iteration we write throwaway Node `.mjs` harnesses (see git
history for `test-wiener.mjs` shape) — write them under the project root, run
against a real log on `~/Desktop/LOG*.BFL`, then `rm` when done. Don't commit
them.

For UI changes, exercise the path in both `pnpm dev` (regular browser) and
`pnpm tauri:dev` (WKWebView). Behaviour can diverge — the file picker is the
classic example.

## Versioning (semver)

`MAJOR.MINOR.PATCH`. Bump rules for this surface area:

- **MAJOR** — log-schema changes that break older saved analyses, breaking
  changes to any exported `dsp/*` function signature, scorecard rubric
  thresholds being moved enough to flip an existing log's status.
- **MINOR** — new tabs, new merge primitives, new DSP analyses, additive UI.
- **PATCH** — bug fixes, internal cleanups, dependency bumps, docs.

Three files must be bumped together at release time:

1. `package.json` → drives the web UI's header pill and Tauri's bundle version
2. `src-tauri/Cargo.toml` → the Rust crate's own version (kept in sync manually)
3. `src-tauri/Cargo.lock` → contains the crate version too (rebuild or hand-edit)

Then `git tag v<version> && git push origin v<version>` triggers `build.yml`,
which produces a draft GitHub Release with cross-platform installers attached.
The `pages.yml` workflow auto-redeploys the hosted web app on every push to
`main` independently.

## Patterns to follow

These exist because we've already needed them — match them when extending.

### Per-log `presence` flags drive tab availability

`ParsedLog.presence: DataPresence` (defined in `types.ts`, populated in
`parser.ts`) tracks which optional fields were actually in the source log
versus zero-filled by the parser. The sidebar / tab bar uses
`tabHasData(tabId, entries)` in `App.tsx` to dim and disable tabs whose data
is missing from every loaded log.

If you add a new analysis that depends on a new optional log field:

1. Add a flag to `DataPresence` in `types.ts`.
2. Populate it in `parser.ts` based on `cols.has(...)` or a sawX-style flag.
3. Add the tab → flag mapping in `tabHasData()`.

Don't check for "all zeros in the array" at the call site — the presence flag
is the source of truth.

### Multi-log views ship with a "Merge logs" toggle

Four tabs already do this: **Scorecard**, **Step response**, **Full spectrum**,
**Motors balance**. Each uses a matching combine primitive from the dsp module:

| Tab | Primitive |
|---|---|
| Step response | `combineStepResponses` (`dsp/stepResponse.ts`) — pools accepted segments |
| Full spectrum | `combinePsds` (`dsp/spectrogram.ts`) — linear-power weighted Welch combine |
| Motors balance | `combineMotorBalances` (`dsp/motorBalance.ts`) — pools stats + PSDs, re-derives axes |
| Scorecard | `buildMergedScorecard` (`dsp/scorecard.ts`) — composes the three above |

UI uses the shared `<MergeLogsToggle>` and `makeMergedSlot(entries)` helpers
in `App.tsx`. When adding another mergeable tab, write a `combineXxx()` in the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Manwe-777/pid-tuna](https://github.com/Manwe-777/pid-tuna) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
