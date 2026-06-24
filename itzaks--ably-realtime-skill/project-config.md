---
trigger: always_on
description: Professional guide to Ably Realtime for building real-time React/TypeScript applications with pub-sub messaging, channels, presence tracking, Spaces (collaborative UI), LiveObjects (shared state), Chat SDK (complete messaging), and LiveSync (database synchronization). Use when working with Ably, real-time messaging, WebSockets, pub-sub, channels, presence, collaborative features, live cursors, avatar stacks, component locking, shared state, conflict-free updates, chat rooms, typing indicators, m
---


# Ably Realtime for React/TypeScript

Ably Realtime is a platform for building scalable real-time applications with pub-sub messaging, presence tracking, collaborative features, chat, and database synchronization.

## When to Use Each Feature

Ably provides different abstractions for different real-time use cases:

- **Channels (Core Pub-Sub)**: Custom real-time messaging, notifications, live updates, event broadcasting
- **Spaces**: Participant state in collaborative UIs (live cursors, avatar stacks, user locations, component locking)
- **LiveObjects**: Application state synchronization (counters, voting, shared configurations, game state) with conflict-free updates
- **Chat SDK**: Complete messaging apps (1:1 chat, group conversations, livestream chat, support tickets)
- **LiveSync**: Database-to-client synchronization (broadcasting Postgres changes, outbox pattern, transactional consistency)

## Installation

```bash
# Core Ably (required)
npm install ably

# Additional packages (install as needed)
npm install @ably/spaces           # For Spaces
npm install @ably/chat             # For Chat SDK
npm install @ably-labs/models      # For LiveSync Models SDK
```

## Basic Setup

All Ably features require a Realtime client. Create the client outside React components to prevent reconnections on re-renders:

```typescript
// main.tsx or app.tsx
import * as Ably from 'ably';
import { AblyProvider } from 'ably/react';

// Create client OUTSIDE components
const realtimeClient = new Ably.Realtime({
  key: import.meta.env.VITE_ABLY_API_KEY,
  clientId: 'unique-user-id', // Required for Spaces and Chat
});

function Root() {
  return (
    <AblyProvider client={realtimeClient}>
      <App />
    </AblyProvider>
  );
}
```

For production applications, use token authentication instead of API keys. See [references/auth-security.md](references/auth-security.md).

## Quick Start: Channels (Core Pub-Sub)

Basic real-time messaging with channels:

```typescript
import { ChannelProvider, useChannel } from 'ably/react';

// Wrap with ChannelProvider
<ChannelProvider channelName="notifications">
  <NotificationComponent />
</ChannelProvider>

// Use in component
function NotificationComponent() {
  const { publish } = useChannel('notifications', (message) => {
    console.log('Received:', message.data);
    // Update local state with message
  });

  const sendNotification = () => {
    publish('alert', { text: 'New update!', timestamp: Date.now() });
  };

  return <button onClick={sendNotification}>Send</button>;
}
```

For detailed channel operations, presence tracking, and history, see [references/channels/](references/channels/).

## Quick Start: Spaces (Collaborative UI)

Track participant state for collaborative features:

```typescript
import Spaces from '@ably/spaces';
import { SpacesProvider, SpaceProvider, useMembers, useCursors } from '@ably/spaces/react';

// Setup (in root)
const spaces = new Spaces(realtimeClient);

<SpacesProvider client={spaces}>
  <SpaceProvider name="my-collaborative-space">
    <CollaborativeEditor />
  </SpaceProvider>
</SpacesProvider>

// Avatar stack
function AvatarStack() {
  const { self, others } = useMembers();

  return (
    <div>
      <Avatar user={self} />
      {others.map(member => (
        <Avatar key={member.connectionId} user={member} />
      ))}
    </div>
  );
}

// Live cursors
function CursorTracking() {
  const { set } = useCursors((update) => {
    // Render other users' cursors
    renderCursor(update.connectionId, update.position);
  });

  useEffect(() => {
    const handleMove = (e: MouseEvent) => {
      set({ position: { x: e.clientX, y: e.clientY } });
    };
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, [set]);

  return <canvas id="cursor-layer" />;
}
```

For locations, component locking, and advanced patterns, see [references/spaces/](references/spaces/).

## Quick Start: LiveObjects (Shared State)

⚠️ **Public Preview**: LiveObjects API may change before general availability.

Conflict-free shared state synchronization:

```typescript
import { LiveCounter, LiveMap } from "ably/liveobjects";

async function setupSharedState() {
  const channel = realtimeClient.channels.get("game:lobby-1");
  const gameState = await channel.object.get();

  // Create shared counter
  await gameState.set("score", LiveCounter.create(0));

  // Create shared map
  await gameState.set(
    "players",
    LiveMap.create({
      player1: { name: "Alice", ready: false },
      player2: { name: "Bob", ready: false },
    }),
  );

  // Subscribe to changes
  gameState.get("score").subscribe(() => {
    console.log("Score:", gameState.get("score").value());
  });

  // Update values
  await gameState.get("score").increment(10);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itzaks/ably-realtime-skill](https://github.com/itzaks/ably-realtime-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
