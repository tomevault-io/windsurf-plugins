---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## 🟢 ACTIVE CONTINUATION — read `docs/NEXT_ARC_PLAN.md` first

The v1 release arc is merged (PR #841, 2026-07-02). The live arc is now
**`docs/NEXT_ARC_PLAN.md`** (the "🟢 HANDOFF — start here" block at the top): Wave 1 =
**ConKay JARVIS flagship** (spatial FUI cockpit, honest stage-beats, artifact→interactive-3D)
+ **shared primitives** (provenance-stamped ingest, X-as-DTU listing, lattice-fork object,
governance design doc), with the audited 43-idea backlog behind it. The ConKay design detail
still lives in `docs/CONKAY_HONEST_HOLOGRAM_PLAN.md` (Track-B design source; note its Phase-2
`@react-three/postprocessing` blocker is STALE — the dependency is installed now).

- **Honest depth floor:** **0.688** (`node scripts/grade-macro-depth.mjs --honest`), climbing toward the ~0.73 ceiling. Read this number precisely: **100% of macros carry shape/smoke coverage** (the CI harness derives a case per macro) and **~70% of the non-`utility` macros stand on a real behavioral test**; 0.688 is a *weighted* score that taxes correctly-small utility code at 0.6 by design — it is NOT "31% untested."
- **Strategic companion:** `docs/SCIFI_FEASIBILITY_MAP.md` — code-grounded audit (2026-06-08). It corrected two load-bearing facts: engineering **CAS + beam-frame FEA are a real STRENGTH**, and external **connectors were scaffold** (only MCP + OAuth-signin were real → was **Track C**). **Update 2026-06-09: Track C done — Gmail + Google Calendar are now real two-way** (send/push + read/inbox/pull on the SSRF-guarded `connectorFetch` chokepoint, encrypted per-user tokens, polished GmailSection inbox client; live use needs only a Google OAuth client — `docs/CONNECTORS_GO_LIVE.md`). Slack/Sheets/GitHub/Notion are the next thin adds.
- **Three tracks (owner-locked, pre-Wave-1 framing):** **(A) Depth sweep** — fleet loop toward the ceiling (`node scripts/depth-backlog.mjs` → subagents → guard → commit → push). **(B) ConKay honest hologram** — superseded: this track's "Phase 2" is now Wave 1's **Track K** in `docs/NEXT_ARC_PLAN.md`, and as of 2026-07-03 Track K (K1–K6, K6-voice) **and** Track P (P-A/P-B/P-C/P-D) are both fully **SHIPPED** — see that doc's §B/§C for per-phase commit hashes. Next work pulls from the ranked backlog in §D, not from a "Phase 2." **(C) Marquee connector honesty** — ✅ **done (2026-06-09): Gmail + Google Calendar real two-way**; remaining connector work is Slack/Sheets/GitHub/Notion (thin adds on the connector-agnostic core).
- **Non-negotiable for Track B:** *honest by construction* — every animated element is a pure function of a real backend event; **no `setInterval`/fake-progress.** Audit: `grep -rE "setInterval|setTimeout" concord-frontend/components/conkay/`.
- The strategic framing (verified + private compute-agent for R&D; verification IS the product) and the full design + research live in that doc. The `/root/.claude/plans/` copy is container-ephemeral — the repo doc is the source of truth.
- **MMO/RPG completeness (2026-06-26):** `docs/MMO_RPG_COMPLETENESS_AUDIT.md` — a 21-pillar "is the game complete & playable" framework + a code-verified Concordia scorecard + a fixed-defect ledger + a re-verified backlog. **Backlog re-checked 2026-06-26 and largely CLOSED:** gear durability/repair SHIPPED; the ~105-site schema-drift batch is CLOSED (`node scripts/verify-schema-drift.mjs --ci 0` → DRIFT 0, CI-gated against recurrence); the POLISH_AUDIT combat-feel seams are FIXED (pinned by `feel-consolidation.test.ts`). The only residual is optional/non-defect (recorded-audio-vs-synthesis taste call, balance-constant tuning, invariant-engine override authoring). Backed by `docs/research/MMO_RPG_GENRE_RESEARCH.md` (cited genre research) and `docs/research/WIRING_INTEGRITY_AUDIT.md` (caller↔receiver audit). This pass fixed the unknown-macro LLM-mask (now fail-fast `unknown_macro`), the `dtu.create` phantom-success data loss, the `player:low-health` phantom listener, the hardcoded minimap position, the malformed maintenance-gates critical, CharacterCustomizer's fabricated wardrobe (now a real `appearance.options` catalog), the `/dialogue/respond` flat-stub fallback, and the 9 orphan socket emits + 12 dead `concordia:*` CustomEvents; and added Character Sheet / Ability Cooldown HUD / Target Nameplate to the world lens.

---

## What This Is


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ConcordDev/concord-cognitive-engine](https://github.com/ConcordDev/concord-cognitive-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
