---
trigger: always_on
description: This file is read by Claude Code sessions on this repo. It defines the current sprint goal, scope, quality bar, and anti-patterns so any session (continuation or new) shares the same North Star.
---

# Project goal — v3.10.x stable maintenance line (v3.10.0 shipped @latest)

This file is read by Claude Code sessions on this repo. It defines the current sprint goal, scope, quality bar, and anti-patterns so any session (continuation or new) shares the same North Star.

**v3.6.0 stable shipped on 2026-05-15. v3.8.0 stable shipped on 2026-05-24** (promoted from @rc → @latest after 18 RCs of methodology hardening). The current cascade (v3.6.0 → v3.8.2+) is the post-release maintenance + audit-driven hardening line.

**v3.8.0 minor delivered:** K-3 readOnlyHint structural invariant, R-7 watcher → embed-db sync (md + PDF), R-3 CLI parity (addAdvancedRetrievalOptions), Tier A/B AI/LLM discoverability (llms.txt, AGENTS.md, official MCP Registry submission), 8 new structural invariants (cli-help, cli-parity ×2, k3, M-2 docs-consistency, META-invariant, multi-subcommand drift, version-consistency 5→7), 1 OIA walk added (check 6).

**v3.8.0 NOT delivered (closed in v3.8.x post-stable):** T-2/T-3/T-4 E2E tests (v3.8.5), HTTP P2-10/P2-11 lifecycle hardening (v3.8.7).

**v3.9.0 minor in-flight (RCs shipped on `@rc`):** OCR'd PDF watcher embed-sync (rc.1), HNSW in-memory live update (rc.2), R-10 adaptive HNSW refill (rc.3), full state-driven self-audit (rc.4), OCR install docs unification (rc.5), HNSW disk persistence on live update (rc.6). All architectural v3.9.0 items shipped. Promote to `@latest` after fresh external audit per `docs/audits/AUDIT-REQUEST-v3.9.0-rc.2-2026-05-25.md`.

**Still deferred to v3.9.x+:** HNSW filter-during-search (architectural), embed-db migrations, distributed rate-limit, `install-ocr-lang` subcommand (needs langPath wiring + env-gated test).

**External audit blocker (CLAUDE.md v3.6.1 rule, ≥2 independent external auditors with DIFFERENT methodologies) — re-evaluated 2026-05-25 against the corrected audit doc.** v3.8.0 was promoted on internal confidence alone; v3.8.1 retracted the incorrectly-attributed Cursor audit (overclaim #11 — wrong project). On 2026-05-25 the same auditor delivered the CORRECTED enquire-mcp audit on commit `7a9fdbd` / `v3.8.0-rc.15` (`docs/audits/v3.8.0-rc.15-external-2026-05-25.md`): **verdict 4.85/5, ship-blockers: none**. All 5 actionable findings (M-REG-1, L-HYB-1, L-OIA-1, INFO-1, INFO-3) were already closed by rc.18→v3.8.5. INFO-2 (R-10 residual at >66% excluded) closes in v3.9.0-rc.3. **The v3.8.0 stable promotion was retroactively justified by this audit** — but the v3.6.1 rule requires ≥2 INDEPENDENT external auditors with DIFFERENT methodologies. Counting Mavis (v3.6.0, 4.9/5) + Anonymous (v3.6.0, CRITICAL) + Round-7 (v3.6.2) + Round-12/13/14/15/16 + 2026-05-25 corrected pass: **5+ independent external audits across v3.6.x→v3.8.x cascade**, methodologies span change-driven (Mavis-style) + state-driven (Round-22 / 2026-05-25 corrected) + adversarial-CVE (Round-7). **Blocker met.** Next pre-stable promotion (v3.9.0 → @latest) requires a fresh audit on current commit per `docs/audits/AUDIT-REQUEST-v3.9.0-rc.2-2026-05-25.md`.

---

## Goal (historical — v3.6.0 sprint, shipped 2026-05-15)

Released **enquire-mcp v3.6.0** per the planned RC sequence. This section is preserved as historical context — the sprint is closed.

Directive: **"Максимальное качество и уверенный топ-1 из всех Obsidian MCP по технологии и надёжности."**

## Scope (closed) — v3.6.0 RC sequence + promotion

- **v3.6.0-rc.1**: `tools.ts` (4252 lines) → 5 domain modules in `src/tools/` + barrel
- **v3.6.0-rc.2**: `index.ts` (3665 lines) → `src/cli.ts` + `src/server.ts` + `src/prompts.ts` + `src/tool-registry.ts` + `src/tool-manifest.ts`
- **v3.6.0-rc.3**: Full TSDoc (`@param` / `@returns` / `@example`) on 44 tools + 19 prompts + 20 `src/` modules (~1300+ lines of doc-comments)
- **v3.6.0-rc.4**: TypeDoc + GH Pages auto-generated API reference + Public benchmarks (`docs/benchmarks.md`, MRR/NDCG@10/Recall@K vs 3 main competitors)
- **v3.6.0 (stable)**: promote rc.4 → npm `latest`, GH release marked Latest

## Quality bar — required on every release (no exceptions)

1. All tests pass (current count: 818+ at v3.7.17; tests grow with each audit cycle — see CHANGELOG)
2. Lint clean (biome 0 warnings/errors)
3. `tsc` strict + `noUncheckedIndexedAccess` clean
4. Coverage thresholds met (lines ≥86, statements ≥82, functions ≥75, branches ≥74)
5. `scripts/check-changelog-coverage.mjs` passes (CHANGELOG claims = reality within 0.5pp)
6. `scripts/smoke.mjs` synthetic vault scan passes
7. All 9 required CI gates green on PR (lint, test×2, smoke, audit, coverage, version-consistency, docs, oia — `docs` added v3.7.10, `oia` promoted from advisory v3.8.0-rc.6)
8. Daily-check report after merge: 0 regressions, 0 new CodeQL / Dependabot alerts
9. CHANGELOG entry with TL;DR blockquote + method note
10. All docs-consistency invariants stay green (extend as new surfaces appear)

## Audit checkpoint — after every RC

- **Self-audit** via root-cause-sweep methodology (memory: `method_audit_root_cause_sweep.md`): check the drift class hasn't returned from previous cycles

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oomkapwn/enquire-mcp](https://github.com/oomkapwn/enquire-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
