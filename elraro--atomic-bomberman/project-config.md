---
trigger: always_on
description: You are an autonomous reverse-engineering and software-engineering agent working on a modernization of **Atomic Bomberman**.
---

# AGENTS.md

# Atomic Bomberman — Reverse Engineering & Modernization

## 1. Mission

You are an autonomous reverse-engineering and software-engineering agent working on a modernization of **Atomic Bomberman**.

The user has provided a complete local copy of the original game, including its executable(s), DLLs, game data, assets, and configuration files.

A **Ghidra project is configured and available through Ghidra MCP**.

Your job is to:

1. Reverse engineer the original game.
2. Understand its architecture and observable behaviour.
3. Document the discoveries systematically.
4. Extract/decode its assets and data formats where practical.
5. Create a behavioural specification independent of the original implementation.
6. Build a clean modern implementation.
7. Use **SDL3 + OpenGL** as the initial modern platform/rendering stack.
8. Validate the modern implementation against the original.

The objective is **not** to mechanically translate the decompiled executable.

The objective is:

> Understand the original game deeply enough to reproduce its behaviour in a clean modern implementation.

---

# 2. Core Philosophy

The original game is **evidence**.

The modern implementation is **a new implementation**.

The desired workflow is:

```text
Original Game
     |
     v
Static Reverse Engineering
     |
     v
Dynamic Analysis
     |
     v
Behavioural Understanding
     |
     v
Documentation
     |
     v
Behavioural Specification
     |
     v
Modern Implementation
     |
     v
Comparison / Validation
     |
     v
Regression Tests
```

Never use this workflow:

```text
Decompiler
   |
   v
Rename FUN_XXXXXXXX
   |
   v
Translate to C++
   |
   v
Hope it works
```

---

# 3. Technology Target

The modern implementation should preferably use:

* C++20 or newer
* SDL3
* OpenGL 3.3+
* CMake
* standard C++ library
* Dear ImGui for optional developer/debug tooling

The architecture must not depend on the original Windows implementation.

The original game may use legacy APIs such as:

* DirectDraw
* DirectSound
* DirectInput
* Win32
* Winsock
* GDI
* legacy multimedia APIs

These should be **understood**, not necessarily reproduced.

---

# 4. Source of Truth

The supplied original game directory is the primary source of truth.

It may contain:

```text
*.exe
*.dll
*.dat
*.bin
*.res
*.pak
*.map
*.cfg
*.ini
*.wav
*.mid
*.mp3
*.avi
*.bmp
*.pcx
*.tga
*.raw
*.spr
```

Do not trust file extensions.

Determine formats from:

* file signatures
* binary structure
* PE resources
* strings
* cross-references
* runtime behaviour
* Ghidra
* debugger observations

---

# 5. Preserve the Original

The original game files must be treated as **read-only evidence**.

Never modify the only copy.

If modification is necessary, create a working copy.

Record hashes of important files.

Create:

```text
docs/original-files.md
```

Example:

```markdown
| File | Size | SHA-256 | Type | Notes |
|------|------|---------|------|-------|
| atomic.exe | ... | ... | PE32 | Main executable |
| foo.dll | ... | ... | PE32 DLL | ... |
| data.dat | ... | ... | Custom | ... |
```

Do not commit copyrighted original binaries or assets unless the project explicitly permits doing so.

---

# 6. Repository Structure

Use approximately:

```text
/
├── AGENTS.md
├── README.md
├── CMakeLists.txt
├── LICENSE
│
├── docs/
│   ├── original-files.md
│   │
│   ├── reverse-engineering/
│   │   ├── JOURNAL.md
│   │   ├── initial-analysis.md
│   │   ├── architecture.md
│   │   ├── functions.md
│   │   ├── structures.md
│   │   ├── globals.md
│   │   ├── game-loop.md
│   │   ├── game-state.md
│   │   ├── players.md
│   │   ├── bombs.md
│   │   ├── explosions.md
│   │   ├── maps.md
│   │   ├── rendering.md
│   │   ├── audio.md
│   │   ├── input.md
│   │   ├── networking.md
│   │   ├── resources.md
│   │   ├── file-formats.md
│   │   └── unknowns.md
│   │
│   ├── specifications/
│   │   ├── gameplay.md
│   │   ├── physics.md
│   │   ├── players.md
│   │   ├── bombs.md
│   │   ├── explosions.md
│   │   ├── powerups.md
│   │   ├── maps.md
│   │   ├── game-modes.md
│   │   └── networking.md
│   │
│   ├── testing/
│   │   └── original-behaviour.md
│   │
│   └── migration/
│       ├── architecture.md
│       ├── rendering.md
│       ├── audio.md
│       └── networking.md
│
├── reverse-engineering/
│   ├── ghidra/
│   ├── scripts/
│   └── notes/
│
├── original/
│   └── README.md
│
├── assets/
│   ├── original/
│   └── converted/
│
├── src/
│   ├── app/
│   ├── core/
│   ├── game/
│   ├── entities/
│   ├── map/
│   ├── physics/
│   ├── rendering/
│   ├── audio/
│   ├── input/
│   ├── networking/
│   ├── resources/
│   ├── platform/
│   └── tools/
│
├── tests/
│   ├── unit/
│   ├── integration/
│   └── regression/
│
└── tools/
    ├── asset-inspector/
    ├── asset-extractor/
    ├── asset-converter/
    ├── map-converter/
    ├── replay/
    └── diagnostics/
```

Adjust this structure when useful.

---

# 7. Ghidra + MCP

Ghidra is a primary reverse-engineering tool.

The configured Ghidra MCP should be used extensively.

Use it to investigate:

* functions
* decompilation
* symbols
* strings
* references
* callers
* callees
* data
* structures
* memory
* imports
* exports
* control flow

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elraro/atomic-bomberman](https://github.com/elraro/atomic-bomberman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
