---
trigger: always_on
description: Detect the user's language from their first message. All communication — interview questions, summaries, confirmations, error messages — MUST be in that language. Pipeline files and code comments stay in English.
---

# Etnamute

**Version**: 1.0.0

---

## LANGUAGE

Detect the user's language from their first message. All communication — interview questions, summaries, confirmations, error messages — MUST be in that language. Pipeline files and code comments stay in English.

---

## PURPOSE

Etnamute transforms **raw app ideas** into **publishable mobile products**. Not demos. Not toys. Not half-products.

| In Scope                           | Out of Scope        |
| ---------------------------------- | ------------------- |
| iOS + Android mobile apps          | Websites            |
| Expo React Native                  | Backend APIs        |
| RevenueCat monetization (optional) | User authentication |
| Offline-first / local storage      | Cloud databases     |

Monetization is **user's choice** — decided during the discovery interview. Free apps get zero monetization code.

---

## USER FLOW

```
USER: describes app idea
  ↓
PHASE 0: Discovery (INTERACTIVE)                                → /spec-app
  0a: Adaptive interview (5-8 questions via AskUserQuestion)
  0b: Web research (competitors, pricing, market)
  0c: PRD generation → user approves
  ↓
PHASE 1: Plan (AUTONOMOUS)                                       → /build-app
  9-section implementation plan (template: pipeline/plan.md)
  ↓
PHASE 2: Build (AUTONOMOUS, milestone-driven)                    → /build-app
  M1: Scaffold → M2: Screens → M3: Features
  M4: Monetization (skip if free) → M5: Polish
  QA after each milestone (pipeline/qa.md)
  ↓
PHASE 3: Finalization
  Final QA → FINAL_VERDICT.md
  ↓
BUILD COMPLETE
  ↓ (user iterates with /improve-app until satisfied)
  ↓
/market-app → ASO + research + marketing materials
  ↓
/release-app → build + screenshots + submit to stores
```

**Improve Mode**: user requests changes to an existing app → `/improve-app`
**Headless Mode**: build from a pre-written PRD without interview → `/headless`

---

## DIRECTORY STRUCTURE

```
etnamute/
├── CLAUDE.md
├── .claude/
│   ├── skills/                      # Slash commands + code quality skills
│   ├── rules/                       # Auto-discovered build standards
│   └── hooks/                       # Post-edit checks
├── pipeline/                        # Shared references (skills are the primary source of truth)
│   ├── qa.md                        # QA procedure (shared by multiple skills)
│   ├── plan.md                      # Plan template (shared by /build-app and /headless)
│   └── prd-schema.md                # PRD format specification
├── scripts/
│   ├── generate-assets.mjs
│   ├── greenlight.sh
│   └── clean.sh
├── .mcp.json                        # mcpdoc (Expo + RevenueCat docs)
└── apps/                            # One directory per app
    └── <app-slug>/
        ├── spec/                    # PRD, research, plan, design
        │   ├── prd.md
        │   ├── research.md
        │   ├── plan.md
        │   └── DESIGN.md           # From Stitch (optional)
        ├── ralph/FINAL_VERDICT.md
        ├── package.json
        ├── app/, src/, assets/
        ├── research/, aso/, marketing/
        └── fastlane/, .maestro/     # Phase 4 (release)
```

### Output Contract

Every app in `apps/<slug>/` MUST have:
- `package.json`, `app.config.js`, `tsconfig.json`
- `app/` with `_layout.tsx`, `index.tsx`, `home.tsx`, `settings.tsx`
- `app/paywall.tsx` + `src/services/purchases.ts` — only if monetization enabled
- `assets/icon.png` (1024x1024), `assets/splash.png`
- `aso/`, `research/`, `marketing/` — generated via `/market-app` after code is finalized
- `README.md`, `RUNBOOK.md`, `TESTING.md`, `LAUNCH_CHECKLIST.md`, `privacy_policy.md`

---

## MODES

| Mode | Trigger | Behavior |
|------|---------|----------|
| **Discovery** | User describes an app | Interactive interview → research → PRD approval |
| **Build** | User approves PRD | Autonomous: plan → milestones → QA. No stops, no questions. |
| **QA (Ralph)** | After each milestone | Adversarial review from PRD. Fix until ≥97%. Max 3 iterations. |
| **Improve** | User requests change to existing app | Read PRD + code → clarify → apply → verify |
| **Headless** | User provides a PRD file path | Validate PRD → plan → build → QA. No interview. |
| **Release** | User asks to deploy | Fastlane config → Maestro screenshots → build → submit |

```
Discovery ──[PRD approved]──▶ Build ──[milestone]──▶ QA ──[≥97%]──▶ Build (next)
Headless  ──[PRD validated]──▶ Build (same as above)
Build ──[all milestones + final QA]──▶ Complete ──[user request]──▶ Release
Improve ──[changes + verify]──▶ Done ──[more changes]──▶ Improve (loop)
```

---

## PHASE DETAILS

### Phase 0: Discovery → `/spec-app`

Interview → research → PRD generation → user approval.

### Phase 1-2: Plan + Build → `/build-app`

Plan (9 sections via `pipeline/plan.md`) → 5 milestones → QA after each (`pipeline/qa.md`).

| Milestone | Deliverables | Tests |
|-----------|-------------|-------|
| M1: Scaffold | versions, package.json, NativeWind, jest setup | — |
| M2: Screens | navigation, core UI | screen render tests |
| M3: Features | core functionality, data | store + util + persistence tests |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bes-dev/etnamute](https://github.com/bes-dev/etnamute) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
