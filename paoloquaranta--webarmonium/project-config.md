---
trigger: always_on
description: Webarmonium is a real-time collaborative music platform that transforms gestures into algorithmic music. This file provides essential orientation for working with the codebase.
---

# CLAUDE.md

Webarmonium is a real-time collaborative music platform that transforms gestures into algorithmic music. This file provides essential orientation for working with the codebase.

## Project Overview

Webarmonium enables multiple users to create music together through intuitive gestures on a shared canvas. The platform uses advanced gesture recognition, algorithmic music generation, and real-time synchronization to create an immersive collaborative experience.

### Core Architecture

**Three-Tier System:**
- **Frontend**: Vanilla JavaScript + Canvas + Web Audio API
- **Backend**: Node.js + Express + Socket.io
- **Audio Engine**: Tone.js + Web Audio API integration

**Key Features:**
- Real-time multi-user collaboration (5-10 users per room)
- Gesture-to-music transformation with algorithmic composition
- 60fps canvas rendering with cursor synchronization
- Environmental memory for pattern learning (24-hour retention)
- Cross-platform support (mouse, touch, gyroscope)

## Development Environment Setup

### Prerequisites
- **Node.js 18+** (use Node 20 for MCP support)
- Modern browser with Web Audio API support

### Quick Start
```bash
# Backend (port 3001)
cd backend
npm install
npm run dev  # Development with nodemon
# or
npm start    # Production

# Frontend (port 3000)
cd frontend
npm install
npm start    # HTTP server for development

# Access application
http://localhost:3000
```

### Testing
```bash
# Run all tests
cd backend && npm test

# Coverage report
cd backend && npm run test:coverage

# Integration tests (specific scenarios)
NODE_ENV=test npm test -- tests/integration/multiuser-sync.test.js
```

## Codebase Structure

### Frontend (`/frontend/src/`)

**Entry Point:** `main.js` → `WebarmoniumApp` class

**Core Services:**
- `AudioService.js` - Web Audio API + Tone.js integration, three-tier audio system
- `EnhancedGestureCapture.js` - Advanced gesture classification and musical mapping
- `SocketService.js` - WebSocket client communication with retry logic
- `CursorManager.js` - Multi-user cursor rendering and position tracking
- `DrawingRenderer.js` - Collaborative drawing canvas with stroke synchronization

**Key Patterns:**
- Canvas-based 60fps rendering loop
- Event-driven architecture with service communication
- Modular service design with dependency injection
- Cross-platform input normalization

### Backend (`/backend/src/`)

**Entry Point:** `server.js` → Express server + Socket.io

**Core Services:**
- `RoomManager.js` - Room lifecycle, user capacity, and session management
- `GestureProcessor.js` - Gesture validation and classification
- `BackgroundCompositionService.js` - Algorithmic composition engine
- `CompositionEngine.js` - Core musical form and voice generation
- `EnvironmentalMemoryCoordinator.js` - Pattern learning and evolution
- `HoverOrchestrator.js` - Cross-layer audio modulation
- `DrawingSyncService.js` - Real-time drawing synchronization
- `ColorAssignmentService.js` - Multi-user color assignment

**Data Models:**
- `Room.js` - Room state and capacity management
- `User.js` - Anonymous user sessions
- `Gesture.js` - Gesture validation and processing
- `SoundPattern.js` - Musical pattern definitions
- `MemoryState.js` - Environmental learning memory
- `CursorPosition.js` - Multi-user cursor tracking
- `DrawingStroke.js` - Collaborative drawing data

### API Architecture

**WebSocket Events** (see `api/socketHandlers.js`):
- `user:join` / `user:leave` - Room management
- `gesture:start` / `gesture:end` - Gesture processing
- `cursor:move` - Cursor synchronization
- `drawing:start` / `drawing:stroke` / `drawing:end` - Canvas drawing
- `room:state` - Room state synchronization

**Performance Requirements:**
- API endpoints: <200ms p95 response time
- UI interactions: <100ms response time
- WebSocket latency: <100ms
- Memory usage: <100MB baseline, <500MB peak

## Key Technologies

### Frontend Stack
- **Web Audio API** - Low-level audio synthesis
- **Tone.js v14.7.77** - Music synthesis framework
- **Socket.io Client v4.7.5** - Real-time communication
- **Canvas API** - High-performance rendering
- **Vanilla JavaScript** - No framework dependencies

### Backend Stack
- **Node.js 18+** - Runtime environment
- **Express.js** - REST API framework
- **Socket.io v4.7.5** - Real-time bidirectional communication
- **UUID v4** - Anonymous session management
- **Rate Limiting** - Security and performance protection

## Development Patterns

### Audio System
The three-tier audio architecture handles:
1. **Background Layer** - Ambient algorithmic music
2. **Remote Layer** - Other users' musical contributions
3. **Local Layer** - User's own gesture-generated audio

### Gesture Processing Pipeline
```
Input (Mouse/Touch/Gyro) → EnhancedGestureCapture →
Classification → Musical Mapping → SocketService →
Backend Processing → Sound Generation → Audio Output
```

### Multi-user Synchronization
- Real-time cursor position sharing
- Collaborative drawing with stroke synchronization
- Color-based user identification
- Room state management with capacity limits

### Algorithmic Music Generation
The composition system uses multiple engines:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PaoloQuaranta) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
