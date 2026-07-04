---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
pnpm dev            # Vite dev server with HMR
pnpm build          # TypeScript check + Vite production build
pnpm lint           # ESLint
pnpm test           # Vitest (single run)
pnpm test:watch     # Vitest (watch mode)
pnpm vitest run src/engine/__tests__/reducer.test.ts  # Run a single test file
pnpm run balance:gate          # AI-vs-AI balance gate tests (all tiers)
npx tsx scripts/simulate.ts    # Deck matchup simulation harness
npx cypress open     # Cypress E2E tests (interactive)
npx cypress run      # Cypress E2E tests (headless)
```

## Project Overview

Alchemy is a browser-based 1v1 elemental card battler (MTG-inspired, kid-friendly for ages 6–10). React 19 + TypeScript + Vite + Zustand + Tailwind CSS. PWA-enabled, deployed to GitHub Pages.

## Architecture

### Path Aliases (defined in vite.config.ts)

`@engine/*`, `@game/*`, `@components/*`, `@hooks/*`, `@storage/*`, `@network/*`, `@audio/*` map to their respective `src/` subdirectories.

### Engine (`src/engine/`) — Pure game logic, no React

- **`types.ts`** — All game types. `Phase`, `GameAction`, `GameEvent` are discriminated unions. `GameState` is the single source of truth.
- **`reducer.ts`** — Pure function `reduce(state, action, actingPlayer, rng) → { newState, events }`. Every game mutation flows through here.
- **`validation.ts`** — `validateAction()` checks legality; `enumerateLegalActions()` lists all valid moves for a player.
- **`cards.ts` / `cards/`** — `CARD_REGISTRY` maps card IDs to `CardDefinition`. Card data is organized by element.
- **`effects.ts` / `effects/`** — `EFFECT_REGISTRY` maps effect IDs to implementations. Effects produce `GameEvent[]`.
- **`keywords.ts` / `keywords/`** — Keyword ability implementations (swift, blast, heal, etc.).
- **`elements.ts`** — Five-element color wheel (fire, water, earth, air, shadow).
- **`prng.ts`** — Seeded RNG (`createRNG(seed)`) for deterministic gameplay and multiplayer sync.
- **`ruleset.ts`** — `TIER_CONFIGS` for apprentice/alchemist/archmage difficulty tiers.
- **`ai.ts`** — AI opponent decision logic (top-level entry point).
- **`aiConfig.ts`** — AI difficulty/personality presets (`AIDifficulty`, `AIPersonality`, `EvalWeights`).
- **`aiEval.ts`** — Pure board-state evaluation function used by AI scoring.
- **`aiStrategy.ts`** — High-level AI strategy selection (aggro, control, tempo).
- **`aiSearch.ts`** — Action search / lookahead tree for AI decision-making.
- **`aiCombat.ts`** — AI combat-specific logic (attacker/blocker assignment).
- **`aiActionPolicy.ts`** — Shared action filtering policy (e.g. preventing targetless-spell loops).
- **`starterDecks.ts`** — Pre-built deck archetypes per element and tier.

### Game State Layer (`src/game/`) — Zustand stores + dispatch orchestration

- **`gameStore.ts`** — Main Zustand store: holds `GameState`, `SeededRNG`, `legalActions`. Wraps the engine reducer.
- **`uiStore.ts`** — UI-only state: card selection, hovering, targeting mode, combat selections.
- **`animationStore.ts`** — Animation queue. Steps block game progression until animations complete.
- **`dispatchWithAnimations.ts`** — Wraps `gameStore.dispatch` to capture element positions and enqueue animation steps from events.
- **`preferencesStore.ts`** — User preferences (difficulty, tier, battlefield, UI scale). localStorage-persisted Zustand store.
- **`learningStore.ts`** — Runtime learning challenge state (opportunity count, streaks) per game session.
- **`learningProfileStore.ts`** — Persistent learner profile (mastery levels, subject preferences).
- **`tutorialStore.ts`** — Tutorial tip dismissal and progression state.
- **`sessionMeta.ts`** — Per-session metadata (game mode, adventure node context).
- **`controllers/`** — `OpponentController` interface with `aiController` (single-player) and `networkController` (peer-to-peer WebRTC).

### Data Flow

```
User interaction → dispatchWithAnimations(action, player)
  → gameStore.dispatch → engine reduce() → new GameState + GameEvent[]
  → groupEventsIntoSteps → animationStore.enqueueSteps
  → AnimationOverlay triggers particle VFX + sound effects per step
  → useGameLoop() waits for animations → auto-advances phases → delegates opponent turns to controller
```

### Components (`src/components/`)

Organized by domain: `board/`, `card/`, `hand/`, `combat/`, `phase/`, `targeting/`, `animation/`, `effects/`, `hero/`, `layout/`, `ui/`. The `ui/` subdirectory has screens (title, deck builder, game over, etc.).

### Learning (`src/learning/`) — Educational challenge system

- **`content.ts`** / **`mathCurriculum.ts`** / **`readingCurriculum.ts`** — Curriculum data and challenge content by subject and grade level.
- **`domain/`** — Domain logic: `cadencePolicy` (when to show challenges), `promptSelectionPolicy` (which prompt to pick), `rewardPolicy` (bonus cards for correct answers), `masteryModel` (spaced-repetition progression).
- **`onboarding.ts`** — First-run learning setup flow (subject/grade selection).
- **`config.ts`** — Learning feature configuration and toggles.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matthewevans/alchemy](https://github.com/matthewevans/alchemy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
