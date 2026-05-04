---
trigger: always_on
description: Questarr is a video game management application inspired by the \*Arr ecosystem (Sonarr, Radarr) and Steam's library view. Users discover, track, and download games via automated indexer search and download client integration. The application features a clean, dark-themed interface focused on visual game covers and efficient browsing.
---

# GitHub Copilot Instructions for Questarr

## Project Overview

Questarr is a video game management application inspired by the \*Arr ecosystem (Sonarr, Radarr) and Steam's library view. Users discover, track, and download games via automated indexer search and download client integration. The application features a clean, dark-themed interface focused on visual game covers and efficient browsing.

## Technology Stack

### Frontend

- **Framework**: React 18 with TypeScript
- **Build Tool**: Vite 5
- **Routing**: Wouter for lightweight client-side routing
- **State Management**: TanStack Query (React Query) for server state and caching
- **UI Components**: Radix UI primitives with shadcn/ui component library
- **Styling**: Tailwind CSS 4 with custom CSS variables for dark-first theming
- **Design System**: Grid-based layout with card-based game display

### Backend

- **Runtime**: Node.js 20+ with Express.js
- **Language**: TypeScript with ES modules
- **Database**: SQLite with better-sqlite3
- **ORM**: Drizzle ORM for type-safe database operations
- **Authentication**: JWT with bcryptjs password hashing
- **Validation**: express-validator for request validation, Zod for schema validation
- **Logging**: Pino
- **Real-time**: Socket.io for WebSocket notifications

### External APIs

- **IGDB API**: Game metadata, cover images, screenshots, ratings, and platform information (via Twitch OAuth)
- **Torznab/Newznab**: Indexer search protocol (torrent/NZB)
- **Download Clients**: qBittorrent, Transmission, rTorrent, sabnzbd, nzbget
- **Steam**: Wishlist import and Steam App ID resolution
- **xREL.to**: Scene release monitoring
- **HowLongToBeat**: Gameplay time data with fuzzy title matching (24h cache)
- **NexusMods**: Mod search and trending mods per game
- **PCGamingWiki**: Game wiki URL lookup via Steam App ID (CargoQuery API, 24h cache)

## Project Structure

```
/client              # Frontend React application
  /src               # React components and client code
    /components      # Reusable React components
    /pages           # Page components (lazy loaded): library, discover, search, wishlist, calendar, downloads, indexers, downloaders, rss, xrel-releases, stats, settings
    /hooks           # Custom React hooks
    /lib             # Utilities & services (auth, queryClient, etc.)
    /components/ui   # shadcn/ui component library
/server              # Backend Express application
  index.ts           # Server entry point, middleware chain
  routes.ts          # All API endpoints (~3360 lines, organized by domain)
  storage.ts         # Database access layer (Drizzle queries)
  middleware.ts      # Rate limiters, validators, sanitizers
  auth.ts            # JWT generation/verification, password hashing
  igdb.ts            # IGDB API client with caching
  downloaders.ts     # Multi-client download management
  search.ts          # Aggregated Torznab/Newznab search
  cron.ts            # Scheduled jobs (auto-search, download checks, etc.)
  ssrf.ts            # SSRF URL validation
  socket.ts          # WebSocket setup (Socket.io)
  torznab.ts         # Torznab protocol client
  newznab.ts         # Newznab protocol client
  hltb.ts            # HowLongToBeat client (gameplay time data)
  nexusmods.ts       # NexusMods API client (mod search)
  steam.ts           # Steam wishlist import and App ID resolution
  steam-routes.ts    # Express router for Steam endpoints
  pcgamingwiki-router.ts  # PCGamingWiki URL lookup via Steam App ID
  config.ts          # System-wide configuration access layer
/shared              # Shared code between client and server
  schema.ts          # Drizzle ORM schema + Zod validation schemas
  title-utils.ts     # Game title normalization & matching
  download-categorizer.ts  # Download categorization (main/update/DLC/extra)
/migrations          # Drizzle migration files
/tests               # Test setup and E2E tests
```

## TypeScript Configuration

- **Module System**: ESNext with ES modules (`"type": "module"`)
- **Strict Mode**: Enabled for type safety
- **Path Aliases**:
  - `@/*` → `./client/src/*`
  - `@shared/*` → `./shared/*`
- **Target**: Modern browsers with DOM and ESNext libraries

## Development Workflow

### Setup and Installation

```bash
npm install                 # Install dependencies
npm run db:migrate          # Run database migrations
npm run dev                 # Start development server with hot reload (port 5000)
```

### Build and Production

```bash
npm run build              # Build frontend (Vite) + backend (tsc)
npm start                  # Start production server from dist/
npm run check              # Type check with TypeScript
```

### Database Management

```bash
npm run db:generate        # Generate migration from schema changes
npm run db:migrate         # Run pending migrations
npm run db:push            # Push schema directly to dev DB (dev only)
```

### Testing

```bash
npm test                   # Vitest watch mode

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Doezer/Questarr](https://github.com/Doezer/Questarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
