---
trigger: always_on
description: Global rules always followed
---

# Vibe Coding Starter Pack: 3D Multiplayer - Technical Guide

This document describes the architecture, setup, workflow, and key development patterns for the Vibe Coding Starter Pack 3D Multiplayer project. **SpacetimeDB v2.0.1** is used throughout.

## 1. Project Overview

*   **Goal:** Provide a foundation for building 3D multiplayer web games using SpacetimeDB as the backend.
*   **Backend:** SpacetimeDB module written in Rust (`server/`). Handles game logic, state management, and data persistence. Compiled to WASM.
*   **Frontend:** Client application built with Vite, React 19, TypeScript, and React Three Fiber (`client/`). Connects to SpacetimeDB via WebSocket, subscribes to data, calls reducers, and renders the game state.

## 2. Game Architecture

The starter pack uses a modular, component-based architecture with the following core systems:

1. **Core Game Loop**: Managed in `client/src/App.tsx`, which initializes all components and runs the input send cycle at 20Hz
2. **Rendering System**: Handled by Three.js via React Three Fiber with scene setup in `client/src/components/GameScene.tsx`
3. **Character System**: Player controls, FBX model loading, animations, and client-side prediction in `client/src/components/Player.tsx`
4. **UI System**: HUD elements and feedback in `client/src/components/PlayerUI.tsx` and `client/src/components/DebugPanel.tsx`
5. **Multiplayer System**: Client-server communication in `client/src/App.tsx` and server-side code in `server/src/` directory
6. **Load Testing**: Bot simulation in `client/src/simulation.ts` for stress-testing with configurable bot count

## 3. Core SpacetimeDB v2 Concepts

*   **Tables:** Relational data storage defined as Rust structs with `#[spacetimedb::table]`. Key tables include `player` and `logged_out_player`. Tables must be marked `public` for client access. The `name = "..."` attribute defines the SQL table name; the `accessor = "..."` attribute defines the Rust accessor name used in `ctx.db`. Table names in client SQL subscriptions are **case-sensitive** (e.g., `name = "player"` requires `SELECT * FROM player`).
*   **Reducers:** Atomic, transactional Rust functions (`#[spacetimedb::reducer]`) that modify table state. Triggered by client calls or internal events. Key reducers: `register_player`, `update_player_input`, lifecycle reducers (`identity_connected`, `identity_disconnected`).
*   **Subscriptions:** Clients subscribe to SQL queries (e.g., `SELECT * FROM player`) using `conn.subscriptionBuilder()`. **IMPORTANT:** Register `.onApplied()` and `.onError()` callbacks *before* calling `.subscribe()` — in v2, backfill data fires immediately on subscribe.
*   **Generated Bindings:** The `spacetime generate` command creates TypeScript code (`client/src/generated/`) based on the Rust module's schema, providing type-safe access to tables, reducers, and types on the client. Types are in `generated/types.ts`.
*   **Identity:** Represents a unique, authenticated user. In v2 reducers: `ctx.sender()` (method call with parentheses). On client: `conn.identity`. Used as the primary key for player-related tables.

## 4. Prerequisites

1.  **Rust & Cargo:** ([https://rustup.rs/](mdc:https:/rustup.rs))
    *   Version: **1.93+** required for SpacetimeDB 2.0.1
    *   Install: `curl https://sh.rustup.rs -sSf | sh`
    *   Add WASM target: `rustup target add wasm32-unknown-unknown`
    *   Ensure `~/.cargo/bin` is in PATH (restart terminal or `source ~/.cargo/env`).
2.  **SpacetimeDB CLI (v2.0.1):** ([https://install.spacetimedb.com](mdc:https:/install.spacetimedb.com))
    *   Install: `curl -sSf https://install.spacetimedb.com | sh`
    *   Ensure install location (e.g., `~/.local/bin`) is in PATH (restart terminal or `source ~/.zshrc`/`.bashrc`).
    *   Verify: `spacetime version` (Should show `2.0.1` or compatible).
3.  **Node.js 22+ & npm:** Install via [nvm](mdc:https:/github.com/nvm-sh/nvm) (`nvm install 22`).
4.  **(Optional) `wasm-opt`:** For optimizing the built Rust module (part of `binaryen`). Install via system package manager (e.g., `brew install binaryen`, `apt install binaryen`). If missing, builds will still work but show a warning.

## 5. Project Structure

```
vibe-coding-starter-pack-3d-multiplayer/
├── client/                   # Vite+React+TS client
│   ├── public/
│   │   └── models/           # FBX character models (wizard/, paladin/)
│   ├── src/
│   │   ├── components/       # GameScene, Player, DebugPanel, PlayerUI, JoinGameDialog
│   │   ├── generated/        # Auto-generated TS bindings (DO NOT EDIT)
│   │   ├── App.css           # Component styles
│   │   ├── App.tsx           # Main React component — connection, input, game loop
│   │   ├── simulation.ts     # Bot load-testing tool
│   │   ├── index.css         # Global styles/resets
│   │   └── main.tsx          # React entry point
│   ├── index.html
│   ├── package.json
│   ├── tsconfig.json         # Main TS config
│   ├── tsconfig.app.json     # App-specific TS config (used by Vite)
│   └── vite.config.ts
├── server/                   # SpacetimeDB Server Module (Rust)
│   ├── src/
│   │   ├── common.rs         # Shared types (Vector3, InputState), constants
│   │   ├── player_logic.rs   # Server-side movement calculation
│   │   └── lib.rs            # Tables, reducers, lifecycle handlers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Glavin001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-13 -->
