---
trigger: always_on
description: **Last Updated:** 2026-08-27
---

# LLM Wiki Plugin Project Development Standards

**Last Updated:** 2026-08-27

**Current state pointer:** see [ROADMAP.md](./ROADMAP.md#current-status) for active development phase + [CHANGELOG.md](./CHANGELOG.md) for shipped history. This file (AGENTS.md) carries process standards only — and is the single canonical source. The historical mirror at `CLAUDE.md` is now a pointer stub to this file; all new content goes here. **Latest shipped:** v1.27.0 MINOR (2026-08-27, 3677 tests / 260 files).

---

## 🛡️ Six-Gate Quality Closure

| Gate | Constraint | How |
|------|-----------|-----|
| **1. Code correct** | `pnpm lint` 0/0 + `npx tsc --noEmit` 0/0 + `pnpm build` clean + `pnpm test` all pass + `pnpm css-lint` 0 | Five-Gate script (build BEFORE test — see §"Gate 1: Five-Gate automated") |
| **2. No side effects** | Call-site audit + data flow + state mutation + error propagation | Structured review |
| **3. No breaking changes** | API/Schema/File format/Default behavior/Command IDs/Obsidian API | Breaking-change matrix |
| **4. No performance regression** | CPU/memory/IO/network/token — 5-dim written assessment | simplify + code-review + Gate 4 table |
| **5. Docs complete** | 10 READMEs + ROADMAP + AGENTS + CHANGELOG + memory | pre-release-gate |
| **6. Release clean** | Supersets 1-5 + TOC + i18n + Release Notes + Contributors + git hygiene | pre-release-gate |

### Gate 1: Five-Gate automated

```bash
pnpm lint && npx tsc --noEmit && pnpm build && pnpm test && pnpm css-lint
```

All five must pass. ESLint checks style, TypeScript checks types, css-lint checks Obsidian review compliance — three complementary checks, single tool passing is insufficient. No `@ts-ignore` / `eslint-disable` to silence failures.

**Order is non-negotiable**: `pnpm build` MUST run before `pnpm test`. The test suite contains build-artifact verifications (e.g. `src/__tests__/llm-sdk/openai-codex-loopback-flow.test.ts:39` reads `main.js` to assert the esbuild bundle shape), so a test-first run fails with ENOENT on a fresh clone. Local Gate 1 typically has `main.js` on disk from a prior `pnpm build:dev`, which is why this ordering bug was missed before PR #487's first CI run on 2026-08-18.

**Bot alignment (pre-release):** local `pnpm lint` ≠ Obsidian review bot. Bot runs newer `eslint-plugin-obsidianmd`. Before each release:
```bash
LOCAL=$(node -p "require('./node_modules/eslint-plugin-obsidianmd/package.json').version")
LATEST=$(npm view eslint-plugin-obsidianmd version)
[ "$LOCAL" != "$LATEST" ] && pnpm add -D "eslint-plugin-obsidianmd@$LATEST" && rm -f pnpm-lock.yaml && pnpm install && npm install --legacy-peer-deps --package-lock-only && pnpm lint
```

### Gate 2: No Side Effects

For each modified function, trace: **call-site audit** (`grep -rn "<fn>" src/`) → **data flow** (input origin → output destination → side effects) → **state mutation** (concurrent safety? overwrite vs append?) → **error propagation** (new error paths caught by all callers?). Deliverable: 3-5 sentence assessment.

### Gate 3: No Breaking Changes

| Dimension | Check | Pass criteria |
|-----------|-------|---------------|
| API signature | `git diff` + `grep` | All call-sites updated; no new required params without defaults |
| Settings schema | `types.ts` + `settings.ts` | New fields have defaults; removed fields ignored |
| File format | Generation templates | Old files load without error |
| Default behavior | Constructor / config init | Old behavior preserved unless opted in |
| Command/setting IDs | `grep` for IDs | IDs unchanged |
| Obsidian API | `manifest.json` | `minAppVersion` >= current |

Deliverable: "None detected" or specific migration plan.

### Gate 4: No Performance Regression

Procedure: (1) Run `simplify` (3 parallel agents); (2) Run `code-review` (max effort); (3) Walk through 5 dimensions below; (4) If regression → mitigate or escalate; (5) If N/A → state so.

| # | Dimension | Project-specific signal |
|---|-----------|-------------------------|
| 1 | CPU | `O(n²) candidate generation` is known risk — do not regress |
| 2 | Memory | `thinkingControlCache` bounded by user count; `Map<string, PageMeta>` in `generateDuplicateCandidates` holds all pages |
| 3 | IO | `vault.read()` per page in loops is expensive |
| 4 | Network | `OpenAICompatibleClient.createMessage` should cache 400-fallback (Issue #245); lint dedup budget 500 |
| 5 | Token | Ingest 1-3K; lint dedup 100 × ~30 = 3K/batch; retries = full prompt |

**Deliverable** (mandatory in commit body):
```
## Gate 4: Performance

| Dim | Status | Notes |
|-----|--------|-------|
| CPU | ✅/⚠️/N/A | ... |
| Memory | ✅/⚠️/N/A | ... |
| IO | ✅/⚠️/N/A | ... |
| Network | ✅/⚠️/N/A | ... |
| Token | ✅/⚠️/N/A | ... |
```
Bare "no regression" is **not acceptable**.

### Gate 5 + Gate 6

Gate 6 supersets Gates 1-5 plus release hygiene. `pre-release-gate` skill (release Step 4a) re-runs Gate 1 + all Gate 4 dimensions (Gate 6 must re-verify perf at release time, not trust commit-time).

### ⚠️ Anti-patterns

- "The tests pass, so it's fine" → Tests only cover what you thought to test
- "It's just a one-line change" → Most dangerous
- "I'll add tests later" → Tests must accompany change
- "The PR review will catch it" → Reviewer has less context than you

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GD4AI/obsidian-llm-wiki](https://github.com/GD4AI/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
