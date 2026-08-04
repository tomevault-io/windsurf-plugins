---
trigger: always_on
description: This document is the frontend working guide for agents and developers.
---

# Frontend AGENTS.md

This document is the frontend working guide for agents and developers.
Keep it aligned with `frontend/src` source code.

## Stack

- React 18 + TypeScript
- Vite
- Vitest + Testing Library
- shadcn/ui primitives
- Tailwind utility classes + local CSS (`index.css`, `styles.css`)
- Sonner (toasts)
- Leaflet / react-leaflet (map)
- `@michaelhart/meshcore-decoder` installed via npm alias to `meshcore-decoder-multibyte-patch`
- `meshcore-hashtag-cracker` + `nosleep.js` (channel cracker)
- Multibyte-aware decoder build published as `meshcore-decoder-multibyte-patch`

## Code Ethos

- Prefer fewer, stronger modules over many thin wrappers.
- Split code only when the new hook/component owns a real invariant or workflow.
- Keep one reasoning unit readable in one place, even if that file is moderately large.
- Avoid dedicated files whose main job is pass-through, prop bundling, or renaming.
- For this repo, "locally dense but semantically obvious" is better than indirection-heavy "clean architecture".
- When refactoring, preserve behavior first and add tests around the seam being moved.

## Frontend Map

```text
frontend/src/
├── main.tsx                # React entry point (StrictMode, root render)
├── App.tsx                 # Data/orchestration entry that wires hooks into AppShell
├── api.ts                  # Typed REST client
├── types.ts                # Shared TS contracts
├── useWebSocket.ts         # WS lifecycle + event dispatch
├── wsEvents.ts             # Typed WS event parsing / discriminated union
├── prefetch.ts             # Consumes prefetched API promises started in index.html
├── index.css               # Global styles/utilities
├── styles.css              # Additional global app styles
├── themes.css              # Color theme definitions
├── contexts/
│   ├── DistanceUnitContext.tsx # Browser-local distance-unit context/provider
│   └── PushSubscriptionContext.tsx # Push subscription state context/provider
├── lib/
│   └── utils.ts            # cn() — clsx + tailwind-merge helper
├── hooks/
│   ├── index.ts            # Central re-export of all hooks
│   ├── useConversationActions.ts   # Send/resend/trace/block conversation actions
│   ├── useConversationNavigation.ts # Search target, selection reset, and info-pane navigation state
│   ├── useConversationMessages.ts  # Conversation timeline loading, cache restore, jump-target loading, pagination, dedup, pending ACK buffering
│   ├── useUnreadCounts.ts          # Unread counters, mentions, recent-sort timestamps
│   ├── useRealtimeAppState.ts      # WebSocket event application and reconnect recovery
│   ├── useAppShell.ts              # App-shell view state (settings/sidebar/modals/cracker)
│   ├── useRepeaterDashboard.ts      # Repeater dashboard state (login, panes, console, retries)
│   ├── useRadioControl.ts          # Radio health/config state, reconnection, mesh discovery sweeps
│   ├── useAppSettings.ts           # Settings, favorites, preferences migration
│   ├── useConversationRouter.ts    # URL hash → active conversation routing
│   ├── useContactsAndChannels.ts   # Contact/channel loading, creation, deletion
│   ├── useBrowserNotifications.ts  # Per-conversation browser notification preferences + dispatch
│   ├── usePushSubscription.ts      # Web Push subscription lifecycle, per-conversation filters
│   ├── useFaviconBadge.ts          # Browser tab unread badge state
│   ├── useRawPacketStatsSession.ts # Session-scoped packet-feed stats history
│   └── useRememberedServerPassword.ts # Browser-local repeater/room password persistence
├── components/
│   ├── AppShell.tsx            # App-shell layout: status, sidebar, search/settings panes, cracker, modals, security warning
│   ├── ConversationPane.tsx    # Active conversation surface selection (map/raw/trace/repeater/room/chat/empty)
│   ├── visualizer/
│   │   ├── useVisualizerData3D.ts   # Packet→graph data pipeline, repeat aggregation, simulation state
│   │   ├── useVisualizer3DScene.ts  # Three.js scene lifecycle, buffers, hover/pin interaction
│   │   ├── VisualizerControls.tsx   # Visualizer legends and control panel overlay
│   │   ├── VisualizerTooltip.tsx    # Hover/pin node detail overlay
│   │   └── shared.ts                # Graph node/link types and shared rendering helpers
│   └── ...
├── utils/
│   ├── urlHash.ts              # Hash parsing and encoding
│   ├── conversationState.ts    # State keys, in-memory + localStorage helpers
│   ├── messageParser.ts        # Message text → rendered segments
│   ├── pathUtils.ts            # Distance/validation helpers for paths + map
│   ├── pubkey.ts               # getContactDisplayName (12-char prefix fallback)
│   ├── contactAvatar.ts        # Avatar color derivation from public key
│   ├── rawPacketIdentity.ts    # observation_id vs id dedup helpers
│   ├── rawPacketStats.ts       # Session packet stats windows, rankings, and coverage helpers
│   ├── regionScope.ts          # Regional flood-scope label/normalization helpers
│   ├── visualizerUtils.ts      # 3D visualizer node types, colors, particles
│   ├── visualizerSettings.ts   # LocalStorage persistence for visualizer options
│   ├── a11y.ts                 # Keyboard accessibility helper

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jkingsman/Remote-Terminal-for-MeshCore](https://github.com/jkingsman/Remote-Terminal-for-MeshCore) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
