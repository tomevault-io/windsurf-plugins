---
trigger: always_on
description: This file provides guidance to AI coding assistants working with code in this repository.
---

# CLAUDE.md / AGENTS.md

This file provides guidance to AI coding assistants working with code in this repository.

> **`CLAUDE.md` and `AGENTS.md` are byte-identical copies, and `CLAUDE.md` is the
> source of truth.** Two files exist because different tools look for different
> names (Claude Code reads `CLAUDE.md`, Codex reads `AGENTS.md`); they are full
> copies rather than one pointing at the other because not every tool resolves a
> `Read @file` reference, and a tool that doesn't would end up with no
> instructions at all. **Edit `CLAUDE.md`, then copy it over `AGENTS.md`** —
> `tests/unit/agentInstructions.test.ts` fails if they drift apart.

> **Dilo** is a Spanish-first fork of [Handy](https://github.com/cjpais/Handy) (`upstream` remote). Product decisions live in `docs/superpowers/specs/`. Keep the Rust core close to upstream so `git merge upstream/main` stays cheap; brand/UI/default changes go in focused commits. All user-facing copy is Spanish-first (es locale is authored, not machine-translated — keep its voice: tuteo, direct, zero corporate filler).

## Dirección de producto — plataforma conversacional abierta

**Dilo es la pieza central open source**, no el cliente cautivo de un backend,
agente o producto particular. Su propósito es ser una interfaz conversacional
universal: recibe voz o texto, transcribe, conversa y presenta la respuesta por
voz, texto o ambos. Cada persona decide a qué modelo, asistente, automatización
o sistema conectarlo.

- Dilo sigue siendo plenamente útil sin conexiones externas: dictado,
  procesamiento local y voz local.
- Las capacidades externas entran por contratos genéricos y reemplazables:
  primero un destino asistente configurable; después conectores/adaptadores
  con permisos explícitos y, cuando corresponda, protocolos como MCP.
- No introducir en el núcleo nombres, reglas de negocio ni dependencias de un
  backend particular. Una instalación privada puede ser muy potente sin
  convertir esa implementación en la arquitectura del producto público.
- Dilo posee la experiencia conversacional —captura, STT, TTS, sesión,
  progreso y presentación—; el backend conectado interpreta, planifica y
  ejecuta. Dilo no se convierte en ERP, orquestador de agentes ni autoridad de
  negocio.
- La voz puede proponer una acción, pero nunca autentica ni autoriza una
  operación sensible. Los conectores declaran permisos y las credenciales no
  se guardan como texto plano en la configuración.
- El nombre/persona y el futuro wake word son configurables por cada usuario;
  ninguna personalidad concreta define el producto.

La dirección aprobada y sus límites están en
`docs/superpowers/specs/2026-07-22-dilo-plataforma-conversacional-abierta.md`.
Es una definición de producto, **no autorización para implementar conectores
sin su diseño técnico y plan correspondientes**.

La misma especificación incluye una nota del 2026-07-24 para explorar **Dilo
Online con Nova 2 Sonic**: wake word local y conversación cloud opcional con
Command Center y agentes. Sonic debe ser un proveedor reemplazable detrás de un
contrato conversacional propio; no reemplaza el dictado offline ni entra por el
conector de texto Bedrock Mantle. La nota no autoriza implementación.

## Development Commands

**Prerequisites:**

- [Rust](https://rustup.rs/) (latest stable)
- [Bun](https://bun.sh/) package manager

**Core Development:**

```bash
# Install dependencies
bun install

# Run in development mode
bun run tauri dev
# If cmake error on macOS:
CMAKE_POLICY_VERSION_MINIMUM=3.5 bun run tauri dev

# Build for production
bun run tauri build

# Frontend only development
bun run dev        # Start Vite dev server
bun run build      # Build frontend (TypeScript + Vite)
bun run preview    # Preview built frontend
```

**Linting and Formatting (run before committing):**

```bash
bun run lint              # ESLint for frontend
bun run lint:fix          # ESLint with auto-fix
bun run format            # Prettier + cargo fmt
bun run format:check      # Check formatting without changes
bun run format:frontend   # Prettier only
bun run format:backend    # cargo fmt only
```

**Model Setup (Required for Development):**

```bash
mkdir -p src-tauri/resources/models
curl -o src-tauri/resources/models/silero_vad_v4.onnx https://blob.handy.computer/silero_vad_v4.onnx
```

For detailed platform-specific build setup, see [BUILD.md](BUILD.md).

## Architecture Overview

Dilo is a cross-platform open conversational interface built with Tauri 2.x
(Rust backend + React/TypeScript frontend). Speech-to-text remains its base,
but the product direction also includes interchangeable assistant destinations
and spoken/text responses without coupling the core to any particular backend.

### Backend Structure (src-tauri/src/)

- `lib.rs` - Main entry point, Tauri setup, manager initialization
- `managers/` - Core business logic:
  - `audio.rs` - Audio recording and device management
  - `model.rs` - Model downloading and management
  - `transcription.rs` - Speech-to-text processing pipeline
  - `history.rs` - Transcription history storage
- `audio_toolkit/` - Low-level audio processing:
  - `audio/` - Device enumeration, recording, resampling
  - `vad/` - Voice Activity Detection (Silero VAD)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aacontn/dilo](https://github.com/aacontn/dilo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
