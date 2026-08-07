---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git policy

**Never commit or push without being explicitly asked.** Finish the work, leave all changes uncommitted in the working tree, and summarize what changed so it can be reviewed first. Plan or task approval is not commit authorization. If a commit split would help, propose the split and messages and wait for the go-ahead.

## Commands

```bash
pnpm install        # install deps
pnpm build          # compile src/ → dist/ (tsup bundles cli.ts + plugin/index.ts)
pnpm dev            # watch mode
pnpm test           # full check: test:unit + check:links + check:surface — network-bound
pnpm test:unit      # vitest run only (builds once via globalSetup) — fast, offline, for the inner loop
pnpm check:links    # validate every docs URL in src/providers (404s, soft 404s, stale redirects)
pnpm check:surface  # diff surface.methods manifests against the latest SDK type declarations (drift guard)
```

**Run `pnpm test` before committing.** The two network-bound guards catch what the
unit suite structurally cannot, because both compare the repo against the outside
world: a provider moving a docs page (`check:links`), and an SDK growing or
renaming methods the surface manifest doesn't list (`check:surface`). Both have
already caught real problems. Nothing enforces this automatically — use
`pnpm test:unit` while iterating and the full `pnpm test` before you commit.

Run a single rule's tests (requires a prior build):

```bash
pnpm build && npx vitest run tests/rules/resend-missing-idempotency-key.test.ts
```

Smoke-test against a fixture directory:

```bash
node dist/cli.mjs tests/fixtures/resend/resend-api-key-hardcoded-broken
```

## Architecture

Two outputs from a single build (`tsup.config.ts`):

| Output | Entry | Description |
|--------|-------|-------------|
| `dist/cli.{mjs,cjs}` | `src/cli.ts` | CLI binary (`api-doctor` bin) |
| `dist/plugin.js` | `src/plugin/index.ts` | Oxlint JS plugin (consumed by the CLI and directly by users) |

### Python engine — present but dormant

Python rule sources live under `src/providers/*/rules/python/` with a stdlib runtime in
`src/engines/python/runtime/`, **but the shipped product is TypeScript-only.** Every call
site that could classify, walk, detect, or lint a `.py` file is commented out behind a
`PYTHON-DORMANT` marker:

```bash
grep -rn PYTHON-DORMANT src tests    # every switch, in one command
```

The master switch is the `.py` branch in `src/engines/classify.ts` — while it is off,
`.py` files are never walked, read, classified, or linted. `src/detector.ts` needs its
own switch because pyproject/requirements detection reads disk directly and does not go
through file classification. `src/scanner.ts` does not import the Python runner at all,
so `dist/` contains no code that can spawn a Python process, and `package.json` `files`
ships no `.py` at all.

Rules for keeping it dormant:

- **Never re-enable one site alone** — the switches are a set; flip them together.
- Python rule tests (`tests/rules/*-python-rules.test.ts`) drive the runtime directly via
  `lintPythonFixture` and bypass `scan()`, so they keep running and keep the rule pack
  under test. Only `tests/scanner-python.test.ts` (end-to-end through `scan()`) is skipped.
- Do not add `src/providers` or `src/engines/python/runtime` to `package.json` `files`:
  an explicit `files` entry force-includes everything beneath it, which `.gitignore`
  cannot override — that leaks local `__pycache__/*.pyc` and every provider `.ts` source
  into the published tarball.
- When Python does ship: a repo containing `.py` files but no `python3` on PATH must
  degrade to a JS-only report, never abort the whole scan (today `runPythonEngine`
  throws `ScanError` → exit 2, discarding valid JS findings).

### Source layout

```
src/
├── cli.ts              Entry point — parses flags, runs scan, emits output, exits
├── scanner.ts          Walks files, classifies language, fans out to engines
├── detector.ts         package.json / pyproject / import / URL heuristics
├── types.ts            Shared contracts (ScanResult, Report, Finding, manifests)
├── engines/
│   ├── classify.ts     Per-file language (javascript | python)
│   ├── js/runner.ts    Oxlint engine
│   └── python/         Node runner + stdlib-ast runtime/
├── reporter/           Terminal, JSON, markdown, and file output
├── coverage/           SDK surface coverage (CLI-only; JS/TS; never scored)
│   └── collect.ts      oxc-parser pass over provider files → used method paths
├── plugin/
│   ├── index.ts        Oxlint plugin — imports providers/*/rules/js/
│   └── rule-registry.ts  Reads meta.docs from each JS rule
└── providers/
    ├── index.ts        Registers all provider manifests
    └── <name>/
        ├── manifest.ts   Detection + rules[] (+ optional surface)
        ├── utils.ts / utils.py
        ├── rules/js/     Oxlint / ESTree rules
        ├── rules/python/ stdlib-ast rules (same rule keys)
        └── README.md
```

### Data flow

```
cli.ts
  └─ scan()                    scanner.ts
       ├─ classify each file   engines/classify.ts
       ├─ detectProviders()    detector.ts + manifests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qualtyco/api-doctor](https://github.com/qualtyco/api-doctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
