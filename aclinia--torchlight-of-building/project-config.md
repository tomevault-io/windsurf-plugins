---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

TanStack Start + Vite + React 19 + TypeScript character build planner for Torchlight Infinite.

## Commands

```bash
pnpm dev          # Development server (http://localhost:3000)
pnpm build        # Production build
pnpm test         # Run all tests
pnpm test <file>  # Single test file
pnpm typecheck    # TypeScript type checking
pnpm check        # Biome linting and formatting
```

## Stack

- TanStack Start, Vite, React 19, TypeScript (strict), Tailwind CSS 4, Vitest
- Utilities: `remeda` (lodash-like), `ts-pattern` (pattern matching), `fflate` (compression)

## Project Structure

```
src/routes/              # TanStack Router file-based routes
├── __root.tsx           # Root layout
├── index.tsx            # Home page (/)
├── builder.tsx          # Builder layout (/builder) - loads save, renders Outlet
└── builder/             # Nested builder routes (each section is a route)
    ├── index.tsx        # Redirect → /builder/equipment
    ├── equipment.tsx    # Equipment section
    ├── talents.tsx      # Talents layout (renders Outlet)
    ├── talents/
    │   ├── index.tsx    # Redirect → /builder/talents/slot_1
    │   └── $slot.tsx    # Dynamic slot param (slot_1, slot_2, slot_3, slot_4)
    ├── skills.tsx       # Skills section
    ├── hero.tsx         # Hero section
    ├── pactspirit.tsx   # Pactspirit section
    ├── divinity.tsx     # Divinity section
    ├── configuration.tsx # Configuration section
    └── calculations.tsx # Calculations section

src/components/          # Feature-organized React components
├── builder/             # BuilderLayout, StatsPanel
├── calculations/        # ModRow, ModGroup, StatBreakdown, SkillSelector
├── configuration/       # ConfigField, NumberInput, ConfigurationTab
├── divinity/            # DivinityTab, DivinityGrid, SlateCrafter, SlateInventory
├── equipment/           # EditGearModal, InventoryItem, AffixSlotComponent
├── hero/                # HeroTab, HeroSelector, MemoryInventory, TraitSelector
├── modals/              # ImportModal, ExportModal, DestinySelectionModal
├── pactspirit/          # PactspiritTab, PactspiritColumn, RingSlot
├── skills/              # SkillSlot, SupportSkillSelector, SkillTooltipContent
├── talents/             # TalentGrid, PrismSection, InverseImageSection, CoreTalentSelector
├── ui/                  # Modal, Tooltip, SearchableSelect, NumberInput
├── PageTabs.tsx         # Main navigation tabs
├── SavesTab.tsx         # Save management UI
├── DebugPanel.tsx       # Debug panel component
└── Toast.tsx            # Toast notification component

src/stores/              # Zustand state management
├── builderStore/        # Main persisted store
│   ├── internal.ts      # Zustand store with persist middleware
│   ├── hooks.ts         # Public selectors (useLoadout, useBuilderState)
│   ├── selectors.ts     # Selector functions
│   ├── raw-access.ts    # Explicit raw access (useSaveDataRaw)
│   └── index.ts         # Public exports only
├── equipmentUIStore.ts  # Equipment crafting/preview state
├── talentsUIStore.ts    # Prism/inverse image crafting state
├── skillsUIStore.ts     # Skills UI state
├── heroUIStore.ts       # Hero UI state
├── divinityUIStore.ts   # Divinity UI state
└── pactspiritUIStore.ts # Pactspirit UI state

src/lib/                 # Utilities & types
├── schemas/             # Zod schemas for validation
│   ├── save-data.schema.ts
│   ├── gear.schema.ts
│   ├── skill.schema.ts
│   ├── talent.schema.ts
│   ├── hero.schema.ts
│   ├── divinity.schema.ts
│   ├── pactspirit.schema.ts
│   ├── config.schema.ts
│   ├── common.schema.ts
│   └── index.ts
├── save-data.ts         # SaveData types and factory functions
├── build-code.ts        # Build code encoding/decoding
├── types.ts             # Shared types
├── constants.ts         # App constants
├── storage.ts           # localStorage helpers
└── *-utils.ts           # Feature-specific utilities

src/hooks/               # Custom React hooks
├── useAffixSelection.ts # Affix selection logic
└── useTooltip.ts        # Tooltip positioning

src/tli/                 # Game engine (pure TypeScript, no React)
├── core.ts              # Base types (Gear, HeroMemory, etc.)
├── mod.ts               # Mod type definitions
├── constants.ts         # Engine constants
├── gear-data-types.ts   # Gear data type definitions
├── talent-tree.ts       # Talent tree utilities
├── talent-affix-utils.ts # Talent affix parsing
├── all-affixes.ts       # All affix aggregation
├── mod-parser/          # Template-based mod parsing system
│   ├── index.ts         # Public API exports
│   ├── compiler.ts      # Template → regex compiler
│   ├── template.ts      # Template parsing and matching
│   ├── templates.ts     # All mod templates
│   ├── template-types.ts # Template type definitions
│   ├── type-registry.ts # Mod type registry
│   ├── types.ts         # Core parser types
│   ├── enums.ts         # Parser enums
│   └── README.md        # Parser documentation
├── calcs/               # Calculation engine
│   ├── offense.ts       # DPS calculations
│   ├── damage-calc.ts   # Damage calculation helpers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aclinia/torchlight-of-building](https://github.com/aclinia/torchlight-of-building) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
