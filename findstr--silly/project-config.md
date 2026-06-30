---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Silly is a lightweight, high-performance Lua server framework with coroutine-based async/await. It handles 200,000+ requests/second with single-threaded business logic and supports TCP, UDP, HTTP, WebSocket, gRPC, TLS, MySQL, Redis, and more.

## Lua Interpreter

The standalone Lua interpreter is at `deps/lua/lua`. Use it for running pure Lua scripts (benchmarks, utilities) that don't need the silly runtime:

```bash
deps/lua/lua script.lua
```

For scripts that use silly modules (network, coroutines, etc.), use the silly runtime instead:

```bash
./silly script.lua
```

## Build Commands

```bash
make                # Standard build (Linux/macOS/Windows)
make OPENSSL=ON     # Build with OpenSSL/TLS support
make test           # Build with address sanitizer (Linux/macOS)
make clean          # Clean build artifacts
make cleanall       # Clean everything including dependencies
make fmt            # Format C code with clang-format
```

## Testing

```bash
make testall                                      # Run all tests
sh test/test.sh -j                                # Run tests in parallel (Linux only)
./silly test/test.lua --set=testtcp2              # Run specific test suite
./silly test/test.lua --set=testtcp2 --case="Test 11"  # Run specific test case
```

### Test File Organization

- Test files are in `test/` directory, named `test*.lua`
- **ALL tests MUST be wrapped with `testaux.case("Test X: Description", function() ... end)`**
- **ALL assertions MUST use `"Test X.Y:"` prefix format** (e.g., `"Test 11.3: description"`)
- Follow the pattern in `test/testredis.lua` or `test/testtcp2.lua` for reference

## Architecture

### Threading Model

Silly uses a hybrid 4-thread architecture:

1. **Worker Thread (Lua VM)**: Single-threaded business logic, runs Lua code and coroutines
2. **Socket Thread**: Event-driven I/O (epoll/kqueue/iocp), handles up to 65K connections
3. **Timer Thread**: 10ms resolution timing, manages timeouts and scheduled tasks
4. **Monitor Thread**: Health checks, detects slow message processing

Key principle: Single-threaded business logic eliminates locks and race conditions.

### Event Loop Ordering

The worker thread calls `task._dispatch_wakeup()` **after every single message** (timer, socket I/O, signal). This is a critical architectural guarantee:

- Any coroutine woken during message processing runs **before** the next message is dispatched
- Timers cancelled by a woken coroutine are guaranteed cancelled before their EXPIRE event fires

```
message N processed → _dispatch_wakeup() → [ready coroutines run] → message N+1 processed
```

This makes patterns like "wake coroutine, coroutine cancels timer" race-free.

### Yield Semantics

Only these operations yield the current coroutine:

| Yields | Does NOT yield |
|--------|---------------|
| `conn:read()`, `conn:recvfrom()`, `conn:readall()` | `conn:write()`, `conn:sendto()`, `conn:closewrite()` |
| `tcp.connect()`, `tls.connect()` | `udp.connect()` (no handshake) |
| `time.sleep()`, `task.wait()` | `time.after()`, `task.fork()` |
| `channel:pop()` (when empty) | `tcp.listen()`, `udp.bind()` |

**Between a send and the next yield point, no other coroutine can run.** This is critical for reasoning about code correctness without locks.

### Timer Internals

`time.after(ms, func, ud)` runs `func` in a **new coroutine** when the timer fires.

`time.cancel(session)` clears the callback from the dispatch table. The EXPIRE handler checks `sleep_session_task[session]` before running, so cancel works even if the EXPIRE event is already queued — because `_dispatch_wakeup` guarantees the cancelling coroutine runs first.

### Coroutine Flow Control

Network modules implement flow control via coroutines:

- **Critical Pattern**: Clear state first, then call `wakeup` to prevent re-entry bugs:
  ```lua
  local co = s.co
  s.co = nil
  s.delim = nil
  wakeup(co, data)
  ```

- **Timeout Pattern**: Use timer + sentinel value:
  ```lua
  local errno = require "silly.errno"
  local TIMEOUT = {}

  local timer = time.after(timeout_ms, function(s)
      local co = s.co
      if co then           -- guard: check nil since finish_req may have cleared it
          s.co = nil
          wakeup(co, TIMEOUT)
      end
  end, socket)

  local data = wait()
  if data == TIMEOUT then
      return nil, errno.TIMEDOUT
  end
  time.cancel(timer)
  ```

### Module Structure

- `lualib/silly/`: Pure Lua modules
- `lualib/silly/net/`: Network protocols (tcp, udp, tls, http, websocket, grpc)
- `lualib/silly/store/`: Database clients (mysql, redis, etcd)
- `lualib/silly/sync/`: Concurrency primitives (channel, mutex, waitgroup)
- `lualib/silly/crypto/`: Cryptography utilities
- `lualib/silly/metrics/`: Prometheus metrics
- `luaclib-src/`: C extensions that compile to `.so` files in `luaclib/`
- `src/`: Core C engine (socket, timer, worker, message queue)

### C-Lua Boundary

C modules in `luaclib-src/` expose APIs to Lua:
- `lnet.c`: Socket operations (connect, listen, send, recv)
- `lhttp.c`: HTTP parsing
- `lsilly.c`: Core runtime (fork, wait, wakeup)
- `ltime.c`: Timer operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [findstr/silly](https://github.com/findstr/silly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
