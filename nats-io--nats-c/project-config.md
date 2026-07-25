---
trigger: always_on
description: NATS client library for C. Supports core NATS, JetStream, KeyValue, Object Store, and microservices. C99 standard, cross-platform (Linux, macOS, Windows).
---

# CLAUDE.md - nats.c

NATS client library for C. Supports core NATS, JetStream, KeyValue, Object Store, and microservices. C99 standard, cross-platform (Linux, macOS, Windows).

## Build Commands

```bash
# Standard build (out-of-source)
mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Debug
make

# Common cmake options
cmake .. \
  -DCMAKE_BUILD_TYPE=Debug \           # Debug, Release, RelWithDebInfo
  -DNATS_BUILD_WITH_TLS=ON \           # TLS via OpenSSL (default ON)
  -DNATS_BUILD_STREAMING=OFF \         # NATS Streaming/STAN (deprecated, default OFF)
  -DNATS_BUILD_USE_SODIUM=OFF \        # libsodium for NKeys (default OFF)
  -DNATS_WITH_EXPERIMENTAL=ON \        # Experimental APIs
  -DNATS_BUILD_EXAMPLES=ON \           # Build examples (default ON)
  -DNATS_COVERAGE=ON                   # Code coverage

# Dev mode build (adds type safety checks for lock/unlock macros)
cmake .. -DNATS_BUILD_DEV_MODE=ON -DCMAKE_BUILD_TYPE=Debug

# Run tests (requires nats-server on PATH)
cd build
ctest -L test --timeout 60 --output-on-failure

# Run a single test
cd build
ctest -L test --timeout 60 -V --repeat-until-fail 1 -R <TestName>    # e.g., ctest -L test --timeout 60 -V --repeat-until-fail 1 -R DefaultConnection

# With sanitizers (set env before cmake)
NATS_SANITIZE=address cmake .. -DNATS_SANITIZE=ON -DCMAKE_C_FLAGS=-fsanitize=address
```

## Project Structure

```
src/                        # Library source code
  nats.h                    # Internal header with public API declarations (~10K lines, Doxygen-documented)
  natsp.h                   # Private header: all internal struct definitions, helper macros
  status.h                  # natsStatus enum, jsErrCode enum (public)
  version.h                 # Version defines (generated from version.h.in)
  mem.h                     # Memory allocation macros (NATS_MALLOC/FREE/STRDUP)
  err.h                     # Error reporting macros and functions
  conn.c/h                  # Connection implementation
  sub.c/h                   # Subscription implementation
  pub.c                     # Publish implementation
  msg.c/h                   # Message type
  js.c/h, jsm.c             # JetStream client and management
  kv.c/h                    # KeyValue store
  object.c/h                # Object store
  micro.c, microp.h         # Microservices framework (micro_*)
  micro_client.c, micro_endpoint.c, micro_error.c, micro_monitoring.c, micro_request.c
  opts.c/h                  # Connection options
  parser.c/h                # Protocol parser
  buf.c/h                   # Buffer utilities
  hash.c/h                  # Hash table
  nuid.c/h                  # NUID generation
  nkeys.c/h                 # NKey authentication
  crypto.c/h                # Crypto utilities
  dispatch.c/h              # Message dispatch (thread pool, per-sub threads)
  asynccb.c/h               # Async callback management
  timer.c/h                 # Timer implementation
  srvpool.c/h               # Server pool management
  comsock.c/h               # Socket communication
  url.c/h                   # URL parsing
  util.c/h                  # General utilities
  stats.c/h                 # Statistics
  natstime.c/h              # Time utilities
  gc.h                      # Garbage collection
  deprnats.h                # Deprecated top-level nats.h redirect
  include/                  # Platform abstraction headers
    n-unix.h, n-win.h       # Platform-specific type definitions
  unix/                     # Unix platform impl (thread, mutex, cond, sock)
  win/                      # Windows platform impl
  glib/                     # Internal GLib-like utilities (timers, SSL, GC, dispatch pool, etc.)
  stan/                     # NATS Streaming (deprecated, requires protobuf-c)
  adapters/                 # Event loop adapter headers (libuv, libevent)
test/
  test.c                    # Single monolithic test file
  test.h                    # Test helpers, server management macros
  list.h                    # X-macro that includes list_test.txt, list_bench.txt, list_stan.txt
  list_test.txt             # Test function registry (~304 tests), format: _test(TestName)
  list_bench.txt            # Benchmark function registry
  CMakeLists.txt            # Test build config, registers each test with CTest
  certs/                    # TLS test certificates
  dylib/                    # Test for dynamic library loading without NATS calls
  check_cpp/                # C++ compatibility check
  no_crash_on_exit/         # Test that library does not seg fault when main thread exit while nats_CloseAndWait is in progress
examples/                   # Example programs
  getstarted/               # Simple getting-started examples
  stan/                     # NATS Streaming examples (deprecated)
doc/                        # Doxygen documentation
```

## Code Style and Conventions

### Naming
- **Public API functions**: `nats<Type>_<Action>()` with `NATS_EXTERN` prefix, e.g., `natsConnection_Connect()`, `natsSubscription_NextMsg()`, `jsStreamConfig_Init()`
- **Internal functions**: `nats<Type>_<action>()` (camelCase action), e.g., `natsConn_publish()`, `natsConn_bufferFlush()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nats-io/nats.c](https://github.com/nats-io/nats.c) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
