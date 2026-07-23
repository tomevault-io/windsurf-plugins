---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This project is the Sentry SDK for Unreal Engine which provides crash and error monitoring as well as performance tracking for games built with Unreal Engine. It wraps multiple Sentry SDKs for supported platforms (`sentry-native` for Windows/Linux/macOS/Xbox/WinGDK/PlayStation/Nintendo, `sentry-cocoa` for macOS/iOS, `sentry-java` for Android) into a single Unreal Engine plugin and extends them with Unreal Engine-specific functionality.

## Plugin Architecture

### Plugin Modules

1. **Sentry** - Main plugin API with platform-specific implementations (Runtime module)
2. **SentryEditor** - Editor utilities, settings UI, build tools (Editor module)

### Plugin Source Structure

```
plugin-dev/Source/
├── Sentry/
│   ├── Public/              # Public API headers (SentrySubsystem.h, SentrySettings.h, SentryLibrary.h)
│   └── Private/
│       ├── Android/         # Android implementation (sentry-java via JNI)
│       ├── Apple/           # Base for Mac and iOS (sentry-cocoa)
│       ├── GenericPlatform/ # Base for Microsoft, Linux, PS, Switch (sentry-native)
│       ├── HAL/             # Hardware Abstraction Layer
│       ├── Interface/       # Abstract interfaces
│       ├── IOS/             # iOS overrides for Apple
│       ├── Linux/           # Linux overrides for GenericPlatform
│       ├── Mac/             # macOS overrides for Apple/GenericPlatform
│       ├── Microsoft/       # Windows/Xbox/WinGDK base, overrides GenericPlatform
│       ├── Null/            # Stubs for unsupported platforms
│       ├── Performance/     # Performance monitors (frame time, GC, network, game stats)
│       ├── SessionReplay/   # Session replay recording (backbuffer capture, MP4 encoder)
│       ├── Tests/           # Unit test specs
│       ├── Utils/           # Common utilities
│       └── Windows/         # Windows overrides for Microsoft
├── SentryEditor/            # Editor utilities
└── ThirdParty/              # Pre-built platform SDKs
```

### Platform Hierarchy

```
sentry-native (C/C++ SDK)
├── GenericPlatform/          # Base implementation for all sentry-native platforms
│   ├── Microsoft/            # Shared Windows + Xbox + WinGDK code
│   │   ├── Windows/          # Windows-specific overrides
│   │   └── Xbox/             # Xbox-specific overrides (private extension)
│   │   └── WinGDK/           # WinGDK-specific overrides (private extension)
│   ├── Mac/                  # macOS-specific overrides (native backend)
│   ├── Linux/                # Linux-specific overrides
│   ├── PlayStation/          # PlayStation-specific (private extension)
│   └── Switch/               # Nintendo Switch-specific (private extension)

sentry-cocoa (Objective-C SDK)
├── Apple/                    # Base implementation for all Apple platforms
│   ├── Mac/                  # macOS-specific overrides (cocoa backend)
│   └── IOS/                  # iOS-specific overrides

sentry-java (Java/Kotlin SDK)
└── Android/                  # Android implementation (JNI interop)

Null/                         # Stubs for unsupported platforms
```

### Crash Handler Backends

For `sentry-native` platforms SDK supports multiple crash handler backends. The backend is selected in `Sentry.Build.cs` at build time via the `UseNativeBackend` setting configured in `SentrySettings`.

| Platform             | Default backend | Alternative backend |
|----------------------|-----------------|---------------------|
| Windows (x64, ARM64) | Crashpad        | Native              |
| Linux (x64, ARM64)   | Crashpad        | Native              |
| macOS                | Cocoa           | Native              |
| Xbox/WinGDK          | Breakpad        | Native              |

Each platform's ThirdParty directory contains subdirectories per backend (e.g. `Crashpad/` + `Native/`, `Cocoa/` + `Native/`, or `Breakpad/` + `Native/`).

### Public API

**Core:**
- `SentrySubsystem` - Main SDK entry point (initialize, capture events, manage scope)
- `SentrySettings` - Plugin configuration (DSN, options, feature toggles)
- `SentryLibrary` - Blueprint function library for instantiating Sentry objects

**Sentry Entities:**
- `SentryEvent`, `SentryBreadcrumb`, `SentryScope`, `SentryUser`, `SentryAttachment`, `SentryHint`, `SentryFeedback`, `SentryMetric`, `SentryLog` - Data objects representing Sentry concepts

**Tracing:**
- `SentryTransaction`, `SentrySpan`, `SentryTransactionContext`, `SentrySamplingContext` - Tracing and performance measurement

**Callbacks:**
- `SentryBeforeSendHandler`, `SentryBeforeBreadcrumbHandler`, `SentryBeforeLogHandler`, `SentryBeforeMetricHandler`, `SentryTraceSampler` - Hooks for customizing SDK behavior

**Utilities:**
- `SentryVariant` - Universal value type for passing data to Sentry APIs (tags, context, etc.)
- `SentryOutputDevice` / `SentryErrorOutputDevice` - UE log output integration

### Platform Implementation Pattern

- Public API is platform-agnostic
- Platform-specific implementations in `Private/{Platform}/` directories
- Abstract interfaces in `Interface/` define contracts
- Wrapper classes translate platform-specific SDKs to Unreal APIs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsentry/sentry-unreal](https://github.com/getsentry/sentry-unreal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
