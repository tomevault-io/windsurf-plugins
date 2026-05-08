---
trigger: always_on
description: This file contains the general rules, principles, and canonical definitions for the AI-Empowered Governance Apps browser extension. As an AI assistant, you MUST adhere to these guidelines when reviewing, refactoring, or generating code for this project.
---

# `.cursorrules` for GyroGovernance Browser Extension

This file contains the general rules, principles, and canonical definitions for the AI-Empowered Governance Apps browser extension. As an AI assistant, you MUST adhere to these guidelines when reviewing, refactoring, or generating code for this project.

## 1. 🌍 Core Principles & Philosophy

- **Mission:** The extension's primary goal is to democratize access to sophisticated AI model evaluation. It translates the complex, mathematical **GyroDiagnostics** framework into an accessible, user-friendly tool.
- **Platform Agnostic:** The core workflow MUST remain clipboard-based and platform-agnostic. We do not use API keys. The user brings their own access to AI models (ChatGPT, Claude, etc.).
- **Openness & Community:** All generated insights and evaluation data are intended for a public knowledge commons, licensed under CC0. The code is open-source.

## 2. 📖 Canonical Terminology & Concepts (Non-Negotiable)

These terms are part of the core methodology and MUST NOT be changed, renamed, or reinterpreted.

- **GyroDiagnostics:** The name of the entire evaluation methodology.
- **Common Governance Model (CGM):** The underlying mathematical-physics theory.
- **The Three "P"s:** This is the canonical user flow.
    1.  **Participation:** Selecting or defining a governance challenge.
    2.  **Preparation:** The synthesis and analysis process (epochs, analysts).
    3.  **Provision:** The final generated report/insight.
- **Core Metrics:**
    - **Quality Index (QI):** The weighted overall performance score (%).
    - **Alignment Rate (AR):** The temporal efficiency metric. Its units are **quality points per minute (`/min`)**. Its categories (`VALID`, `SUPERFICIAL`, `SLOW`) are fixed.
    - **Superintelligence Index (SI):** The structural coherence metric derived from K₄ graph topology and the target aperture **A* ≈ 0.02070**.
- **Process Steps:**
    - **Epochs:** The two distinct 6-turn synthesis phases. Always refer to them as `Epoch 1` and `Epoch 2`.
    - **Turns:** Each epoch consists of exactly 6 turns.
    - **Synthesizer:** The AI model generating responses during an epoch.
    - **Analyst:** The AI model evaluating the transcripts. There are always two (`Analyst 1`, `Analyst 2`).
- **Pathologies:** The five canonical failure modes are fixed: `sycophantic_agreement`, `deceptive_coherence`, `goal_misgeneralization`, `superficial_optimization`, `semantic_drift`.

## 3. 🏗️ Code & Architecture Style

- **Single Source of Truth (SSoT):** The application state (`NotebookState`) MUST treat `sessions: Session[]` and `activeSessionId: string` as the source of truth for all in-progress and completed evaluations.
    - **AVOID** duplicating session data into legacy top-level fields like `challenge`, `epochs`, `analysts`. These are being phased out and should be replaced completely so we dont have dead code or compatibility bloat.
    - **ALWAYS** derive the active session's data using a helper function like `getActiveSession(state)`.
- **Atomic Storage Operations:**
    - All operations that modify stored data (e.g., creating, updating, or deleting sessions/insights in `storage.ts`) should be atomic.
    - Prefer functions that perform the storage write and then return the complete, updated `NotebookState` to prevent UI/storage desynchronization.
- **No Polling for State Sync:**
    - **DO NOT** use `setInterval` or `setTimeout` to periodically refresh state.
    - **ALWAYS** use the `chrome.storage.onChanged` listener to react to changes from other tabs or contexts. This is the correct pattern for browser extensions.
- **Modular & Extracted Logic:**
    - **DO NOT** embed complex business logic (calculations, formatting, progress tracking) directly inside React components.
    - **ALWAYS** extract logic into utility files in the `src/lib/` directory (e.g., `session-utils.ts`, `ui-utils.ts`, `report-generator.ts`).
- **App-Based Architecture:**
    - The UI is structured into "Apps": `WelcomeApp`, `ChallengesApp`, `JournalApp`, `InsightsApp`.
    - `Notebook.tsx` is the top-level router that switches between these apps.
    - Each app manages its own internal views (e.g., `ChallengesApp` routes between `TypeSelector`, `SDGGallery`, etc.).
- **Robust Asynchronous Operations:**
- **Storage API Patterns:**
    - Use `sessions.*` for session CRUD operations (create, createMany, update, delete, clone, getById)
    - Use `storage.*` only for global state updates (UI navigation, suite tracking, etc.)
    - Use `insights.*` for insights library operations
    - Use `drafts.*` for temporary draft management
    - **NEVER** mix these APIs in a single operation - one operation should use one API family
    - **ALWAYS** use session helpers (`getActiveSession`, `getSessionById`) instead of manual `state.sessions.find()`

## 4. ✨ User Experience (UX) & Flow

- **From Form to Workspace:** The primary UX goal is to transform the extension from a rigid, linear form into a flexible, multi-session workspace with what we call Apps (inner sections).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gyrogovernance/apps](https://github.com/gyrogovernance/apps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
