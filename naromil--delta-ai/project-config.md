---
trigger: always_on
description: Electron desktop app (BYOK) with a ChatGPT-like chat UI (React 19) and an
---

# Delta AI

Electron desktop app (BYOK) with a ChatGPT-like chat UI (React 19) and an
always-on-top lookup popup that performs OCR on screen captures via global hotkey.
Targets Linux (X11 + KDE Wayland first-class), macOS/Windows secondary.

## Commands

- `npm run dev` — Launch app with HMR (electron-vite)
- `npm run typecheck` — `tsc --noEmit` (node: `src/main/**`, `src/preload/**`; web: `src/renderer/src/**`)
- `npm run lint` — ESLint + Prettier
- `npm run build` — Typecheck then `electron-vite build`
- `npm run format` — Prettier write across the repo

**Always run `npm run typecheck && npm run lint` after changing `.ts`/`.tsx`.**
The build will fail on typecheck errors, so fix them before committing.

## Critical rules

- Don't import Electron main APIs into the renderer; use the preload bridge.
- Don't bypass `registerHotkey` for global shortcuts — the portal path won't fire on Wayland.
- Don't commit secrets. API keys live only in `{userData}/config/providers.json`.
- Use custom scrollbars everywhere — never the native OS scrollbar. Style `::-webkit-scrollbar` on scrollable elements.
- Store all AI-facing prompt strings as exported constants in `src/shared/prompts.ts`. Never inline a prompt in a handler or component.
- When architecture changes, update this file.

## Detailed guidelines

### Architecture & Module Map

#### High-level call chain

```
Renderer (React 19) → Preload (contextBridge) → Main process (Node)
├── App.tsx / LookupApp.tsx ├── index.ts Streaming IPC (chat-send/chat-expand)
├── useChatStreaming hook │ + lookup-trigger-grow, lookup-transfer
├── Conversation/Turn/ExpansionFrame components ├── provider.ts AI dispatch (callProviderStream)
└── window.api.chatSend / chatExpand / lookup* ├── config.ts Persistence, hotkey, Wayland
├── lookup/ OCR → popup
└── services/ Wayland D-Bus portals
```

#### IPC channels

| Channel                                   | Direction       | Purpose                                                                       |
| ----------------------------------------- | --------------- | ----------------------------------------------------------------------------- |
| `chat-send`                               | Renderer → Main | Send a streaming chat message (role payload discriminates 'chat' vs 'lookup') |
| `chat-chunk`                              | Main → Renderer | Streaming chunk for a chat-send request (keyed by `requestId`)                |
| `chat-response`                           | Main → Renderer | Final response for a chat-send                                                |
| `chat-error`                              | Main → Renderer | Error for a chat-send                                                         |
| `chat-expand`                             | Renderer → Main | Request an inline word-expansion stream                                       |
| `chat-expand-chunk`                       | Main → Renderer | Streaming chunks for an expand request (keyed by `requestId`)                 |
| `chat-replace-conversation`               | Main → Renderer | Hydrate transferred state + conversationId/title into the chat window         |
| `lookup-trigger-grow`                     | Lookup → Main   | On first ask, signal main to animate window growth                            |
| `lookup-transfer`                         | Lookup → Main   | Send `{state, conversationId?}` to chat; main transforms context, persists    |
| `ai-error`                                | Main → Lookup   | Capture or OCR error message (sent from lookup.ts & window.ts)                |
| `lookup-context`                          | Main → Lookup   | OCR context state (`{status, text, hint}`)                                    |
| `lookup-grow`                             | Main → Lookup   | (`width, height`) signal grow animation on the renderer side                  |
| `lookup-paste-text`                       | Lookup → Main   | Pasted text replaces OCR context                                              |
| `lookup-paste-image`                      | Lookup → Main   | Pasted image → OCR → context                                                  |
| `lookup-ocr-image`                        | Lookup → Main   | Invoke OCR on an image (returns `{text, error?}`)                             |
| `lookup-input-changed`                    | Lookup → Main   | Whether Ask field has text (guards blur-to-close)                             |
| `lookup-close`                            | Lookup → Main   | Close the lookup window                                                       |
| `load-model-config` / `save-model-config` | Renderer ↔ Main | Model config CRUD                                                             |
| `load-settings` / `save-settings`         | Renderer ↔ Main | App settings CRUD                                                             |
| `conversation-save` / `-load` / `-delete` | Renderer → Main | Conversation CRUD to `{userData}/conversations/`                              |
| `conversation-list`                       | Renderer → Main | List chat-source conversations (metadata sorted by updatedAt desc)            |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [naromil/delta-ai](https://github.com/naromil/delta-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
