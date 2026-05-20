---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Workflow Orchestration

### 1. Plan Node Default
- Enter plan mode for ANY non-trivial task (3+ steps or architectural decisions)
- If something goes sideways, STOP and re-plan immediately - don't keep pushing
- Use plan mode for verification steps, not just building
- Write detailed specs upfront to reduce ambiguity

### 2. Subagent Strategy
- Use subagents liberally to keep main context window clean
- Offload research, exploration, and parallel analysis to subagents
- For complex problems, throw more compute at it via subagents
- One tack per subagent for focused execution

### 3. Self-Improvement Loop
- After ANY correction from the user: update `tasks/lessons.md` with the pattern
- Write rules for yourself that prevent the same mistake
- Ruthlessly iterate on these lessons until mistake rate drops
- Review lessons at session start for relevant project

### 4. Verification Before Done
- Never mark a task complete without proving it works
- Diff behavior between main and your changes when relevant
- Ask yourself: "Would a staff engineer approve this?"
- Run tests, check logs, demonstrate correctness

### 5. Demand Elegance (Balanced)
- For non-trivial changes: pause and ask "is there a more elegant way?"
- If a fix feels hacky: "Knowing everything I know now, implement the elegant solution"
- Skip this for simple, obvious fixes - don't over-engineer
- Challenge your own work before presenting it

### 6. Autonomous Bug Fixing
- When given a bug report: just fix it. Don't ask for hand-holding
- Point at logs, errors, failing tests - then resolve them
- Zero context switching required from the user
- Go fix failing CI tests without being told how

## Task Management

1. **Plan First**: Write plan to `tasks/todo.md` with checkable items
2. **Verify Plan**: Check in before starting implementation
3. **Track Progress**: Mark items complete as you go
4. **Explain Changes**: High-level summary at each step
5. **Document Results**: Add review section to `tasks/todo.md`
6. **Capture Lessons**: Update `tasks/lessons.md` after corrections

## Core Principles

- **Simplicity First**: Make every change as simple as possible. Impact minimal code.
- **No Laziness**: Find root causes. No temporary fixes. Senior developer standards.
- **Minimat Impact**: Changes should only touch what's necessary. Avoid introducing bugs.

## Project Overview

Layerz Wallet is a Bitcoin-focused multi-platform wallet supporting multiple Layer 2 solutions. It's architected as a monorepo with three main directories:

- **`mobile/`** - React Native (Expo) app for iOS/Android
- **`ext/`** - Chrome browser extension (popup + background script)
- **`shared/`** - Shared business logic, wallet implementations, network abstractions, and types

Both `mobile/` and `ext/` import from `shared/` via symbolic links (`src/shared-link -> ../../shared`), ensuring a single source of truth for all wallet logic.

## Development Commands

### Extension (`ext/`)

```bash
cd ext
source ../../env.sh      # Enviroment file
npm install              # Install dependencies
npm start                # Start dev server (builds and watches)
npm run build            # Production build
npm run unit             # Run unit tests (vitest)
npm run integration      # Run integration tests (vitest)
npm run e2e              # Run end-to-end tests (Playwright)
npm run test             # Run all tests + circular dependency check
npm run lint             # Check code formatting and types
npm run lint:fix         # Auto-fix linting issues
npm run circular         # Check for circular dependencies
```

Load extension in Chrome:
1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked extension"
4. Select the `ext/build` folder

### Mobile (`mobile/`)

```bash
cd mobile
source ../../env.sh      # Enviroment file
npm install              # Install dependencies
npm start                # Start Expo dev server
npm run android          # Run on Android
npm run ios              # Run on iOS
npm run unit             # Run unit tests (vitest)
npm run integration      # Run integration tests (vitest)
npm run e2e              # Run Maestro e2e tests
npm run lint             # Check code formatting and types
npm run lint:fix         # Auto-fix linting issues
```

EAS builds:
If you are on macos prefer ios
Android: 
- Development build: `eas build --platform android --profile development-simulator --local`
- Preview build: `eas build --platform android --profile preview --local`
iOS:
- Development build: `eas build --platform ios --profile development-simulator --local`
- Preview build: `eas build --platform ios --profile preview --local`

## Architecture

### Monorepo Structure

Each platform implements platform-specific adapters while sharing core logic:

- **Storage**: Both implement `IStorage` interface
  - `mobile`: AsyncStorage (LayerzStorage) + Expo SecureStore (SecureStorage)
  - `ext`: chrome.storage.local for both LayerzStorage and SecureStorage

- **Background Execution**:
  - `mobile`: Direct execution (single JS context)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [layerztec/layerzwallet](https://github.com/layerztec/layerzwallet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
