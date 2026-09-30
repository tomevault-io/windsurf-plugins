---
trigger: always_on
description: Flotilla is a Nostr "relays as groups" community chat client. It implements NIP-29 (relay-based groups) to create Discord-like spaces (servers) and rooms (channels).
---

## Project Overview

Flotilla is a Nostr "relays as groups" community chat client. It implements NIP-29 (relay-based groups) to create Discord-like spaces (servers) and rooms (channels).

**Tech Stack:**

- SvelteKit 5.48+ with TypeScript 5.9+
- Capacitor for cross-platform (Web/PWA, Android, iOS)
- TailwindCSS for styling
- Welshman library suite for Nostr protocol
- IndexedDB for local storage
- Vite for building

**Key Concepts:**

- **Spaces** - Relays used as community groups (like Discord servers)
- **Rooms** - NIP-29 groups within spaces (like Discord channels), identified by `h`
- **Chats** - Direct message conversations (NIP-04/NIP-44 encrypted)

## Architecture & Dependency Graph

The project follows a **strict acyclic dependency hierarchy**:

```
routes/                    (top layer - can depend on anything)
  ↓
app/components/            (can depend on app/* and lib/*)
  ↓
app/*                      (can only depend on lib/*)
  ↓
lib/                       (can only depend on external libraries)
  ↓
external libraries         (bottom layer)
```

**Import Ordering Convention (CRITICAL):**
Always sort imports by dependency level:

1. Third-party libraries first
2. Then `lib/` imports
3. Then `app/` imports

Example:

```typescript
import {derived} from "svelte/store"
import {throttle} from "throttle-debounce"
import {Profiles} from "@welshman/app"
import {Dialog} from "$lib/components"
import {app} from "@app/core"
```

## Cleanup Pass

These are the things that most often get fixed by hand after the fact. Go through
them before handing work back. The full style reference is under Development
Conventions below.

- **Comments** — a comment is one line, for a genuinely surprising thing: a workaround, a constraint imposed by a backend or platform, an invariant that isn't visible from the code in front of you. Never comment props, never restate what the next line does, never justify an ordinary decision. Where one line can't hold it, the argument belongs in the pull request rather than in the code. If a small refactor would make the comment stale, don't write it.
- **Used once, inlined** — a derived value, helper, type, or named constant with a single use is indirection. Write `setTimeout(pollOnce, 3500)`, not a `POLL_INTERVAL` referenced twice in one file. Name something only when the name is what makes the code readable.
- **Positive conditionals** — prefer `if (ready) { ... }` over `if (!ready) return`. Nesting is fine; when it gets deep that's the signal the function is doing too much, so split it. Many early returns belong in validation or pipeline functions, not everywhere else.
- **Truthiness** — test values directly (`if (invoice.paid_at)`) instead of comparing against `null`/`undefined`, and put the truthy branch first in a ternary.
- **Standard names** — `loading` for in-flight state, `on*` for handlers bound to an event, a plain verb (`submit`, `save`) for the action itself. Use `{prop}` shorthand when the names match.
- **Errors** — toast a human-readable message and `console.error` anything unexpected (e.g. a non-`HostingError`). Never swallow an error you didn't anticipate.

## State Management

**Core Principles:**

- Use Svelte 4 **stores** for all state (NOT runes outside UI components)
- Everything hangs off the single `App` instance in `app/core.ts`. There are no welshman globals —
  reach features with `app.use(Plugin)`, which is memoized per app and cheap to call inline.
- `app/core.ts` also exports `pubkey`, `signer`, `user` and `session` stores (all with `.get()`),
  `login`, and `deriveUserItem(plugin)` for the current user's entry in a keyed collection.
- Most global state flows through the app's `repository` (unidirectional)
- Query state with a plugin's `one(key)` / `index` / `all`, or `deriveEventsById` /
  `deriveItemsByKey` from `@welshman/store` against `app.repository`
- Update state by building a domain writer and publishing the resulting `Command`

**Projections:**

A `Projection<T>` is `{get(): T, $: Readable<T>}` — bind `.$` in markup, call `.get()` in
callbacks and hot paths.

**Thunks:**

- Reduce UI latency by handling signatures and sending in background
- Return status that should be displayed to user
- Allow cancellation and error handling
- Immediately publish to local repository for optimistic updates

## Nostr Integration

**Welshman Library Suite:**

- `@welshman/app` - The `App` instance and its plugins (Profiles, Rooms, Thunks, Router, …)
- `@welshman/domain` - A typed Reader/Writer pair per event kind, plus `Relay` and `Zapper`
- `@welshman/net` - Network layer (Pool, Socket, adapters, request/publish/pull)
- `@welshman/store` - Svelte integration (deriveEventsById, deriveItemsByKey, etc.)
- `@welshman/util` - Event utilities (kinds, tags, validation, the RelaySelection routing DSL)
- `@welshman/signer` - Signing abstraction (NIP-01, NIP-07, NIP-46)
- `@welshman/editor` - Rich text editor with Nostr
- `@welshman/content` - Content parsing
- `@welshman/feeds` - Feed management

**Key NIPs Implemented:**

- NIP-01: Basic protocol
- NIP-44/59/17: Encrypted DMs
- NIP-07: Browser extension signing
- NIP-19: Bech32 encoding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coracle-social/flotilla](https://github.com/coracle-social/flotilla) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
