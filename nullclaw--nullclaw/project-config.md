---
trigger: always_on
description: This file defines the default working protocol for coding agents in this repository.
---

# AGENTS.md — nullclaw Agent Engineering Protocol

This file defines the default working protocol for coding agents in this repository.
Scope: entire repository.

## 1) Project Snapshot (Read First)

nullclaw is a Zig-first autonomous AI assistant runtime optimized for:

- minimal binary size (target: < 1 MB ReleaseSmall)
- minimal memory footprint (target: < 5 MB peak RSS)
- zero dependencies beyond libc and optional SQLite
- full feature parity with ZeroClaw (Rust reference implementation)

Core architecture is **vtable-driven** and modular. All extension work is done by implementing
vtable structs and registering them in factory functions.

Key extension points:

- `src/providers/root.zig` (`Provider`) — AI model providers
- `src/channels/root.zig` (`Channel`) — messaging channels
- `src/tools/root.zig` (`Tool`) — tool execution surface
- `src/memory/root.zig` (`Memory`) — memory backends
- `src/observability.zig` (`Observer`) — observability hooks
- `src/runtime.zig` (`RuntimeAdapter`) — execution environments
- `src/peripherals.zig` (`Peripheral`) — hardware boards (Arduino, STM32, RPi)

Current scale: **245 source files, ~204K lines of code, 5,640+ tests**.

Build and test:

```bash
zig build                           # dev build
zig build -Doptimize=ReleaseSmall  # release build
zig build test --summary all        # run all tests
```

## 2) Deep Architecture Observations (Why This Protocol Exists)

These codebase realities should drive every design decision:

1. **Vtable + factory architecture is the stability backbone**
   - Extension points are explicit and swappable via `ptr: *anyopaque` + `vtable: *const VTable`.
   - Callers must OWN the implementing struct (local var or heap-alloc). Never return a vtable interface pointing to a temporary — the pointer will dangle.
   - Most features should be added via vtable implementation + factory registration, not cross-cutting rewrites.

2. **Binary size and memory are hard product constraints**
   - `zig build -Doptimize=ReleaseSmall` is the release target. Every dependency and abstraction has a size cost.
   - Avoid adding libc calls, runtime allocations, or large data tables without justification.
   - `MaxRSS` during `zig build test` must stay well under 50 MB.

3. **Security-critical surfaces are first-class**
   - `src/gateway.zig`, `src/security/`, `src/tools/`, `src/runtime.zig` carry high blast radius.
   - Defaults are secure-by-default (pairing, HTTPS-only, allowlists, AEAD encryption). Keep it that way.

4. **Zig 0.16.0 API is the baseline — no newer features**
   - HTTP client: `std.http.Client.fetch()` with `std.Io.Writer.Allocating` for response body capture.
   - Child processes: `std.process.Child.init(argv, allocator)`, `.Pipe` (capitalized).
   - stdout: `std.fs.File.stdout().writer(&buf)` → use `.interface` for `print`/`flush`.
   - `std.io.getStdOut()` does NOT exist in the current Zig stdlib — use `std.fs.File.stdout()`.
   - SQLite: linked via `/opt/homebrew/opt/sqlite/{lib,include}` on the compile step, not the module.
   - `ArrayListUnmanaged`: init with `.empty`, pass allocator to every method.

5. **All 5,640+ tests must pass at zero leaks**
   - The test suite uses `std.testing.allocator` (leak-detecting GPA). Every allocation must be freed.
   - `Config.load()` allocates — always wrap in `std.heap.ArenaAllocator` in tests and production.
   - `ChaCha20Poly1305.decrypt` can segfault on tag failure with heap-allocated output on macOS with older Zig toolchains — use a stack buffer then `allocator.dupe()`.

## 3) Engineering Principles (Normative)

These principles are mandatory. They are implementation constraints, not suggestions.

### 3.1 KISS

Required:
- Prefer straightforward control flow over meta-programming.
- Prefer explicit comptime branches and typed structs over hidden dynamic behavior.
- Keep error paths obvious and localized.

### 3.2 YAGNI

Required:
- Do not add config keys, vtable methods, or feature flags without a concrete caller.
- Do not introduce speculative abstractions.
- Keep unsupported paths explicit (`return error.NotSupported`) rather than silent no-ops.

### 3.3 DRY + Rule of Three

Required:
- Duplicate small local logic when it preserves clarity.
- Extract shared helpers only after repeated, stable patterns (rule-of-three).
- When extracting, preserve module boundaries and avoid hidden coupling.

### 3.4 Fail Fast + Explicit Errors

Required:
- Prefer explicit errors for unsupported or unsafe states.
- Never silently broaden permissions or capabilities.
- In tests: `builtin.is_test` guards are acceptable to skip side effects (e.g., spawning browsers), but the guard must be explicit and documented.

### 3.5 Secure by Default + Least Privilege

Required:
- Deny-by-default for access and exposure boundaries.
- Never log secrets, raw tokens, or sensitive payloads.
- All outbound URLs must be HTTPS. HTTP is rejected at the tool layer.
- Keep network/filesystem/shell scope as narrow as possible.

### 3.6 Determinism + No Flaky Tests

Required:
- Tests must not spawn real network connections, open browsers, or depend on system state.
- Use `builtin.is_test` to bypass side effects (spawning, opening URLs, real hardware I/O).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nullclaw/nullclaw](https://github.com/nullclaw/nullclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
