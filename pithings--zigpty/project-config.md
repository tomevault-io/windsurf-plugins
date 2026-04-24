---
trigger: always_on
description: Zig-based PTY library. Dual-use: standalone **Zig package** and **Node.js NAPI addon**.
---

# zigpty

Zig-based PTY library. Dual-use: standalone **Zig package** and **Node.js NAPI addon**.

## Goals

- Smallest build (19KB ReleaseSmall vs node-pty's ~500KB+ with C++ runtime)
- Pure Zig PTY library with no NAPI dependency (`lib.zig`)
- Thin NAPI wrapper layer for Node.js (`pty.zig` + `pty_unix.zig` + `win/napi.zig` + `root.zig`)
- Raw NAPI — no third-party Zig NAPI bindings, no node-gyp
- Cross-platform (Linux + macOS + Windows)
- Statically linked via Zig

## Architecture

```
zigpty/
├── build.zig               # Zig build: exposes "zigpty" module + NAPI shared libs
├── build.zig.zon           # Zig package metadata (min Zig 0.15.1)
├── build.config.ts         # obuild config (bundle src/ → dist/)
├── zig/                    # Zig sources (two layers)
│   ├── lib.zig             # Pure Zig PTY library — platform dispatcher + shared code
│   ├── root.zig            # NAPI module entry: exports platform-specific functions
│   ├── napi.zig            # Raw NAPI bindings (~240 lines, extern declarations)
│   ├── pty.zig             # NAPI shared helpers + platform dispatch (re-exports)
│   ├── pty_unix.zig        # NAPI↔lib.zig bridge for Unix (fork, open, resize, process)
│   ├── termios.zig         # Default terminal config (Linux + macOS)
│   ├── pty_linux.zig       # Linux-specific: execvpe, ptsname_r, /proc, close_range
│   ├── pty_darwin.zig      # macOS-specific: execvp+environ, sysctl, FD close loop
│   ├── pty_windows.zig     # Windows-specific: ConPTY (CreatePseudoConsole + pipes)
│   ├── errno_shim.c        # Android errno compat (__errno_location → __errno)
│   └── win/
│       ├── napi.zig        # NAPI↔lib.zig bridge for Windows (spawn, write, resize, kill, close)
│       └── node_api.def    # NAPI import definitions (→ .lib via zig dlltool)
├── src/                    # TypeScript wrapper + tests
│   ├── index.ts            # Public API: spawn(), open() — platform dispatch
│   ├── napi.ts             # Native module loader (platform-aware, INativeUnix/INativeWindows)
│   ├── terminal.ts         # Terminal class (Bun-compatible) + TerminalOptions type
│   ├── pty/                # PTY class hierarchy
│   │   ├── _base.ts        # BasePty abstract class — shared state, events, waitFor, buildEnvPairs
│   │   ├── types.ts        # IPty, IPtyOptions, IDisposable, IEvent interfaces
│   │   ├── unix.ts         # UnixPty: tty.ReadStream + async fs.write
│   │   ├── windows.ts      # WindowsPty: native callbacks + deferred init
│   │   └── pipe.ts         # PipePty: pure-TS fallback when native bindings unavailable
│   ├── spawn.test.ts       # E2E tests (platform-conditional)
│   ├── pipe.test.ts        # PipePty fallback tests
│   └── terminal.test.ts    # Terminal class tests
├── dist/                   # Built output (obuild → .mjs + .d.mts)
├── prebuilds/              # Zig build output (8 binaries, see below)
├── scripts/
│   └── cross-platform.sh   # Docker-based cross-platform smoke tests
└── .github/workflows/
    └── ci.yml              # PR/push: build + test (Linux, macOS, Windows)
```

### TypeScript Class Hierarchy

```
IPty (interface, src/pty/types.ts)
  └── BasePty (abstract, src/pty/_base.ts)
        ├── UnixPty (src/pty/unix.ts)       — native PTY via Zig NAPI
        ├── WindowsPty (src/pty/windows.ts) — native ConPTY via Zig NAPI
        └── PipePty (src/pty/pipe.ts)       — pure-TS fallback (child_process pipes)

Terminal (standalone, src/terminal.ts) — Bun-compatible callbacks, AsyncDisposable
```

**BasePty** contains shared logic:

- State: `_dataListeners`, `_exitListeners`, `_closed`, `_exitCode`, `_exited` promise, `_terminal`
- Event getters: `onData`, `onExit`, `exited`, `exitCode`
- `waitFor(pattern, { timeout? })` — waits for string in output, hooks into both `onData` and Terminal data paths
- `_handleExit(info)` — common exit callback (set closed, fire listeners, resolve promise)
- `buildEnvPairs(env, termName?, sanitizeKeys?)` — shared env builder

**UnixPty** adds: fd management, `tty.ReadStream`, async write queue with EAGAIN retry, flow control, `process` via native, signal-based `kill()`.

**WindowsPty** adds: ConPTY handle, deferred calls until ready, `napi_threadsafe_function` data/exit callbacks.

**PipePty** (fallback, no native dependency): Spawns child via `child_process.spawn` with `stdio: ["pipe", "pipe", "pipe"]`. Emulates terminal behavior in userspace:

- Signal character translation (`^C`→SIGINT, `^Z`→SIGTSTP, `^\`→SIGQUIT, `^D`→EOF)
- Canonical mode with echo (line buffering, backspace, `^W` word erase, `^U` line kill, `^R` reprint)
- Raw mode (`setRawMode()` / `setCanonicalMode()`) — disables echo and line buffering
- XON/XOFF flow control interception when `handleFlowControl` is enabled
- Force-color env hints (`FORCE_COLOR=1`, `COLORTERM=truecolor`) auto-set
- `SIGWINCH` sent on `resize()` as best-effort hint
- Foreground process tracking via `/proc/<pid>/stat` → `/proc/<pgrp>/cmdline` (Linux only)
- Merges stdout + stderr into single data stream (matching real PTY behavior)

**Native loading** (`napi.ts`): `loadNative()` returns `null` on failure instead of throwing. `hasNative` boolean exported for runtime detection. `spawn()` in `index.ts` routes to `PipePty` when `hasNative === false`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pithings/zigpty](https://github.com/pithings/zigpty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
