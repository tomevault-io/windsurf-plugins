---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## lib-jitsi-meet Architecture

This is the JavaScript library for accessing Jitsi Meet server-side deployments. It provides WebRTC functionality, XMPP communication, and media handling for Jitsi Meet clients.

## Common Development Commands

### Build Commands
```bash
npm run build           # Full build (webpack UMD bundle + TypeScript compilation)
npm run build:webpack   # Build UMD bundle only for browser <script> tags
npm run build:webpack-dev # Development webpack bundle
npm run build:tsc       # TypeScript compilation only for ESM modules
npm run watch           # Development build with file watching
```

### Development Commands

**Testing:**
- `npm test` - Run all tests via Karma (single run)
- `npm run test-watch` - Run tests in watch mode
- Tests use Jasmine framework with Chrome headless browser

**Code Quality:**
- `npm run lint` - ESLint + TypeScript type checking
- `npm run lint-fix` - Auto-fix linting issues
- `npm run type-check` - TypeScript type checking only

**Documentation:**
- `npm run typedoc` - Generate TypeScript documentation

**Other:**
 - `npm run gen-types` - Generate TypeScript declaration file

### TypeScript Migration

New features should be implemented only with TypeScript. When modifying existing JavaScript files, consider converting to TypeScript. The codebase is actively migrating from JavaScript to TypeScript.

**TypeScript Conventions:**
- 4-space indentation, LF line endings
- TypeScript enums for constant groups
- Interfaces for major components during transition
- Use strict type checking and avoid `any`, `unknown` and `object` type
- Strong type checking enabled in tsconfig.json

### Key Dependencies

**Core Libraries:**
- `strophe.js` - XMPP client library (custom Jitsi fork)
- `webrtc-adapter` - WebRTC compatibility shim
- `sdp-transform` - SDP parsing and manipulation
- `@jitsi/logger` - Logging framework
- `@jitsi/js-utils` - Jitsi JavaScript utilities

**Development:**
- Webpack for bundling (UMD and ES modules)
- Karma + Jasmine for testing
- ESLint with @jitsi/eslint-config
- TypeScript compiler

### Configuration Files
- `webpack.config.js` + `webpack-shared-config.js` - Webpack configuration for UMD builds
- `tsconfig.json` - TypeScript configuration with ES2020 target
- `karma.conf.js` - Test runner configuration
- `.eslintrc.js` - ESLint with TypeScript support
- `tools/gen-version.js` - Version generation script

### Testing Guidelines

- Tests located alongside source files with `.spec.ts` extension
- Karma runs tests in Chrome headless browser
- Tests include both TypeScript and JavaScript files during migration
- Use Jasmine framework for assertions and test structure

## Code Architecture

### Core API Structure
- **JitsiMeetJS.ts** - Main library entry point exposing the public API
- **JitsiConnection.ts** - XMPP connection management and authentication
- **JitsiConference.js** - Video conference session representation
- **JitsiParticipant.ts** - Conference participant abstraction
- **JitsiTrack/JitsiLocalTrack/JitsiRemoteTrack** - Media track management

### Module Organization

**RTC Module** (`/modules/RTC/`):
- Core WebRTC functionality, track management, and screen sharing
- **TraceablePeerConnection.js** - Enhanced PeerConnection with debugging
- **RTCUtils.js** - Browser compatibility and WebRTC utilities

**XMPP Module** (`/modules/xmpp/`):
- XMPP/Jingle protocol implementation for signaling
- **ChatRoom.js** - Multi-user chat room with presence management
- **JingleSessionPC.js** - Jingle protocol for media negotiation
- **SignalingLayerImpl.js** - Abstraction layer for signaling
- **Strophe plugins** - Protocol extensions (disco, ping, stream-management, etc.)

**E2EE Module** (`/modules/e2ee/`):
- End-to-end encryption using insertable streams and SFrame
- **Worker.js** - Web worker for E2EE processing (separate webpack entry)
- **OlmAdapter.js** - Integration with Olm for key management

**Quality Control** (`/modules/qualitycontrol/`):
- Video quality adaptation and codec selection
- **ReceiveVideoController/SendVideoController** - Stream management

**Service Layer** (`/service/`):
- Type-safe constants, events, and enums
- Well-defined event system used throughout the library

### Build System Architecture
- **Dual output**: UMD bundle (`dist/umd/`) and ESM modules (`dist/esm/`)
- **Webpack configuration**: Shared config with separate UMD and E2EE worker builds
- **TypeScript migration**: Gradual migration with both `.js` and `.ts` files coexisting
- **Testing**: Karma + Jasmine with webpack preprocessing for both JS and TS files

### Key Design Patterns
- **Event-driven architecture**: Extensive use of EventEmitter throughout all modules
- **Protocol abstraction**: Clean separation between XMPP signaling and WebRTC media
- **Modular design**: Self-contained modules with clear dependencies
- **Browser compatibility**: webrtc-adapter integration and capability detection

### Development Notes
- Tests are located alongside source files with `.spec.js/.spec.ts` extensions
- TypeScript types are maintained in `/types` directory for gradual migration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jitsi/lib-jitsi-meet](https://github.com/jitsi/lib-jitsi-meet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
