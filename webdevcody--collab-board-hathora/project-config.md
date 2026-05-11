---
trigger: always_on
description: refactoring the code or adding new features which might require touching many components
---


# Collaborative Whiteboard/Collaborative Boards - Architecture Documentation

## Overview

This is a scalable real-time collaborative board application that demonstrates the stateful routing pattern for building horizontally scalable real-time applications. The system allows users to create and join collaborative boards where they can draw shapes, add text, and see real-time cursors of other users.

## System Architecture

### High-Level Components

```
┌─────────────┐    HTTP API     ┌──────────────┐    Scheduler     ┌──────────────┐
│   Client    │ ─────────────► │ Backend      │ ──────────────► │ Session      │
│  (React)    │                │ Server       │                  │ Server       │
│             │                │ (Express.js) │                  │ (WebSocket)  │
└─────────────┘                └──────────────┘                  └──────────────┘
                                        │                                 ▲
                                        │ Room allocation                 │ WebSocket
                                        ▼ and host lookup               │ connection
                                ┌──────────────┐                         │
                                │  Scheduler   │ ─────────────────────────┘
                                │   Module     │
                                └──────────────┘
```

### Component Breakdown

1. **Client (React SPA)** - Frontend user interface
2. **Backend Server (Express.js)** - Authentication and room management API
3. **Session Server (Node.js + WebSocket)** - Real-time collaboration server
4. **Scheduler Module** - Room allocation and server discovery system

## Component Details

### 1. Backend Server (`backend-server/`)

**Purpose**: Stateless HTTP API server for authentication and room management.

**Key Files**:

- `src/server.ts` - Express.js server with three main endpoints
- `src/auth.ts` - JWT token creation and validation
- `src/scheduler.ts` - Room allocation logic

**API Endpoints**:

```typescript
// Authentication
POST /api/login
Body: { userId: string }
Response: { token: string }

// Room Creation
POST /api/rooms
Headers: Authorization: Bearer <jwt-token>
Response: { roomId: string }

// Room Lookup
GET /api/rooms/:roomId
Headers: Authorization: Bearer <jwt-token>
Response: { host: string | null, token: string | null }
```

**Authentication Flow**:

1. Client sends username to `/api/login`
2. Server creates JWT with `{ userId }` payload
3. Client stores JWT in sessionStorage
4. All subsequent requests include JWT in Authorization header
5. Server validates JWT and extracts userId

**Room Management**:

- Creates rooms via Scheduler module
- Maps roomId to session server host
- Issues session-specific JWT tokens for WebSocket connections

### 2. Session Server (`session-server/`)

**Purpose**: Stateful WebSocket server handling real-time collaboration for specific rooms.

**Key Files**:

- `src/server.ts` - WebSocket server and connection handling
- `src/room.ts` - Room state management and collaboration logic
- `src/auth.ts` - JWT token verification

**Connection Flow**:

1. Accepts WebSocket upgrades with JWT token as query parameter
2. Validates JWT contains `{ userId, roomId, host }`
3. Verifies request host matches token host (security)
4. Loads or creates Room instance for roomId
5. Adds user to room and starts message handling

**Real-time Message Types**:

```typescript
// Cursor movement
{ type: "cursor_move", x: number, y: number }

// Shape creation
{ type: "shape_create", shapeType: "rectangle"|"oval"|"text", x, y, width, height, text?, fill?, stroke? }

// Shape updates
{ type: "shape_update", shapeId: string, x?, y?, width?, height?, text?, fill?, stroke? }

// Shape deletion
{ type: "shape_delete", shapeId: string }
```

**Room State Management**:

- Maintains connected users list
- Tracks real-time cursor positions
- Manages collaborative shapes (rectangles, ovals, text)
- Broadcasts state snapshots to all connected clients
- Enforces limits (100 users, 1000 shapes per room)

### 3. Scheduler Module

**Purpose**: Core component that allocates rooms to session server instances.

**Interface**:

```typescript
interface Scheduler {
  createRoom(): Promise<string>; // Assigns new room to server instance
  getRoomHost(roomId: string): Promise<string | null>; // Looks up server host for room
}
```

**Two Implementations**:

**StaticScheduler** (Development):

- Takes comma-separated list of session server hosts via `SESSION_SERVER_HOST` env var
- Randomly assigns rooms to available hosts
- Stores room-to-host mapping in memory
- Limitations: No dynamic scaling, no fault tolerance, single-instance backend

**HathoraScheduler** (Production):

- Integrates with Hathora Cloud hosting platform
- Creates session server instances on-demand
- Handles room concurrency (multiple rooms per server instance)
- Provides fault tolerance and horizontal scaling

### 4. Client (`client/`)

**Purpose**: React single-page application providing the user interface.

**Key Files**:

- `src/main.tsx` - App entry point and routing
- `src/components/Auth.tsx` - Authentication wrapper
- `src/components/Lobby.tsx` - Room creation/joining interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [webdevcody/collab-board-hathora](https://github.com/webdevcody/collab-board-hathora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
