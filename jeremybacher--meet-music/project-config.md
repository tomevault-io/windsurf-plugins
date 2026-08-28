---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**Meet Music** — a Chrome MV3 extension (Preact + TypeScript, bundled with esbuild) that adds a
shared music queue inside Google Meet. The defining constraint: **everyone else in the meeting hears
the music without installing anything**, because the audio is mixed into the microphone track of
whoever is playing.

Every architectural oddity in this repo follows from that one decision. Read `README.md` for the
product-level explanation and `CONTRIBUTING.md` for the manual test checklist.

## Commands

```bash
corepack enable          # this project uses pnpm (packageManager is pinned)
pnpm install
pnpm dev                 # esbuild watch → dist/
pnpm build               # production bundle → dist/
pnpm test                # vitest, 61 tests
pnpm typecheck           # tsc --noEmit
```

Before opening a PR, run exactly what CI runs:

```bash
pnpm typecheck && pnpm test && pnpm build
```

Load `dist/` in `chrome://extensions` with Developer mode on, and hit reload there after each build.
Chrome 116+ is the floor (`minimum_chrome_version` in the manifest).

## Architecture

Four execution contexts, none of which share variables:

```
┌─ Browser of whoever is playing ────────────────────────────────────┐
│  YouTube tab (background)                                          │
│  ├─ [ISOLATED] yt-content: Web Audio over the <video>              │
│  └─ [MAIN]     yt-main: loadVideoById, no reload between songs     │
│            │                                                       │
│            │ local WebRTC          Service worker                  │
│            │ (host candidates)     └─ tab + signalling relay       │
│            ▼                                                       │
│  Meet tab                                                          │
│  ├─ [ISOLATED] panel + transport over the Meet chat                │
│  └─ [MAIN]     getUserMedia patch + mixer                          │
└────────────────────────────────────────────────────────────────────┘
        │ mixed audio, through Meet
        ▼  everyone else, without installing anything
```

- MAIN ↔ ISOLATED talk over `window.postMessage`, filtered by the `BRIDGE` marker in
  `src/core/messages.ts` (Meet posts its own noise on that channel).
- Content scripts ↔ service worker talk over `chrome.runtime` ports (`PORT_MEET`, `PORT_PLAYER`).
- Audio travels YouTube tab → Meet tab over a **local** WebRTC link; both ends are in the same
  browser, so ICE resolves with host candidates and there is no network in between.
- **Host/guest model**: whoever starts the audio is the host and owns the state. Guests emit
  intentions over the chat and paint the last snapshot they received. No CRDT, no conflict
  resolution.

## Invariants

These are load-bearing. Breaking one is either deliberate or a bug — never incidental.

1. **Installed but idle, the extension touches nothing.** With no music, `getUserMedia` returns the
   microphone untouched and no `AudioContext` is created. Anything you add to the voice path must
   only happen while music is playing.
2. **The voice passes through no processing node**, only a gain. The limiter hangs off the music
   branch; the two branches sum only at the destination. `setLevels({ musicBroadcast })` writes to
   `musicBroadcastGain` and never to `micGain`. `test/mixer.test.ts` verifies the topology.
3. **Meet's DOM is not an API.** Selectors are centralised and heuristic on purpose, and there is
   always a degradation path: if something is not found, the panel says so and keeps working in solo
   mode instead of falling over.
4. **The chat is an expensive channel.** Every message is a visible line for anyone without the
   extension. Before adding a message type, ask whether it is needed and whether it should be
   coalesced (see `LATEST_WINS` and the volume debounce in `chat-transport.ts`).
5. **Sending never clobbers a draft.** If the chat field has text, the send is deferred and the
   draft plus focus are saved and restored.
6. **Encryption is keyed from the meeting code.** AES-GCM authenticates, so anything not produced by
   the extension fails to decrypt and is discarded — that is what stops a participant from typing a
   command into the chat. It does not claim to resist a malicious participant: whoever has the code
   has the key.
7. **Identity and display name are separate.** Each participant has a stable id kept apart from the
   name, so two people on the default name never count as one.
8. **No new dependencies** unless they solve something that cannot reasonably be done by hand. Today
   there are two: `preact` and `esbuild`.

## Conventions

- **Code comments are in Spanish** (the maintainer's language). **Everything else is in English** —
  panel copy, tooltips, options page, README, CONTRIBUTING, **commit messages, branch names and PR
  titles**. Comments are the only Spanish in the repository.
- Comment the **why**, not the what. The comments that earn their keep explain a decision that looks
  odd and is not. Most files open with a doc-comment stating the file's reason to exist; keep that
  shape when adding one.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jeremybacher/meet-music](https://github.com/jeremybacher/meet-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
