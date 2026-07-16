---
trigger: always_on
description: Language-learning immersion app: Electron + SolidJS + TypeScript frontend, Capacitor mobile target, Python/FastAPI NLP backend (port 7752). SRS flashcards, video subtitles, OCR, TTS, LLM tutoring.
---

# mLearn Knowledge Base

## OVERVIEW
Language-learning immersion app: Electron + SolidJS + TypeScript frontend, Capacitor mobile target, Python/FastAPI NLP backend (port 7752). SRS flashcards, video subtitles, OCR, TTS, LLM tutoring.

## STRUCTURE
```
src/
├── electron/        # Main process (CommonJS). IPC, window management, voice/LLM/OCR services
├── renderer/        # SolidJS UI (ESNext). Components, windows, hooks, contexts
├── shared/          # Types, constants, platform bridges/backends. Renderer-only abstractions
├── root-of-app/     # Python FastAPI backend. NLP tokenization, translation, OCR, TTS
└── html/            # 15 Electron window entries + mobile.html (Capacitor)
extension/           # Chrome browser extension
android/, ios/       # Capacitor native projects
examples/plugins/    # Plugin templates (shiritori, discord-activity)
```

## WHERE TO LOOK
| Task | Location |
|------|----------|
| Add IPC channel | `shared/constants.ts` → `preload.ts` → `shared/global.d.ts` → `shared/bridges/types.ts` → both bridges → `electron/services/` |
| Add UI window | `src/html/{name}.html` → `vite.config.ts` input → `src/renderer/windows/{name}/` |
| Add component | `src/renderer/components/common/{Name}/{Name}.tsx` + `.css` → `common/index.ts` |
| Add backend endpoint | `shared/backends/types.ts` → `shared/backends/httpBackend.ts` → `src/root-of-app/routes/{name}.py` |
| Add setting | `shared/types.ts` (Settings + DEFAULT_SETTINGS) → settings context |
| Add language runtime capability | `src/shared/types.ts` language metadata schema + `src/root-of-app/generic_language.py` |
| Add language package/data | `~/Desktop/projects/mlearn-website` language-data packaging, then install via catalog |
| Platform-specific code | `src/shared/platform.ts` helpers; never hardcode OS checks in renderer |

## CONVENTIONS
- **Two tsconfigs**: root (ESNext, renderer+shared) + `src/electron/tsconfig.json` (CommonJS, excludes bridges/backends/platform)
- **Path aliases**: `@/` → `src/`, `@shared/` → `src/shared/`, `@renderer/` → `src/renderer/`
- **Strict TS**: `noUnusedLocals`, `noUnusedParameters`, `noImplicitReturns`
- **CSS**: Co-located `.css` per component, no CSS modules. 6 override theme files in `src/renderer/styles/themes/` + default light in `src/renderer/styles/index.css`. Applied via `body.theme-{name}`. No hardcoded colors in TSX. Do not add CSS variable fallbacks.
- **Localization**: `t('mlearn.Section.Key')` with `{param}`. 5 UI languages in `src/root-of-app/locales/`. Validate JSON after editing locale files.
- **Flashcard keys**: SHA-256 hashes (64-char hex), not raw text.
- **Tests**: Co-located `*.test.ts`/`*.test.tsx`. Vitest 3 projects: `node` (electron+shared+extension), `examples` (plugins), `renderer` (happy-dom). Pool: `forks`, maxWorkers: 4, setup: `test/setup.ts`. Write tests for every new feature.
- **Bridge composition**: `PlatformBridge` is 22 sub-interfaces. `getBridge()` is renderer-only; never import bridges/backends/platform into `src/electron/`.
- **Backend modes**: `settings.backendMode` is `'local' | 'tethered'` only. `getBackend()` returns `HttpBackend` for both. Cloud LLM calls bypass `getBackend()` entirely and use `CloudLLMAdapter` (SSE streaming).
- **Context nesting order** (via `WindowWrapper`): `ServerProvider → LocalizationProvider → ResponsiveProvider → SettingsProvider → LowPowerGateProvider → LanguageProviderBridge → MigrationHandler → FlashcardProvider`
- **Settings updates**: Always use `updateSetting()`/`updateSettings()` from Settings context — triggers `reconcile()`, DOM theme application, backend reconfig, bridge save, and `BroadcastChannel` cross-window sync. Never use raw `setStore`.
- **Setting fallbacks**: When reading optional or migrated settings, use `DEFAULT_SETTINGS.<key>` as the fallback. Do not hardcode literal defaults like `?? true`, `?? 300`, or `|| 'local'`.
- **State patterns**: Settings uses `createStore` + `reconcile()`. Flashcards use `createStore` + `produce()`.
- **Capacitor stub**: `electron` imports are aliased to `src/shared/stubs/electron.ts` in Capacitor builds.
- **Barrel exports**: Every new common component must be exported from `src/renderer/components/common/index.ts`.
- **Icons**: Use SVGs from `https://blendicons.com/free-icons/all-styles`. Do not use emojis.
- **Language data**: Runtime language metadata, dictionaries, frequencies, and optional adapters are downloaded into user `language-data/`. Do not add bundled app-source language modules or dictionaries.
- **Language-owned runtime dependencies**: Language-specific OCR/tokenizer/TTS/STT Python packages belong in language package metadata under `runtime.python.packagesByComponent` in `~/Desktop/projects/mlearn-website`, not in app-level `pip_requirements.json` defaults. If a clean install is missing OCR or tokenizer libraries for one language, fix the cloud language package/catalog and redeploy language data.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adrianvla/mLearn](https://github.com/adrianvla/mLearn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
