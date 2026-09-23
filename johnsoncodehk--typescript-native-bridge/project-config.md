---
trigger: always_on
description: **One path. No fallback. No ceremony.**
---

# AGENTS.md

## Philosophy

**One path. No fallback. No ceremony.**

- **One path** — no mode flags or opt-out env vars. A fallback nobody sets is an untested code path that rots while the default gets all the coverage. If two behaviors are both defensible, pick one and delete the other; verify it with the gates, not with a switch.
- **No fallback** — do not degrade around hypothetical edges. Enumerate real edges and kill them by construction (idempotency, mechanism guarantees). If a failure mode cannot occur on the supported floor (Node ≥ 20, Go 1.26, current V8), do not code for it — fail loudly so real problems surface instead of hiding behind a silent degraded mode.
- **No ceremony** — if the language/runtime contract already provides the behavior (e.g. returning a NULL-initialized value propagates a pending napi exception), do not restate it in code. Comments explain *why* (ownership, contracts, invariants), never *what the next line does*.

Reference implementation of the style: `patches/typescript-go/overlay/bridge/napi_shim.c`.

### Convergence rules (so the tree never needs a sweep)

- **Env knobs need a committed consumer** — otherwise they're deleted on sight. One name per knob.
- **No debug scaffolding in product code** — instrumentation and its witness live and die together.
- **The pinned bundle is the floor** — call its API directly; no presence guards or compat branches. A mismatch must throw.
- **Abandoned approaches die in the same commit** — dead code is part of the pivot's diff.
- **Triage scripts are disposable** — wired gate, exit-coded probe, or deleted when the issue closes.
- **One harness, parameterized** — never fork a script to vary it.
- **Mechanism changes sweep their vocabulary** — grep the old name across patches/, tools/, comments in the same change.
- **Deletions are verified by the gates, not switches.**
- **No framework hardcoding** — behavior keys on registered mechanisms (`extraFileExtensions` / `supportedTSExtensionsFlat`), never on literal framework suffixes or names (`.vue`, `vue`, `svelte`…). If a fix needs a framework literal, the mechanism is what's missing.
- **One Go→JS file-name boundary** — `wireFileNameToHost` (tsgoLibPaths.ts) is the only normalization point for wire-decoded file names; every new transport/payload must funnel through it at its decode point, call sites never re-normalize. (Key folding is `canonicalSourceFilePath`; JS→Go input is `resolveHostFileName`.)
- **Review the landed diff for elegance before committing** — top smells: duplicated truth (call the source instead), caches that hide the problem, hidden contracts. Don't defend the first draft.

## Repo in one paragraph

TNB is a tsgo-backed TypeScript fork: upstream `microsoft/TypeScript` and `microsoft/typescript-go` pinned as submodules, a small patch set on top (`patches/`), and a NAPI bridge (`bridge.node`) that runs the tsgo checker in-process. Sources of truth: `patches/typescript/overlay` + `patches/typescript/*.patch` (edit via the submodule then `npm run save-ts-patches`), `patches/typescript-go/overlay` + `*.patch` (`npm run save-patches`). Build: `npm run build:lib` (fork bundles) and `npm run build:bridge` (NAPI addon).

## Gates (run before committing behavior changes)

- `npm run check:lib` / `check:enums` / `check:sourcefile-guard`
- `npm run check:go-as-guards` — Go-side Type-cast guard: every `Type.As*()` chain deref and unguarded nil-family assignment in the tsgo patches/overlay is fixed or carries an inline `// asguard:exempt` reason (issues #69/#70 bug class)
- Witnesses: 85 across wg0–wg6, all wired in CI (`.github/workflows/ci.yml`), single source of truth `tools/ci-witness-groups.mjs` — `node tools/ci-witness-groups.mjs all` validates dup/missing/orphan/local-only/baseline wiring and emits the matrix (`matrix` mode feeds the ci.yml prepare job).
- Local-only — run on demand, not in the matrix (reasons in the matrix header comment): framework-checks, external-edits, generation-retention, napi-fuzz, completion-latency, postedit-latency, perf-edit-rpc, perf-qi-rpc, typing-cpuprof.
- Locally hostable gates: `check:sim-nav` and `check:sourcefile-guard` need a volar checkout. ci.yml's recipe, which is the one to copy (`VOLAR_SHA` is the ci.yml env pin — set it in your shell, a shallow clone does not carry the tag):
  ```sh
  export VOLAR_SHA=<.github/workflows/ci.yml value>
  git clone --depth 50 --branch master https://github.com/vuejs/language-tools.git /tmp/volar
  git -C /tmp/volar checkout "$VOLAR_SHA" || { git -C /tmp/volar fetch --deepen 50 origin master; git -C /tmp/volar checkout "$VOLAR_SHA"; }
  # point /tmp/volar/pnpm-workspace.yaml's typescript override at this repo (link:<repo>)
  cd /tmp/volar && corepack enable && pnpm install --no-frozen-lockfile && pnpm run build
  cd <repo> && VOLAR_ROOT=/tmp/volar STOCK_TSSERVER_PATH=/tmp/stock-ts-p3/package/lib/tsserver.js npm run check:sim-nav
  ```
  `check:sourcefile-guard` needs only `VOLAR_ROOT`. Both were run in full for #72/#73. `check-pristine-attribution` clones pristine tsgo into `/tmp/tnb-pristine-tsgo` and refuses to run only if one of its own repro files is already present there — a stray file of another name goes unnoticed, so check `git -C /tmp/tnb-pristine-tsgo status --porcelain` yourself.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johnsoncodehk/typescript-native-bridge](https://github.com/johnsoncodehk/typescript-native-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
