---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->

# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` (resolved from this file's directory; in monorepos the `next` package may not be visible from the repo root) before writing any code. Heed deprecation notices.

This block is written and re-added by `next dev` — verify at `node_modules/next/dist/server/lib/generate-agent-files.js`. Removing it from a diff only re-creates the uncommitted change; committing it with your work keeps the tree clean.

<!-- END:nextjs-agent-rules -->

# wspbot

A WhatsApp bot that answers when tagged in a group. Next.js app deployed as a Docker container on
a Dokploy VPS at `wspbot.crafter.run`. Built by Jibaru of Crafter Station (jibaru.dev).

## Shape

Two ways in, and they share almost nothing. Messages arrive as a webhook push and are answered
by the model; the dashboard is a set of gated pages that configure what the model is allowed to
do. The `features` table is the only thing both halves touch.

```
WhatsApp ──▶ wapi ──POST /api/wapi/webhook──▶ this app
                                                 ├─▶ OpenAI via the Vercel AI SDK (+ web search)
                                                 ├─▶ ffmpeg (stickers, voice notes, video)
                                                 ├─▶ Postgres ◀── the dashboard writes here
                                                 └─▶ wapi, via the vendored SDK ──▶ WhatsApp

you ──▶ / (public) ──▶ /login ──▶ proxy.ts ──▶ /dashboard/{features,limits,stickers,
                                       │           memory,reminders,summaries,move,usage}
                                       └─▶ features table ──▶ which tools a turn is given
```

**Inbound**

```
app/api/wapi/webhook/route.ts    the only entry point for messages; verify, ack, work in after()
lib/signature.ts                 webhook signature verification (plain compare or HMAC)
lib/mentions.ts                  parsing message nodes, "is this for me?"
lib/inbound-media.ts             decrypting what arrived attached
```

**The turn**

```
lib/agent.ts                     prompt + every tool; the whole model turn
lib/features.ts                  the registry: switches, tool ownership, self-description
lib/about.ts                     what the bot knows about itself
lib/memory.ts                    facts, per chat or global
lib/tasks.ts                     the per-chat checklist
lib/reminders.ts                 scheduled work; lib/reminder-runner.ts fires it
lib/rate-limit.ts                per-person quotas, checked before anything costs money
lib/transfer.ts                  moving a group's context into another group (dashboard only)
lib/summaries.ts                 scheduled digests: schedules, the log, the transcript
lib/summary-recorder.ts          writing down a recorded group; lib/summary-runner.ts fires it
lib/cron.ts                      five-field cron, evaluated as "does this minute match?"
lib/usage.ts                     token accounting, cost estimate
```

**Dashboard**

```
proxy.ts                         gates every page (Next 16 renamed middleware -> proxy)
lib/auth.ts                      bcrypt at sign-in, signed cookie thereafter
app/login/                       sign-in page and its server action
app/page.tsx                     the public landing page (the only ungated route)
app/landing.css                  its brand styles, scoped under .lp
app/crafter-mark.tsx             the real Crafter Station mark and horizontal lockup
app/dashboard/                   one route per section, each with its own actions.ts
app/dashboard/layout.tsx         shell + nav; nav.tsx is the only client component
public/                          generated icon set; rebuild from public/icon.config.json
```

**Outbound and media**

```
lib/wapi.ts                      thin facade over the SDK: server-only, identity cache, 2 clients
lib/wapi-sdk/                    the official wapi SDK, vendored (see below)
lib/stickers.ts                  the shared sticker library
lib/sticker-maker.ts             ffmpeg: anything -> 512x512 WebP
lib/audio.ts                     TTS output -> Ogg/Opus
lib/video.ts                     anything -> H.264/AAC MP4
lib/ffmpeg.ts                    shared ffmpeg runner + scratch dirs
lib/fetch-media.ts               guarded remote downloads (SSRF)
```

**Integrations and plumbing**

```
lib/notion.ts                    Notion OAuth + page operations
lib/oauth-state.ts               signed OAuth state (no server-only, so it is testable)
lib/sheets.ts                    Google Sheets read and write
lib/session.ts                   reconnecting a dropped WhatsApp session
instrumentation.ts               starts the session watchdog and the reminder tick at boot
lib/db.ts                        Postgres pool + the idempotent DDL
lib/config.ts                    environment, validated at the point of use
```

## Things that will be re-broken if you don't know them

Each of these cost real debugging time. They are counter-intuitive, and every one of them looks
like a simplification opportunity.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Jibaru/wspbot](https://github.com/Jibaru/wspbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
