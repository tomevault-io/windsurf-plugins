---
trigger: always_on
description: Zotero plugin: watches a folder, imports PDFs, mirrors a Zotero library to disk, and (Mode 3) syncs deletions two-way. Targets Zotero 7/8/9 (`strict_min_version 6.999`, live-verified on 9.0.4). Plugin ID `watch-folder@zotero-plugin.org`. Per-release detail lives in git history + `.private/docs/RELEASE_HISTORY.md` — do NOT grow this file with changelogs.
---

# CLAUDE.md

Zotero plugin: watches a folder, imports PDFs, mirrors a Zotero library to disk, and (Mode 3) syncs deletions two-way. Targets Zotero 7/8/9 (`strict_min_version 6.999`, live-verified on 9.0.4). Plugin ID `watch-folder@zotero-plugin.org`. Per-release detail lives in git history + `.private/docs/RELEASE_HISTORY.md` — do NOT grow this file with changelogs.

# CRITICAL RULES — MUST FOLLOW

## RESPONSES
- Be concise and direct. Lead with the answer; skip preamble and option-surveys unless asked.
- Report outcomes faithfully: failing tests, skipped steps, unverified work — say so plainly.
- This is a data-safety-critical plugin (it can trash a real Zotero library + the user's files). When a choice risks live data, prefer the safe default and say what you chose; surface genuinely irreversible decisions before acting.

## PLANNING MODE
- For non-trivial work, scout first (list files, find the call sites, scope the diff), then plan; don't assume the design or invariants — read the "DON'T TOUCH" section below.
- Use read-only sub-agents (Explore/Plan) to research breadth and to adversarially review a plan or a risky diff before acting. For data-safety changes, run an adversarial multi-agent review (find → verify → synthesize) before shipping.
- Green-preserving staging: when a change is large, build the new path ALONGSIDE the old (gated by a pref/flag, suite stays green), then flip the default at the end.

## EDIT / CHANGE MODE
- **Bundle trap (the #1 footgun):** editing a `.mjs` does NOT change what Zotero runs. Always `npm run bundle && npm run build`, then reload (`zotero_plugin_reload` or reinstall the `.xpi`). The dev order is bundle→build; `npm run release` runs build→bundle internally — don't hand-run them backwards.
- **No linter exists** — the esbuild bundle IS the syntax/import-cycle check. A clean `npm run bundle` is the lint gate. Be deliberate.
- Version lives in `package.json` AND `manifest.json` — keep them identical. `dist/manifest.json` is regenerated. `update.json` is served from `main` and is NOT auto-uploaded — commit + push it.
- **Zotero major-version compatibility (`strict_max_version`):** `manifest.json` AND the served `update.json` both carry `strict_max_version` (currently `9.*`); `build/package.mjs` mirrors it from the manifest into `update.json` so they never drift. The `update.json` copy is the lever Zotero's background compat check reads to keep an installed copy enabled, so it must NOT be omitted (omitted = "compatible with everything", which would let an install silently run on an untested future major — wrong-way risk for a delete-capable plugin). To onboard a new Zotero major: test on its beta (betas IGNORE `strict_max_version`), then bump `strict_max_version` in `manifest.json`, rebuild, and re-serve `update.json` — installed copies re-enable on the next background check WITHOUT a new XPI. Do NOT blind-widen ahead of a testable beta.
- Conventions: classes `PascalCase`, functions `camelCase`, private `_`-prefixed. Named exports only. Async-first. Errors → `Zotero.logError` (user) / `Zotero.debug` (dev). JSDoc public fns. Identity by 8-char Zotero keys, not numeric itemIDs.
- For complex multi-part work, prefer delegating disjoint slices to parallel sub-agents and coordinating; keep live-MCP testing serial (one flaky bridge).

## TESTING — never assume it works
- Run `npx vitest run` (single file: `npx vitest run test/unit/<m>.test.mjs`; by name: `-t "<name>"`). Suite must stay green before any commit/checkpoint. **965 tests across 28 files** — update this count when it changes.
- New module → `test/unit/<m>.test.mjs`, import SUT from `../../content/<m>.mjs`, `vi.mock` deps per-file, reset in `beforeEach`. `test/setup/geckoMocks.js` stubs `Zotero`/`IOUtils`/`PathUtils`/`Services`/`crypto.subtle`. The `_hashCache.mjs` singleton must be cleared in `beforeEach` if you mock `getFileHash`.
- Live verification = `.private/mcp-runbooks/` (maintainer-only). Run **SMOKE.md S.1–S.7** before tagging a release. Version-guard first: `zotero_plugin_list` must equal source version, else you're testing stale code.

## DOCS IN SYNC — at every checkpoint
- Three hand-authored single-file HTML pages at repo root (embedded CSS, no JS, no build): `index.html` (landing), `test-plan.html` (5-chapter user stories), `test-cases.html` (inclusion/exclusion behavior spec). Every behavior must appear in `test-cases.html`.
- On any feature ship / version bump / behavior change: refresh version badges, footer dates, the hero meta strip (test/pref/mode counts), the configure table (mirrors `prefs.js`), and the affected story/case cards. Never describe features that aren't in the current bundle.
- Public dev docs in `docs/` (`DEVELOPERS.md`, `architecture.md`). Pref count + test count also appear in `CLAUDE.md` here and the HTML — keep all occurrences equal.

# SCOPE MODEL (v2.7)
- `scopeMode` pref, the primary dial:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [josesiqueira/zotero-watch-folder](https://github.com/josesiqueira/zotero-watch-folder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
