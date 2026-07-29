---
trigger: always_on
description: This document defines mandatory rules and invariants for any agent modifying the Chromium Embedded Framework (CEF) integration in this repository.
---

# AGENTS.md - CEF Integration Safety & Engineering Contract

This document defines mandatory rules and invariants for any agent modifying the Chromium Embedded Framework (CEF) integration in this repository.

The codebase uses CEF C API dynamically loaded via `dlopen` and embeds Chromium into an AppKit-based macOS application with single-threaded message loop mode.

CEF is extremely sensitive to:

- thread correctness
- object lifetimes
- message loop configuration
- ABI compatibility
- shutdown ordering
- process architecture

Violating these rules will frequently cause crashes inside `cef_do_message_loop_work()`, renderer process failures, or hard-to-debug memory corruption.

This file exists to ensure no agent ever repeats those mistakes.

## 1. Core Architecture Overview

The CEF runtime in this repository follows this architecture:

```text
Host App (AppKit)
        |
        v
CEF Bridge (Objective-C++ / C++)
        |
        v
Dynamically loaded CEF Framework
        |
        v
CEF Browser Process
        |
        |- Renderer Process
        |- GPU Process
        `- Utility Processes
```

Key facts:

- The host app is AppKit-only (no SwiftUI).
- CEF runs in single-threaded message loop mode.
- The host app manually drives Chromium via `cef_do_message_loop_work()`.
- The CEF framework is loaded dynamically via `dlopen`.

## 2. ABSOLUTE RULES (DO NOT BREAK)

These are hard invariants.
Any violation will likely crash Chromium.

### Rule 1 - ALL CEF API calls must run on the CEF UI thread

In this integration the CEF UI thread is the macOS main thread.

Therefore:

- All CEF object usage must run on the main thread.

Examples of APIs that MUST run on main thread:

- `cef_browser_host_create_browser_sync`
- `cef_browser_t` methods
- `cef_frame_t` methods
- `cef_browser_host_t` methods
- `cef_process_message_create`
- `cef_shutdown`
- `cef_initialize`
- `cef_execute_process`
- `cef_do_message_loop_work`

If code touches any of these types:

- `cef_browser_t`
- `cef_frame_t`
- `cef_browser_host_t`
- `cef_process_message_t`

it must execute on the main thread.

Enforce using:

- `runOnCefMainThread(...)`

Never call these from background threads.

### Rule 2 - NEVER use CEF objects across threads

CEF objects are not thread-safe.

Example of a bug:

```cpp
std::thread {
    browser->reload(browser);
}
```

This will crash.

Instead:

```cpp
runOnCefMainThread(^{
    browser->reload(browser);
});
```

### Rule 3 - Never unload the CEF framework while callbacks may run

This is one of the most dangerous bugs.

If `dlclose()` is called while CEF callbacks may still execute:

- CEF may call function pointers inside unloaded code
- crash occurs inside `cef_do_message_loop_work`

Correct sequence:

1. close all browsers
2. wait for browser close completion
3. call `cef_shutdown()`
4. only then `dlclose()`

Agents must never call `dlclose()` while browsers exist.

### Rule 4 - Never access `gCefApi` without synchronization

The global function table:

```cpp
CefApi gCefApi;
```

is mutated during:

- shutdown
- framework unload
- test reset

Access must follow this pattern:

```cpp
CefDoMessageLoopWorkFn fn;

{
    std::lock_guard lock(gStateLock);
    fn = gCefApi.doMessageLoopWork;
}

if (fn) {
    fn();
}
```

Never do:

```cpp
gCefApi.doMessageLoopWork()
```

directly.

### Rule 5 - NEVER mix message loop modes

CEF has three modes:

- `multi_threaded_message_loop`
- `external_message_pump`
- manual `cef_do_message_loop_work`

This project uses:

- `multi_threaded_message_loop = 0`
- `external_message_pump = 0` by default
- `external_message_pump = 1` only behind the external-pump bridge path

and one scheduling mode at a time.

Therefore:

Agents must never enable:

- `external_message_pump = 1`

unless they also implement:

- `cef_browser_process_handler_t::OnScheduleMessagePumpWork`
- a browser-process `cef_app_t` wrapper that stays alive until shutdown
- host-side cancellation so manual pumping is disabled while external pump mode is active

### Rule 6 - CEF shutdown must happen after ALL browsers close

Shutdown ordering must be:

1. create browser
2. use browser
3. close browser
4. wait for browser invalid
5. release browser refs
6. `cef_shutdown()`
7. unload framework

Agents must never call `cef_shutdown()` while any browser still exists.

### Rule 7 - Browser replacement is dangerous

Replacing a browser instance:

- `oldBrowser -> newBrowser`

is a major lifecycle risk.

Agents should prefer:

- rebind host view
- resize host view
- reuse browser

instead of destroying and recreating.

Replacement should be rare and intentional.

### Rule 8 - CEF callbacks may occur during shutdown

Callbacks such as:

- `on_address_change`
- `on_title_change`
- `on_favicon_urlchange`
- `on_load_end`

may fire while shutdown is happening.

All callbacks must assume:

- browser may already be closing
- state may be partially torn down

Always guard lookups.

### Rule 9 - Never hold bridge locks while calling into CEF

Do not hold `gStateLock` while invoking:

- any `cef_*` object method
- any function pointer from `gCefApi`

Correct pattern:

1. acquire lock
2. copy plain state / retain refs / copy function pointer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [navigator-zip/navigator](https://github.com/navigator-zip/navigator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
