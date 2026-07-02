---
trigger: always_on
description: - This project is a `Next.js 14` App Router application for three AI tools:
---

# Project Guide For Future Agents

## Overview

- This project is a `Next.js 14` App Router application for three AI tools:
  - chat: `/chat`
  - image generation: `/gen-image`
  - translation: `/translate`
- UI stack: `React 18`, `Tailwind CSS`, `shadcn/ui`, `Radix UI`, `framer-motion`.
- State management: `Zustand` with `persist`.
- Primary deployment target: `Cloudflare Pages` with `@cloudflare/next-on-pages`.
- Internal models are served through Next edge routes backed by the Cloudflare `AI` binding.
- External models are called directly from the browser via SDK/HTTP using user-provided API keys stored in local state/localStorage.

## Run And Deploy

- Install: `pnpm install`
- Local dev UI: `pnpm dev`
- Cloudflare Pages build: `pnpm pages:build`
- Local Pages preview: `pnpm preview`
- Deploy to Cloudflare Pages: `pnpm deploy`
- Deploy to Cloudflare Pages dev preview branch: `pnpm deploy:dev`

## Environment And Platform Notes

- Cloudflare config is in `wrangler.toml`.
- Pages dev deployments are branch-based preview deployments. This repo uses `pnpm deploy:dev` to deploy to the `dev` branch preview environment.
- Edge API routes rely on `getRequestContext().env`, so cloud deployment assumptions are built into `app/api/**/route.js`.
- `.env.example` only covers some browser-side vars. Real production secrets for Cloudflare/Gemini/private routes come from Cloudflare env/bindings, not from this file.
- README explicitly states the project is intended for Cloudflare deployment only.

## Directory Map

- `app/`: App Router pages, layout, and API routes.
- `components/`: UI components, grouped by domain (`chat`, `navbar`, `sidebar`, `common`, `ui`, `anim`).
- `hooks/`: page/model/chat hooks.
- `service/`: request layer and provider integrations.
- `store/`: global Zustand stores.
- `utils/`: shared enums, message builders, stream parsing, model metadata, storage helpers.
- `provider/`: theme provider.

## Main User Flows

### 1. Chat

- Page entry: `app/chat/page.js`
- Main container: `components/chat/index.jsx`
- User input + submission orchestration: `hooks/useChat.js`
- Internal chat API: `service/api.js -> /api/chat`
- External chat APIs:
  - `service/openai.js`
  - `service/deepseek.js`
  - `service/grok.js`
  - `service/gemini.js`
  - `service/ollama.js`

### 2. Image Generation

- Page entry: `app/gen-image/page.js`
- Shares the same chat UI shell, but `type` switches behavior in `useChat`.
- Internal image API: `service/api.js -> /api/gen-image`
- Optional prompt pre-translation is handled client-side in `genImageWithApi(..., preTrans)`.

### 3. Translation

- Page entry: `app/translate/page.js`
- Internal translation API: `service/api.js -> /api/trans`
- Translation target selection is stored in `useModelStore.currentModelInfo.transTarget`.

## Frontend Architecture

### Root Layout

- `app/layout.js` wires:
  - theme provider
  - sidebar provider
  - navbar
  - global toaster
  - analytics helpers
  - decorative cursor/loading components

### Shared Chat Shell

- `components/chat/index.jsx` is the shared screen for all three tools.
- It initializes models with `useInitModel()`.
- It reads conversation state from `useChat(type)`.
- It handles auto-scroll and regenerate-last-user-message behavior.

### Chat Component Roles

- `chat-topbar.jsx`: top actions/context for current tool.
- `chat-list.jsx`: message list container.
- `chat-message.jsx`: message rendering, markdown/image/pending state.
- `chat-bottombar.jsx`: input area and submit actions.
- `chat-scroll.jsx`: floating scroll-to-bottom helper.

## State Management

State is centralized in `store/index.js`.

### `useModelStore`

- Stores:
  - fetched internal model lists by route key
  - current selected model info by route key
  - third-party provider config, including API keys and Ollama host/model
- Persisted under `model-store`.
- Route keys are derived by `mapPathToKey()` in `utils/router.js`.

### `useChatStore`

- Stores three separate histories:
  - `messages`
  - `transMessages`
  - `imgMessages`
- Also stores `preTrans` for image prompt pre-translation.
- Persisted under `chat-store`.
- History caps:
  - chat: 500
  - translate: 500
  - image: 100

### `useChatStatusStore`

- Stores transient request loading state only.
- Not persisted.

### `useUserStore`

- Stores local nickname only.

## Request/Data Flow

### Internal Models

1. Page loads shared chat shell.
2. `useInitModel()` calls `getModels(path.replace('/', ''))`.
3. `service/api.js` requests `/api/models`.
4. `app/api/models/route.js` returns model metadata from `app/api/utils/models.js`.
5. User submits input in `useChat(type)`.
6. Based on tool type, request goes to:
   - `/api/chat`
   - `/api/gen-image`
   - `/api/trans`
7. Response is stored in Zustand and rendered by shared chat UI.

### External Models

1. `useChat(type)` chooses provider via `getChatApi(currentModel.type)`.
2. API keys/hosts are read from local persisted store via `utils/storage.js`.
3. Provider clients stream directly in the browser.
4. Stream chunks are normalized into the shared append-chunk flow.

## API Routes

### `/api/chat`

- File: `app/api/chat/route.js`
- Runtime: `edge`
- Uses Cloudflare `env.AI.run(...)`
- `POST` uses streaming output.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wuyangwang/my-chat-web](https://github.com/wuyangwang/my-chat-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-02 -->
