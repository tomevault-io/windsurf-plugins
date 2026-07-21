---
trigger: always_on
description: Real-time voice + text + file chat. No sign-up. Cloudflare-native stack.
---

# free4chat — Agent Development Guide

## Project Overview

Real-time voice + text + file chat. No sign-up. Cloudflare-native stack.

- **Live URL**: https://free4.chat
- **Branch**: `cloudflare` (default)
- **Stack**: Next.js 15 → Cloudflare Worker via `@opennextjs/cloudflare`

## Directory Layout

```
free4chat/
├── app/
│   ├── scripts/
│   │   └── patch-worker.mjs          # post-build: bundles BotSession.ts and appends export to .open-next/worker.js
│   ├── src/
│   │   ├── common/
│   │   │   ├── consts.tsx            # LOCAL_PEER_ID = "local-peer-id"
│   │   │   ├── types.tsx             # UserInfo, Message, Color interfaces
│   │   │   └── utils.ts             # strToBgColor, umamiEvent, hashRoom, etc.
│   │   ├── do/
│   │   │   └── BotSession.ts         # Durable Object: Luna chat history + hourly rate limit
│   │   ├── hooks/
│   │   │   └── useChatRoom.ts        # Core RTK hook — all meeting logic lives here
│   │   ├── components/
│   │   │   ├── TurnstileGate.tsx     # Full-page bot challenge wrapper (used in _app.tsx)
│   │   │   ├── RoomContent.tsx       # Room layout (participant grid + chat panel + @luna relay)
│   │   │   ├── UserCard.tsx          # Per-participant card (audio + avatar + mute + screenshare)
│   │   │   ├── AudioVisualizer.tsx
│   │   │   └── TextChatCard.tsx      # Chat sidebar (messages, activity strip, Luna pill)
│   │   └── pages/
│   │       ├── _app.tsx              # App wrapper — loads TurnstileGate around all pages
│   │       ├── index.tsx             # Landing / room join
│   │       ├── room.tsx              # Dynamic import of RoomContent (ssr: false)
│   │       └── api/
│   │           ├── token.ts          # POST /api/token — token server (runs in Worker)
│   │           └── bot.ts            # POST /api/bot — proxies message to BotSession DO
│   ├── wrangler.jsonc                # main: .open-next/worker.js, KV + DO bindings
│   ├── open-next.config.ts
│   └── package.json                  # cf-build = opennextjs build + patch-worker.mjs
└── .github/workflows/
    └── deploy-web.yml                # Lint + type-check → deploy (push to cloudflare branch)
```

## RTK SDK Usage Pattern

The app uses **`useRealtimeKitClient`** (low-level hook) — NOT the higher-level React hooks. All RTK state is managed imperatively through the `meeting` object inside `useChatRoom.ts`.

```ts
const [meeting, initMeeting] = useRealtimeKitClient();
```

### Key meeting APIs currently used

| Object                        | API                                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------------------- |
| `meeting.self`                | `.name`, `.audioEnabled`, `.audioTrack`, `.enableAudio()`, `.disableAudio()`                   |
| `meeting.self` events         | `"audioUpdate"`                                                                                |
| `meeting.participants.joined` | `.toArray()`, events: `"participantJoined"`, `"participantLeft"`, `"audioUpdate"`              |
| `meeting.chat`                | `.messages`, `.sendTextMessage()`, `.sendImageMessage()`, `.sendFileMessage()`, `"chatUpdate"` |
| `meeting`                     | `.join()`, `.leaveRoom()`                                                                      |

### Screen share APIs (RTK native, fully supported)

```ts
meeting.self.enableScreenShare();
meeting.self.disableScreenShare();
meeting.self.screenShareEnabled; // boolean
meeting.self.screenShareTracks; // { video: MediaStreamTrack, audio?: MediaStreamTrack }

participant.screenShareEnabled;
participant.screenShareTracks;

meeting.self.on("screenShareUpdate", buildParticipants);
meeting.participants.joined.on("screenShareUpdate", buildParticipants);
```

Permission check: `meeting.self.permissions.canProduceScreenshare // "ALLOWED" | "NOT_ALLOWED" | "CAN_REQUEST"`

## Data Flow

```
useRealtimeKitClient()
  └── meeting (imperative RTK object)
        └── useChatRoom.ts
              ├── buildParticipants() → UserInfo[]
              └── returns { participants, messages, muteSelf, toggleScreenShare,
                            sendText, sendFile, sendAction, error, resolvedRoomType, ... }
                    └── RoomContent.tsx
                          ├── @luna intercept → POST /api/bot → BotSession DO
                          ├── ScreenShareViewer (one at a time)
                          ├── UserCard.tsx (per participant)
                          │     ├── <audio> element
                          │     └── AudioVisualizer
                          └── TextChatCard.tsx
                                ├── message list (text / file / image / bot / action)
                                ├── Activity strip (Draw · Poll · Games · Luna pill)
                                └── text input + send + file upload
```

## Type Contracts

### UserInfo (common/types.tsx)

```ts
export interface UserInfo {
  name: string;
  room: string;
  className?: string;
  audioStream?: MediaStream | null;
  screenShareStream?: MediaStream | null;
  screenShareEnabled?: boolean;
  peerId: string;
  muteState?: boolean;
}
```

### Message (common/types.tsx)

```ts
export interface Message {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [i365dev/free4chat](https://github.com/i365dev/free4chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
