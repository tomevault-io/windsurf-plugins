---
trigger: always_on
description: Context for AI agents working with `@runwayml/avatars-react` - a React SDK for real-time AI avatar interactions via WebRTC.
---

# Agent Guidelines

Context for AI agents working with `@runwayml/avatars-react` - a React SDK for real-time AI avatar interactions via WebRTC.

## Quick Reference

| Resource | Location |
|----------|----------|
| Package README | `README.md` |
| Next.js example | `examples/nextjs/` |
| Types | `src/types.ts` |

## Architecture

```
src/
├── components/    # Headless React components with render props
├── hooks/         # useAvatarSession, useAvatar, useLocalMedia
├── api/           # Session consumption endpoint
├── types.ts       # SessionCredentials, SessionState, Props types
└── index.ts       # Public exports
```

## Component Hierarchy

```
AvatarCall (handles session creation + styled container)
└── AvatarSession (WebRTC provider)
    ├── AvatarVideo (remote avatar)
    ├── UserVideo (local camera)
    ├── ControlBar (mic/camera/end)
    └── ScreenShareVideo

AvatarProvider (handles session creation, no container element)
└── AvatarSession (WebRTC provider)
    └── {children}  ← full layout control
```

## Key Patterns

**Session states:** `idle` → `connecting` → `active` → `ending` → `ended` (or `error`)

**Render props:** All display components accept `children` as render function:
```tsx
<AvatarVideo>
  {(avatar) => {
    switch (avatar.status) {
      case 'connecting': return <Spinner />;
      case 'waiting': return <Placeholder />;
      case 'ready': return <VideoTrack trackRef={avatar.videoTrackRef} />;
    }
  }}
</AvatarVideo>
```

**Hooks require context:** Must be inside `<AvatarCall>`, `<AvatarProvider>`, or `<AvatarSession>`

## Components

| Component | Purpose |
|-----------|---------|
| `AvatarCall` | High-level component, handles session creation + styled container |
| `AvatarProvider` | Headless provider — same credential handling, no container element |
| `AvatarSession` | Low-level, requires pre-fetched credentials |
| `AvatarVideo` | Renders remote avatar video |
| `UserVideo` | Renders local camera |
| `ControlBar` | Mic/camera/end-call buttons |
| `ScreenShareVideo` | Renders screen share |
| `PageActions` | Handles click/scroll/highlight events from the avatar (renders null) |

## Hooks

| Hook | Purpose |
|------|---------|
| `useAvatarStatus` | Discriminated union of full avatar lifecycle (recommended) |
| `useAvatarSession` | Session state and `end()` control |
| `useAvatar` | Remote avatar video track |
| `useLocalMedia` | Local mic/camera toggles |
| `useClientEvent` | Subscribe to a single client event type by tool name (state + callback) |
| `useClientEvents` | Listen for all client events from the avatar |
| `usePageActions` | Subscribes to page-action events and executes DOM click/scroll/highlight |
| `useTranscription` | Callback for each `RoomEvent.TranscriptionReceived` segment (side effects, logging) |
| `useTranscript` | Accumulated transcript for UI; upserts by LiveKit segment `id` (dedup) with optional buffer cap |

## Commands

```bash
bun run dev        # Watch mode
bun run build      # Build package
bun run typecheck  # TypeScript check
bun run lint       # Biome linter
bun test           # Run tests
```

## Source Files

| Purpose | Path |
|---------|------|
| All types | `src/types.ts` |
| High-level component | `src/components/AvatarCall.tsx` |
| Headless provider | `src/components/AvatarProvider.tsx` |
| Session provider | `src/components/AvatarSession.tsx` |
| Avatar video | `src/components/AvatarVideo.tsx` |
| User video | `src/components/UserVideo.tsx` |
| Controls | `src/components/ControlBar.tsx` |
| Session hook | `src/hooks/useAvatarSession.ts` |
| Avatar hook | `src/hooks/useAvatar.ts` |
| Media hook | `src/hooks/useLocalMedia.ts` |
| Page actions component | `src/components/PageActions.tsx` |
| Page actions hook | `src/hooks/usePageActions.ts` |
| Page actions tools (server) | `src/api/page-actions.ts` |
| Server example | `examples/nextjs/app/api/avatar/connect/route.ts` |

## Design Principles

- **No direct LiveKit imports in examples/consumer code.** If an example needs to import from `@livekit/components-react` or `livekit-client`, that's a signal the SDK isn't exposing enough. Treat every direct LiveKit import as a missing SDK API surface.
- **Default control chrome:** bundled `styles.css` treats mic/camera/screen-share “off” as neutral dimmed (not error red), uses a blue accent while screen sharing, and reserves red for end-call; tune off-state with `--avatar-control-bg-off` and `--avatar-control-color-off`. When sharing, default layout shows screen share as the main region with avatar picture-in-picture; `useLocalMedia.toggleScreenShare` uses `CaptureController` + `setFocusBehavior('no-focus-change')` when the browser supports it so picking another tab to share is less likely to steal focus (otherwise degrades gracefully).

## Learned User Preferences

- Do not merge or close pull requests on the user's behalf unless they explicitly ask; they merge and close PRs themselves.
- Changelog entries should stay user-facing — describe what works now in the SDK, not implementation details. Don't leak event names, JSON shapes, transport layer names, or internal option flags into `CHANGELOG.md`.

## Learned Workspace Facts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runwayml/avatars-sdk-react](https://github.com/runwayml/avatars-sdk-react) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
