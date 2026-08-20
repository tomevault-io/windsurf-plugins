---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev              # Start frontend (Vite) + backend (Convex) in parallel
npm run dev:frontend     # Start only Vite dev server
npm run dev:backend      # Start only Convex dev server
npm run build            # Build frontend (vite build)
npm run convert-images   # Convert PNG images to WebP format
npm run lint             # Full check: tsc (convex + app) → convex dev --once → vite build
npx convex dev --once    # Push schema/functions to Convex without watching
```

Tests: `npm run test` (vitest + convex-test; `*.test.ts` em `convex/`). Seed data is available via `convex/seed.ts`.

## Architecture

Multi-tenant CRM with human-AI team collaboration. Convex backend, React + TailwindCSS frontend.

**Multi-tenancy:** Every table has `organizationId`. All queries must be scoped to the user's org.

**Auth flow:** `requireAuth(ctx, organizationId)` from `convex/lib/auth.ts` handles auth + org membership. Returns the team member. `requirePermission(ctx, organizationId, category, level)` extends this with granular RBAC checks. A few functions without org context (e.g. `getUserOrganizations`) still use `getAuthUserId` directly.

**Permissions (RBAC):** 9 permission categories (leads, contacts, inbox, tasks, reports, team, settings, auditLogs, apiKeys) with hierarchical levels. Defined in `convex/lib/permissions.ts`. Each role (admin, manager, agent, ai) has sensible defaults; admins can override per-member. Frontend uses `usePermissions(organizationId)` hook and `<PermissionGate>` component.

**Invite flow:** Admins invite humans via `inviteHumanMember` action (creates auth account with bcrypt-hashed temp password). New users must change password on first login (`mustChangePassword` flag). Users auto-linked to org via `afterUserCreatedOrUpdated` auth callback.

**Side effects in mutations:** Most mutations insert into `activities` + `auditLogs` and trigger webhooks via `ctx.scheduler.runAfter(0, internal.nodeActions.triggerWebhooks, ...)`.

**Email/Notifications:** `@convex-dev/resend` component for email delivery. Central dispatch via `convex/email.ts` — all events go through `internal.email.dispatchNotification`. Templates in `convex/emailTemplates.ts` (PT-BR). Per-member preferences in `notificationPreferences` table (opt-out model — no row means all enabled). Convex components registered in `convex/convex.config.ts`. **Email env vars:** `RESEND_API_KEY`, `APP_URL`, `RESEND_FROM_EMAIL` (default: `HNBCRM <noreply@mail.hnbcrm.com>`), `RESEND_WEBHOOK_SECRET`. Domain: `mail.hnbcrm.com` (subdomain to avoid Gmail MX conflict).

**HTTP API:** `convex/router.ts` has RESTful endpoints at `/api/v1/` authenticated via `X-API-Key` header. API keys resolve permissions from key → team member → role defaults. Routes wired in `convex/http.ts`.

**WhatsApp channel:** Per-organization channel configs (`convex/channelConfigs.ts`, `channelConfigs` table) with `provider: "meta" | "bridge"` — two transports on the same `whatsapp` channel. `meta` = official WhatsApp Cloud API (Graph API, 24h service window, message templates); ingress at `GET/POST /webhooks/whatsapp` (`convex/whatsapp.ts`). `bridge` = self-hosted wuzapi gateway (WhatsApp Web protocol via whatsmeow, QR pairing, no 24h window, no templates); ingress at `POST /webhooks/bridge` verified by HMAC-SHA256 via env `WA_BRIDGE_HMAC_SECRET` (`convex/bridge.ts`). Managed provisioning: envs `WA_BRIDGE_DEFAULT_URL` + `WA_BRIDGE_ADMIN_TOKEN` (both set = "Servidor HNBCRM" default in the provisioning form — user only types a display name; admin token never leaves the server; advanced users can still point to their own gateway). Bridge is opt-in per org and carries a permanent-ban risk (unofficial protocol, violates WhatsApp ToS). Key files: `convex/bridge.ts`, `convex/lib/bridgeParse.ts` (pure parser), `convex/lib/bridgeSend.ts` (text), `convex/lib/bridgeMedia.ts` (media), `convex/lib/bridgeSession.ts` (QR/status/provisioning). Outbound dispatch in `convex/whatsapp.ts` branches by provider (voice notes are transcoded to ogg/opus via the Whisper service's `/convert`). Typing presence and delivery/read receipts flow both ways (`sendTypingState`, `markConversationRead`, `ChatPresence`/`ReadReceipt` webhook events). UI in `src/components/settings/ChannelsSection.tsx` (+ `ChannelHealthPanel` with 7-day delivery stats).

**Inbox features:** Voice-note transcription via self-hosted Whisper (`convex/transcription.ts`, env `WHISPER_SERVICE_URL`/`WHISPER_SERVICE_TOKEN`, opt-in per org via `channelConfigs.autoTranscribeAudio`; transcript mirrored to `messages.transcriptText`). Full-text message search (search indexes `search_content` + `search_transcript` on `messages`, query `conversations.searchMessages`). Quick replies (`convex/quickReplies.ts`, "/" trigger in the composer). Scheduled messages (`convex/scheduledMessages.ts`, `ctx.scheduler.runAt`). Conversation archiving + org-scoped labels (`conversations.ts`, `conversationLabels` table) with bulk actions. Inbox UI pieces live in `src/components/inbox/`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ericmil87/hnbcrm](https://github.com/ericmil87/hnbcrm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
