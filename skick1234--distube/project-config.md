---
trigger: always_on
description: This document provides guidance for working with the DisTube codebase.
---

# CLAUDE.md - DisTube Development Guide

This document provides guidance for working with the DisTube codebase.

## Project Overview

DisTube is a powerful Discord.js music bot library written in TypeScript that simplifies music commands and enables effortless playback from various sources with integrated audio filters.

- **Node.js**: >= 22.12.0
- **Discord.js**: v14
- **Package Manager**: Bun
- **Build Tool**: tsup (dual ESM/CJS output)
- **Test Framework**: Vitest
- **Linter/Formatter**: Biome

## Quick Commands

```bash
bun install           # Install dependencies
bun run build         # Build the project (tsup)
bun run test          # Run tests (vitest)
bun run lint          # Check linting (biome ci)
bun run lint:fix      # Fix linting issues (biome check --write --unsafe)
bun run prettier      # Format code (biome format --write)
bun run type          # Type check (tsc --noEmit)
bun run docs          # Generate documentation (typedoc)
```

## Project Structure

```
src/
├── DisTube.ts              # Main class - extends TypedEmitter, orchestrates everything
├── index.ts                # Public API exports
├── type.ts                 # TypeScript type definitions, enums, interfaces
├── constant.ts             # Constants, default filters, audio config
├── util.ts                 # Utility functions (validation, type guards)
├── core/
│   ├── DisTubeBase.ts      # Abstract base class providing access to managers
│   ├── DisTubeHandler.ts   # URL/string resolution to Song/Playlist
│   ├── DisTubeOptions.ts   # Options validation and FFmpeg config
│   ├── DisTubeStream.ts    # FFmpeg stream handling
│   ├── DisTubeVoice.ts     # Voice connection management
│   └── manager/
│       ├── BaseManager.ts          # Generic Collection-based manager
│       ├── GuildIdManager.ts       # Base for guild-keyed managers
│       ├── QueueManager.ts         # Queue lifecycle management
│       ├── DisTubeVoiceManager.ts  # Voice connection management
│       └── FilterManager.ts        # Audio filter application
└── struct/
    ├── Song.ts             # Song class with metadata
    ├── Queue.ts            # Queue management (filters, repeat, autoplay)
    ├── Playlist.ts         # Playlist container
    ├── DisTubeError.ts     # Custom error class with 60+ predefined codes
    ├── Plugin.ts           # Abstract plugin base
    ├── ExtractorPlugin.ts  # Full-featured plugin (validate, resolve, search, stream)
    ├── InfoExtractorPlugin.ts    # Info-only plugin
    ├── PlayableExtractorPlugin.ts # Playable plugin (no search)
    └── TaskQueue.ts        # Sequential task queuing
```

## Architecture Patterns

### Event-Driven Architecture
- Uses `tiny-typed-emitter` for type-safe event emission
- 11 event types: `addList`, `addSong`, `playSong`, `finishSong`, `error`, `debug`, `ffmpegDebug`, `empty`, `finish`, `initQueue`, `noRelated`, `disconnect`, `deleteQueue`
- Queue emits events through parent DisTube instance

### Manager Pattern
- `BaseManager<V>` provides Collection-based storage
- `GuildIdManager` extends with guild-keyed storage
- Each manager handles lifecycle and state for its domain

### Plugin Architecture
Three plugin types with different capabilities:
- `ExtractorPlugin`: Full-featured (validate, resolve, search, getStreamURL)
- `InfoExtractorPlugin`: Info only (validate, resolve, createSearchQuery)
- `PlayableExtractorPlugin`: Playable only (validate, resolve, getStreamURL)

## Constants

Audio and timing constants are defined in `src/constant.ts`:

```typescript
AUDIO_SAMPLE_RATE = 48000      // Hz
AUDIO_CHANNELS = 2             // Stereo
DEFAULT_VOLUME = 50            // 0-100%
JOIN_TIMEOUT_MS = 30_000       // 30 seconds
RECONNECT_TIMEOUT_MS = 5_000   // 5 seconds
RECONNECT_MAX_ATTEMPTS = 5     // Max rejoin attempts
HTTP_REDIRECT_CODES            // Set of redirect status codes
MAX_REDIRECT_DEPTH = 5         // Max redirect follows
```

Always use these constants instead of magic numbers.

## Coding Conventions

### TypeScript
- Strict mode enabled with all strict checks
- Use `type` imports: `import type { ... } from "..."`
- Path aliases: `@/*` maps to `src/*`, `@` maps to `src/index.ts`
- No unused variables or parameters (enforced by tsconfig)
- Always use explicit return types for public methods
- Use `catch (e: unknown)` instead of `catch (e: any)`

### Error Handling
```typescript
// Good - type-safe error handling
catch (e: unknown) {
  const error = e instanceof Error ? e : new Error(String(e));
  const message = e instanceof Error ? (e.stack ?? e.message) : String(e);
}

// Bad - loses type safety
catch (e: any) {
  console.log(e.message);  // Unsafe
}
```

### Formatting (Biome)
- 2 spaces indentation
- 120 character line width
- Double quotes for strings
- Trailing commas everywhere
- Arrow functions: parentheses only when needed (`x => x`, `(a, b) => a + b`)

### Naming Conventions
- Classes: PascalCase (`DisTubeVoice`, `QueueManager`)
- Methods/Properties: camelCase (`getQueue`, `playSong`)
- Private fields: prefix with `#` or `_` for internal (`#getQueue`, `_beginTime`)
- Constants: UPPER_SNAKE_CASE (`AUDIO_SAMPLE_RATE`, `DEFAULT_VOLUME`)
- Enums: PascalCase with UPPER_SNAKE_CASE members

## Error Handling

### DisTubeError Class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skick1234/DisTube](https://github.com/skick1234/DisTube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
