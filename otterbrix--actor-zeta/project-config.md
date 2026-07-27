---
trigger: always_on
description: Guidance for Claude Code when working with this repository.
---

# CLAUDE.md

Guidance for Claude Code when working with this repository.

## Core Rules

1. **READ FULL FILES** before making changes - this codebase has intricate template metaprogramming
2. **NO RTTI, NO EXCEPTIONS** - code MUST compile with `-fno-rtti -fno-exceptions`
3. **USE PMR** - never use `new`/`delete` directly, always `spawn<Actor>(memory_resource, args...)`
4. **BUILD AND TEST** after every change

## Project Overview

actor-zeta is a C++20 **header-only** actor model with cooperative scheduling, PMR memory management, and no RTTI/exceptions dependencies.

## Quick Start

```bash
# Setup
conan profile detect --force
conan install . -of build -s build_type=Debug --build=missing

# Build
cmake -B build -GNinja \
  -DCMAKE_BUILD_TYPE=Debug \
  -DALLOW_EXAMPLES=ON \
  -DALLOW_TESTS=ON \
  -DRTTI_DISABLE=ON \
  -DEXCEPTIONS_DISABLE=ON \
  -DCMAKE_TOOLCHAIN_FILE=./build/Debug/generators/conan_toolchain.cmake
cmake --build build

# Test
cd build && ctest --output-on-failure
```

**CLion:** Uses `cmake-build-debug/`. Set toolchain: `-DCMAKE_TOOLCHAIN_FILE=cmake-build-debug/conan/Debug/generators/conan_toolchain.cmake`

## File Structure

```
header/
├── actor-zeta.hpp              # Main API
├── actor-zeta/
│   ├── core.hpp                # Core types
│   ├── spawn.hpp               # Actor allocation
│   ├── send.hpp                # Message sending
│   ├── actor/                  # Actor implementation
│   │   ├── basic_actor.hpp     # Basic actor alias
│   │   ├── dispatch.hpp        # Message dispatch
│   │   └── dispatch_traits.hpp # Dispatch traits
│   ├── mailbox/                # Message system
│   └── detail/                 # Internal (future.hpp, generator.hpp, rtt.hpp)
test/                           # Catch2 tests
examples/                       # Usage examples
```

## Architecture

### Actor Lifecycle
1. Define actor class inheriting from `basic_actor<Actor>`
2. Define `dispatch_traits<&Actor::method1, &Actor::method2>`
3. Implement `behavior_t behavior(message*)` - coroutine with `co_await dispatch(...)`
4. Spawn: `auto actor = spawn<MyActor>(memory_resource, args...)`
5. Send: `auto [needs_sched, future] = send(actor.get(), &MyActor::method, args...)`
6. Schedule: `if (needs_sched) scheduler->enqueue(actor.get())`

### Actor Shutdown (CRITICAL)

**`scheduler->stop()` MUST be called BEFORE destroying actors.**

The scheduler holds raw pointers (`job_ptr`) to actors. If an actor is destroyed while the scheduler is running, worker threads may call `resume()` on freed memory (use-after-free).

#### Safe Pattern 1: Stop Scheduler First (Recommended)

```cpp
void safe_shutdown() {
    auto scheduler = std::make_unique<sharing_scheduler>(4, 1000);
    scheduler->start();

    auto actor = spawn<MyActor>(resource);

    // Send messages (fire-and-forget is OK)
    for (int i = 0; i < 100; ++i) {
        auto [needs_sched, future] = send(actor.get(), &MyActor::process, i);
        if (needs_sched) scheduler->enqueue(actor.get());
        future.detach();  // Fire-and-forget
    }

    scheduler->stop();  // All workers exit, no more resume() calls
}  // Actor destroyed here - SAFE
```

#### Safe Pattern 2: Wait for All Work

```cpp
void wait_for_work() {
    auto scheduler = std::make_unique<sharing_scheduler>(4, 1000);
    scheduler->start();

    std::vector<unique_future<int>> futures;

    {
        auto actor = spawn<MyActor>(resource);

        for (int i = 0; i < 10; ++i) {
            auto [needs_sched, future] = send(actor.get(), &MyActor::compute, i);
            if (needs_sched) scheduler->enqueue(actor.get());
            futures.push_back(std::move(future));  // Keep ALL futures
        }

        // Wait for ALL futures (scheduler workers produce; drive via run_until_complete).
        for (auto& f : futures) {
            auto result = actor_zeta::run_until_complete(f, []{ std::this_thread::yield(); });
        }
    }  // Actor destroyed - SAFE (all work complete)

    scheduler->stop();
}
```

#### Safe Pattern 3: Actor Outlives Scheduler (RAII)

```cpp
class Application {
    std::unique_ptr<MyActor, pmr::deleter_t> actor_;    // Declared FIRST
    std::unique_ptr<sharing_scheduler> scheduler_;       // Declared SECOND

public:
    Application(std::pmr::memory_resource* res)
        : actor_(spawn<MyActor>(res))
        , scheduler_(std::make_unique<sharing_scheduler>(4, 1000)) {
        scheduler_->start();
    }

    ~Application() {
        // C++ destroys in REVERSE order:
        // 1. ~scheduler_ → stop() called, workers exit
        // 2. ~actor_ → SAFE
    }
};
```

#### Unsafe Patterns (DO NOT USE)

```cpp
// WRONG: Actor destroyed while scheduler running
{
    auto actor = spawn<MyActor>(resource);
    auto [needs_sched, fut] = send(actor.get(), &MyActor::process, data);
    if (needs_sched) scheduler->enqueue(actor.get());
}  // CRASH: workers may call resume() on freed memory
scheduler->stop();

// WRONG: Destroy actor with pending futures
unique_future<int> future;
{
    auto actor = spawn<MyActor>(resource);
    auto [_, f] = send(actor.get(), &MyActor::slow_compute, 42);
    future = std::move(f);
}  // CRASH: Actor freed while slow_compute running
auto result = std::move(future).take_ready();  // Use-after-free
```

| Scenario | Safe? |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [otterbrix/actor-zeta](https://github.com/otterbrix/actor-zeta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
