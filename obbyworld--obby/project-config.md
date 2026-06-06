---
trigger: always_on
description: React + TypeScript + TailwindCSS + DaisyUI + Zustand + Tauri (desktop/mobile).
---

# ObsidianIRC — Agent Reference

React + TypeScript + TailwindCSS + DaisyUI + Zustand + Tauri (desktop/mobile).
WebSocket-only IRC client. Tauri wraps the web app with native bindings for TCP sockets
(WebSocket-compatible wrapper), file viewing (Swift plugin on iOS/macOS via `src-tauri/plugins/`),
and share-sheet. The web build also runs standalone via Docker/Nginx.

---

## Commands — run these every time

```bash
npm run format; npm run fix:unsafe; npm run test; npm run build
```

- `format` / `fix:unsafe` — Biome lint + format (pre-commit hook does this automatically)
- `test` — Vitest, all 50 test files must pass
- `build` — TypeScript compile + Vite bundle (must be clean)

---

## Tool versions

- **JavaScript:** **Node ≥22** ([`package.json` `engines`](package.json)); **`npm install`** from the repo root.
- **Rust (Tauri):** [`rust-toolchain.toml`](rust-toolchain.toml) — **stable** + **rustfmt**/**clippy**; MSRV in [src-tauri/Cargo.toml](src-tauri/Cargo.toml).

---

## Nix

- **`nix develop`** — full dev environment (Node 22 + Tauri Linux deps + rustup). Linux only (`x86_64`/`aarch64`).
- **`nix build .#obsidianirc`** — produces `result/bin/ObsidianIRC`. When `package-lock.json` changes, run `nix run .#update-npm-deps-hash` if you have Nix locally; otherwise `update-linux-nix` in [`publish.yaml`](.github/workflows/publish.yaml) syncs the hash to `main` on version tags.
- Details: [BUILD.md — Nix (flake)](BUILD.md#nix-flake)
---

## Project Layout

```
src/
  components/
    layout/       # AppLayout, ChatArea, ChannelList, MemberList, ChatHeader, ResizableSidebar
    mobile/       # Mobile-specific variants
    message/      # MessageItem, MediaPreview, MessageAvatar, MessageReply
    ui/           # Modals, dropdowns, settings panels
  hooks/          # Custom React hooks (useScrollToBottom, useTabCompletion, …)
  lib/
    irc/
      IRCClient.ts          # IRC client class
      handlers/             # IRC protocol dispatch (one file per domain)
        index.ts            # IRC_DISPATCH table + handleMessage()
        connection.ts / messages.ts / users.ts / channels.ts
        whois.ts / metadata.ts / auth.ts / monitoring.ts
    ircClient.ts            # Singleton: `export default new IRCClient()`  ← all imports point here
    mediaProbe.ts           # HEAD/GET probing (see URL Safety below)
    mediaUtils.ts           # Media type detection + trust logic
    settings/               # Settings definitions and helpers
  store/
    index.ts                # Zustand store: state shape + all action methods
    handlers/               # Store-side IRC event subscriptions (one file per domain)
      index.ts              # registerAllHandlers(store) — called by store/index.ts
      messages.ts / users.ts / channels.ts / batches.ts
      whois.ts / metadata.ts / auth.ts / connection.ts
    helpers.ts              # generateDeterministicId(serverId, name) — uuidv5 channel/user IDs
    types.ts                # UISelections and other store-specific types
    localStorage.ts         # loadUISelections / saveUISelections
  types/
    index.ts                # Shared types: Server, Channel, Message, User, …
tests/                      # Vitest tests — mirror src/ structure
src-tauri/                  # Tauri config, Rust backend, plugins (Swift share-sheet)
```

---

## IRC Event Flow — two layers

### Layer 1: Protocol parsing (`src/lib/irc/`)

`IRCClient.handleMessage()` calls `handleMessage(ctx, serverId, raw)` from `src/lib/irc/handlers/index.ts`,
which dispatches via `IRC_DISPATCH`:

```ts
const IRC_DISPATCH: Record<string, (ctx: IRCClientContext, serverId: string, msg: ParsedMessage) => void> = {
  PRIVMSG: handlePrivmsg,
  JOIN: handleJoin,
  "332": handleRplTopic,
  // …
};
```

Each handler in `src/lib/irc/handlers/*.ts` receives `ctx: IRCClientContext` (the client instance)
and calls `ctx.triggerEvent("EVENT_NAME", payload)` to emit to the store.

**To add a new IRC command:** add a handler function to the relevant `src/lib/irc/handlers/*.ts`
file and add it to `IRC_DISPATCH` in `index.ts`.

### Layer 2: Store subscriptions (`src/store/handlers/`)

`src/store/handlers/index.ts` exports `registerAllHandlers(store: StoreApi<AppState>)`,
which is called once at the bottom of `src/store/index.ts` after `useStore` is created.

Each handler file subscribes to `ircClient` events and updates the Zustand store:

```ts
// Pattern in every src/store/handlers/*.ts
export function registerXxxHandlers(store: StoreApi<AppState>) {
  ircClient.on("EVENT", (payload) => {
    store.setState((state) => ({ /* return Partial<AppState> — no mutation */ }));
  });
}
```

**To add a new store reaction to an IRC event:** add `ircClient.on(...)` in the relevant
`src/store/handlers/*.ts` and call the new register function from `handlers/index.ts`.

**Important:** `store.setState()` callbacks must **return** `Partial<AppState>`. The store
uses Immer (`immer` package is a dependency), but the Immer middleware is not currently wired
into `create()` — direct mutation of `state` inside `setState` will silently not work.

---

## Tests

Location: `tests/` — mirrors `src/` structure.

```
tests/
  hooks/        # React hook tests (renderHook + act)
  lib/          # Pure logic tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obbyworld/obby](https://github.com/obbyworld/obby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-06 -->
