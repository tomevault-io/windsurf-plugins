---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Grimoire is a Nostr protocol explorer and developer tool. It's a tiling window manager interface where each window is a Nostr "app" (profile viewer, event feed, NIP documentation, etc.). Commands are launched Unix-style via Cmd+K palette.

**Stack**: React 19 + TypeScript + Vite + TailwindCSS + Jotai + Dexie + Applesauce

## Core Architecture

### Dual State System

**UI State** (`src/core/state.ts` + `src/core/logic.ts`):
- Jotai atom persisted to localStorage
- Pure functions for all mutations: `(state, payload) => newState`
- Manages workspaces, windows, layout tree, active account

**Nostr State** (`src/services/event-store.ts`):
- Singleton `EventStore` from applesauce-core
- Single source of truth for all Nostr events
- Reactive: components subscribe via hooks, auto-update on new events
- Handles replaceable events automatically (profiles, contact lists, etc.)

**Relay State** (`src/services/relay-liveness.ts`):
- Singleton `RelayLiveness` tracks relay health across sessions
- Persisted to Dexie `relayLiveness` table
- Maintains failure counts, backoff states, last success/failure times
- Prevents repeated connection attempts to dead relays

**Nostr Query State Machine** (`src/lib/req-state-machine.ts` + `src/hooks/useReqTimelineEnhanced.ts`):
- Accurate tracking of REQ subscriptions across multiple relays
- Distinguishes between `LIVE`, `LOADING`, `PARTIAL`, `OFFLINE`, `CLOSED`, and `FAILED` states
- Solves "LIVE with 0 relays" bug by tracking per-relay connection state and event counts
- Pattern: Subscribe to relays individually to detect per-relay EOSE and errors

**Critical**: Don't create new EventStore, RelayPool, or RelayLiveness instances - use the singletons in `src/services/`

**Event Loading** (`src/services/loaders.ts`):
- Unified loader auto-fetches missing events when queried via `eventStore.event()` or `eventStore.replaceable()`
- Custom `eventLoader()` with smart relay hint merging for explicit loading with context
- `addressLoader` and `profileLoader` for replaceable events with batching
- `createTimelineLoader` for paginated feeds

**Action System** (`src/services/hub.ts`):
- `ActionRunner` (v5) executes actions with signing and publishing
- Actions are async functions: `async ({ factory, sign, publish }) => { ... }`
- Use `await publish(event)` to publish (not generators/yield)

### Window System

Windows are rendered in a recursive binary split layout (via `react-mosaic-component`):
- Each window has: `id` (UUID), `appId` (type identifier), `title`, `props`
- Layout is a tree: leaf nodes are window IDs, branch nodes split space
- **Never manipulate layout tree directly** - use callbacks from mosaic

Workspaces are virtual desktops, each with its own layout tree.

### Command System

`src/types/man.ts` defines all commands as Unix man pages:
- Each command has an `appId` (which app to open) and `argParser` (CLI → props)
- Parsers can be async (e.g., resolving NIP-05 addresses)
- Command pattern: user types `profile alice@example.com` → parser resolves → opens ProfileViewer with props

**Global Flags** (`src/lib/global-flags.ts`):
- Global flags work across ALL commands and are extracted before command-specific parsing
- `--title "Custom Title"` - Override the window title (supports quotes, emoji, Unicode)
  - Example: `profile alice --title "👤 Alice"`
  - Example: `req -k 1 -a npub... --title "My Feed"`
  - Position independent: can appear before, after, or in the middle of command args
- Tokenization uses `shell-quote` library for proper quote/whitespace handling
- Display priority: `customTitle` > `dynamicTitle` (from DynamicWindowTitle) > `appId.toUpperCase()`

### Reactive Nostr Pattern

Applesauce uses RxJS observables for reactive data flow:
1. Events arrive from relays → added to EventStore
2. Queries/hooks subscribe to EventStore observables
3. Components re-render automatically when events update
4. Replaceable events (kind 0, 3, 10000-19999, 30000-39999) auto-replace older versions

Use hooks like `useProfile()`, `useNostrEvent()`, `useTimeline()` - they handle subscriptions.

**The `use$` Hook** (applesauce v5):
```typescript
import { use$ } from "applesauce-react/hooks";

// Direct observable (for BehaviorSubjects - never undefined)
const account = use$(accounts.active$);

// Factory with deps (for dynamic observables)
const event = use$(() => eventStore.event(eventId), [eventId]);
const timeline = use$(() => eventStore.timeline(filters), [filters]);
```

### Applesauce Helpers & Caching

**Critical Performance Insight**: Applesauce helpers cache computed values internally using symbols. **You don't need `useMemo` when calling applesauce helpers.**

```typescript
// ❌ WRONG - Unnecessary memoization
const title = useMemo(() => getArticleTitle(event), [event]);
const text = useMemo(() => getHighlightText(event), [event]);

// ✅ CORRECT - Helpers cache internally
const title = getArticleTitle(event);
const text = getHighlightText(event);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [purrgrammer/grimoire](https://github.com/purrgrammer/grimoire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
