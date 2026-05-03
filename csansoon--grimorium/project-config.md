---
trigger: always_on
description: This is a **Blood on the Clocktower** storyteller companion app built with React 18, TypeScript, Vite, Tailwind CSS, and Radix UI. It guides the storyteller through all game phases — role revelation, night actions, day discussion, nominations, voting, and game-over. The storyteller controls the device at all times and shows the screen to players when needed (no device passing).
---

# Grimorium — Agent Manual

This is a **Blood on the Clocktower** storyteller companion app built with React 18, TypeScript, Vite, Tailwind CSS, and Radix UI. It guides the storyteller through all game phases — role revelation, night actions, day discussion, nominations, voting, and game-over. The storyteller controls the device at all times and shows the screen to players when needed (no device passing).

This document explains every system in the codebase, how they interconnect, and the patterns you must follow when implementing new features.

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Event-Sourced Game State](#2-event-sourced-game-state)
3. [The Game Controller](#3-the-game-controller)
4. [Roles](#4-roles) — includes [Night Action Steps](#night-action-steps-nightsteplistlayout), [Setup Actions](#setup-actions-pre-revelation)
5. [Effects](#5-effects) — includes `canRegisterAs` for misregistration, [Malfunction System](#the-malfunction-system)
6. [The Intent Pipeline](#6-the-intent-pipeline)
7. [The Perception System](#7-the-perception-system) — includes [Perception Query Utilities](#perception-query-utilities), [PerceptionConfigStep](#perceptionconfigstep-component), [Auto-Calc Perception Flow](#how-auto-calculating-roles-use-perception-overrides)
8. [Day Actions](#8-day-actions)
9. [Night Follow-Ups](#9-night-follow-ups)
10. [Win Conditions](#10-win-conditions)
11. [The Screen State Machine](#11-the-screen-state-machine)
12. [UI Components](#12-ui-components) — includes `NightStepListLayout`, `PerceptionConfigStep`
13. [Internationalization (i18n)](#13-internationalization-i18n)
14. [How to Implement a New Role](#14-how-to-implement-a-new-role)
15. [How to Implement a New Effect](#15-how-to-implement-a-new-effect)
16. [How to Add a New Intent Type](#16-how-to-add-a-new-intent-type)
17. [Testing](#17-testing)
18. [Rules and Anti-Patterns](#18-rules-and-anti-patterns)

---

## 1. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      GameScreen.tsx                          │
│            (Screen state machine + UI orchestrator)          │
│                            │                                │
│    ┌───────────────────────┼──────────────────────────┐     │
│    │                       │                          │     │
│    ▼                       ▼                          ▼     │
│  Screens               game.ts                   Pipeline   │
│  (DayPhase,         (Game controller)          (Intent      │
│   NightAction,         │                      resolution,   │
│   Voting, ...)         │                      Perception)   │
│                        │                          │         │
│                        ▼                          │         │
│                   Event-sourced state             │         │
│                   (Game.history)                  │         │
│                        ▲                          │         │
│                        │                          │         │
│              ┌─────────┼─────────┐               │         │
│              │         │         │               │         │
│              ▼         ▼         ▼               ▼         │
│           Roles    Effects    Teams           Resolvers     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### Core Principles

1. **Event Sourcing** — Game state is never mutated directly. All changes are appended as `HistoryEntry` objects to `game.history`. Each entry contains an immutable `stateAfter` snapshot.

2. **Modularity through Pipelines** — Roles and effects do not reference each other's logic. Interactions happen through the **Intent Pipeline**: roles emit intents ("I want to kill this player"), and effects register handlers to intercept/modify/prevent them.

3. **Perception over Identity** — Information roles never check a player's actual role directly. They use the **Perception System** (`perceive()`), which allows effects to modify how a player is perceived (e.g., Recluse registering as evil).

4. **Effects carry behavior** — Roles are thin. Passive abilities live on **Effects** (attached to players at game start via `initialEffects`). Effects declare intent handlers, day actions, win conditions, and perception modifiers.

5. **No hardcoded role logic in the game controller** — `game.ts` knows nothing about individual roles. It orchestrates the game loop generically.

---

## 2. Event-Sourced Game State

**File:** `src/lib/types.ts`

### Core Types

```typescript
type Game = {
    id: string;
    name: string;
    createdAt: number;
    history: HistoryEntry[];    // The entire game state is here
};

type HistoryEntry = {
    id: string;
    timestamp: number;
    type: EventType;
    message: RichMessage;           // i18n-ready display messages

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [csansoon/grimorium](https://github.com/csansoon/grimorium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
