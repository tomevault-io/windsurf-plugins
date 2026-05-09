---
trigger: always_on
description: Mobile-first LLM frontend for AI roleplay. SillyTavern alternative as a native mobile app.
---

# Glaze

Mobile-first LLM frontend for AI roleplay. SillyTavern alternative as a native mobile app.
**Stack:** Vue 3.5 + Vite 7 + Capacitor 8 (iOS/Android). **Language:** JavaScript only. **Status:** Active alpha (v0.4.0). **License:** AGPL-3.0.

## Commands

```bash
npm run dev          # Dev server
npm run build        # Production build
npm test             # Vitest unit tests
npm run test:ui      # Tests with UI

# Mobile
npm run build && npx cap sync android && npx cap open android
npm run build && npx cap sync ios && npx cap open ios
```

## Architecture

```
src/
├── App.vue                 # Root component — view orchestration, navigation, global state
├── core/
│   ├── config/             # Settings singletons (APISettings.js, APPSettings.js)
│   ├── services/           # Business logic — functional, no classes
│   └── states/             # Reactive state modules — Vue ref/reactive/computed exports
├── utils/                  # Helpers (db.js, macroEngine.js, tokenizer.js, textFormatter.js)
├── composables/            # Vue composables (useVirtualScroll, useViewer)
├── components/
│   ├── chat/               # ChatInput, ChatMessage, MagicDrawer
│   ├── editors/            # GenericEditor, FullScreenEditor
│   ├── layout/             # AppHeader, BottomNavigation, AppLoader
│   ├── media/              # HoloCardViewer, ImageViewer
│   ├── sheets/             # Bottom sheet content (LorebookSheet, RegexSheet, etc.)
│   └── ui/                 # Generic UI primitives (BottomSheet, FabButton, etc.)
├── views/                  # Page-level components (ChatView, CharacterList, PresetView)
├── workers/                # Web Workers (generationWorker.js — off-thread prompt building)
├── locales/                # i18n JSON (en.json, ru.json)
└── assets/                 # CSS, logos, default presets
```

**Key files:**
- `src/utils/db.js` — IndexedDB wrapper, all persistent storage operations
- `src/workers/generationWorker.js` — prompt assembly (blocks, lorebooks, macros, token counting)
- `src/core/services/llmApi.js` — LLM API calls and SSE stream parsing
- `src/core/services/generationService.js` — generation orchestration
- `src/core/states/lorebookState.js` — lorebook scanning and activation logic
- `src/core/services/regexService.js` — regex transformation engine
- `src/utils/macroEngine.js` — macro substitution ({{char}}, {{user}}, {{random}}, etc.)

## Code Conventions

### Vue Components
- **Composition API with `<script setup>` only** — no Options API anywhere
- **JavaScript only** — no TypeScript
- Props via `defineProps()`, emits via `defineEmits()`
- Heavy views loaded with `defineAsyncComponent()` to reduce initial bundle

### State Management
- **No Pinia, no Vuex** — custom reactive state modules in `src/core/states/`
- Each module exports `ref()`, `reactive()`, `computed()` values and mutation functions
- Computed with getter/setter for bidirectional binding where needed
- State modules are imported directly where needed (no global store instance)

```js
// Pattern: src/core/states/exampleState.js
import { ref, computed } from 'vue';

const items = ref([]);
export const activeItem = computed({
    get() { return items.value[0]; },
    set(val) { items.value[0] = val; }
});
export function addItem(item) { items.value.push(item); }
```

### Services
- **Functional** — exported async functions, no classes
- Located in `src/core/services/`

### Navigation
- **No Vue Router** — custom view switching via `currentView` ref in App.vue
- Navigate with: `window.dispatchEvent(new CustomEvent('navigate-to', { detail: 'view-chat' }))`
- Cross-component communication uses `window.dispatchEvent(new CustomEvent(...))` pattern

### File Naming
| Type | Convention | Example |
|------|-----------|---------|
| Components/Views | PascalCase | `ChatView.vue`, `AppHeader.vue` |
| Services | camelCase | `generationService.js`, `llmApi.js` |
| States | camelCase + State | `presetState.js`, `lorebookState.js` |
| Composables | use + PascalCase | `useVirtualScroll.js`, `useViewer.js` |
| Utils | camelCase | `db.js`, `macroEngine.js` |

### CSS
- Scoped `<style>` blocks (no CSS modules, no preprocessor)
- CSS custom properties for theming (`--header-height`, `--vk-blue`, `--element-opacity`, etc.)
- Glass effect pattern:
  ```css
  background-color: rgba(255, 255, 255, var(--element-opacity, 0.8));
  backdrop-filter: blur(var(--element-blur, 20px));
  ```
- Light/dark themes via `body.dark-theme` class

## Storage

| Data | Backend | Key Pattern |
|------|---------|-------------|
| Characters | IndexedDB `characters` store | by `id` |
| Personas | IndexedDB `personas` store | by `id` |
| Chats | IndexedDB `keyvalue` store | `gz_chat_{charId}` |
| Lorebooks | IndexedDB `keyvalue` store | `gz_lorebooks` |
| Presets | localStorage | `silly_cradle_presets` |
| API config | localStorage | `api-*`, `gz_api_*` |
| Regex scripts | localStorage | `regex_scripts` |
| App settings | localStorage | `gz_*` |
| Session vars | localStorage | `gz_vars_{charId}_{sessionId}` |

**DB queue pattern** — all IndexedDB writes go through a queue to prevent races:
```js
let dbQueue = Promise.resolve();
function queueDbOp(op) {
    dbQueue = dbQueue.then(op);
    return dbQueue;
}
```

## Domain Concepts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hydall/Glaze](https://github.com/hydall/Glaze) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
