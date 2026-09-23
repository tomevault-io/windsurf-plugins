---
trigger: always_on
description: Hical is a modern C++20 high-performance web framework built on Boost.Asio, featuring a native HTTP/WebSocket stack (picohttpparser + self-developed WebSocket implementation), PMR three-tier memory pools, coroutine-based async I/O (`asio::awaitable<T>`), C++20 Concepts for compile-time type safety, a C++26 reflection layer (dual-track: native P2996 or C++20 macro fallback), and an optional coroutine-based database middleware (Boost.MySQL and libpq/PostgreSQL backends).
---

# Repository Guidelines

## Project Overview

Hical is a modern C++20 high-performance web framework built on Boost.Asio, featuring a native HTTP/WebSocket stack (picohttpparser + self-developed WebSocket implementation), PMR three-tier memory pools, coroutine-based async I/O (`asio::awaitable<T>`), C++20 Concepts for compile-time type safety, a C++26 reflection layer (dual-track: native P2996 or C++20 macro fallback), and an optional coroutine-based database middleware (Boost.MySQL and libpq/PostgreSQL backends).

## Project Structure & Module Organization

- `src/core/` — Abstract interfaces, HTTP framework, routing, middleware, logging, server code, reflection layer. Must **not** include `src/asio/` headers.
- `src/asio/` — Boost.Asio concrete implementations (event loop, TCP connection, TCP server, event-loop pool)
- `src/db/` — Optional database middleware (enable with `-DHICAL_WITH_DATABASE=ON`, guarded by `HICAL_HAS_DATABASE` macro)
- `src/third_party/` — Bundled dependencies (picohttpparser)
- `tests/` — GoogleTest test suite, one `test_<feature>.cpp` per module
- `examples/` — Example servers (echo, benchmark, OpenAPI, reflection, PMR)
- `docs/` — Longer guides (architecture, coroutines, logging, OpenAPI, deployment)
- `docker/` — CI test matrix, production deployment, benchmark tooling
- `benchmark/` — Multi-framework HTTP benchmark suite (hical vs actix/drogon/cinatra/etc.)

Namespaces: public API in `hical::`, reflection in `hical::meta::`, database in `hical::db::`, internals in anonymous namespace or `detail::`.

## Build, Test, and Development Commands

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

### Optional Modules
```bash
cmake -B build -DHICAL_WITH_DATABASE=ON ...    # Database middleware (requires Boost.MySQL >= 1.85)
cmake -B build -DHICAL_WITH_PGSQL=ON ...       # PostgreSQL backend (requires libpq, implies HICAL_WITH_DATABASE)
cmake -B build -DHICAL_WITH_OPENAPI=OFF ...    # Disable OpenAPI (enabled by default)
cmake -B build -DHICAL_ENABLE_REFLECTION=ON ... # C++26 reflection (requires compatible compiler)
```

### Run Tests
```bash
# Full suite
ctest --test-dir build --output-on-failure --timeout 60 -j4
# MSVC needs: ctest ... -C Release

# Single test
./build/tests/test_router
ctest --test-dir build -R test_router --output-on-failure

# CI-like Linux suite
cd docker/test && docker compose up --build --abort-on-container-exit
```

### Format & Static Analysis
```bash
# Format check (CI enforces on GCC job)
find src tests examples -name '*.h' -o -name '*.cpp' | xargs clang-format --dry-run --Werror

# Format fix
find src tests examples -name '*.h' -o -name '*.cpp' | xargs clang-format -i

# Static analysis (Clang job, non-blocking)
find src -name '*.cpp' | xargs clang-tidy -p build
```

### Architecture

**`src/core/` modules:**
- `EventLoop.h` / `Timer.h` / `TcpConnection.h` — Abstract base classes
- `Concepts.h` — C++20 concepts (`EventLoopLike`, `TcpConnectionLike`, `TimerLike`, `NetworkBackend`)
- `MemoryPool.h` — Three-tier PMR: global synchronized pool → thread-local unsynchronized pool → request-level monotonic buffer. **Critical constraint:** objects from request-level monotonic buffer must not escape request lifetime (use-after-free).
- `HttpServer.h` — Top-level facade: TcpServer + Router + MiddlewarePipeline + WebSocket + IdleScanner. SO_REUSEPORT multi-acceptor (Linux/macOS), single-acceptor fallback (Windows). Graceful stop via `releaseWork()` (no `io_context::stop()`).
- `HeaderMap.h` — `vector<pair<string,string>>` backed, case-insensitive lookup, L1-cache-friendly for typical <20 headers
- `HttpRequest.h/cpp` — Zero-copy request wrapper: `string_view` referencing connection-level read buffer, stack-allocated `array<Entry,64>` headers. Public API: `method()`, `path()`, `header()`, `body()`, `cookie()`, `queryParam()`, `formParam()`, `readJson<T>()`
- `HttpResponse.h/cpp` — Response wrapper with `FileBody` deferred async file sending, `serializeHeadTo(FixedBuffer<512>&)` zero-heap scatter-gather I/O, `setHeader()` accepts `std::string_view`
- `HttpSessionImpl.cpp` — Compilation firewall for picohttpparser + WebSocket. ReadBufferPool borrow/return (8KB thread_local pool), optimistic sync write (≤512B single-buffer), response prefix template (~90B pre-built wire bytes), IdleScanner::Guard RAII idle timeout

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hical61/Hical](https://github.com/Hical61/Hical) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
