---
trigger: always_on
description: Always follow these instructions precisely and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.
---

# Satisfactory Factories Development Instructions

Always follow these instructions precisely and only fallback to additional search and context gathering if the information here is incomplete or found to be in error.

## Project Overview

Satisfactory Factories is a Vue 3 + TypeScript web application for planning production chains in the game Satisfactory. The project has three main components:
- **Web**: Vue 3 frontend with Pinia state management and Vitest testing
- **Parsing**: TypeScript tool to process game data from Satisfactory
- **Backend**: Express.js API for authentication and data synchronization

## Build & Development Requirements

### Prerequisites
- **Node.js**: Version 20.17+ (current: 20.19.4) - Use `nvm install 20.17 && nvm use 20.17`
- **pnpm**: Version 9.14.4+ - Install with `npm install -g pnpm`
- **Docker**: Required for backend database operations
- **NEVER CANCEL builds or long-running commands** - Build processes can take 5+ minutes

### Component Build Process & Timing

**CRITICAL: Always set timeout values of 300+ seconds for build commands and 600+ seconds for test commands. NEVER CANCEL long-running operations.**

#### Web Component (Primary Interface)
```bash
cd web
pnpm install    # Takes ~22 seconds
pnpm lint-check # Takes ~6 seconds (has 4 warnings - acceptable)
pnpm build      # Takes ~17 seconds - NEVER CANCEL, wait for completion
pnpm test       # Takes ~20 seconds - NEVER CANCEL, runs 413 tests with coverage
pnpm dev        # Starts dev server on http://localhost:3000
```

#### Parsing Component (Game Data Processing)
```bash
cd parsing
pnpm install    # Takes ~8 seconds
pnpm build      # Takes ~3 seconds
pnpm test       # Takes ~9 seconds - NEVER CANCEL, runs 23 tests with 94.7% coverage
pnpm lint-check # Takes ~2 seconds (has warnings about TypeScript 'any' usage - acceptable)
```

#### Backend Component (API Server)
```bash
cd backend
pnpm install    # Takes ~6 seconds
pnpm build      # Takes ~3 seconds
pnpm lint-check # Takes ~2 seconds (has 3 warnings about 'any' usage - acceptable)
# Note: Backend has no tests currently
```

## Development Workflow

### Starting Development
1. **Always build and test first**: Run the complete validation sequence before making changes
2. **Web development**: Use `cd web && pnpm dev` to start the development server
3. **Manual testing**: Always test actual functionality - load the demo plan to validate core features

### Code Standards & Conventions
- **TypeScript Only**: All new files must use `.ts` or `.vue` with `<script setup lang="ts">`
- **Package Manager**: Use `pnpm` ONLY - never use `npm install`
  - Do not, under **ANY** circumstances, create a `package-lock.json` file in your final code. We use `pnpm` here, we do not use `npm`. Add a step to ensure you are not introducing this file, or your PR will be rejected.
- **Linting**: Run `pnpm lint` in any component before committing (CI will block PRs with lint failures)
- **Testing**: New code should have tests, especially in parsing component (mandatory 90%+ coverage)
- **Code Style**: 2 spaces indentation, LF line endings, newline at EOF
- **Commits**: Use conventional commit structure (e.g., `feat:`, `fix:`, `docs:`) with issue references

### Manual Validation Requirements
After making changes, ALWAYS validate by:
1. Running the web application (`cd web && pnpm dev`)
2. Loading the demo plan via "Start with a demo plan" button
3. Verifying complex factory calculations work correctly
4. Testing factory interactions (products, imports, satisfaction)

### Essential Commands Reference

**Build All Components:**
```bash
# Web (17s build + 20s tests) - NEVER CANCEL
cd web && pnpm install && pnpm build && pnpm test

# Parsing (3s build + 9s tests) - NEVER CANCEL  
cd parsing && pnpm install && pnpm build && pnpm test

# Backend (3s build) - NEVER CANCEL
cd backend && pnpm install && pnpm build
```

**Pre-commit Validation:**
```bash
cd web && pnpm lint-check && pnpm build && pnpm test
cd parsing && pnpm lint-check && pnpm build && pnpm test  
cd backend && pnpm lint-check && pnpm build
```

## Project Structure & Key Locations

### Web Component (`/web`)
- **Main Interface**: Vue 3 SPA with complex factory planning functionality
- **State Management**: Pinia stores in `src/stores/`
- **Components**: Vue SFCs in `src/components/` (composition API + TypeScript)
- **Tests**: Vitest files with `.spec.ts` extension throughout `src/`
- **Key Files**: 
  - `src/stores/app-store.ts` - Main application state
  - `src/utils/factory-management/` - Core calculation logic
  - `src/components/planner/` - Primary UI components

### Parsing Component (`/parsing`) 
- **Purpose**: Processes `Docs.json` from Satisfactory game into usable format
- **Critical Component**: MUST have 90%+ test coverage (currently 94.7%)
- **Output**: `gameData.json` used by web component
- **Key Files**:
  - `src/parts.ts`, `src/recipes.ts`, `src/buildings.ts` - Core parsing logic
  - `tests/` - Comprehensive test suite (mandatory for all changes)

### Backend Component (`/backend`)
- **Purpose**: Authentication and data sync API
- **Optional**: Not required for local development  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [satisfactory-factories/application](https://github.com/satisfactory-factories/application) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
