---
trigger: always_on
description: provides a new image digest; bump both references and (optionally) re-create
---

# AGENTS.md — nx.js

Guide for AI agents contributing to this repo.

## What is nx.js?

A JavaScript runtime for **Nintendo Switch homebrew**, built on QuickJS + libnx + cairo. It implements Web APIs (Canvas 2D, Fetch, Crypto, URL, EventTarget, etc.) so you can write Switch apps in JS/TS.

## Repo Structure

```
source/          — C code (native modules compiled for aarch64 Switch)
packages/
  runtime/       — TypeScript runtime (bundled via QuickJS bytecode compiler)
    src/$.ts     — Native binding types (the `$` object bridges JS↔C)
    src/index.ts — Entry point, registers all globals
    test/        — Host-platform test binary & TAP conformance tests
  nro/           — .nro builder (Switch homebrew executable format)
  nsp/           — .nsp builder
  create-nxjs-app/ — Scaffolding tool
apps/            — Example apps (each is a standalone pnpm workspace package)
Makefile         — Cross-compiles C code via devkitPro toolchain
```

## How Native Modules Work

Every feature follows the same pattern:

### C side (`source/foo.c` + `source/foo.h`)

```c
#include "types.h"

static JSValue nx_foo_do_thing(JSContext *ctx, JSValueConst this_val,
                               int argc, JSValueConst *argv) {
    // Implementation using libnx or other C libraries
    return JS_UNDEFINED;
}

static const JSCFunctionListEntry function_list[] = {
    JS_CFUNC_DEF("fooDoThing", 1, nx_foo_do_thing),
};

void nx_init_foo(JSContext *ctx, JSValueConst init_obj) {
    JS_SetPropertyFunctionList(ctx, init_obj, function_list,
                               countof(function_list));
}
```

Header (`source/foo.h`):
```c
#pragma once
#include <quickjs.h>
void nx_init_foo(JSContext *ctx, JSValueConst init_obj);
```

Then in `source/main.c`:
- `#include "foo.h"` at the top
- `nx_init_foo(ctx, nx_ctx->init_obj);` in the init block (~line 680, alphabetical)

### JS side (`packages/runtime/src/foo.ts`)

```typescript
import { $ } from './$';
import { def } from './utils';
import { EventTarget } from './polyfills/event-target';

export class Foo extends EventTarget {
    constructor() {
        super();
        $.fooInit();
        addEventListener('unload', $.fooExit);
    }
}
def(Foo); // registers class name for toString/instanceof
```

- The `$` object contains ALL native function bindings (defined in C `function_list`s)
- Add types for new `$` functions in `packages/runtime/src/$.ts`
- Register in `packages/runtime/src/index.ts`
- Use `assertInternalConstructor` for classes that shouldn't be user-constructible
- Use `proto()` for classes that return a native object from C (like Image)

### Async Work Pattern

For expensive operations (decoding, crypto, etc.), use the thread pool:

```c
// In work_cb (runs on thread pool — NO JS API calls allowed):
static void my_work_cb(nx_work_t *req) {
    my_data_t *data = (my_data_t *)req->data;
    // Do heavy work here
}

// In after_work_cb (runs on main thread — JS API calls OK):
static JSValue my_after_work_cb(JSContext *ctx, nx_work_t *req) {
    my_data_t *data = (my_data_t *)req->data;
    // Return result to JS
    return JS_NewArrayBuffer(...);
}

// Queue it:
return nx_queue_async(ctx, req, my_work_cb, my_after_work_cb);
```

See `source/async.c` for the implementation, `source/image.c` and `source/crypto.c` for examples.

## Key Types & Macros (from `source/types.h`)

- `nx_context_t` — per-runtime context (thread pool, work queue, rendering mode, etc.)
- `NX_DEF_GET(obj, "name", getter_fn)` — define a getter
- `NX_DEF_GETSET(obj, "name", get_fn, set_fn)` — define getter + setter
- `NX_DEF_FUNC(obj, "name", fn, arg_count)` — define a method
- `countof(x)` — array length macro

## File Loading Pattern

Files are loaded via `fetch()` with `romfs:/` URLs (Switch ROM filesystem) or regular paths:

```typescript
// In Image class (good reference for loading resources):
fetch(url)
    .then(res => res.arrayBuffer())
    .then(buf => $.imageDecode(this, buf))
```

The `$.entrypoint` gives the base URL for resolving relative paths.

### RomFS mounts (`romfs:` vs `nxjs:`)

`main.cc` mounts two RomFS devices:

- **`nxjs:`** — the nx.js NRO's OWN embedded RomFS (holds the runtime's source
  map, `nxjs:/runtime.js.map`). Always mounted. The embedded runtime is run
  under the name `nxjs:/runtime.js` so its stack frames symbolicate.
- **`romfs:`** — "the app". For a standalone app this is the same NRO's RomFS
  (so existing `romfs:/asset` references work). For a **bootstrap launch**
  (`argv[1]` is an app `.nro`), it is the launched app's RomFS instead.

### Entrypoint resolution (`argv[1]`)

`main.cc` resolves the user entrypoint as:

1. If `argv[1]` is set (bootstrap launch — a thin launcher hands nx.js the app):
   - `*.nro` → mount its embedded RomFS as `romfs:` and run `romfs:/main.js`.
     The RomFS is not at file offset 0, so `mount_nro_romfs()` parses the NRO
     header + asset header to find the RomFS offset, then calls
     `romfsMountFromFsdev(argv[1], romfs_offset, "romfs")`.
   - otherwise (typically `*.js`) → run `argv[1]` directly.
2. Else (standalone): mount self as `romfs:`, run `romfs:/main.js`; fall back to
   `<argv0>.js` next to the `.nro` on the SD card.

The entrypoint resolution (`resolve_entrypoint()`) runs **early in `main()`, before

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TooTallNate/nx.js](https://github.com/TooTallNate/nx.js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
