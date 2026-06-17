---
trigger: always_on
description: Provides type-safe communication with the main process. Used for:
---

# AGENTS.md

This document provides guidance for AI agents and developers using agentic workflows on the Matter Controller project.

## Project Overview

**Matter Controller** is an Electron + React desktop application for controlling Matter protocol home automation devices. It provides a user-friendly interface for managing accessories, scenes, rooms, and settings with multi-language support (English & Spanish).

**Tech Stack:**
- Frontend: React 19.2.6, TypeScript 5.9.3
- State Management: Zustand 5.0.14
- Internationalization: i18next 26.3.0, react-i18next 17.0.8
- Build: Electron Vite 3.1.0, electron-builder 26.8.1
- Desktop Runtime: Electron 42.3.0
- Auto-Updater: electron-updater 6.6.2 (GitHub releases)
- CI/CD: GitHub Actions (build, bump-version, publish)

## Architectural Overview

### Directory Structure

```
.github/
├── workflows/
│   ├── build.yml                  # CI: type check & build
│   ├── bump-version.yml           # Manual: bump version & create PR
│   └── publish.yml                # Auto: release on PR merge
src/
├── main/                          # Electron main process
│   ├── index.ts                   # App entry point (auto-updater)
│   └── ipc-handlers.ts            # IPC handlers (incl. updates)
├── preload/                       # Preload scripts for IPC
│   ├── index.ts                   # Bridge (incl. app, updates)
│   └── types.ts                   # API type definitions
├── renderer/                      # React application
│   ├── src/
│   │   ├── App.tsx                # Root component
│   │   ├── main.tsx               # React entry point (i18n initialized here)
│   │   ├── services/
│   │   │   ├── ipc.ts             # IPC bridge to main process
│   │   │   └── i18n.ts            # i18n configuration
│   │   ├── state/
│   │   │   ├── device-store.ts    # Zustand store for devices/accessories
│   │   │   ├── preferences-store.ts # Zustand store for UI preferences (includes language)
│   │   │   ├── room-store.ts      # Zustand store for rooms
│   │   │   ├── scene-store.ts     # Zustand store for scenes
│   │   │   └── ui-store.ts        # Zustand store for UI state
│   │   ├── locales/
│   │   │   ├── en.json            # English translations
│   │   │   └── es.json            # Spanish translations
│   │   ├── utils/
│   │   │   └── language-detector.ts # Language detection logic
│   │   ├── ui/
│   │   │   ├── components/        # Reusable React components
│   │   │   ├── controls/          # Device control components
│   │   │   ├── views/             # Page-level views (Home, Settings, Rooms, etc.)
│   │   │   └── styles/            # CSS files
│   │   └── types/                 # TypeScript type definitions
└── electron.vite.config.ts        # Build configuration
```

### State Management

The project uses **Zustand with persistence middleware** for state management. Key stores:

- **device-store**: Accessories and bridges (loaded from main process)
- **room-store**: Room organization and grouping
- **scene-store**: Scene definitions and automation
- **preferences-store**: UI preferences (theme, background, **language**, card sizes, order)
- **ui-store**: Transient UI state (current view, edit mode)

All stores except `ui-store` persist to localStorage.

### Internationalization (i18n)

**Status:** Recently implemented (2026-06-02)

**Architecture:**
- **i18next + react-i18next** for translation framework
- **Translation files:** `src/renderer/src/locales/{en,es}.json`
- **Language detection:** Tries Electron API → browser navigator.language → defaults to English
- **Preference storage:** Saved in `preferences-store` with `language: 'en' | 'es' | 'system'`
- **Initialization:** On app startup in `main.tsx` before React render
- **Usage:** Components use `const { t } = useTranslation()` hook to access translations

**Translation Structure:**
```json
{
  "header": { ... },           // Header component strings
  "modal": { ... },            // Modal dialog strings
  "sidebar": { ... },          // Navigation sidebar strings
  "settings": { ... },         // Settings view strings
  "colors": { ... },           // Color preset names
  "common": { ... }            // Common UI strings
}
```

**Key Points for Agents:**
- Add new UI strings: Add translation keys to both `en.json` and `es.json` (same structure)
- For dynamic text: Use interpolation with `{{variable}}` syntax in JSON, `t('key', { variable })` in code
- Language selector in Settings allows users to override system language
- All components use `useTranslation()` hook - no hardcoded strings in UI

## Development Workflow

### Common Tasks

#### Adding a New Feature

1. **Plan with brainstorming skill** - Understand requirements and design
2. **Create implementation plan** with writing-plans skill
3. **Execute plan** using subagent-driven-development or executing-plans
4. **For UI strings:** Add translation keys to `en.json` and `es.json` with matching structure
5. **Test in both languages** via Settings language selector
6. **Code review** with requesting-code-review skill before merge

#### Fixing a Bug

1. **Systematic debugging** - Use systematic-debugging skill to understand the issue
2. **Isolate the problem** - Identify which component/store/service is affected

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiguelRipoll23/home-app](https://github.com/MiguelRipoll23/home-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
