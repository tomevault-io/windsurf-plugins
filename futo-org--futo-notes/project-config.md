---
trigger: always_on
description: @README.md for project overview. @justfile for all commands.
---

# AGENTS.md — FUTO Notes Operating Manual

@README.md for project overview. @justfile for all commands.

FUTO Notes is an offline-first markdown app with a Svelte 5 editor, shared Rust core, Tauri
desktop, native SwiftUI/Compose mobile shells, and optional E2EE sync.

This root file contains only cross-layer decisions and recurring traps. **CRITICAL** rules protect
user data or shipped behavior; never weaken one to make a test, build, or pipeline pass.
Engineering defaults: the simplest implementation that fully meets the current requirement, and an
established, well-maintained library over a custom one.

**Read the nearest nested `AGENTS.md` before editing a layer.** Every crate has one
(`crates/futo-notes-{core,model,store,search,sync,ffi}/`), as does `src/`, `packages/editor/`,
each app in `apps/`, `scripts/`, `tests/`, and `docs/spec/`.

For structural work, read `docs/architecture/codebase-organization.md`: use the narrowest real
owner, make shared code earn its scope, keep entry points as orchestration, co-locate tests, and
complete moves across code, tests, config, and docs. Its `spec/` means this repo's `docs/spec/`.
CRITICAL rules and behavioral specs take priority; report any conflict.

## 1. Quick start

Use `just` from the repo root (`just install`, `just tauri-dev`, `just check`). The justfile owns
commands, overlays, dev IDs, worktree isolation, and device detection. Never call `cargo tauri`.
`just tauri-dev` is desktop dev: Wayland, port 5180.

## 2. CRITICAL — mobile is native, not Tauri

There is no Tauri mobile shell; the old `cargo tauri ios/android` recipes were removed. Use the
native apps in `apps/ios` and `apps/android` through `just ios-native` / `just android-native`.
Their nested manuals own build, device, release, and test variants. Missing
`vite-plugin-singlefile` means stale node_modules; error 7 launching iOS usually means a locked phone.

## 3. Monorepo map

- `src/`: shared UI, reactive state, and coordination.
- `packages/editor/`: hot-path TS rules, bridge contract, toolbar manifest.
- `crates/`: `-model` (pure note rules, no fs) · `-core` (hashing, E2EE crypto, 3-way merge, path
  safety + atomic files) · `-store` (THE local note engine) · `-sync` (push-first `run_sync`, SSE) ·
  `-license` (the paid-client-license rule: v2 FUTOpay activation, offline verification) ·
  `-search` (Tantivy BM25) · `-ffi` (UniFFI projection; bindings gitignored).
- `apps/`: Tauri desktop plus native iOS and Android shells.
- `docs/spec/`: behavioral truth; `tests/` (unit, Playwright, and the editor gauntlet): fixture/oracle systems.

Generated and gitignored: native bindings/JNI libraries and `editor.html`. The external sync server
(its own Go repo) receives only client-encrypted opaque blobs; sync tests download the release
pinned in `scripts/sync-server-pin.json`, so no checkout of it is needed here.

## 4. Where logic lives (decision procedure)

1. **Note rule or note-tree mutation?** Rust model/core/store, projected through Tauri or UniFFI;
   never reimplement it in TS, Swift, or Kotlin.
2. **Needed per keystroke?** The only exception is a conformance-locked hot-path TS mirror, reached
   from the app through `src/lib/rules.ts`; Rust remains canonical and `packages/editor/AGENTS.md`
   owns the procedure (§7.3).
3. **View, reactive state, coordination, or shell?** TypeScript/Svelte in `src/`.
4. **Compute-heavy or protocol-shaped?** Rust.
5. **OS capability?** Extend `PlatformFS`; components never branch on platform, never invoke note
   commands or plugin-fs directly (`pnpm run lint:platform`, `pnpm run check:platform-discipline`).
6. **Two domain calls in sequence?** Make one atomic Rust workflow, not a shell-side stitch.
   This has no reliable gate: an experiment produced a green check-then-act race that resurrected
   a deleted note. If every caller must remember an ordering invariant, push it down.

Before copying auth, validation, parsing, or cleanup at call sites, find or create its narrow
infrastructure owner.

## 5. Cross-cutting conventions

- **Svelte 5 runes only** (`$state`/`$derived`/`$effect`; module state in `.svelte.ts`). Never
  `svelte/store`, `on:click`, or `createEventDispatcher` — use `onclick=` attributes and callback props.
- Never hand-build note paths: use TS `pathSafety.ts` or Rust `safe_note_path`.
- **Every new user-visible string is a catalog entry.** Authored UI text — labels, headings,
  buttons, placeholders, toasts, errors, accessibility and OS-facing labels — is added to
  `languages/en.json` and read back through the platform's accessor (`localizedText` in TS/Swift/
  Kotlin), never written as a literal at the call site. An English entry is the minimum bar; a
  translation is welcome but never required to land. User data is the opposite rule: note titles,
  filenames, folder names, tags, paths and URLs render verbatim (M2). `languages/README.md` owns
  the naming and placeholder grammar; `pnpm run check:languages` validates the catalogs.
- The note cache (`notesCache` in `src/features/notes/notes.svelte.ts`) is a projection. Apply the
  complete post-commit `LocalNoteMutation`; do not optimistically reconstruct collision or backlink
  outcomes.
- FFI builds (iOS and Android) use the `release-ffi` profile; plain release uses `panic = "abort"`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [futo-org/futo-notes](https://github.com/futo-org/futo-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
