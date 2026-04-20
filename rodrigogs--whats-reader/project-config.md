---
trigger: always_on
description: **Hybrid SvelteKit + Electron app** for parsing and viewing WhatsApp chat exports offline. Runs as both a web app (GitHub Pages) and desktop app (Electron).
---

# WhatsApp Backup Reader - AI Agent Instructions

## Architecture Overview

**Hybrid SvelteKit + Electron app** for parsing and viewing WhatsApp chat exports offline. Runs as both a web app (GitHub Pages) and desktop app (Electron).

### Core Design Principles
- **100% Offline**: No external API calls, all processing happens client-side
- **Privacy-first**: No analytics, no telemetry, no data transmission
- **Web Workers**: CPU-intensive tasks (search indexing, transcription) run in workers
- **Svelte 5 Runes**: All state management uses `$state`, `$derived`, `$effect` (no stores)

## State Management (Svelte 5 Runes)

**Critical**: This project uses **Svelte 5 runes exclusively**. Never use Svelte 4 stores (`writable`, `derived`, `readable`).

### Global State Pattern

State lives in `*.svelte.ts` files using factory functions that return getter/action objects:

```typescript
// src/lib/state.svelte.ts
export function createAppState() {
  let chats = $state<ChatData[]>([]);
  let selectedIndex = $state<number | null>(null);
  
  const selectedChat = $derived(
    selectedIndex !== null ? chats[selectedIndex] : null
  );
  
  return {
    get chats() { return chats; },
    get selectedChat() { return selectedChat; },
    addChat(chat: ChatData) { chats = [...chats, chat]; }
  };
}

export const appState = createAppState();
```

**Key files**: `src/lib/state.svelte.ts`, `src/lib/bookmarks.svelte.ts`, `src/lib/transcription.svelte.ts`

### Reactivity Rules
- Use `$derived` for computed values, not functions
- Use `$effect` for side effects (DOM sync, cleanup)
- Use `untrack()` to read reactive values without subscribing
- Never mutate state directly - always reassign: `items = [...items, newItem]`

## Web Workers

Three workers handle heavy operations:

1. **Index Worker** (`src/lib/workers/index-worker.ts`): Builds message search index when chat loads
2. **Search Worker** (`src/lib/workers/search-worker.ts`): Performs search queries with progress updates
3. **Transcription Worker** (in `@huggingface/transformers`): Runs Whisper model for audio transcription

### Worker Communication Pattern

```typescript
// Main thread initiates worker
const worker = new Worker(new URL('./search-worker.ts', import.meta.url), {
  type: 'module'
});

// Send work with unique ID for cancellation
worker.postMessage({ id: searchId, query, messages });

// Handle progress + results
worker.onmessage = (e) => {
  if (e.data.type === 'progress') {
    searchProgress = e.data.progress;
  } else if (e.data.type === 'complete') {
    searchResultIds = e.data.resultIds;
  }
};
```

**Never** run expensive operations on main thread - always delegate to workers.

## Internationalization (i18n)

Uses **Paraglide JS** (compile-time i18n, not runtime lookups).

### Adding Translations

1. Add key to `messages/en.json` (source of truth):
   ```json
   {
     "search_placeholder": "Search messages..."
   }
   ```

2. Run machine translation: `npm run machine-translate`

3. Import and use in components:
   ```svelte
   <script>
   import * as m from '$lib/paraglide/messages';
   </script>
   <input placeholder={m.search_placeholder()} />
   ```

**Never** hardcode user-facing strings. All 10 languages must stay in sync.

## Parser System

WhatsApp exports use different date formats by locale. Parser supports:
- US format: `12/25/2024, 3:45 PM`
- European: `25/12/2024, 15:45`
- ISO: `2024-12-25 15:45:00`

`src/lib/parser/chat-parser.ts` tries all formats with regex patterns. Test new formats in `examples/parser-tests/`.

### Chat Data Structure

```typescript
interface ChatData {
  title: string;
  messages: ChatMessage[];
  participants: string[];
  messageIndex: Map<string, number>;  // O(1) lookup
  flatItems: FlatItem[];  // For virtual scrolling
  messagesById: Map<string, ChatMessage>;
}
```

## Build System

### Dual-Target Builds

**SvelteKit adapter-static** generates single-page app:
- **Web**: Base path `/whats-reader` for GitHub Pages (when `GITHUB_PAGES=true`)
- **Electron**: Relative paths (`base: './'`) to load from `file://`

`svelte.config.js` switches between these using `process.env.GITHUB_PAGES`.

### Build Commands

```bash
npm run build              # SvelteKit build → build/
npm run electron:build     # Build + Electron packaging → dist-electron/
npm run electron:build:mac # Platform-specific (mac/win/linux)
```

Electron config in `package.json` under `"build"` key. Icons: `static/icon.{icns,ico,png}`.

## Development Workflows

### Running Locally

```bash
npm run dev              # Web app on localhost:5173
npm run electron:dev     # Electron app (concurrently runs Vite + Electron)
```

### Code Quality

```bash
npm run lint            # Biome check (not ESLint!)
npm run lint:fix        # Auto-fix
npm run check           # TypeScript + Svelte validation
```

**Biome config** in `biome.json`:
- Tab indentation (width 2)
- Single quotes
- Semicolons required
- Svelte files: many rules disabled (false positives)

### Testing Changes

Use example files in `examples/chats/`:
- `private-chat/` - Individual chat with audio
- `family-group-chat/` - Group chat

Create test ZIPs: `cd examples/chats && ./build-zips.sh`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rodrigogs/whats-reader](https://github.com/rodrigogs/whats-reader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
