---
trigger: always_on
description: libHttpClient is a **cross-platform C/C++ library** providing a platform abstraction layer for **HTTP** and **WebSocket** communication. It is used by Xbox Live Service API (XSAPI) and the PlayFab SDK.
---

# Copilot Instructions — libHttpClient

## Project Overview

libHttpClient is a **cross-platform C/C++ library** providing a platform abstraction layer for **HTTP** and **WebSocket** communication. It is used by Xbox Live Service API (XSAPI) and the PlayFab SDK.

- **Language:** C/C++ (C++17 standard required)
- **Public API:** Flat C API — all public functions use `HC*` prefix with C linkage (`STDAPI`)
- **Platforms:** Win32, UWP, GDK (Xbox/PC gaming), XDK (legacy Xbox One), iOS, macOS, Android, Linux
- **Build systems:** MSBuild (Windows), CMake (Linux/Android), Xcode (iOS/macOS)
- **CI:** Azure DevOps (see `Utilities/Pipelines/libHttpClient.CI.yml`)

## Architecture

```
Include/httpClient/    — Public C API headers (httpClient.h, httpProvider.h, mock.h, trace.h, async.h)
Include/               — XAsync.h, XAsyncProvider.h, XTaskQueue.h (async task queue API)
Source/
  Common/              — Shared utilities, types, Result<T>, error macros, pch, memory allocator
  Global/              — Global state, singleton, custom memory (mem.h)
  HTTP/                — Core HTTP call logic, retry, compression
    WinHttp/           — Win32/GDK HTTP provider (WinHTTP)
    XMLHttp/           — UWP HTTP provider
    Curl/              — Linux HTTP provider (libcurl)
    Android/           — Android HTTP provider (JNI bridge)
    Apple/             — iOS/macOS HTTP provider
  WebSocket/           — Core WebSocket logic
    Websocketpp/       — Linux WebSocket provider (websocketpp + asio)
    Android/           — Android WebSocket provider
  Mock/                — Mock HTTP/WebSocket layer for testing
  Platform/            — Platform initialization (PlatformComponents pattern)
    Win32/, UWP/, GDK/, XDK/, Android/, Apple/, Linux/, Generic/
  SSL/                 — SSL/TLS support
  Task/                — Async task infrastructure
  Logger/              — Logging/tracing
Build/                 — Platform-specific build projects (.vcxproj, CMakeLists.txt, .xcworkspace)
Tests/UnitTests/       — TAEF/TE unit tests
External/              — Git submodules: openssl, curl, websocketpp, asio, zlib
Samples/               — Sample apps (Win32, UWP, GDK)
Utilities/Pipelines/   — Azure DevOps CI pipeline definitions
```

### Platform Abstraction Pattern

Each platform implements `PlatformInitialize()` in `Source/Platform/<Platform>/PlatformComponents_<Platform>.cpp`, which creates platform-specific `IHttpProvider` and `IWebSocketProvider` implementations. Platform selection is controlled by:

- **MSBuild:** `platform_select.props` auto-detects based on `ApplicationType`/`Platform` → sets `HC_PLATFORM_MSBUILD_GUESS`
- **Code:** `HC_PLATFORM` preprocessor constant (e.g., `HC_PLATFORM_WIN32`, `HC_PLATFORM_GDK`, `HC_PLATFORM_ANDROID`)
- **Feature flags:** `HC_NOZLIB`, `HC_NOWEBSOCKETS` to exclude optional features

## Coding Conventions

### Naming

| Element | Convention | Example |
|---------|-----------|---------|
| Public C API functions | `HC` prefix, PascalCase | `HCHttpCallCreate()`, `HCWebSocketConnectAsync()` |
| Internal functions | camelCase | `ShouldRetry()`, `ResetResponseProperties()` |
| Member variables | `m_` prefix | `m_provider`, `m_refCount` |
| Types/Classes | PascalCase | `HC_CALL`, `WinHttpProvider`, `Result<T>` |
| Constants/Macros | UPPER_SNAKE_CASE | `MAX_DELAY_TIME_IN_SEC`, `HC_PLATFORM_WIN32` |
| File names | PascalCase or descriptive | `PlatformComponents_Win32.cpp`, `httpcall.h` |

### Namespaces

```cpp
NAMESPACE_XBOX_HTTP_CLIENT_BEGIN   // namespace xbox { namespace httpclient {
NAMESPACE_XBOX_HTTP_CLIENT_END     // }}
```

Sub-namespaces: `xbox::httpclient::log`, `xbox::httpclient::detail`, `xbox::httpclient::test`

### Error Handling

- All functions return `HRESULT` (S_OK on success)
- **No exceptions thrown** from public API — all functions are `noexcept` with `try {} CATCH_RETURN()` wrapping
- Use error macros from `Source/Common/ResultMacros.h`:
  - `RETURN_IF_FAILED(hr)` — early return on failure
  - `RETURN_HR_IF(hr, condition)` — conditional return
  - `RETURN_IF_NULL_ALLOC(ptr)` — returns E_OUTOFMEMORY if null
  - `LOG_IF_FAILED(hr)` — log without returning
- Internal result type: `Result<T>` (Source/Common/Result.h) — wraps HRESULT + optional payload + error message

### Memory Management

- Caller-controlled allocation via `HCMemSetFunctions()` callback
- Use custom allocator types from `Source/Common/Types.h`:
  - `http_internal_string`, `http_internal_vector<T>`, `http_internal_map<K,V>`
  - `HC_UNIQUE_PTR<T>`, `SharedPtr<T>`, `UniquePtr<T>`
  - `http_allocate_unique<T>(...)` for creating unique_ptr with custom allocator
- **Never use raw `new`/`delete`** — use `Make<T>()`/`Delete<T>()` from `Source/Global/mem.h`
- RAII patterns for all resource management

### Headers

- Use `#pragma once` (no traditional include guards)
- Include order: `pch.h` → own header → internal headers (quoted) → platform headers (angle brackets) → STL
- Public API headers use angle brackets: `#include <httpClient/httpClient.h>`
- Internal headers use quoted relative paths: `#include "HTTP/httpcall.h"`

### Other Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/libHttpClient](https://github.com/microsoft/libHttpClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
