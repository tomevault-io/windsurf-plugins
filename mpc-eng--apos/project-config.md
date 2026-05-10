---
trigger: always_on
description: APOS is the Autonomous App Portfolio Operating System — a fully agentic pipeline for identifying, validating, building, and monetising software products. It operates across five stages: **Idea Pool → Triage → Validate → Build → Convert**. Designed for a solo PM operating as Chief Decision Officer whose only required inputs are binary approve/reject decisions.
---

# APOS Development Pipeline

## Project Overview

APOS is the Autonomous App Portfolio Operating System — a fully agentic pipeline for identifying, validating, building, and monetising software products. It operates across five stages: **Idea Pool → Triage → Validate → Build → Convert**. Designed for a solo PM operating as Chief Decision Officer whose only required inputs are binary approve/reject decisions.

## Multi-Track Architecture

APOS operates on three independent dimensions. Full architecture: `agents/config/multi-track-architecture.md`.

### Platforms (what gets built)
| Platform | Stack | Distribution | Design System |
|---|---|---|---|
| **iOS** | Swift 6 / SwiftUI / XcodeGen | App Store | APOSDesignSystem (Swift Package) |
| **Web** | TypeScript / React / Next.js | Web URL | shadcn/ui + Tailwind tokens |

Platform is set per-app at registration. Agent definitions use a core + platform overlay pattern: `core/agent.core.md` + `platforms/{platform}/agent.{platform}.md`.

### Ideation Modes (how ideas are generated)
| Mode | Direction | Command |
|---|---|---|
| **Market-Pull** (default) | Problem → Solution. Scan signals for struggle behaviour | `/generate-ideas` |
| **Technology-Push** | Domain Pain → Claude-Fit. Start from domain expertise, validate Claude enables a product that couldn't exist before | `/builder-ideate` |
| **Clone** | Reference → Differentiation. Gap analysis against existing app | `/clone` |

All modes output to `ideas.json` and enter the same downstream pipeline.

### Evaluation Profiles (who judges quality)
| Profile | Extra Criteria | Config |
|---|---|---|
| **Standard** (default) | APOS signal strength, JTBD, switching trigger | Built into agent definitions |
| **Builder Program** | Claude runtime dependency, displacement test (HG-5), structural vs. replicable capabilities, demo impact, VC narrative | `agents/config/builder-program-profile.json` |

Profiles are additive overlays — they add modifiers and hard gates on top of standard scoring.

## Schema Version

All agents and schemas use **schema_version: "3.4.0"** (JSON Schema draft-07).

## Pipeline Stages

| Stage | What Happens | Owner Input |
|---|---|---|
| **Idea Pool** | Scan signal sources, frame user value (JTBD), score ideas with owner proximity and trend coupling modifiers, surface top candidates | Review expanded ideas, rate proximity to problem, collapse/ignore low-score |
| **Research** *(optional)* | Deep feasibility research on an owner-suggested idea: market sizing, competitor deep-dive, user research synthesis, technical feasibility, monetisation benchmarks. Alternative on-ramp to Triage | Review decision card → COMMIT_TO_TRIAGE / PARK / KILL |
| **Clone** *(optional)* | Takes an existing app or concept, analyses gaps across 7 dimensions (incl. value-chain coupling), generates 3-5 differentiated angles as complete idea entries. Alternative on-ramp alongside Research | Select which angles to commit to ideas.json |
| **Triage** | Adversarial two-pass evaluation: prosecution (kill brief) then defence (four checks). Batch-ranked competitively — only one idea promoted per batch. If eligible: PROMOTE_TO_RAPID_PROTOTYPE (low complexity + direct experience + 0 concessions). Researched ideas enter with enriched context | Review decision card (leads with uncertainty, not advocacy). If rapid prototype offered: approve or choose standard validation |
| **Rapid Prototype** *(conditional)* | If triage recommends PROMOTE_TO_RAPID_PROTOTYPE: skip landing page, compressed build (Foundation + 1 sprint). MVP is the validation artifact. Usage Validation (5 users, 3 days) replaces email-signup validation | Approve rapid prototype path, recruit 5 test users, review usage validation results |
| **Validate** | Pre-flight check, landing page with pricing signal, multi-channel traffic strategy, 7-day signal collection with conversion rate as primary metric. OR: Usage Validation after rapid prototype (5 users, 3-day observation) | Approve community post + outreach list, review Day 7 decision card |
| **Design** *(optional)* | Visual design iteration using Refero (precedent research), Stitch (screen generation), Figma (refinement). Critical-path screens only: onboarding, core loop, empty states, paywall. Max 3 iteration rounds → design freeze | Approve/iterate screens each round, approve design freeze |
| **Build** | Orchestrator coordinates spec → Stitch validation (if mockups) → code → compile → test → run tests → review → UX walkthrough cycle. Approved design screens feed into spec + code context | Review spec before code, approve/reject at each gate |
| **Convert** | Weekly analytics, A/B test proposals, win-back tracking | Approve/reject single A/B test per week |

## Context Routing

CLAUDE.md is routing and overview — enforcement rules live in the agent definitions. When looking for details on a specific topic:

| Topic | Read |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mpc-eng/apos](https://github.com/mpc-eng/apos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
