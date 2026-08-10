---
trigger: always_on
description: This file is auto-loaded at the start of every Claude Code session. It is the **orientation
---

# CLAUDE.md — start here every session

This file is auto-loaded at the start of every Claude Code session. It is the **orientation
layer** for the project. The planner (Sandeep) decides direction; the agent (you) builds.
Read the four linked docs below before doing strategic work — never start research or planning
from scratch when a doc already holds the answer.

## What this project is

A framework/runtime for **kriya software**: apps where AI agents are first-class users,
driving the app through **typed actions** (not screenshots, not DOM scraping). Same handler
serves a human button-click and an agent tool-call. Built on **Tauri 2 + Rust agent host +
TypeScript SDK + React**, with a safety layer (permissions, human approval, signed audit,
budget, persistent memory) built in, not bolted on.

The long-term ambition is a **full, hard-to-copy, YC-ready** product — not a toy demo. When
scoping work, bias toward the copy-resistant depth, and don't oversell small wins.

## The source of truth lives in the repo (not in memory)

Claude's machine-local memory is a fast cache only. **These repo docs are canonical** — if
they ever disagree with memory, the repo wins (decision [D-008](docs/DECISIONS.md)).

| Doc | What it holds | Read it when |
|---|---|---|
| [docs/ROADMAP.md](docs/ROADMAP.md) | **What to build next**, prioritized. The checklist the planner points you at. | Starting any build work |
| [docs/strategy/](docs/strategy/) | **Why** — market landscape + the strategic wedge | Any positioning / strategy / "should we change X" question |
| [docs/PRODUCT_GAPS.md](docs/PRODUCT_GAPS.md) | **Feature-completion state** — what's done / partial / missing | Checking what exists before building |
| [docs/DECISIONS.md](docs/DECISIONS.md) | **Decisions + why**, append-only | Before re-opening a settled question |
| [docs/PUBLISHING.md](docs/PUBLISHING.md) | npm/crates.io publish runbook — initial publish **done** (R2, 2026-06-15); now the **republish** runbook | Republishing after changing a published package/crate |
| [docs/LAUNCH.md](docs/LAUNCH.md) | Show HN plan, opening comment, objection answers, Twitter thread | Working roadmap item R12 / launch |
| [architecture.md](architecture.md) | How the bolt-on / in-process **pattern** works end to end | Touching the protocol / host loop |
| [docs/SERVICE-ARCHITECTURE.md](docs/SERVICE-ARCHITECTURE.md) | **The service architecture** (D-016): one governance core + three reach fronts + the shippable `kriya-gateway` product; tech + sequence diagrams + build-over plan | Building the gateway / proxy / fronts (R22–R26) |
| [docs/THREE-FRONTIERS.md](docs/THREE-FRONTIERS.md) | **Local-only & SUPERSEDED (2026-06-27).** The original "three frontiers" positioning essay — premise (desktop has no agent tooling standard) is outdated (desktop has MCP-stdio too). The public positioning is now the **governance / control-plane** thesis in the README. Kept for history; **do not re-surface the three-frontiers framing publicly.** | Positioning history only — never as the current pitch |
| **PRIVATE Console repo → `kriya-console/docs/ideas/`** | **The 2026-06 product-strategy + GTM + control-plane direction lives in the private Console repo** (`governex/kriya-console`), not here. Start at `docs/ideas/README.md`; the definitive build target is `docs/ideas/13-control-plane-full-spec-and-gtm.md` (the on-prem/online agent-governance control plane for regulated/sovereign/air-gapped buyers) + the build roadmap `docs/ideas/14-build-roadmap.md`. | **Any product/GTM/control-plane/"where is kriya going" question — the answer is in the private repo, not this one.** A shared `kriya-verify` crate (extracted from this runtime's verifier) is the planned seam. |

## Current direction (as of 2026-06-24)

- **Working thesis (planner-affirmed, not yet market-validated):** the bet is the governed
  **in-process** action layer for capabilities that live only inside a running local app (no API
  to wrap, local/private data) — desktop/in-process is the mechanism, governance the moat, **one
  bet, not two**. Dual GTM: build-time adoption for new local-first AI apps + bolt-on migration
  of existing apps (`wrapAction`) + enterprise governance. ICP: POS (planner is building one),
  CRM, regulated workstations, near-future agentic SaaS. Don't regress to "generic governance" —
  see [docs/strategy/governed-local-first-wedge.md](docs/strategy/governed-local-first-wedge.md)
  and decision D-009.
- **Pivot (D-016, 2026-06-24) — the new front door is a PRODUCT, not the library.** MCP-stdio
  commoditized the *transport*, not the *enforcement* (native MCP has no enforced governance:
  approval is SHOULD-not-MUST + client-side, auth is OPTIONAL + excludes stdio). So ship a
  **standalone `kriya-gateway`** — a zero-change governance **proxy** that wraps any existing MCP
  server with policy + approval + signed audit — and reposition `wrapAction` as enterprise depth.
  Architecture = **one transport-agnostic core + the govern/audit fronts** (hooks / proxy / broker /
  LLM proxy / egress lanes), **build-over not rewrite**. See
  [docs/SERVICE-ARCHITECTURE.md](docs/SERVICE-ARCHITECTURE.md) + D-016. The wedge (D-009) is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [governex/kriya](https://github.com/governex/kriya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
