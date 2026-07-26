---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Discord Soundboard is a Spring Boot + React application that allows users to play sound files through a Discord bot in voice channels. The application features Discord OAuth2 authentication, role-based permissions, real-time updates via Server-Sent Events, and both web UI and Discord chat command interfaces.

**Tech Stack:**
- Backend: Java 17, Spring Boot 3.5.7, JDA 6.1.1 (Discord API), Lavaplayer (audio), H2 Database
- Frontend: React 18.3.1, TypeScript, Vite 6.3.5, Tailwind CSS, Radix UI
- Build: Gradle with node-gradle plugin for frontend integration

## Common Commands

### Backend Development

```bash
# Run application locally (requires bot token in application-local.properties)
./gradlew bootRun

# Run backend tests
./gradlew test

# Build entire application (includes frontend build)
./gradlew build

# Create distribution zip for release
./gradlew assembleBootDist

# Clean all build artifacts (includes frontend)
./gradlew clean
```

### Frontend Development

```bash
cd src/frontend

# Install dependencies
npm install

# Run frontend dev server (Vite)
npm run dev

# Build frontend for production
npm run build

# Run frontend tests
npm test

# Run tests with coverage
npm run test:coverage

# Run tests in watch mode
npm run test:watch
```

### Running Single Tests

**Backend (JUnit):**
```bash
./gradlew test --tests "net.dirtydeeds.discordsoundboard.controllers.SoundControllerTest"
./gradlew test --tests "*.SoundServiceTest.testFindAll"
```

**Frontend (Vitest):**
```bash
cd src/frontend
npm test -- SoundButton.test.tsx
npm test -- --grep "should render favorite icon"
```

## Architecture Overview

### Backend Architecture

**Entry Point:** `src/main/java/net/dirtydeeds/discordsoundboard/MainController.java`

The backend follows a layered Spring Boot architecture with Discord bot integration:

```
Controllers (REST API + SSE)
    ↓
Services (Business Logic)
    ↓
Repositories (JPA/H2 Database)
    ↓
Discord Bot Layer (JDA + Lavaplayer)
```

**Key Backend Components:**

1. **REST Controllers** - Located in `src/main/java/net/dirtydeeds/discordsoundboard/controllers/`
   - `AuthController` - JWT authentication and CSRF token endpoints
   - `SoundController` - Sound file CRUD, upload, download, favorites, SSE streaming
   - `BotCommandController` - Play sounds, control volume, stop playback
   - `BotVolumeController` - Volume management with SSE updates
   - `DiscordUserController` - User management, entrance/leave sounds, SSE streaming
   - `PlaybackController` - Real-time playback events via SSE

2. **Services** - Located in `src/main/java/net/dirtydeeds/discordsoundboard/service/`
   - `SoundService` / `SoundServiceImpl` - Sound file operations
   - `DiscordUserService` / `DiscordUserServiceImpl` - User operations
   - `PlaybackService` / `PlaybackServiceImpl` - SSE emitter management for playback events

3. **Discord Bot Integration** - Located in `src/main/java/net/dirtydeeds/discordsoundboard/`
   - `JDABot` - Discord connection initialization with gateway intents
   - `PlayerManager` - Lavaplayer audio player management per guild
   - `AudioHandler` - Low-level audio transmission to Discord
   - `SoundPlayer` - Orchestrates sound playback across services
   - Event Listeners: `UserEventListener`, `EntranceSoundBoardListener`, `LeaveSoundBoardListener`, `MovedChannelListener`, `CommandListener`
   - Chat Commands: Located in `src/main/java/net/dirtydeeds/discordsoundboard/chat/commands/` (PlayCommand, RandomCommand, VolumeCommand, etc.)

4. **Security & Authentication**
   - `SecurityConfig` - OAuth2 login, logout, CORS configuration
   - `JwtUtil` - JWT token generation/validation (HMAC-SHA256)
   - `OAuth2LoginSuccessHandler` - Intercepts Discord OAuth, generates JWT with roles/permissions
   - `UserRoleConfig` - Role-based permission mapping (Admin, DJ, Moderator, User)
   - `CrsfCheckFilter` - CSRF token validation for mutation requests

5. **Database** - H2 file-based at `./discordDB/discordDB`
   - `SoundFileRepository` - JPA repository for sound metadata
   - `DiscordUserRepository` - JPA repository for user data
   - Entities: `SoundFile` (soundFileId, category, timesPlayed, favorite, volumeOffsetPercentage), `DiscordUser` (id, username, entranceSound, leaveSound, inVoice, avatarUrl)

**Real-Time Communication:**
- Multiple SSE endpoints provide live updates to frontend (5-minute timeout, 25-second heartbeats)
- Events: sound file updates, volume changes, user status updates, playback tracking

### Frontend Architecture

**Entry Point:** `src/frontend/src/App.tsx`

The frontend uses React hooks for state management with no external state library:

```
App Component (Orchestration)
    ↓
Custom Hooks (Data + Actions)
    ↓
UI Components (Presentation)
    ↓
API Utils (HTTP + SSE)
```

**Key Frontend Components:**

1. **Custom Hooks** - Located in `src/frontend/src/hooks/`
   - `useAuth.ts` - Discord OAuth flow, JWT token management, user permissions
   - `useSounds.ts` - SSE connection to sound files stream, favorites management
   - `useSoundActions.ts` - Play, favorite, delete, upload, download operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Darkside138/DiscordSoundboard](https://github.com/Darkside138/DiscordSoundboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
