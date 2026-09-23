---
trigger: always_on
description: Viritura is a web-native collaborative music notation editor. It uses:
---

# Viritura - AGENTS.md

## Project Description

Viritura is a web-native collaborative music notation editor. It uses:

- **Rust → WASM** for the layout/engraving engine
- **TypeScript + React** for the editor UI
- **Canvas 2D** for score rendering with SMuFL fonts (Bravura)
- **MNX** (W3C JSON standard) as the native score format

## Project Structure

```
engine/viritura-engine/     # Rust core (model, layout, render)
  src/layout/               # Layout engine (~42 modules)
  src/layout/tests/          # 55 test files by feature area
  src/parse/                 # MNX parser (Rust)
  src/model/                 # Data model structs
  src/render/                # Render commands + SMuFL
engine/viritura-wasm/       # WASM bindings (wasm-bindgen)
apps/desktop/               # Tauri v2 shell (VST3 host, Lua mapper, native playback)
apps/editor/            # React app (Vite)
  src/stories/               # Storybook stories (MNX kitchen sink)
  .storybook/                # Storybook config
packages/renderer/          # Canvas renderer (TS)
packages/core/              # Data model types, enums, and constants (TS)
packages/format/            # MNX parser/serializer (TS, split into 5 modules)
   fixtures/mnx/              # 71 upstream examples + 15 local fixtures
  schemas/                   # viritura-extensions.json schema
packages/musicxml/          # MusicXML → MNX converter (TS, isomorphic)
packages/crdt/              # Yjs bridge, live session, awareness
packages/playback/          # Playback context + transport
packages/video-sync/        # Score-to-picture sync (video, native PiP)
packages/audio/             # Samplers, reverb, mixing
packages/midi/              # MIDI timeline, dynamics, tempo
packages/piano-roll/        # Piano-roll view
packages/instrument-profiles/ # Instrument profile model + registry
packages/sound-profiles/    # Sound profile model
packages/ui/                # UI primitives (own Storybook)
apps/website/               # Marketing site + /mnx tooling routes
apps/server-ui/             # Server-rendered admin/consent UI
server/Viritura.Api/        # ASP.NET Core: auth, MCP relay, signalling, snapshots
server/Viritura.GitHub/     # GitHub App / OAuth / git proxy
server/Viritura.Infrastructure/
docs/                       # Architecture documents
```

## Key References

- Documentation index: [`docs/README.md`](docs/README.md)
- Architecture overview: `docs/overview/project-overview.md` → `docs/overview/architecture.md`
- Music Notation Reference coverage: `docs/spec/music-notationref-coverage.md`
- Viritura vendor extensions: `docs/spec/viritura-extensions.md` (schema: `packages/format/schemas/viritura-extensions.json`)
- MNX spec examples: `packages/format/fixtures/mnx/*.mnx` (71 upstream official examples + 15 locally-authored fixtures; all pass MNX schema validation)
- SMuFL glyphs: `engine/viritura-engine/src/render/smufl.rs`
- SMuFL specification: `../smufl/`
- MNX specification: `../mnx-spec/`

## Validation

- **Build:** `pnpm build`
- **TypeScript build alias:** `pnpm build:ts`
- **Complete core polyglot build:** `pnpm build:all`
- **Test:** `pnpm test`
- **Lint:** `pnpm lint`
- **Rust build:** `pnpm build:rust`
- **Rust test:** `pnpm test:rust` (process-locked to prevent concurrent full suites)
- **WASM build:** `pnpm build:wasm` (`pnpm wasm:build` is preserved)
- **.NET + server UI build:** `pnpm build:dotnet`
- **Desktop build:** `pnpm build:desktop`
- **API image:** `pnpm build:api-image`
- **VSIX package:** `pnpm build:vsix`
- **Storybook (UI primitives + design language):** `pnpm dev:storybook:ui` (port 6005)
- **Storybook (MNX spec + Viritura extensions):** `pnpm dev:storybook:mnx` (port 6006)
- **Storybook (composed app surfaces):** `pnpm dev:storybook` (port 6007)
- **MNX schema validation:** `pnpm --filter @viritura/format test`

## Worktree development servers

When an agent needs to start, run, preview, or visually inspect any web or API
service, it must use `pnpm dev:stack` rather than launching Vite,
Storybook, or `dotnet watch` directly. The wrapper gives every worktree isolated
containers, data, internal service DNS, and `*.<slug>.localhost` routes without
host-port collisions.

- Default app work: `pnpm dev:stack up` (editor + API + server UI watcher).
- Frontend-only work: `pnpm dev:stack up ui`.
- API/server work: `pnpm dev:stack up backend`.
- Storybook work: `pnpm dev:stack up storybook-ui`,
  `storybook-mnx`, or `storybook-app`; use `storybook` only when all three are needed.
- Cross-stack or uncertain scope: `pnpm dev:stack up full`.
- Report routes with `pnpm dev:stack url`; inspect with `status` and
  `logs [service]`.
- Dependency-manifest changes select a new content-addressed image
  automatically. Use `rebuild [target]` only to discard stale compiler output,
  `stop` for a short pause, `down` to remove containers, and `prune` to delete
  worktree-local compiler output. Shared dependency and API data volumes are
  preserved.
- Rust-only builds and tests do not need a server. UI-capable profiles
  automatically run the cache-aware Docker WASM builder before startup; use
  `pnpm dev:stack wasm` to invoke it explicitly.
- The wrapper requires Docker to be running. Start Docker Desktop, Docker Engine,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Viritura/Viritura](https://github.com/Viritura/Viritura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
