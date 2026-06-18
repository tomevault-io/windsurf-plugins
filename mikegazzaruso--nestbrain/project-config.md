---
trigger: always_on
description: LLM-powered personal knowledge base packaged as a native desktop workspace. Raw sources go in, a structured Markdown wiki comes out — compiled, linked, and maintained entirely by AI. Ships as an Electron app with an embedded Next.js UI, a real PTY terminal, and deep Claude Code integration.
---

# NestBrain

LLM-powered personal knowledge base packaged as a native desktop workspace. Raw sources go in, a structured Markdown wiki comes out — compiled, linked, and maintained entirely by AI. Ships as an Electron app with an embedded Next.js UI, a real PTY terminal, and deep Claude Code integration.

Published binaries are sold at [nestbrain.app](https://nestbrain.app) ($29 one-time, signed + notarized). The source in this repo is **GPL-3.0** and fully buildable.

## Project Overview

NestBrain ingests raw documents (URLs, PDFs, GitHub repos, arXiv papers, YouTube transcripts, RSS feeds) and uses an LLM to compile them into an interconnected wiki of Markdown files. The wiki is Obsidian-compatible and queryable through hybrid search + LLM-grounded Q&A. The desktop app wraps all of this in a workspace UI: VS Code-style file tree, CodeMirror editor, integrated xterm.js terminal, session-aware Claude Code skills.

## Repo Layout (pnpm monorepo)

```
nestbrain/
├── apps/
│   ├── desktop/                # Electron 33 shell (main + preload + builder config)
│   │   ├── src/main.ts         # Electron main: PATH fix, IPC, embedded server
│   │   ├── src/dev-module.ts   # Open-core seam: guarded require of src/dev-impl/ (gitignored,
│   │   │                       #   lives in the PRIVATE nestbrain-modules repo; CI overlays it,
│   │   │                       #   local dev uses scripts/sync-modules.sh)
│   │   ├── src/preload.ts      # Renderer-safe IPC bridge
│   │   └── build/              # electron-builder hooks, icons, NSIS installer
│   └── web/                    # Next.js 16 + React 19 UI (runs as standalone inside Electron)
│       └── src/{app,components,lib,types}
├── packages/
│   ├── cli/                    # `nestbrain` CLI (commander)
│   ├── core/                   # Domain logic
│   │   └── src/{compiler,ingest,llm,qa,search,lint,vectorstore}
│   ├── db/                     # Chroma client + embeddings wrapper
│   ├── shared/                 # Types and constants (auth/sync types live here so main + renderer share them)
│   └── sync/                   # Drive-backed multi-device sync engine
│       └── src/{engine,drive-adapter,watcher,manifest,excludes,walker,hash}
├── skeleton/                   # Workspace template copied to NestBrain/ on first run
│   ├── CLAUDE.md
│   └── Skills/{start_session,end_session}/SKILL.md
├── docker/                     # Dockerfile + compose (includes a chromadb service)
├── data/                       # Local-dev workspace (raw/, wiki/, chromadb/)
├── docs/screenshots/
├── nestbrain.yaml              # Default workspace config (provider, embeddings, search, server)
├── pnpm-workspace.yaml
├── turbo.json
└── package.json                # version: 1.14.3
```

## Tech Stack

- **Language**: TypeScript 5.9 (Node 20+, ESM where possible, CJS where Electron forces it)
- **Package manager**: pnpm 10 + Turborepo
- **Desktop shell**: Electron 33, `node-pty` (lazy-loaded), `electron-builder` (mac DMG signed/notarized, Windows NSIS, Linux placeholder)
- **UI**: Next.js 16.2 + React 19, CodeMirror 6, xterm.js + addon-fit, mermaid, react-force-graph-2d, lucide-react, Tailwind v4
- **CLI**: `commander`
- **LLM providers** (`packages/core/src/llm/`): `claude-cli` (default — spawns the user's `claude` CLI) and `openai`
- **Embeddings**: `@huggingface/transformers` running ONNX locally (`Xenova/all-MiniLM-L6-v2`)
- **Vector store**: `packages/core/src/vectorstore` is the primary path used by the compiler; `packages/db` wraps `chromadb` for the dockerized setup
- **Ingest**: `@mozilla/readability` + `linkedom` + `turndown` for URLs, `pdf-parse` for PDFs, `rss-parser`, `youtube-transcript`, custom GitHub + arXiv adapters
- **Auth**: Google OAuth 2.0 Desktop flow (PKCE, loopback redirect). Refresh token in OS keychain via Electron `safeStorage`. Code in `apps/desktop/src/auth/`.
- **Sync**: `@nestbrain/sync` package — chokidar watcher + Drive REST adapter + manifest + engine. Wired into the main process by `apps/desktop/src/sync/`. Scope: `drive.file` (no security audit, app only sees what it created).
- **Lint/QA**: LLM-driven against the compiled wiki
- **Testing**: Vitest (configured at root, very thin coverage today)
- **Lint/format**: ESLint 9 + Prettier 3

## Key Commands

### Repo-level (pnpm + turbo)

```bash
pnpm install                       # bootstrap workspace
pnpm dev                           # turbo dev across all packages
pnpm build                         # turbo build
pnpm lint                          # turbo lint
pnpm test                          # turbo test (vitest)
pnpm format                        # prettier write
```

### Desktop app

```bash
pnpm desktop:dev                   # build TS + launch Electron with NESTBRAIN_DEV=1
pnpm desktop:build                 # build web standalone + copy assets + build desktop TS
pnpm desktop:package:mac           # DMG into apps/desktop/release/
pnpm desktop:package:win           # NSIS .exe into apps/desktop/release/
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mikegazzaruso/nestbrain](https://github.com/mikegazzaruso/nestbrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
