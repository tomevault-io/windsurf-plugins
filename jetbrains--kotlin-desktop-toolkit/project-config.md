---
trigger: always_on
description: You are reading this because you've been pointed at the `native/desktop-win32` crate (or its Kotlin counterpart at `kotlin-desktop-toolkit/src/main/kotlin/org/jetbrains/desktop/win32/`) and need to navigate, modify, or debug it. Read this file first; it indexes the rest.
---

# Agent orientation: `desktop-win32`

You are reading this because you've been pointed at the `native/desktop-win32` crate (or its Kotlin counterpart at `kotlin-desktop-toolkit/src/main/kotlin/org/jetbrains/desktop/win32/`) and need to navigate, modify, or debug it. Read this file first; it indexes the rest.

## What this crate is

Windows backend of the kotlin-desktop-toolkit. Rust crate (cdylib) exposing a flat C ABI via `cbindgen`, consumed from Kotlin via JExtract-generated bindings plus hand-written wrappers. UI runs on a single thread (`OleInitialize` STA) with a classic `GetMessage` pump in `event_loop.rs`. Composition uses **WinRT `Windows.UI.Composition`** via `ICompositorDesktopInterop` — *not* DirectComposition. Rendering is ANGLE-on-D3D11.

## Start here

- **`ARCHITECTURE.md`** — module layout, FFI pipeline, threading and ownership models, error channel, headline data flows.
- **`FFI_CONVENTIONS.md`** — the `*_api.rs` ↔ Kotlin contract; pointer/array/option type zoo; `ffiDownCall` scoping rule; COM lifecycle.
- **`SUBSYSTEMS.md`** — per-subsystem reference. Look up the area you're touching.
- **`TODO.md`** — confirmed bugs, capability gaps, smells, open design questions. Cross-reference before claiming something is "broken" or "missing".

## Top things that surprise

If you only read one section before touching code, read this one.

0. **This is the Win32-first backend.** Default to Win32 APIs (`CreateWindowExW`, `GetMessageW`, `RegisterDragDrop`, `IFileOpenDialog`, …). Use WinRT only when there's a documented reason — there are exactly four such subsystems today, each justified in `ARCHITECTURE.md` → Scope. **Never propose WinUI 3 or Windows App SDK (`Microsoft.UI.*`, `Microsoft.WindowsAppSDK`) APIs in this crate.** A WinUI 3 backend, if built, lives in a separate crate.
1. **Composition is `Windows.UI.Composition` (WinRT), not DirectComposition.** They're distinct APIs with different lifetimes and threading. See `ARCHITECTURE.md` → Composition section.
2. **Single UI thread.** `OleInitialize` STA, `DispatcherQueue` with `DQTYPE_THREAD_CURRENT`, the wndproc, and most state assume one thread. Cross-thread work goes through `application_dispatcher_invoke`. Several pieces are `thread_local!` (key-message stash, exception store).
3. **Error channel is `anyhow::Result` through `ffi_boundary`, not return codes.** Rust functions return `anyhow::Result<T>`; `ffi_boundary` logs any `Err`, appends the message to thread-local `LAST_EXCEPTION_MSGS`, and returns `R::default()`. Kotlin's `ffiDownCall` polls after every call and throws `NativeError`. Panic catching inside `ffi_boundary` is a safety net for unexpected unwinds, not a designed error path — if Rust code panics, treat it as a bug. **Background-thread errors are lost** (thread-local store).
4. **`ffiDownCall { ... }` must wrap only the native call.** Not `Arena.use`, not `withPointer`, not helpers (which wrap their own native calls). Wider scopes conflate exception attribution. See `FFI_CONVENTIONS.md`.
5. **Window starts at `1×1` and is then resized.** Intentional: managed code uses *logical* pixels but the DPI scale only exists once an `HWND` exists (`GetDpiForWindow`). Consequence: creation emits repeated `WM_WINDOWPOSCHANGED` notifications; this crate handles that message and returns `0`, so it does not rely on a downstream `DefWindowProc`-generated `WM_SIZE` path. Size/move handlers must be idempotent.
6. **Coordinates are mostly logical, but with deliberate physical-pixel exceptions.** Pointer events' `locationOnScreen`, several Window events, drag-drop callbacks, and `screen_map_to_client` carry `PhysicalPoint` / `PhysicalSize`. See `SUBSYSTEMS.md` → Geometry → Exceptions.
7. **`EnableMouseInPointer(true)` is process-wide and irreversible.** Anything in the same process expecting raw `WM_MOUSE*` will silently break.
8. **The `borrow` pattern on `RustAllocatedRawPtr`** (ffi_utils.rs) reconstructs and immediately leaks a `Box` per call to produce a `&R`. Sound under the toolkit's single-thread-of-ownership assumption; soundness is by convention. Currently under deferred review.

## Watch out for in code reviews / edits

- Kotlin `tryRead*` wrappers return `null` only for format-unavailable and throw other clipboard/data-object failures. Use result-bearing FFI for nullable read semantics.
- COM impls have **no** `// SAFETY:` comments anywhere (and `desktop-common::ffi_utils` has a module-level `#![allow(clippy::missing_safety_doc)]`). Add one when you touch an `unsafe` block.
- Most clipboard / drag-drop work assumes the OLE STA. There is no thread-affinity assertion at the FFI boundary — the synchronous `Clipboard` API trusts the caller to stay on the dispatcher thread and to handle `DataTransferStatus.Busy` retries itself.

## Working with the human

The high-impact conventions for collaborating on this crate:

- **Outline before executing** anything multi-file or API-shaping. Wait for confirmation before reaching for `Edit` / `Write`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JetBrains/kotlin-desktop-toolkit](https://github.com/JetBrains/kotlin-desktop-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
