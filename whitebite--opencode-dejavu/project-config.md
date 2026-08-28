---
trigger: always_on
description: **Generated:** 2026-08-22 (refreshed 2026-08-23)
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-08-22 (refreshed 2026-08-23)
**Commit:** ec439cd+
**Branch:** main

## OVERVIEW

dejavu — OpenCode plugin ("memory prosthesis with teeth"): mechanically detects recurring tool-call failures and promotes them into enforced gates (3 failures across 2 distinct sessions). Remind first, hard-block on same-session repeat offense. TypeScript ESM, runs under Bun, ships as raw `.ts` (no build step).

## STRUCTURE

```
dejavu-opencode-plugin/
├── index.ts            # Plugin entry — exports Dejavu (Plugin factory) + all 4 hooks
├── src/
│   ├── patterns.ts     # Pure engine: signatures, normalization, secret scrub, detection, blocking policy
│   ├── store.ts        # GateStore/Stores: two-scope persistence, locks, promotion, TTL, migration, reconcile
│   └── validate.ts     # Invariant layer: strict gate parsing + mechanical repair (parse-don't-validate boundary)
├── test/smoke.ts       # Behavioral smoke test — plain bun script, no framework, temp-dir isolated
├── scripts/            # doctor.ts (pathology report), analyze.ts (store summary), migrate.ts (demote+scrub)
├── command/dejavu.md   # /dejavu slash-command definition (install → ~/.config/opencode/command/)
├── skills/dejavu/      # Companion agent-protocol skill (install → ~/.config/opencode/skills/)
└── .omo/, .codegraph/  # Tooling artifacts — not project code
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Gate enforcement (remind/block/override) | `index.ts` `tool.execute.before` | session state lives ON THE GATE (`remindedSessions`/`failedSessions`), read fresh under the store lock |
| Failure detection + recording | `index.ts` `tool.execute.after` + `event` | two channels: exit/text vs message stream |
| Signature/normalization | `src/patterns.ts` | `callSignature`, `normalizeCommand`, `parameterizeError` |
| Enforcement policy | `src/patterns.ts:canBlock()`/`canRemind()` | three tiers — bash non-diagnostics block, diagnostics remind-only, everything else just watches |
| Persistence, locks, promotion, global escalation | `src/store.ts` | `Stores.recordFailure()` is the core; cross-project evidence lives in global `index.json` |
| Self-healing / reconcile | `src/store.ts` + `src/validate.ts` | `Stores.reconcileAll()` at every init; `doctor --repair` on demand |
| Tunables | top of `index.ts` **and** `src/store.ts` | split: TTL/review caps in index, promote thresholds in store |
| Pathology checks | `scripts/doctor.ts` | 5 defect classes: stale-blocking, not-teaching, annoying, secrets, version drift |

## CODE MAP

| Symbol | Type | Location | Role |
|--------|------|----------|------|
| `Dejavu` | Plugin factory | index.ts:66 | entry point; wires 4 hooks, also `export default` |
| `GateSignal` | class | index.ts:37 | sentinel error — the ONLY error rethrown from hooks |
| `callSignature` | fn | src/patterns.ts:272 | stable call identity per tool (bash/read/edit/write/glob/grep) |
| `patternKey` | fn | src/patterns.ts:300 | sha1 prefix-12 of signature — the gate key |
| `canBlock` | fn | src/patterns.ts:184 | blocking policy gate |
| `canRemind` / `isRepoLocal` | fn | src/patterns.ts:197/215 | remind-only tier (diagnostics) / repo-local verbs that never escalate globally |
| `normalizeCommand` | fn | src/patterns.ts:80 | bash → signature; fingerprints interpreter one-liner payloads |
| `hashInterpreterPayload` | fn | src/patterns.ts:56 | `-c`/`-e` code payload → `<code:hash>` (identity of one-liners) |
| `fuzzySimilar` | fn | src/patterns.ts:349 | near-duplicate merge; length-band pre-filter + `FUZZY_MAX_LEN` cap |
| `detectFailure` | fn | src/patterns.ts:393 | line-by-line bash-output failure scan |
| `isNoiseError` | fn | src/patterns.ts:419 | aborted/cancelled executions are not failures |
| `scrubSecrets` | fn | src/patterns.ts:36 | redaction before ANY persistence |
| `GateStore` | class | src/store.ts:190 | one scope: gates.json + index.json + log.jsonl, TTL caches, key index |
| `Stores` | class | src/store.ts:513 | two-scope manager: findGate/recordFailure/migrate/expire/reconcileAll/forgetSession |
| `mergeGate` | fn | src/store.ts:483 | evidence merge for dedupe/escalation (never demotes blocking, preserves session state) |
| `coerceGateShape` | fn | src/validate.ts:24 | strict parse of a persisted gate record (hopeless → null) |
| `repairGate` | fn | src/validate.ts:76 | mechanical repair: inverted dates, truncation, re-scrub, demote, session-state hygiene |
| `atomicWrite` / `withLock` | fn | src/store.ts:120/151 | Windows-safe fs primitives |

## CONVENTIONS

- ESM (`"type": "module"`) + Bun runtime; scripts run directly (`bun scripts/x.ts`); no build, no bundling
- No semicolons, double quotes, explicit return types on everything, `node:` prefix on builtins
- No linter/formatter config exists — style is maintained by hand, match neighboring code
- JSDoc `/** */` on exports; inline `//` comments explain WHY (design rationale), not WHAT
- Catch blocks swallow deliberately with a rationale comment; only `GateSignal` is rethrown
- Tunables are named UPPER_SNAKE constants grouped under `// --- Section ---` dividers

## ANTI-PATTERNS (THIS PROJECT)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WhiteBite/opencode-dejavu](https://github.com/WhiteBite/opencode-dejavu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
