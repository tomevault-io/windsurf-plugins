---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Hical is a modern C++20 high-performance web framework built on Boost.Asio/Beast, featuring PMR memory pools, coroutine-based async I/O (`asio::awaitable<T>`), C++20 Concepts for compile-time type safety, a C++26 reflection layer (dual-track: native P2996 or C++20 macro fallback), and an optional coroutine-based database middleware (Boost.MySQL backend).

## Build Commands

### Linux / macOS
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
```

### Windows (MSYS2 MINGW64)
```bash
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

### Windows (MSVC + vcpkg)
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_TOOLCHAIN_FILE=C:/vcpkg/scripts/buildsystems/vcpkg.cmake
cmake --build build --config Release
```

### Enable Database Middleware (requires Boost.MySQL)
```bash
cmake -B build -DHICAL_WITH_DATABASE=ON ...
```

### Enable C++26 Reflection (requires compatible compiler)
```bash
cmake -B build -DHICAL_ENABLE_REFLECTION=ON ...
```

### Run All Tests
```bash
ctest --test-dir build --output-on-failure --timeout 60 -j4
# MSVC needs: ctest ... -C Release
```

### Run a Single Test
```bash
./build/tests/test_router
# Or via ctest with filter:
ctest --test-dir build -R test_router --output-on-failure
```

### Format Check (CI enforces this on GCC job)
```bash
find src tests examples -name '*.h' -o -name '*.cpp' | xargs clang-format --dry-run --Werror
```

### Static Analysis (Clang job, non-blocking)
```bash
find src -name '*.cpp' | xargs clang-tidy -p build
```

## Architecture

### Two-Layer Design

**`src/core/`** — Abstract interfaces, shared types, HTTP framework, and reflection layer:
- `EventLoop.h` / `Timer.h` / `TcpConnection.h` — Abstract base classes (pure virtual). `TcpConnection` includes `sendFile()` and `lastActiveTime()` virtual methods
- `Concepts.h` — C++20 concepts (`EventLoopLike`, `TcpConnectionLike`, `TimerLike`, `NetworkBackend`) for compile-time backend constraints
- `MemoryPool.h` — Three-tier PMR memory strategy: global synchronized pool, thread-local unsynchronized pool, request-level monotonic buffer
- `HttpServer.h` — Top-level facade integrating TcpServer + Router + MiddlewarePipeline + WebSocket middleware pre-build + fd exhaustion handling
- `Router.h` — Static routes (hash map O(1) with transparent hashing via `RouteKeyView`/`is_transparent` for zero-alloc `string_view` lookup) + parameter routes (`{id}` pattern, per-method grouping via `unordered_map<HttpMethod, vector>`) + WebSocket routes with `WsOptions` (Origin whitelist)
- `Middleware.h` — Onion-model middleware pipeline with `MiddlewareNext` chaining; supports pre-built chain (`build()`), dynamic chain (`buildChain()`), and `buildFor()` for external pre-build, with separate `execute()` overloads for cached vs dynamic paths
- `Coroutine.h` — `Awaitable<T>` alias for `boost::asio::awaitable<T>`, plus `sleep()` / `coSpawn()` helpers
- `Reflection.h` / `MetaJson.h` / `MetaRoutes.h` — C++26 reflection layer (see below)
- `StaticFiles.h` — Async static file serving (`Awaitable<HttpResponse>`) with `BOOST_ASIO_HAS_FILE` async I/O + ifstream fallback, PathCache (4096/60s TTL), ETag/304, MIME detection, path traversal protection, 64MB file size limit
- `Multipart.h/cpp` — RFC 7578 multipart/form-data parser (256 part DoS limit), dual API: `getFile(req, field)` (re-parses) and `getFile(parts, field)` (searches pre-parsed vector, recommended)
- `Session.h/cpp` — In-memory session manager with `shared_mutex` (read-write lock), lazy GC, OpenSSL RAND_bytes 128-bit IDs, `makeSessionMiddleware` factory, `maxSessions` DoS limit, atomic `lastAccess` (lock-free), `regenerate()` for session fixation prevention, `migrateFrom()` for atomic data migration with address-ordered double locking
- `IdleFd.h` — Cross-platform idle fd reservation (POSIX: `/dev/null` fd; Windows: no-op stub) for EMFILE accept loop protection
- `WriteNode.h` — Polymorphic write buffer nodes: `WriteNode` base, `MemoryWriteNode` (shared_ptr\<string\>), `FileWriteNode` (path/offset/length) for heterogeneous send queue
- `Version.h.in` — CMake-configured version header (single source of truth from `project(VERSION)`)

**`src/asio/`** — Boost.Asio concrete implementations:
- `AsioEventLoop` — Wraps `boost::asio::io_context`, implements `EventLoop`
- `GenericConnection<SocketType>` — Template supporting both `tcp::socket` (plain) and `ssl::stream<tcp::socket>` (SSL). Write queue uses `deque<shared_ptr<WriteNode>>` supporting both memory and file nodes. `sendFile()` + `sendFileNode()` for async file I/O with `BOOST_ASIO_HAS_FILE` guard + ifstream fallback. `lastActiveTimeMs_` atomic for idle detection. `reading_` is `atomic<bool>` for thread-safe `stopRead()`
- `SslConnection.h` — Lightweight SSL connection type alias (`SslConnection = GenericConnection<ssl::stream<tcp::socket>>`), lazy OpenSSL include
- `EventLoopPool` — Multi-threaded pool (1 thread : 1 io_context), round-robin connection distribution

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hical61/Hical](https://github.com/Hical61/Hical) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
