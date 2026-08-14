---
trigger: always_on
description: Notes for future Claude. Keep this short and useful.
---

# CLAUDE.md

Notes for future Claude. Keep this short and useful.

## What this is

Unofficial **Discord Bot Client** ([Uncover-it/bot-client](https://github.com/Uncover-it/bot-client)). A Next.js 16 web app that logs in with a Discord **bot token** and acts as a full Discord client for that bot: browse guilds, channels, members, send/edit/delete messages, manage roles/channels/permissions, etc.

- Auth: bot token stored in an HTTP-only `token` cookie. No DB. No user accounts.
- Real-time: direct browser WebSocket to `wss://gateway.discord.gg` (Discord Gateway v10).
- REST: server actions in `src/api/*/actions.ts` proxy to `https://discord.com/api/v10` with `Authorization: Bot <token>`.

## Branch / git

- **Working branch: `rewrite`** (do work here unless told otherwise).
- `main` is the published / deployed branch.
- Recent rewrite history is shallow: `0ed3783 init` is the rewrite baseline.

## Stack

- Next.js 16 (App Router) + React 19, TypeScript strict.
- **React Compiler is on** (`reactCompiler: true` in `next.config.ts`, via
  `babel-plugin-react-compiler`). Components and hooks are auto-memoized, so
  new code does not need `useMemo`/`useCallback` for plain render work. Keep
  them only where the value must be referentially stable for a non-React
  reason. Existing manual memoization is harmless and was left alone.
- Bun runtime (`bun --bun next ...`). Lockfile is `bun.lock`.
- Tailwind v4 (`@tailwindcss/postcss`), `tw-animate-css`.
- shadcn/ui-style primitives in `src/components/ui/` (Radix under the hood via `radix-ui`).
- State: Zustand (`src/lib/store.ts` — `useRealtimeStore`).
- Markdown: `react-markdown` + `remark-gfm`.
- Toasts: `sonner`. Icons: `lucide-react`. Emoji picker: `frimousse`.

## Scripts

- `bun run dev` — dev server (port 3000).
- `bun run build` / `bun run start` — prod.
- `bun run lint` — Biome (`biome check`). `bun run format` writes fixes.
- `bunx tsc --noEmit` — typecheck.

## Path alias

`@/*` → `src/*` (see `tsconfig.json`).

## Directory map

```
src/
  app/                       Next.js App Router
    page.tsx                 Login screen (token entry)
    layout.tsx               Root layout
    dashboard/
      layout.tsx             Auth gate + GatewayProvider + Sidebar shell
      page.tsx               Dashboard landing
      servers/[serverId]/
        settings/page.tsx    Per-guild settings
        channels/[channelId]/page.tsx   Channel view (the main UI)
      dms/[channelId]/page.tsx          DM view
  api/                       "use server" actions (NOT route handlers)
    session/actions.ts       getSessionToken, getCurrentUser
    validate/actions.ts      validateToken
    data/actions.ts          ALL Discord REST: guilds, channels, messages,
                             roles, members, bans, emojis, stickers, reactions,
                             pins, typing, kick/ban/timeout, etc.
  components/
    sidebar.tsx              Guild + channel navigation sidebar
    login.tsx                Token form
    settings.tsx             Bot/app settings
    stickerList.tsx
    contextMenuHandellers.tsx  (sic — typo, kept as-is)
    theme-toggle.tsx, year-footer.tsx
    providers/
      gateway-provider.tsx   Boots gateway + REST ping interval
    discord/                 Feature components (the "chat" UI)
      channel-view.tsx       Channel container (text/voice/forum dispatch)
      dm-view.tsx            DM container (same message stack, no guild)
      dm-sidebar-section.tsx DM list + "new conversation" dialog
      message-list.tsx       Message list + day dividers + jump-to-reply + hover toolbar
      message.tsx            Single message row (avatar, name, content, reply ref)
      message-input.tsx      Composer with reply state, typing, attachments
      message-content.tsx    Markdown + mention rendering
      message-reactions.tsx  Reaction pills (toggle + hover reactor preview)
      reaction-viewer.tsx    Who-reacted tooltip body + per-emoji dialog
      message-embed.tsx, message-attachment.tsx
      member-list.tsx        Right sidebar members
      timeout-banner.tsx     Shown when the bot itself is timed out
      unread-badge.tsx       Per-channel unread count
      session-overview.tsx   Dashboard landing readout
      forum-view.tsx, voice-view.tsx
      channel-settings-dialog.tsx, server-settings.tsx, role-editor.tsx
      user-profile-popover.tsx, status-bar.tsx, emoji-picker-pro.tsx
      intent-warning.tsx     Banner when privileged intents are missing
    ui/                      shadcn primitives (button, dialog, sidebar, ...)
  hooks/
    use-gateway.ts           Connects DiscordGateway, pipes events to store
    use-permissions.ts       Channel/guild permission resolution
    use-self-timeout.ts      Is the bot timed out here, plus countdown
    use-open-dm.ts           Open/create a DM and navigate to it
    use-sidebar-resize.ts, use-mobile.ts, use-hydrated.ts
  lib/
    store.ts                 Zustand realtime store (guilds, messages, members,
                             selfMembers, presences, typing, dms, unread)
    utils.ts                 cn() etc.
    merge-button-refs.ts
    discord/
      gateway.ts             DiscordGateway class (WS, heartbeat, resume, intents)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Uncover-it/bot-client](https://github.com/Uncover-it/bot-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
