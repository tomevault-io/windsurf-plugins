---
trigger: always_on
description: If `CLAUDE.md` exists in this repository, it is a symlink to this file; edit only `AGENTS.md`, never `CLAUDE.md` directly.
---

# AGENTS.md

If `CLAUDE.md` exists in this repository, it is a symlink to this file; edit only `AGENTS.md`, never `CLAUDE.md` directly.

## Commands

```bash
# Install dependencies before building or packaging
npm ci

# Rust build auto-cleanup: Tauri build scripts run `npm run clean:rust`
# first, which deletes Rust build artifacts older than 14 days from
# src-tauri/target via scripts/clean-rust.js. cargo-sweep is optional;
# without it the cleanup is skipped and builds still work.

# Development (frontend only, no Tauri shell)
npm run dev

# Development (full desktop app)
npm run tauri dev

# Build
npm run build              # frontend only: vue-tsc --noEmit && vite build
npm run tauri build        # production Tauri bundle

# Frontend unit tests
npm test

# Frontend unit tests in watch mode
npm run test:watch

# E2E UI tests (WebDriver driving the real Tauri app; Linux only, needs
# tauri-driver + WebKitWebDriver; build the debug binary first)
npm run test:e2e

# Type-check the e2e harness and specs (not covered by `npm run build`)
npm run typecheck:e2e

# Platform packaging
npm run build:desktop          # native bundle for the current platform only
npm run build:linux:appimage   # Linux AppImage for host arch; Linux only
npm run build:macos:dmg        # Apple Silicon macOS DMG; macOS only

# Version bump
npm run bump-version -- <version>   # updates npm, Tauri, and Cargo version files

# Preview built frontend
npm run preview

# Type-check frontend
npx vue-tsc --noEmit

# Rust backend checks/tests
cd src-tauri && cargo check
cd src-tauri && cargo test
```

Tagged releases are driven by `.github/workflows/release-desktop.yml`: pushing a `v*` tag first runs the E2E UI tests workflow as a reusable job, and only if it passes builds Linux x86_64 and Linux ARM64 AppImages on Ubuntu, an unsigned Apple Silicon DMG on macOS, creates the GitHub Release if needed, and uploads all artifacts. Linux release asset names include the architecture, e.g. `Dez_<version>_linux_x86_64.AppImage` and `Dez_<version>_linux_arm64.AppImage`.

Frontend unit tests use Vitest with colocated `*.test.ts` files. Observed automated tests include Rust unit tests in `src-tauri/src/commands.rs`.

## Git conventions

Commit messages must use an 80-character maximum line width for both subject and body.

## Architecture

Dez follows an SBP MVC pattern: selectors are the cross-layer API, and code should live in the owning MVC layer. Tauri v2 provides the Rust backend; Vue 3 / TypeScript / Vite provide the frontend. State management uses Pinia setup stores as internal model implementation details.

Top-level source sketch:

- `src/model/`: domain state, persistence formats, providers, chat/stream model logic, and model selectors.
- `src/controller/`: user workflows, startup/shutdown, native/Tauri boundaries, HTTP runtime orchestration, updates, and diagnostics.
- `src/view/`: Vue UI, view composables, CodeMirror editor services, toast/theme/clipboard UI selectors, and reactive read adapters.
- `src/utils/`: cross-layer utilities only.
- `src-tauri/`: Rust commands, native HTTP bridge, persistence file I/O, credential storage, and app bootstrap.

SBP is covered in more detail in a section below.

### Runtime data flow

```text
User edits ThreadEditor (CodeMirror document)
  -> ThreadEditor syncs CodeMirror doc to tabStore.activeTab.sections
  -> Mod-Enter calls dez.controller/submitTab
    -> dez.stream/start creates a stream session + native HTTP request id
      -> dez.provider/streamChat builds provider-specific requests in TypeScript
      -> dez.http/stream owns a Tauri Channel and byte stream
        -> dez.native/streamHttp invokes Rust http_bridge::stream_http
          -> Rust spawns a Tokio task, streams reqwest chunks, and emits Headers/Chunk/Done/Error
      -> TypeScript decodes SSE frames and provider token deltas
    -> dez.stream/receiveToken appends tokens to the captured agent section
    -> dez.editor/appendTokenIfVisible patches the active CodeMirror view only when visible
    -> finish/error/cancel reconciles the editor, appends a trailing user section, and persists
```

### Frontend (`src/`)

- **Single editor surface**: `ThreadEditor.vue` owns one CodeMirror `EditorView`; the old contenteditable/chat-message architecture is no longer current. Avoid DOM-first fixes. The CodeMirror document is flattened text with sentinel markers, and store state is reconstructed through helpers in `src/view/thread/cmEditor.ts`.
- **Conversation model**: A tab owns `Section[]`, where each section is `{ id, role: 'user'|'agent', content: ContentNode[] }`. `ContentNode` is `TextNode` or `PromptNode`. Shared helpers are in `src/model/chat/content.ts`.
- **CodeMirror sentinels**: `cmEditor.ts` uses ASCII Record Separator `SECTION_SEP = '\u001E'` as an in-buffer section boundary. Prompt pills are encoded with private-use sentinels `PILL_OPEN`, `PILL_BODY`, `PILL_CLOSE`, and `PILL_NL`; these are live-editor implementation details and must not be written to disk or exposed in clipboard text.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taoeffect/dez](https://github.com/taoeffect/dez) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
