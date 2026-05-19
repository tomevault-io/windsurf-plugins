---
trigger: always_on
description: Codebase context + AI-agent rules of engagement. **Read this before any task on this repo.**
---

# CLAUDE.md

Codebase context + AI-agent rules of engagement. **Read this before any task on this repo.**

> **Behavioral guardrails:** see [`~/.claude/CLAUDE.md` → Karpathy Behavioral Guidelines](~/.claude/CLAUDE.md).

## What this repo is

WaveX OS — open-source operating system for running an AI agent company on
localhost. The full-fidelity onboarding pipeline is owned by wavex-os
(vendored at `vendor/wavex-os/`) and surfaced through wavex-os adapter
packages. Wavex-os contributes the dashboard (`MissionControl` +
`components/mission/*`), the runtime layer (healing, observability, launchd
templates, agent skills, install scripts), and the boundary services
(auth, composio, db, inference) that the vendored plugin consumes.

They meet at `~/.wavex-os/instances/<companyId>/`.

## Repo map

```
wavex-os/
├── apps/installer/                  npx wavex-os init CLI
├── packages/
│   ├── core/                        Paperclip vendored via subtree
│   ├── db/                ★ NEW     PGlite (dev) / Postgres (prod) + Drizzle schema
│   ├── plugin-sdk-shim/   ★ NEW     Re-exports @paperclipai/plugin-sdk surface
│   ├── auth-shim/         ★ NEW     assertBoard / assertCompanyAccess (WAVEX_AUTH_MODE)
│   ├── composio-shim/     ★ NEW     listConnections + featured toolkits (WAVEX_COMPOSIO_DISABLED)
│   ├── inference-adapter/ ★ NEW     tier-router claudeBin (WAVEX_INFERENCE_MODE: oauth/apikey)
│   ├── wavex-os-server/   ★ NEW     Fastify routes for vendored onboarding
│   ├── onboarding-ui/               Browser app (MissionControl + WavexOsOnboarding)
│   ├── mock-core/                   Fastify server: hosts /api/* + wavex-os routes
│   ├── healing/             ★ FROZEN    OAuth refresh, worker restart, 401 fallback
│   ├── observability/       ★ FROZEN    bottlenecks, attribution, token budget (drizzle-orm peer)
│   ├── standard-skills/     ★ FROZEN    cross-cutting agent skills
│   └── agent-templates/                  per-role skill definitions
├── vendor/wavex-os/         ★ NEW     vendored wavex-os @ d84983a1 (2026-05-03)
│   ├── plugin-sdk/                       @paperclipai/plugin-sdk
│   ├── shared/                           @paperclipai/shared
│   ├── tier-router/                      @wavex-os/plugin-tier-router
│   ├── flywheel-kernel/                  @wavex-os/plugin-flywheel-kernel
│   ├── onboarding/                       @wavex-os/plugin-onboarding (50 src files)
│   ├── wavex-os-flow-types/              @wavex-os/plugin-flow-types
│   ├── ui-onboarding-components/         upstream UI source (future migration target)
│   ├── tsconfig.base.json                shared sibling base for vendored packages
│   └── VENDOR.md                         source SHA + vendor exceptions + update procedure
├── scripts/wrappers/        ★ FROZEN    claude-anthropic-direct.sh, claude-spawn.sh
├── scripts/wavex-claude-spawn.sh   ★ NEW   T2 spawn shim (prepends `exec` for tier-router)
├── scripts/render-launchd-templates.mjs ★ FROZEN
├── scripts/provision-*.mjs               ★ FROZEN
├── templates/launchd/       ★ FROZEN    macOS plist templates
├── examples/*.example.json  ★ FROZEN    runtime contract spec
├── baseline/                ★ NEW     captured fixture KPI baselines
├── docs/ops/                ★ NEW     surface-tuning-map.md / -chart.html (40 tunables)
├── docs/onboarding/migration-plan.md     full-fidelity port plan + 7 committed decisions
├── docs/MINIMAL_INCEPTION.md             kernel topology spec
└── docs/SELF_HEALING.md                  4-layer recovery architecture
```

## Frozen paths (DO NOT MODIFY)

```
packages/healing/**
packages/observability/src/**            (package.json may add type deps only)
packages/standard-skills/**
packages/onboarding-ui/public/agent-templates/**
apps/installer/**
scripts/wrappers/*.sh
scripts/render-launchd-templates.mjs
scripts/provision-*.mjs
scripts/setup-hierarchy-and-kpis.sample.mjs
templates/launchd/**
examples/*.example.json
vendor/wavex-os/**                       (excepting documented patches in VENDOR.md)
```

If a frozen path needs to change to complete a task, **STOP** and surface the concern. Do not proceed.

## Where to start by task

| Task | Touch |
|---|---|
| Add a pillar field | `vendor/wavex-os/onboarding/src/schema/pillar-responses.ts` upstream + re-vendor |
| Add a connector | `vendor/wavex-os/onboarding/src/phases/phase-2-connector/decision-matrix.ts` upstream |
| Add a Fastify route | `packages/wavex-os-server/src/routes/*.ts` |
| Add an inference call | Plugin's tier-router; wavex-os adapter sets bin via inference-adapter |
| Add a wizard subview | `packages/onboarding-ui/src/wavex-os/{pillars,phases}/` (wavex layout) |
| Add a dashboard read | `packages/wavex-os-server/src/routes/instance.ts` (new endpoint) |
| Add a DB table | `packages/db/src/schema/<file>.ts` + new migration in `packages/db/migrations/` |
| Activate a finalized manifest into the DB | `POST /api/instance/<id>/activate` → `packages/wavex-os-server/src/routes/activate.ts` → `bridge/finalize-bridge.ts` |
| Add a slot↔template mapping | `packages/wavex-os-server/src/bridge/catalog.ts` (`SLOT_TO_TEMPLATE`) |

## Onboarding contract (filesystem)

```
~/.wavex-os/instances/<companyId>/         ← wavex projection layer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aimerdoux/wavex-os](https://github.com/aimerdoux/wavex-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
