---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

---

## Development Commands

This is a **Vite+ monorepo**. All tooling goes through the `vp` global CLI — never use `pnpm`, `npm`, or tool binaries (`vitest`, `eslint`, `oxlint`) directly. See `apps/docs/viteplus.md` for the full reference.

```bash
# Install dependencies (after pulling)
vp install

# Run the Tauri desktop app (Vite dev server + Tauri window)
vp run tauri

# Run the website dev server
vp dev

# Format + lint + typecheck (run before committing)
vp check

# Run tests (watch mode)
vp test

# Build all packages
vp run -r build

# Check everything is ready (fmt, lint, test, build)
vp run ready
```

**Critical rules for Vite+:**

- `vp check` replaces `vue-tsc`, `eslint`, and `oxfmt` — use it, not them
- `vp test` replaces `vitest` — never install or run vitest directly
- `vp lint` and `vp fmt` work standalone if you need just one step
- To run a custom `package.json` script that shares a name with a built-in (e.g. a custom `dev`), use `vp run dev` not `vp dev`
- All imports use `vite-plus`: `import { defineConfig } from 'vite-plus'` and `import { expect, test } from 'vite-plus/test'`

---

## Monorepo Structure

```
capubridge/
├── apps/
│   ├── desktop/        # Tauri 2 desktop app (Vue 3 + TypeScript)
│   │   ├── src/        # Frontend Vue source
│   │   └── src-tauri/  # Rust backend (Tauri commands)
│   ├── docs/           # VitePress documentation app
│   └── website2/       # Marketing/docs website (Vite)
├── packages/
│   └── cdp-protocol/   # Typed CDP client and domain adapters
└── vite.config.ts      # Root Vite+ config (fmt, lint, staged hooks)
```

The `desktop` app imports the CDP package as `import { ... } from "@capubridge/cdp-protocol"` via workspace.

---

## Project Overview

Capubridge is a Tauri 2 desktop app built with Vue 3 + TypeScript.
It is a developer tool for hybrid app (Capacitor/Ionic) developers that combines:

- ADB device management GUI
- Deep browser storage inspector (IndexedDB, LocalStorage, Cache API, OPFS)
- Remote Chrome DevTools Protocol (CDP) connection to physical Android devices
- Capacitor-specific tooling

Full specification: see `apps/docs/SPEC.md`

---

## Tech Stack Quick Reference

| Layer              | Technology                                  |
| ------------------ | ------------------------------------------- |
| Frontend framework | Vue 3 (Composition API, `<script setup>`)   |
| Language           | TypeScript (strict)                         |
| Build              | Vite 5                                      |
| Styling            | UnoCSS (atomic CSS)                         |
| State              | Pinia (setup store syntax) + TanStack Query |
| Tables             | TanStack Table v8                           |
| Terminal           | xterm.js v5                                 |
| Code editor        | Monaco Editor                               |
| Desktop            | Tauri 2                                     |
| Backend            | Rust (thin — shell plugin + file I/O only)  |

---

## Mandatory Coding Conventions

### CRITICAL: No new `adb.exe` process spawns

**Never** spawn a new `adb.exe` process via `std::process::Command::new("adb")`. All ADB commands must go through the shared `ADB_SERVER` static `LazyLock<Mutex<ADBServer>>` which maintains a single TCP connection to the ADB daemon on port 5037.

Spawning new `adb.exe` processes causes:

- RunDLL error dialog popups on Windows (third-party DLL injection into spawned processes)
- Performance overhead (process startup, connection negotiation)
- Potential race conditions with multiple ADB server instances

**Correct pattern:**

```rust
let mut server = get_server().lock();
let mut device = server
    .get_device_by_name(&serial)
    .map_err(|e| format!("Device not found: {e}"))?;
device.shell_command(&cmd, Some(&mut stdout), None)?;
device.pull(&remote_path, &mut out)?;
```

**Wrong pattern — NEVER do this:**

```rust
let adb = which::which("adb")?;
let output = std::process::Command::new(adb)
    .args(["-s", &serial, "shell", "cmd"])
    .output()?;
```

The only exception is the global `suppress_error_dialogs()` call in `lib.rs` setup, and `CREATE_NO_WINDOW` flags for essential process spawns (like Chrome launch).

### Comments policy

- **TypeScript**: Only JSDoc comments. No inline `//` comments.
- **Rust**: Only doc comments (`///`) for public functions. No inline `//` comments.

### Vue components

```vue
<!-- ALWAYS use <script setup lang="ts"> -->
<script setup lang="ts">
import { ref, computed, onMounted } from "vue";
import type { IDBRecord } from "@/types/storage.types";

// Props with defineProps
const props = defineProps<{
  dbName: string;
  records: IDBRecord[];
  isLoading?: boolean;
}>();

// Emits with defineEmits
const emit = defineEmits<{
  refresh: [];
  recordEdit: [record: IDBRecord];
  recordDelete: [key: IDBValidKey];
}>();
</script>

<template>
  <!-- template here -->
</template>
```

### Composables

```typescript
// Always named export, always start with 'use'
// File: src/composables/useIDB.ts
export function useIDB(targetId: Ref<string>) {
  // implementation
  return {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aybinv7/capubridge](https://github.com/aybinv7/capubridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
