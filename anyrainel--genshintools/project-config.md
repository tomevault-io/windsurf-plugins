---
trigger: always_on
description: React 19 + TypeScript + Vite 7 app for Genshin Impact player tools. UI uses Tailwind CSS 3, shadcn/ui, Radix primitives, Vaul drawers, Lucide icons, and Sonner toasts. State is Zustand 5 with Immer and persist middleware. Cloudflare Workers use `worker/`, `dist/`, and Wrangler for deployment.
---

# Genshin Tools — Codex Context

## Project Snapshot

React 19 + TypeScript + Vite 7 app for Genshin Impact player tools. UI uses Tailwind CSS 3, shadcn/ui, Radix primitives, Vaul drawers, Lucide icons, and Sonner toasts. State is Zustand 5 with Immer and persist middleware. Cloudflare Workers use `worker/`, `dist/`, and Wrangler for deployment.

## Common Commands

- `npm run dev` — Vite dev server
- `npm run dev:vite` — Vite only
- `npm run build` — production web build
- `npm run lint` / `npm run lint:fix` — Biome check / auto-fix
- `npm run type-check` — TypeScript check for app and tests
- `npm run depcheck` — dependency boundary rules via dependency-cruiser
- `npm run test` / `npm run test:watch` / `npm run test:coverage` — Vitest
- `npm run regtest` — artifact generator golden-file regression test

## Page Map

Use this as the first routing hint when the user refers to a page or tab in natural language. Navigation is defined in `src/components/layout/appNavigation.tsx`.

- Home (`/`) — `src/pages/Home.tsx`: app entry page with tool cards and onboarding links.

- Account Data (`/account-data`) — `src/pages/AccountData.tsx`: account import/export shell and tab router.
  - Characters (`/account-data/characters`) — `src/pages/account-data/CharacterView.tsx`: imported character roster, ownership filters, edit mode, and artifact score display.
  - Inventory (`/account-data/inventory`) — `src/pages/account-data/InventoryView.tsx`: imported weapons/artifacts inventory browsing, scanner entry, and edit/delete flows.
  - Recommendations (`/account-data/recommendations`) — `src/pages/account-data/RecommendationView.tsx`: artifact upgrade recommendations from account data, build configs, and scoring thresholds.
  - Evaluation (`/account-data/evaluation`) — `src/pages/account-data/EvaluationView.tsx`: build completion evaluation for owned characters against configured artifact builds.
  - Resources (`/account-data/resources`) — `src/pages/account-data/ResourceView.tsx`: resource spending suggestions for craft, reroll, and artifact level-up actions.
  - Triage (`/account-data/triage`) — `src/pages/account-data/TriageView.tsx`: artifact keep/lock/fodder triage using account data, build configs, and triage settings.

- Artifact Filter (`/artifact-filter`) — `src/pages/ArtifactBuilds.tsx`: artifact build import/export shell and tab router.
  - Configure (`/artifact-filter/configure`) — `src/pages/artifact-builds/CharacterBuildView.tsx`: per-character artifact build target configuration.
  - Compute Filters (`/artifact-filter/filters`) — `src/pages/artifact-builds/ArtifactBuildsView.tsx`: computed artifact set/slot filters from configured character builds.
  - AutoTune (`/artifact-filter/weights`) — `src/pages/artifact-builds/AutoTuneView.tsx`: team-aware AutoTune workflow for deriving weighted artifact formulas.

- Team Comp (`/team-comp`) — `src/pages/TeamComp.tsx`: team import/export shell and tab router.
  - Damage (`/team-comp/damage`) — `src/pages/team-comp/DamageView.tsx`: team grid plus selected-team damage detail calculator.
  - Frozen (`/team-comp/frozen`) — `src/pages/team-comp/FrozenView.tsx`: frozen artifacts/teams review and batch equip export.
  - Investment (`/team-comp/investment`) — `src/pages/team-comp/InvestmentView.tsx`: selected-team investment comparison/detail workflow.
  - Weapon Choice (`/team-comp/weapon`) — `src/pages/team-comp/WeaponChoiceView.tsx`: selected-team weapon comparison/detail workflow.

- Tier List (`/tier-list`) — `src/pages/TierList.tsx`: tier list import/export shell and tab router.
  - Characters (`/tier-list/characters`) — `src/pages/tier-list/CharacterTierListView.tsx`: editable character tier table with filters, presets, export, and customization.
  - Weapons (`/tier-list/weapons`) — `src/pages/tier-list/WeaponTierListView.tsx`: editable weapon tier table with weapon filters, presets, export, and customization.

- Archive (`/archive`) — `src/pages/Archive.tsx`: game data archive shell and tab router.
  - Characters (`/archive/characters`) — `src/pages/archive/CharacterArchiveView.tsx`: searchable character archive with filters and detail panel.
  - Weapons (`/archive/weapons`) — `src/pages/archive/WeaponArchiveView.tsx`: searchable weapon archive with type/rarity/stat filters.
  - Artifacts (`/archive/artifacts`) — `src/pages/archive/ArtifactArchiveView.tsx`: artifact set archive with half-set filtering and expandable rows.
  - Bosses (`/archive/bosses`) — `src/pages/archive/BossArchiveView.tsx`: ley line boss archive with search, schedule-aware default selection, and detail panel.

## Important Locations

- `src/components/{domain}/` — domain UI components
- `src/components/shared/` — reusable cross-domain UI
- `src/components/layout/` — page/layout shells
- `src/stores/` — persisted Zustand stores
- `src/data/` — static data, generated game data, localization files, shared types
- `src/lib/{domain}/` — domain-specific pure logic; avoid putting cross-cutting primitives here unless they truly belong to that domain

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Anyrainel/GenshinTools](https://github.com/Anyrainel/GenshinTools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
