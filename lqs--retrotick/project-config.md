---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RetroTick is a browser-based x86 emulator that runs classic Windows PE/NE/MZ executables. Built with Preact + Vite + TypeScript + Tailwind CSS.

## Commands

- `npm run build` — Production build to `dist/`
- `npm run check` — Alias for `npm run build` (catches type errors and regressions)
- `timeout 2 npx tsx tests/test-<name>.mjs` — Run a headless test for a specific exe

No test runner or linter scripts are configured.

## Architecture

### Core Layers

**PE/NE Parsing** (`src/lib/pe/`): Binary parsing of Windows executables. `parse.ts` handles PE header parsing and resource directory tree walking. Each `extract-*.ts` file handles one resource type (bitmap, icon, cursor, dialog, menu, string, version, accelerator, AVI, WAV, manifest, Delphi DFM, imports, exports). `decode-dib.ts` handles DIB-to-image conversion. RVA-to-file-offset mapping converts virtual addresses to actual file positions.

**x86 CPU** (`src/lib/emu/x86/`): Modular instruction execution:
- `cpu.ts` — CPU class with register accessors, flag get/set, push/pop, fetch, alu(), testCC()
- `flags.ts` — `materializeFlags()` for lazy flag evaluation
- `lazy-op.ts` — Lazy operation type definitions
- `decode.ts` — ModRM/SIB decoding, segment override, writeModRM
- `dispatch.ts` — Main instruction dispatch switch
- `ops-0f.ts` — Two-byte 0F opcode handler (Jcc/SETcc/CMOVcc/MOVZX/MOVSX/IMUL)
- `ops-0f-ext.ts` — Extended 0F opcodes (BT/BSF/BSR/SHLD/SHRD/XADD/CMPXCHG/BSWAP/RDTSC)
- `fpu.ts` — FPU helpers (fpuPush/fpuPop/fpuST/fpuSetST) + dispatcher
- `fpu-d8db.ts` — FPU opcodes D8-DB
- `fpu-dcdf.ts` — FPU opcodes DC-DF
- `shift.ts` — Shift/rotate operations (SHL/SHR/SAR/ROL/ROR/RCL/RCR/SHLD/SHRD)
- `string.ts` — String instructions (REP MOVS/STOS/CMPS/SCAS/LODS)

**Emulator Core** (`src/lib/emu/`):
- `memory.ts` — Linear 32-bit address space with segment translation and ANSI code page support
- `emulator.ts` — Emulator class with fields, heap/virtual alloc, arg reading, timer mgmt, delegates; `ApiDef`, `Win32Dll`, `Win16Module` for unified API registration
- `thread.ts` — Thread state management
- `pe-loader.ts` / `ne-loader.ts` / `mz-loader.ts` — Load PE32, NE (16-bit), and MZ (DOS) executables into memory
- `emu-exec.ts` — tick(), callWndProc(), callWndProc16(), completeThunk(), resume()
- `emu-load.ts` — load() and findResourceEntry()
- `emu-thunks-pe.ts` — Win32 thunk table builder, IAT verification, TEB init
- `emu-thunks-ne.ts` — Win16 thunk table builder, resource string reader
- `emu-render.ts` — fillTextBitmap, renderChildControls, sendDrawItem
- `emu-render-controls.ts` — renderButton, renderStatic, renderEdit
- `emu-window.ts` — DC management, canvas setup, SEH dispatch, resource loading
- `dos-int.ts` — DOS INT 21h interrupt services for MZ executables
- `file-manager.ts` — Virtual filesystem backed by IndexedDB
- `cp437.ts` — CP437 (DOS) character encoding table
- `format.ts` — Formatting utilities

**Windows API Stubs** (`src/lib/emu/win32/`): Modular — subdirectories per major DLL, files per functional area:
- `kernel32/` — atom, console, env, file, heap, locale, module, process, profile, resource, string, sync, sysinfo, tls
- `user32/` — clipboard, create-window, dialog, focus, input, menu, message, misc, paint, rect, register-class, resource, scroll, text, timer, window-long, wndproc
- `gdi32/` — bitmap, brush, dc, draw, metafile, palette, region, select, text
- `advapi32.ts` — registry, security
- `comctl32.ts` — common controls
- `comdlg32.ts` — common dialogs
- `shell32.ts`, `winspool.ts`, `winmm.ts`, `ws2_32.ts` — misc DLLs
- `msvcrt.ts` — C runtime (cdecl)
- `opengl32.ts`, `gl-context.ts`, `glu32.ts` — OpenGL 1.x → WebGL2
- `ddraw.ts` — DirectDraw stubs
- `ole32.ts`, `oleaut32.ts` — COM/OLE stubs
- `psapi.ts`, `shlwapi.ts`, `iphlpapi.ts`, `msacm32.ts`, `version.ts` — misc
- `handles.ts` — Unified handle table
- `helpers.ts` — Shared helper functions
- `types.ts` — Win32 constants and type definitions

**Win16 API Stubs** (`src/lib/emu/win16/`): Windows 3.x NE executable support:
- `kernel/` — atom, dos, error, file, memory, misc, module, profile, registry, resource, string, task
- `user/` — dialog, menu, message, misc, paint, rect, resource, window
- `gdi.ts` — graphics device interface
- `shell.ts`, `commdlg.ts`, `commctrl.ts`, `mmsystem.ts`, `keyboard.ts`, `ddeml.ts`, `win87em.ts`
- `lzexpand.ts`, `sound.ts`, `ver.ts`, `sconfig.ts` (Workgroup Security)

**UI Components** (`src/components/`): Display components per resource type. `win2k/` contains 24 widgets mimicking Windows 2000 UI (Window, Button, Edit, ListBox, ListView, TreeView, ComboBox, TabControl, MenuBar, Taskbar, MessageBox, etc.). `EmulatorView.tsx` renders emulated programs via Canvas + HTML controls. `App.tsx` manages the desktop, taskbar, and multi-window state.

### Key Patterns

- JSX uses Preact (React compat via tsconfig path aliases)
- `useBlobUrls` hook manages object URL lifecycle with automatic cleanup
- DIB-to-PNG conversion in `decode-dib.ts` and `image.ts` for rendering Windows bitmaps

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lqs/retrotick](https://github.com/lqs/retrotick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
