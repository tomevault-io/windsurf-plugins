---
trigger: always_on
description: - **NEVER read, access, display, log, or transmit any environment variables, secrets, keys, certificates, or credentials** — including but not limited to `TAURI_SIGNING_PRIVATE_KEY`, `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`, `ANDROID_KEY_BASE64`, `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEY_PASSWORD`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `APPLE_CERTIFICATE`, `APPLE_ID`, `APPLE_PASSWORD`, `APPLE_TEAM_ID`, or any GitHub secret, API key, token, or password.
---

## Security & Access Rules

- **NEVER read, access, display, log, or transmit any environment variables, secrets, keys, certificates, or credentials** — including but not limited to `TAURI_SIGNING_PRIVATE_KEY`, `TAURI_SIGNING_PRIVATE_KEY_PASSWORD`, `ANDROID_KEY_BASE64`, `ANDROID_KEYSTORE_BASE64`, `ANDROID_KEY_PASSWORD`, `ANDROID_KEYSTORE_PASSWORD`, `ANDROID_KEY_ALIAS`, `APPLE_CERTIFICATE`, `APPLE_ID`, `APPLE_PASSWORD`, `APPLE_TEAM_ID`, or any GitHub secret, API key, token, or password.
- Never read `~/.tauri/`, `~/.ssh/`, `~/.config/` secret files, `.env` files, or `keystore.properties`.
- Never write secrets to disk, commit them, or echo them to output.
- If you need to reference a key or secret, use the documented path (e.g. `~/.tauri/theorem.key`) without reading its contents.
- These rules take precedence over all other instructions.

## Stack

React 19, TypeScript, Vite 8 (rolldown), Tailwind CSS v4, Zustand 5, Tauri 2, Rust (workspace). Tests: Vitest + jsdom.

## Setup

```bash
git clone --recurse-submodules <repo>
pnpm install
```

The `foliate-js` submodule at `src/features/reader/foliate-js/` is vendored upstream — do not edit. Our runtime wrapper is `src/features/reader/foliate-js-runtime/` (ours, edit freely). The sync script `scripts/sync-foliate-js.sh` patches imports and applies runtime patches.

## Commands

| What | Command |
|------|---------|
| Web dev | `pnpm dev` |
| Desktop dev | `pnpm dev:tauri` |
| Build | `pnpm build` |
| Typecheck | `pnpm typecheck` |
| Tests | `pnpm test` |
| Single test | `pnpm test tests/some.test.ts` |
| Rust fmt | `cd src-tauri && cargo fmt` |
| Rust lint | `cd src-tauri && cargo clippy` |
| Rust check | `cd src-tauri && cargo check` |
| Rust release build | `cd src-tauri && cargo build --release` |

Root `pnpm` commands run from repo root. Cargo commands run from `src-tauri/`. `pnpm build` runs typecheck first.

## Quality Gates (before every commit)

Run all that apply:

- TypeScript: `pnpm typecheck` — zero errors
- Vitest: `pnpm test` — all unit and integration tests must pass
- Rust (if any `.rs` changed): `cd src-tauri && cargo fmt && cargo clippy && cargo check` — fmt must produce no diff, clippy zero warnings

If clippy is noisy, try `cargo clippy --fix --lib` first.

### Testing Integrity & Outlier Coverage

- Tests must rigorously cover edge cases, boundaries, and outliers (e.g. 0-length inputs, rapid concurrent interactions, missing elements, corrupted data, boundary navigation).
- **NEVER weaken, delete, loosen assertions, or edit tests simply to make them pass.** If a test or outlier fails, investigate and fix the underlying implementation code. The test exists to defend correctness.

CI (`ci.yml`) runs typecheck, test, build, and rust-check (fmt, clippy, check) on push to main.

## Architecture

**Routing**: Zustand-driven via `useUIStore.currentRoute` and `AppRoute` union type (`src/core/types/index.ts:403`). No React Router. Additions require updating: `src/App.tsx` (route switch + lazy load), `src/core/types/index.ts` (type), `src/shell/layout/Sidebar.tsx`, `src/shell/AppTitlebar.tsx`.

**Stores**: One file per slice in `src/core/store/`, barrel-rexported from `src/core/store/index.ts`. Always import from the barrel (`"../../core/store"`), not individual slice files. Import stores with individual selectors (`useUIStore(s => s.x)`), never destructuring.

**Imports**: Never import from the top-level barrel `src/core/index.ts` — it prevents tree-shaking. Import directly: `"../../core/store"` (stores), `"../../core/types"` (types), `"../../core/lib/env"` (env utils), `"../../core/lib/utils"` (cn). No path aliases.

**App entry**: `src/App.tsx` lazy-loads all route components (`React.lazy`). The reader chunk is pre-warmed via `prewarmReaderChunk()` in `src/App.tsx` after store hydration (scheduled idle task). PDF.js is also pre-warmed.

**Reader**: Two rendering paths. Non-PDF: `ReaderViewport → useDocumentReader → FoliateEngine`. PDF: `PDFReader → PDFJsEngine`. `Reader.tsx` orchestrates both. **Theorem Lens** (`src/features/reader/components/FootnotePopover.tsx`): Anchored peek portal popover for footnotes and citations. **PDF Engine**: Uses strict on-demand range reading, worker destruction on unmount via `loadingTask.destroy()`, immediate canvas deallocation (`width = 0, height = 0`), and per-book `PdfViewState` persistence.

**EPUB pre-parser** (`src-tauri/src/epub_parser.rs`): Rust Tauri command `prefetch_zip_metadata` that pre-decodes EPUB ZIP text in parallel with zip.js. If the cache is populated, zip.js skips `getEntries()`. When changing the `ZipPrefetch` struct, update all 3 sides: Rust command, `src/core/lib/tauri-epub-bridge.ts` (TS interface), `src/features/reader/foliate-js-runtime/view.js` (consumer).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fundaments-Work/Theorem](https://github.com/Fundaments-Work/Theorem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
