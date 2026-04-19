---
trigger: always_on
description: - Canvas 2D + `requestAnimationFrame`
---

# Space Trader Development Notes

## Stack

- Electron
- React + TypeScript
- Vite
- Canvas 2D + `requestAnimationFrame`

## Project Layout

### Electron Shell

- `electron/main.ts`
- `electron/preload.ts`

### React Entrypoints

- `src/main.tsx`
- `src/App.tsx`
- `src/App.css`
- `src/GameCanvas.tsx`

### Core Runtime

- `src/game/runtime.ts`
  - persistent runtime singleton
  - authoritative mutable game state
  - animation queue
  - pending targeting flow
  - local vs network command routing
  - worker-backed bot decisions
  - debug helpers
  - active content-selection + runtime-profile context
- `src/game/systems.ts`
  - `updateGame`
  - `renderGame`
- `src/game/types.ts`
  - frame, viewport, and animation types
- `src/game/derived.ts`
  - cached derived state keyed off runtime version
- `src/game/presentation.ts`
  - player/resource/faction presentation helpers

### Model Layer

- `src/game/model/state.ts`
  - canonical `GameState`
  - entity shapes
  - zones
  - initial-state creation
  - dynamic player-order bootstrap
  - elimination tracking
  - resource and runtime-profile aware defaults
  - deterministic generated-id counter
- `src/game/model/enums.ts`
  - phases, factions, resources, unit roles
  - note: factions/resources are dynamic string ids populated by loaded content
- `src/game/model/ids.ts`
  - typed IDs and player constants
- `src/game/model/hex.ts`
  - axial hex math
- `src/game/model/queries.ts`
  - state query helpers
- `src/game/model/selectors.ts`
  - UI selectors
- `src/game/model/migrations.ts`
  - state migration / hot-state repair
- `src/game/random/seeded.ts`
  - seeded RNG helpers for deterministic match setup

### Content System

- `src/game/content/loader.ts`
  - explicit set loading
  - registry reset / reload lifecycle
  - default built-in content initialization via `initializeDefaultContent()`
- `src/game/content/sets/catalog.ts`
  - built-in set manifest catalog
  - built-in set id lookup
  - default built-in selection (`alpha`, which depends on `foundation`)
- `src/game/content/registry.ts`
  - registered sets, cards, stack effects, factions, resources, maps, deck recipes, runtime profiles
- `src/game/content/cards/builders.ts`
  - shared card authoring helpers used across sets
- `src/game/content/sets/types.ts`
  - set manifests
  - faction/resource/runtime-profile module types
- `src/game/content/sets/foundation/**`
  - cardless shared gameplay foundation
  - shared stack effects, play effects, AI scoring, and runtime installers
- `src/game/content/sets/alpha/**`
  - first real playable set content and installers
  - depends on `foundation`

### Content Facades

- `src/game/content/cards/catalog.ts`
  - registry-backed card access
  - card metadata helpers
- `src/game/content/cards/types.ts`
  - generic card definition types
  - generic play-effect config / modifier shapes
- `src/game/content/stackEffects.ts`
  - registry-backed stack-effect access
- `src/game/content/stackEffects/types.ts`
  - generic stack behavior types
- `src/game/content/decks/starterDecks.ts`
  - starter deck access + validation
- `src/game/content/maps/catalog.ts`
  - registry-backed map access
- `src/game/content/mechanics/stateAccess.ts`
  - helper for namespaced mechanic state

### Mechanics

- `src/game/mechanics/index.ts`
  - generic mechanic-state lifecycle
- `src/game/content/sets/alpha/mechanics/**`
  - current live Alpha mechanics
  - `stealth`
  - `sprout`
  - `relay`
  - `surge`
  - `bloom`
  - `salvage`
  - `bastion`
  - `predation`
  - `emplaced`
  - `uncounterable`

### Registries

- `src/game/registries/triggerConditions.ts`
- `src/game/registries/autoTargets.ts`
- `src/game/registries/instructionHandlers.ts`
- `src/game/registries/playEffects.ts`
- `src/game/registries/cardPlayModifiers.ts`
- `src/game/registries/cardCounterability.ts`
- `src/game/registries/cardResolveAnimations.ts`
- `src/game/registries/directInteraction.ts`
- `src/game/registries/cascadeBranches.ts`
- `src/game/registries/combatHooks.ts`
- `src/game/registries/unitDeployment.ts`
- `src/game/registries/unitStatHooks.ts`
- `src/game/registries/mechanicInstructions.ts`
- `src/game/registries/mechanicAnimations.ts`
- `src/game/registries/mechanicState.ts`
- `src/game/registries/mechanicApis.ts`
- `src/game/registries/spellScoring.ts`
- `src/game/registries/stackEffectMagnitudes.ts`
- `src/game/registries/stackResolveAnimations.ts`
- `src/game/registries/stackPreviews.ts`
- `src/game/registries/boardBlastEffects.ts`
- `src/game/registries/debugStackResponses.ts`
- `src/game/registries/presentation.ts`

### Actions Pipeline

- `src/game/actions/commands.ts`
- `src/game/actions/events.ts`
- `src/game/actions/reducers.ts`
- `src/game/actions/instructions.ts`
- `src/game/actions/instructionHandlers.ts`
- `src/game/actions/handlers/cards.ts`
- `src/game/actions/handlers/combat.ts`
- `src/game/actions/handlers/phase.ts`
- `src/game/actions/handlers/selection.ts`

### Rules

- `src/game/rules/validators.ts`
  - command legality
- `src/game/rules/cardPlayOptions.ts`
  - shared legal-target enumeration
- `src/game/rules/cardPlayLegality.ts`
  - playability/stack targeting helpers

### Turn Management

- `src/game/turn/playerOrder.ts`
- `src/game/turn/phaseMachine.ts`
- `src/game/turn/stack.ts`
- `src/game/turn/autoFlow.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/williamcotton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
