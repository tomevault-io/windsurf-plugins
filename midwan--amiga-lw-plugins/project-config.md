---
trigger: always_on
description: - **Docker image**: `sacredbanana/amiga-compiler:m68k-amigaos`
---

# Agent Knowledge Base — Amiga LightWave 5.x Plugin

## Toolchain

- **Docker image**: `sacredbanana/amiga-compiler:m68k-amigaos`
- **Compiler**: `m68k-amigaos-gcc` 6.5.0b (GCC cross-compiler for m68k AmigaOS)
- **Build wrapper**: `./build.sh` runs `make` inside Docker, mounting the project at `/work`
- **Key compiler flags**: `-noixemul -m68020 -O2 -Wall`
- **Key linker flags**: `-noixemul -nostartfiles -m68020`
- **Required link libraries**: `-lm -lgcc` (math and GCC runtime)

### Docker container paths

| Path | Contents |
|---|---|
| `/opt/amiga/bin/` | Cross-tools (`m68k-amigaos-gcc`, `ar`, `as`, `ld`, etc.) |
| `/opt/amiga/m68k-amigaos/ndk-include/` | AmigaOS NDK headers (`exec/`, `dos/`, `intuition/`, etc.) |
| `/opt/amiga/m68k-amigaos/lib/` | System libraries (`libamiga.a`, `libc.a`, `libm.a`) |
| `/opt/amiga/m68k-amigaos/libnix/` | libnix C runtime (standalone, no ixemul.library) |

### GCC compatibility notes

- GCC defines `__AMIGA__`, `__amigaos__`, `__amiga__` — NOT `_AMIGA` (SAS/C) or `AZTEC_C`
- `XCALL_(t)` and `XCALL_INIT` are defined as no-ops for GCC in `plug.h`
- SAS/C object files (`.o`, `.lib`) are NOT link-compatible with GCC — everything must be rebuilt from source
- `stubs.c` provides a dummy `exit()` because libnix references it but plugins never call it

### CRITICAL: No libnix runtime functions

**Problem (solved)**: Using `-nostartfiles` skips libnix's C startup, which initializes the
heap, stdio, ctype tables, and other runtime state. The following libnix functions are
**BROKEN** in our plugin environment and must NOT be used:

- **`malloc` / `free` / `realloc`** — heap not initialized → hangs or corrupts memory
- **`sprintf` / `printf` / `fprintf`** — stdio not initialized
- **`qsort`** — may internally use malloc
- **`strncasecmp` / `strcasecmp`** — needs ctype/locale tables → address error crash
- **Any function requiring locale, stdio, or heap**

**Safe alternatives**:
- Memory: `AllocMem`/`FreeMem` via exec.library (SysBase opened in `_Startup`)
- Use a `plugin_alloc`/`plugin_free` wrapper that stores size before the returned pointer:
  ```c
  static void *plugin_alloc(unsigned long size) {
      unsigned long *p = (unsigned long *)AllocMem(size + 4, MEMF_PUBLIC | MEMF_CLEAR);
      if (!p) return 0;
      *p = size + 4;
      return (void *)(p + 1);
  }
  static void plugin_free(void *ptr) {
      unsigned long *p;
      if (!ptr) return;
      p = ((unsigned long *)ptr) - 1;
      FreeMem(p, *p);
  }
  ```
- Strings: `strcpy`/`strncpy`/`strcat`/`strncat`/`strlen`/`memcpy`/`memset` are safe (inline/no state)
- Case-insensitive compare: use custom `ci_strncmp` instead of `strncasecmp`
- Integer to string: use custom `int_to_str` instead of `sprintf`
- Sorting: use custom sort (bubble/insertion with byte-swap) instead of `qsort`

### CRITICAL: Minimize stack usage in plugin callbacks

LightWave plugin callbacks may run with limited stack. Large local arrays cause
address error crashes (guru $80000003). Rules:
- **Never** put large buffers on the stack (>64 bytes)
- Write directly into `plugin_alloc`'d buffers or struct fields instead
- Use byte-level swap for sorting structs (don't copy whole struct to stack local)
- Total stack usage per callback should stay under ~200 bytes

### CRITICAL: Math library initialization

**Problem (solved)**: Using `-nostartfiles` skips libnix's C startup, which normally opens
`mathieeedoubbas.library` and `mathieeedoubtrans.library`. GCC's soft-float math stubs
(`__adddf3`, `__muldf3`, `__divdf3`, `sqrt`, etc.) call through `MathIeeeDoubBasBase` and
`MathIeeeDoubTransBase` library base pointers. Without initialization, these contain garbage
→ address error (guru $80000003) on first floating-point operation.

**Fix**: `slib4.c` `_Startup()` now opens both math libraries via weak references, and
`_Shutdown()` closes them. Plugins that don't use floating-point won't pull in these symbols
and are unaffected.

**Verified**: spikey.p (uses `sqrt`, multiply, divide on doubles) runs correctly in LightWave
Modeler on real AmigaOS after this fix.

---

### Handler version numbers (LW 5.x)

LightWave 5.x sends these version numbers to Activate:

| Handler class | Version sent |
|---|---|
| `ObjReplacementHandler` | 2 |
| `ObjReplacementInterface` | 1 |
| `ImageFilterHandler` | 1 (assumed, negative sample checks `!=1`) |

Accept version >= 1 and fill the V1 fields. For version 2, the extra V2 fields
(descln, useItems, changeID) appear to be pre-zeroed by LW and can be left unfilled.

---

## LightWave 5.x Plugin Architecture

### Binary format

Plugins are AmigaOS `LoadSeg()`-able executables (`.p` files) with a specific header:

```
moveq  #0,d0           ; Return 0 if run from CLI
rts
dc.l   $04121994        ; Magic number
dc.l   $2               ; Flags
dc.l   $100             ; Version
dc.l   _Startup         ; Pointer to startup function
dc.l   _Shutdown        ; Pointer to shutdown function
dc.l   ServerDesc       ; Pointer to server description array
```

This header is provided by `sdk/source/serv_gcc.s` (assembled to `sdk/lib/serv_gcc.o`).

### Plugin lifecycle

1. LightWave calls `_Startup()` when the plugin is first loaded → returns `serverData` (non-null = success)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [midwan/amiga-lw-plugins](https://github.com/midwan/amiga-lw-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
