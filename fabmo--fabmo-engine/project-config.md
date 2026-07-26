---
trigger: always_on
description: FabMo Engine is a host software for G2 motion control platforms. It manages CNC motion systems, streams G-Code, interprets OpenSBP (ShopBot language), and hosts a web-based dashboard for controlling digital fabrication tools.
---

# FabMo Engine - AI Coding Agent Instructions

## Project Overview
FabMo Engine is a host software for G2 motion control platforms. It manages CNC motion systems, streams G-Code, interprets OpenSBP (ShopBot language), and hosts a web-based dashboard for controlling digital fabrication tools.

**Key Technologies**: Node.js v16.14.0, Restify (REST API), Socket.IO (WebSockets), TingoDB (database), SerialPort (G2 driver), Webpack (dashboard build)

## Architecture: The Big Picture

### 1. Application Lifecycle
- **Entry point**: `server.js` → `engine.js` → `engine.start()`
- **Initialization sequence** in `engine.start()`:
  1. Configure engine (`EngineConfig`)
  2. Connect to G2 motion controller via serial (`machine.connect()`)
  3. Initialize config subsystems (driver, opensbp, dashboard, machine, user)
  4. Setup HTTP/WebSocket servers (Restify + Socket.IO on port 80/9876)
  5. Load routes from `/routes` directory
  6. Load dashboard apps and profiles
  7. Start network manager (platform-specific)

### 2. Core Architectural Components

**Machine Model** (`machine.js`): Central singleton that abstracts the physical CNC machine. Manages state, coordinates runtimes, and sits one layer above the G2 driver. Access via `require('./machine').machine`.

**G2 Driver** (`g2.js`): Low-level serial communication with G2 motion controller. Handles command queuing, status reports, and error codes. JSON-based protocol over SerialPort.

**Runtimes**: Pluggable execution contexts that control the machine for specific tasks:
- `GCodeRuntime` - G-Code execution
- `SBPRuntime` - OpenSBP language interpreter (primary language)
- `ManualRuntime` - Manual control (keyboard/pendant)
- `PassthroughRuntime` - Direct G2 access
- `IdleRuntime` - Default when inactive

Only ONE runtime active at a time. Switch via `machine.setRuntime()`.

**Configuration System** (`/config`): Multi-branch tree structure:
- `engine` - Engine settings (ports, platform)
- `driver` - G2 motion controller config
- `opensbp` - OpenSBP runtime settings
- `machine` - Machine-specific parameters
- `dashboard` - UI configuration
- `user` - Authentication data

All configs sync between disk (`/opt/fabmo` or `C:\opt\fabmo`) and memory.

**Profiles** (`profiles.js`): Packages of settings/macros/apps for different machine types. Located in `/profiles`. Applying a profile obliterates existing configs and requires engine restart.

**Database** (`db.js`): TingoDB (MongoDB-like) stores:
- File metadata (actual files on disk)
- Job queue/history
- Thumbnails

Collections: `files`, `jobs`, `thumbnails`. Max storage: 500MB (auto-pruning).

**Dashboard** (`/dashboard`): Web frontend built with Webpack. Apps are modular installable components stored in `/opt/fabmo/approot`. System apps in `/dashboard/apps` are core (editor, job manager, network manager, etc.).

**Routes** (`/routes`): REST API endpoints. Each `.js` file exports a function accepting the Restify server object. Auto-loaded by `routes/index.js`. See `/routes/config.js`, `/routes/jobs.js`, etc.

### 3. Data Flow Patterns

**Status Updates**: Machine state flows via EventEmitter pattern:
```
G2 Driver → Machine Model → WebSocket → Dashboard
```
Machine emits `status` events (`machine.emit('status', this.status)`). Listen in routes/websockets via `machine.on('status', ...)`.

**Job Execution**:
1. File uploaded → Database (`db.js`)
2. Job created → Queue
3. Machine picks job → Selects runtime
4. Runtime interprets code → Streams to G2
5. Status updates flow back through chain

**Configuration Changes**: 
- HTTP POST → Route handler → Config object `.update()` → Sync to disk
- Profile changes trigger `process.exit(1)` expecting systemd restart

## Critical Development Workflows

### Build & Run
```bash
npm install              # Install deps + webpack build
npm run debug           # Debug mode (enhanced logging, no webpack)
npm run dev             # Webpack + debug server
npm start               # Production mode
npm run webpack         # Build dashboard only
```

**Important**: Dashboard requires webpack build before running. Debug mode reloads apps more aggressively.

### Platform-Specific Considerations
- **Serial ports**: Configured in `engine_config.js` based on `process.platform`:
  - Linux: `/dev/fabmo_g2_motion` (via udev rules)
  - macOS: `/dev/cu.usbmodem*`
  - Windows: COM ports in `C:\fabmo\config\engine.json`
- **Data directory**: `/opt/fabmo` (Linux/Mac) or `C:\opt\fabmo` (Windows)
  - Must be writable by engine user (no sudo required)

### Testing
Jest tests in `/test`. Run with `npm test`. Limited coverage currently (`util.test.js`, `machine.test.js`).

### Code Quality
- **ESLint**: Uses `eslint-config-prettier`. Config in `.eslintrc.js`. Many files have selective `eslint-disable` comments.
- **Prettier**: `printWidth: 120`, `endOfLine: lf`. Config in `.prettierrc`.
- **Husky + lint-staged**: Pre-commit hooks (configured in `package.json`).

## Project-Specific Conventions

### Error Handling Pattern

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FabMo/FabMo-Engine](https://github.com/FabMo/FabMo-Engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
