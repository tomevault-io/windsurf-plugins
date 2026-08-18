---
trigger: always_on
description: Standalone DeepSeek Harness plugin repository (`dsh-permission-rules`). Development follows the dsh-plugin-guide skill and the official plugin contract; this file records repo-local decisions.
---

# AGENTS.md

Standalone DeepSeek Harness plugin repository (`dsh-permission-rules`). Development follows the dsh-plugin-guide skill and the official plugin contract; this file records repo-local decisions.

## Layout

- `src/index.ts` — function-plugin contract (`name`/`inject`/`Config`/`apply`; NO default export — the Loader unwraps `exports.default ?? exports`). Injects `commands` + `tools`.
- `src/config.ts` — Schemastery schema + explicit `resolveConfig` (no hidden `?? default` in `run()` paths); closed enums and boolean flags are validated at resolution so plain-JS mounts fail loud too.
- `src/glob.ts` — strict glob→RegExp compiler + the backtracking guards: `maxGlobStars` caps unbounded star expansions, and regex-mode patterns reject nested unbounded quantifiers and quantified overlapping literal alternations. Bad patterns throw at compile time (load), never silently match nothing.
- `src/rules.ts` — the pure core: YAML document validation, pattern compilation (incl. `!pattern` negation), first-match evaluation, the `agents` identity dimension (`main`/`subagent`/`preset:<name>` selectors against session-header candidates; unknown identity never matches — fail closed), nested path-candidate extraction, `when`/`absent` dimensions, chain merging (`compileRulesChain`), shadow detection (`findUnreachableRules`). No fs/clock/process state.
- `src/prose.ts` — `/rules` output vocabulary in five languages (`en`/`zh` reference, `es`/`pt`/`hi` community) + `describeRule` dimension tokens (incl. the `src` source-attribution token). Rule `reason`s are never translated.
- `src/events.ts` — `permissionRules/decision` SessionEventMap member (declaration merging, incl. the `outcome` field) + `AuditAppend`, the append surface that requests the envelope's `ignorable: true` marker.
- `src/runtime.ts` — `tools/pre-execute` listener, per-cwd rule-chain loading (project chain by cwd / `searchUp` walk → fallback → empty), `permissionRules/decision` audit, `/rules` command (`list | reload | decisions [n] | test [flags incl. --platform]`), Chokidar watch (LRU cache with resolved/case-folded keys, watcher reconciliation, timer cleanup, candidate watches on expected-but-absent rule files so mid-session creation is adopted). Registers itself as `ctx.permissionRulesRuntime`.
- `test/` — vitest; real `Context` + real `Session`/`Commands`/`ApprovalService` from the `0.1.0-rc.6` peers; chokidar mocked with a fake EventEmitter; the dsh-auto-review integration uses its tarball with a scripted reviewer mock.
- `docs/rules-format.md` (+ `.en.md`) — the rule file schema and the 5-rule security baseline; `docs/rules-format.schema.json` is the machine-readable schema for editor completion.
- `scripts/repair-session-logs.mjs` — one-off repair for session logs written before the marker: rewrites targeted audit rows to carry `ignorable: true` (frame-preserving zstd rewrite, backups, `scan`/`repair`/`--dry-run` modes).
- `scripts/check-readme-sync.mjs` — five-language README sync gate (section structure, config-table keys, `/rules` command docs); wired into CI.
- `.github/ISSUE_TEMPLATE/*` + `.github/PULL_REQUEST_TEMPLATE.md` + `SECURITY.md` — structured issue forms (bug/feature), PR checklist, and the private vulnerability-reporting policy.

## Hard rules applied here

- Waterfall listener (`tools/pre-execute`) always calls `next()` unless it claims the call with `deny`/`ask`. An `allow` hit is NEVER short-circuited. Under `enforce: false` (dry-run) even deny/ask hits delegate — the record keeps the would-be action with `dryRun: true` plus the real downstream `outcome`.
- Model-visible ⟺ logged: the only model-visible plugin content is the deny/ask reason materialized by the tools registry into the tool result; the `permissionRules/decision` audit event carries the same `callId` and reason for reconstruction, and its `outcome` records the FINAL pre-execute decision (an allow hit followed by a downstream deny is logged as denied).
- Log-only audit: `permissionRules/decision` is never injected into the model context, and is appended with `{ ignorable: true }` via the `AuditAppend` surface. Post-rc.6 hosts stamp the marker; the `0.1.0-rc.6` line silently drops it, so the runtime detects unmarked hosts BEFORE the first append (peer-version pre-check, then a probe of the appended envelope's return value) and disables session-log audit with a one-time warning — `allowUnmarkedAudit: true` opts back in, and `scripts/repair-session-logs.mjs` repairs already-polluted logs.
- Loud misconfiguration: invalid YAML, unknown fields/actions, bad globs/regexes, backtracking-prone patterns, and rule counts over `maxRules` fail the load (`badFilePolicy` chooses fail vs ignore-with-warning). Deployment-level files (absolute `rulesFile`, `fallbackPath`) fail the mount. `searchUp` + absolute `rulesFile` fails `resolveConfig`.
- Backtracking bounds: a compiled glob's degree equals its star count — `maxGlobStars` (default 2) caps it exactly; regex mode rejects nested unbounded quantifiers and quantified overlapping literal alternations, while independent quantifier chains stay allowed (documented escape hatch).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PerryLink/dsh-permission-rules](https://github.com/PerryLink/dsh-permission-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
