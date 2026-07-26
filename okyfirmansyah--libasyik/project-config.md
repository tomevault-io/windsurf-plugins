---
trigger: always_on
description: A C++17 microframework combining **Boost.Asio** (async I/O), **Boost.Beast** (HTTP/WebSocket), and **Boost.Fiber** (cooperative user-space threads) into a synchronous-looking API for building HTTP/REST/WebSocket services. Each connection runs in its own lightweight fiber — enabling simple sequential code that handles thousands of concurrent connections on a single thread.
---

# AGENT.md — AI Assistant Guide for libasyik

## What is libasyik?

A C++17 microframework combining **Boost.Asio** (async I/O), **Boost.Beast** (HTTP/WebSocket), and **Boost.Fiber** (cooperative user-space threads) into a synchronous-looking API for building HTTP/REST/WebSocket services. Each connection runs in its own lightweight fiber — enabling simple sequential code that handles thousands of concurrent connections on a single thread.

## Architecture at a Glance

```
┌──────────────────────────────────────────┐
│  asyik::service  (1 OS thread)           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ │
│  │ fiber #1 │ │ fiber #2 │ │ fiber #N │ │  ← cooperative scheduling
│  │ (conn A) │ │ (conn B) │ │ (conn N) │ │  ← 1 connection = 1 fiber
│  └──────────┘ └──────────┘ └──────────┘ │
│         boost::asio::io_context          │
│         boost::fiber scheduler           │
├──────────────────────────────────────────┤
│  Worker thread pool  (async())           │  ← CPU/blocking I/O offload
└──────────────────────────────────────────┘
```

**Key principle**: Fibers within the same `asyik::service` share one OS thread — no locks needed for shared state. Fibers yield cooperatively at libasyik API calls.

## Critical: Fiber Programming Rules

### RULE 1 — Never block the fiber scheduler

Fibers share one OS thread per service. Any blocking call freezes **all** fibers on that service.

```cpp
// ✗ WRONG — blocks entire scheduler
as->execute([]() {
    std::this_thread::sleep_for(std::chrono::seconds(1)); // BLOCKS all fibers!
    usleep(1000);                                          // BLOCKS all fibers!
    sleep(1);                                              // BLOCKS all fibers!
    auto data = fread(fp, ...);                            // BLOCKS all fibers!
    boost::asio::connect(socket, endpoint);                // BLOCKS (sync Asio)!
});

// ✓ CORRECT — use fiber-friendly sleep
as->execute([]() {
    asyik::sleep_for(std::chrono::seconds(1));  // yields to scheduler
});
```

### RULE 2 — Use fiber-aware synchronization primitives

`std::mutex` suspends the OS thread. Use `boost::fibers::mutex` instead — it yields the current fiber while waiting.

```cpp
// ✗ WRONG — std::mutex blocks the thread (all fibers starve)
std::mutex mtx;
as->execute([&mtx]() {
    std::lock_guard<std::mutex> lock(mtx);  // thread-level block!
    // ...
});

// ✓ CORRECT — fibers::mutex yields to other fibers while waiting
boost::fibers::mutex mtx;
as->execute([&mtx]() {
    std::lock_guard<boost::fibers::mutex> lock(mtx);  // fiber yields
    // ...
});
```

Similarly, use `boost::fibers::promise` / `boost::fibers::future` / `boost::fibers::condition_variable` / `boost::fibers::buffered_channel` instead of their `std::` equivalents.

### RULE 3 — Offload CPU-intensive or blocking work with async()

`service::async()` dispatches work to a background thread pool and returns a `fibers::future`. The calling fiber yields until the result is ready.

```cpp
as->execute([as]() {
    // Offload heavy computation to thread pool
    auto result = as->async([]() -> std::string {
        // OK to use blocking APIs here — runs on worker thread
        sleep(3);
        return do_heavy_computation();
    }).get();  // fiber yields until worker thread completes

    LOG(INFO) << "Result: " << result << "\n";
});
```

The thread pool size is `hardware_concurrency × ASYIK_THREAD_MULTIPLIER` (env var, default 5).

### RULE 4 — Wrapping external async operations with use_fiber_future

To integrate any Boost.Asio-compatible async operation into fiber code, use `asyik::use_fiber_future` as the completion token:

```cpp
#include "libasyik/internal/use_fiber_future.hpp"

// Converts an Asio async call into a fiber-blocking call
// The fiber yields until the async operation completes
size_t n = stream.async_write_some(
    boost::asio::buffer(data),
    asyik::use_fiber_future   // completion token
).get();                      // .get() blocks *only* this fiber
```

For non-Asio async libraries, use `fibers::promise`/`fibers::future` manually:

```cpp
boost::fibers::promise<std::string> p;
auto f = p.get_future();

external_lib::async_fetch(url, [p = std::move(p)](auto result) mutable {
    p.set_value(result);  // fulfills promise from callback
});

std::string data = f.get();  // fiber yields until callback fires
```

## Service Lifecycle

```cpp
#include "libasyik/service.hpp"

// 1. Create service (creates io_context + fiber scheduler)
auto as = asyik::make_service();

// 2. Spawn fibers and register handlers
as->execute([as]() {
    // ... setup work ...
});

// 3. Run (blocks current thread — runs event loop + fiber scheduler)
as->run();

// To stop from inside a fiber:
as->stop();
```

`get_current_service()` retrieves the active service from within any fiber or `async()` worker:
```cpp
auto as = asyik::get_current_service();
```

## HTTP Server

```cpp
#include "libasyik/service.hpp"
#include "libasyik/http.hpp"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [okyfirmansyah/libasyik](https://github.com/okyfirmansyah/libasyik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
