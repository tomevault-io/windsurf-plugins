---
trigger: always_on
description: Whenever a new memory is created or an existing memory is updated in the auto-memory directory (`~/.claude/projects/.../memory/`), it MUST also be written or updated in the corresponding `.claude/memory/` file in this repository. This includes both `MEMORY.md` and all topic sub-files.
---

# RDForward Project Instructions

## Memory Sync Rule

Whenever a new memory is created or an existing memory is updated in the auto-memory directory (`~/.claude/projects/.../memory/`), it MUST also be written or updated in the corresponding `.claude/memory/` file in this repository. This includes both `MEMORY.md` and all topic sub-files.

This ensures memories are portable with the repository. When a developer pulls new changes, they can sync the `.claude/memory/` files into their local auto-memory directory to keep all machines up to date.

The full project memory lives in `.claude/memory/MEMORY.md` and its referenced sub-files — not duplicated here in CLAUDE.md to avoid doubling the context window (since auto-memory is loaded automatically by Claude Code).

## Data File Protection

- NEVER delete or reset world save files (server-world.dat, server-players.dat, or similar) without explicit user permission. These contain accumulated test data built up over many sessions.
- Before ANY destructive file operation (rm, overwrite, reset) on non-code data files in the project root, ALWAYS ask the user first.

## Backwards Compatibility (HIGHEST PRIORITY)

Not breaking existing clients is the single most important constraint in this project. Every code change — whether adding a new protocol version, refactoring shared code, or fixing a bug — MUST preserve full compatibility with ALL previously supported clients. This takes precedence over clean code, performance, and new features.

When adding support for new protocol versions (MCPE, Netty, Alpha, or otherwise):
- ALWAYS test ALL previously supported versions after making changes. New version support must not break older versions.
- Version-specific code paths (packet formats, field sizes, flag meanings) can differ between ANY two versions — never assume adjacent versions share the same format without verifying.
- When adding version thresholds (e.g. `if (version >= V27)`), verify that ALL versions in the affected range behave the same way. A threshold that's correct for v27 may be wrong for v17.
- When touching shared code paths (packet dispatch, chunk sending, login sequence), re-test at least one client from each supported version range.

When refactoring or modifying shared methods:
- If a method is used by multiple protocol versions, NEVER change its behavior globally. Add a version parameter and branch, or create a version-specific override.
- Before editing any shared code, trace ALL callers to verify which clients are affected.
- Guard new behavior behind version checks so only the targeted client type is affected.

## Lazy Loading and Code Decoupling

Performance is a top priority. Protocol adapters, registries, and version-specific code must be loaded and executed only when actually needed:

- **Lazy-load infrastructure**: Bedrock block palettes, MCPE servers, packet registry overlays, block state mappers, and translation tables should NOT be loaded at startup. Defer initialization to the first client connection of that protocol type. Use JVM inner-class holders (thread-safe without synchronization) or synchronized lazy getters (double-checked locking with volatile) as appropriate.
- **Decouple version branches**: When multiple protocol versions share a handler method via cascading if-else chains on protocol version numbers, extract version-specific wire format logic into separate codec/strategy classes selected at connection time. Each session should only execute its own version's code path, never code for other versions.
- **No unnecessary processing**: If only Alpha clients are connected, zero MCPE or Bedrock code should execute. If only 1.8 clients are connected, no 1.21-specific packet registry overlays should be loaded. Broadcast methods that write to sessions of mixed versions must dispatch on the RECIPIENT's codec, not the sender's.
- **Existing pattern to follow**: `BedrockProtocolConstants` uses synchronized lazy getters as the reference pattern. `ProtocolDetectionHandler` dynamically reconfigures the Netty pipeline per-connection as the reference pattern for per-client protocol selection. `BlockReplacementRegistry` follows the same shape for per-version YAML deltas: each `/replacements/<protocolKey>.yml` is loaded the first time a coercion needs it, cached, and a missing resource is recorded as `NO_FILE` so the disk lookup never repeats.

## Bridge Stub Coverage

When a Bukkit/Paper plugin invokes a method on a bridge stub that has no real implementation, the stub MUST call `StubCallLog.logOnce(pluginId, signature)`. On the first hit per `(plugin, signature)` pair, the shared `StubCallLog` emits a WARNING-level JUL line for server operators (server console only). Subsequent hits for the same `(plugin, signature)` are silent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinambrus/RDForward](https://github.com/martinambrus/RDForward) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
