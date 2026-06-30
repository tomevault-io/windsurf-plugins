---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Reldens is an MMORPG Platform (v4.0.0-beta.39) built on Node.js, designed for developers to create multiplayer games. The platform integrates:
- **Server**: Colyseus 0.16 for multiplayer game server
- **Client**: Phaser 3 for game engine, Parcel for bundling
- **Database**: Supports multiple storage drivers (objection-js, mikro-orm, prisma)
- **Architecture**: Client-server with authoritative server, real-time synchronization via WebSockets

**Node Version**: >= 20.0.0

### Sub-Packages
- **@reldens/utils** - Core utilities, Shortcuts class (imported as `sc`), EventsManagerSingleton, Logger
- **@reldens/server-utils** - Server utilities, FileHandler, configuration helpers
- **@reldens/storage** - Multi-ORM database layer (ObjectionJS, MikroORM, Prisma)
- **@reldens/cms** - Content management system and admin panel
- **@reldens/items-system** - Items and inventory system
- **@reldens/modifiers** - Stats and modifiers system
- **@reldens/skills** - Skills and abilities system

## Essential Commands

```bash
# Testing
npm test
node tests/manager.js --filter="test-name" --break-on-error

# Building
reldens buildSkeleton                   # Build both styles and client
reldens fullRebuild                     # Complete rebuild from scratch

# Database
reldens generateEntities [--override]   # Generate entities from database schema

# User management
reldens createAdmin --user=username --pass=password --email=email@example.com
reldens resetPassword --user=username --pass=newpassword
```

**Full command reference**: See `.claude/commands-reference.md`

## Architecture Overview

### Client-Server Organization
The codebase follows a **client/server split architecture** within each feature module:

```
lib/
  ├── {feature}/
  │   ├── client/           # Client-side code (Phaser, UI, rendering)
  │   ├── server/           # Server-side code (Colyseus rooms, logic)
  │   ├── constants.js      # Shared constants
  │   └── schemas/          # Colyseus state schemas (if applicable)
```

### Core Entry Points
- **Server**: `server.js` → `lib/game/server/manager.js` (ServerManager)
- **Client**: `client.js` → `lib/game/client/game-manager.js` (GameManager)
- **Theme**: `theme/default/index.js` initializes client with custom plugins

### Feature Modules (23 Total)
The platform includes 23 feature modules: Game, Rooms, World, Config, Features, Actions, Inventory, Respawn, Rewards, Scores, Teams, Users, Chat, Audio, Prediction, Admin, Firebase, Ads, Import, Objects, Snippets, Bundlers.

**Detailed list**: See `.claude/feature-modules.md`

## Configuration System

Reldens uses a **database-driven configuration** with runtime overrides:

1. **Database Config** (`config` table): Path-based keys, scoped by `scope` field
2. **Environment Variables** (`.env`): Prefix `RELDENS_*` for all settings
3. **Custom Classes**: Passed via `customClasses` to override defaults

**Key Environment Variables:**
- `RELDENS_STORAGE_DRIVER` - Storage driver (objection-js, mikro-orm, prisma)
- `RELDENS_DB_HOST`, `RELDENS_DB_PORT`, `RELDENS_DB_NAME`, `RELDENS_DB_USER`, `RELDENS_DB_PASSWORD`
- `RELDENS_HOT_PLUG` - Enable hot-plug configuration updates (0/1)

**Full environment variables list**: See `.claude/environment-variables.md`

## Events System

The platform uses **@reldens/utils EventsManagerSingleton** for extensibility:

**Common Event Patterns**:
- `reldens.{action}Before` - Hook before operation
- `reldens.{action}After` - Hook after operation
- Events are synchronous (`emitSync`) or async (`emit`)

**Key Events**:
- `reldens.serverConfigFeaturesReady` - Features loaded
- `reldens.beforeJoinGame` - Before player joins
- `reldens.startGameAfter` - Game initialized
- `reldens.roomLoginOnAuth` - Custom authentication logic

**Plugin Pattern**:
```javascript
class ServerPlugin {
    setup({events}) {
        events.on('reldens.serverConfigFeaturesReady', (props) => {
            // Custom logic here
        });
    }
}
```

## Storage & Entity Management

### CRITICAL: Understanding getEntity()

`dataServer.getEntity()` returns a `BaseDriver` instance from `@reldens/storage`, NOT an Entity or Model class.

```javascript
// Correct - returns BaseDriver
let statsRepository = this.dataServer.getEntity('stats');

// BaseDriver provides unified interface:
await statsRepository.create({key: 'hp', label: 'Health Points'});
await statsRepository.loadAll();
await statsRepository.loadOneBy('key', 'hp');
await statsRepository.updateById(1, {label: 'HP'});
```

**Type Annotation:**
```javascript
/** @type {import('@reldens/storage').BaseDriver} */
this.statsRepository = this.dataServer.getEntity('stats');
```

**Storage Drivers:**
- `prisma` (current default): Modern ORM with type safety, custom validation
- `objection-js`: Uses Knex.js, direct SQL, no validation
- `mikro-orm`: ORM with decorators, supports MongoDB

**Detailed architecture**: See `.claude/storage-architecture.md`
**Entity list**: See `.claude/entities-reference.md`

## Theme & Customization

**Theme Structure** (`theme/`):
- `plugins/` - Custom client/server plugins for game-specific logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [damian-pastorini/reldens](https://github.com/damian-pastorini/reldens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
