---
trigger: always_on
description: **Generated:** 2026-03-23
---

# BBPlayer Project Knowledge Base

**Generated:** 2026-03-23
**Project:** BBPlayer - Bilibili Audio Player (React Native)
**Repository:** https://github.com/bbplayer-app/bbplayer

---

## OVERVIEW

BBPlayer is a local-first Bilibili audio player built with React Native and Expo. It features offline playback, lyrics support (SPL format), Bilibili integration, and Material Design 3 UI.

**Core Stack:**

- React Native 0.83.2 + Expo 55 + React 19
- TypeScript with project references
- pnpm workspaces (monorepo)
- Zustand (state) + TanStack Query (data)
- Drizzle ORM + expo-sqlite
- Material Design 3 (React Native Paper)

---

## STRUCTURE

```
.
├── apps/
│   ├── mobile/          # Main React Native app (Expo)
│   ├── backend/         # Cloudflare Workers API (Hono)
│   └── docs/            # VitePress documentation
├── packages/
│   ├── orpheus/         # Native audio module (Media3/AVFoundation)
│   ├── splash/          # Lyric parser (SPL format)
│   ├── image-theme-colors/  # Color extraction
│   ├── logs/            # Logging utility
│   ├── heatmap/         # Audio visualization
│   └── eslint-plugin/   # Custom ESLint rules
├── .agent/              # AI agent rules & skills
└── .github/workflows/   # CI/CD
```

---

## WHERE TO LOOK

| Task                | Location                         | Notes                          |
| ------------------- | -------------------------------- | ------------------------------ |
| **Mobile Screens**  | `apps/mobile/src/app/`           | Expo Router file-based routing |
| **UI Components**   | `apps/mobile/src/components/`    | Shared components              |
| **Feature Modules** | `apps/mobile/src/features/`      | Domain-organized features      |
| **Global State**    | `apps/mobile/src/hooks/stores/`  | Zustand stores                 |
| **API Calls**       | `apps/mobile/src/hooks/queries/` | TanStack Query hooks           |
| **Business Logic**  | `apps/mobile/src/lib/`           | Facades, Services, DB          |
| **Audio Player**    | `packages/orpheus/src/`          | Native module entry            |
| **Lyrics Parsing**  | `packages/splash/src/`           | LRC/SPL parser                 |
| **Custom ESLint**   | `packages/eslint-plugin/rules/`  | Project-specific rules         |
| **Documentation**   | `apps/docs/docs/`                | VitePress site                 |

---

## COMMANDS

```bash
# Development
pnpm install                    # Install deps (pnpm only!)
pnpm lefthook install          # Setup git hooks

# Code Quality
pnpm lint                      # oxlint + eslint
pnpm lint:fix                  # Auto-fix
pnpm format                    # oxfmt
pnpm check:deps                # syncpack dependency check

# Mobile App
cd apps/mobile
pnpm android                   # Run Android (dev build required)
pnpm start                     # Start Metro (WITH_ROZENITE=true)
pnpm test                      # Jest tests

# Backend
cd apps/backend
pnpm dev                       # Wrangler dev
pnpm deploy                    # Deploy to Cloudflare

# Native Modules
cd packages/orpheus
pnpm build                     # expo-module build
pnpm test                      # expo-module test
```

---

## CONVENTIONS

### Import Aliases

- **Mobile app:** `@/*` → `./apps/mobile/src/*`
- **Configured in:** `eslint.config.mjs` (via `@dword-design/eslint-plugin-import-alias`)
- **Must use** for all imports in mobile app (not relative paths)

### Linting Stack

| Tool       | Purpose             | Config              |
| ---------- | ------------------- | ------------------- |
| **oxlint** | Primary linter      | `.oxlintrc.json`    |
| **eslint** | Secondary + plugins | `eslint.config.mjs` |
| **oxfmt**  | Formatter           | CLI only            |

### Commit Format

```
<type>(<scope>): <message>

# Types: feat, fix, docs, style, refactor, chore
# Scopes: mobile, backend, docs, orpheus, splash, logs, root
# Example: feat(mobile): add playlist shuffle
```

### Git Hooks (Lefthook)

- **pre-commit:** oxfmt + oxlint + eslint + gitleaks
- **commit-msg:** commitlint validation
- Stage-fixed files auto-committed

### Package Manager

- **ONLY pnpm** - npm/yarn will break workspace resolution
- Version: `pnpm@10.30.3`

---

## ANTI-PATTERNS

### 🚫 NEVER

- Use Expo Go - requires custom dev build (native code)
- Throw errors in business logic - use `neverthrow` Result pattern
- Define `renderItem` inside component - FlashList performance
- Skip `extraData` with `useMemo` for FlashList dependencies
- Use npm/yarn - pnpm only

### ⚠️ CAUTION

- iOS support is minimal ("birth without nurture") - Android focus
- `console.log` is forbidden (error in oxlint) except in packages/
- MMKV migration code exists - don't remove until migration complete
- Multi-P Bilibili videos may have duplicate DB records

### Type Workarounds

- 27 `@ts-expect-error` in codebase (mostly Zustand/MM migrations)
- Each has explanatory comment - understand before modifying
- Key locations: `useAppStore.ts`, `mmkv.ts`, `LyricsControlOverlay.tsx`

---

## UNIQUE STYLES

### Architecture: Facade + Service Pattern

```
UI Layer (app/, features/)
    ↓ calls
Facade Layer (lib/facades/) - orchestrates, manages transactions
    ↓ calls
Service Layer (lib/services/) - single domain logic, DB access
```

### Error Handling

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bbplayer-app/BBPlayer](https://github.com/bbplayer-app/BBPlayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
