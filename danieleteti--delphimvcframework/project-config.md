---
trigger: always_on
description: DelphiMVCFramework - RESTful framework for Delphi (Object Pascal). REST services, JSON-RPC APIs, web apps. MVC architecture, ORM (MVCActiveRecord), auth (JWT, Basic), middleware, WebSocket, serialization.
---

# CLAUDE.md

## Overview

DelphiMVCFramework - RESTful framework for Delphi (Object Pascal). REST services, JSON-RPC APIs, web apps. MVC architecture, ORM (MVCActiveRecord), auth (JWT, Basic), middleware, WebSocket, serialization.

**Language**: Object Pascal / Delphi
**Version**: `sources/dmvcframeworkbuildconsts.inc` (DMVCFRAMEWORK_VERSION) — currently `3.5.0-silicon-rc4`
**License**: Apache License 2.0

## Build Commands

```bash
# Tests — the matrix matters: the same suite runs against every host
python -m invoke tests                 # Win32+Win64, classic (WebBroker) server
python -m invoke tests32               # Win32 only
python -m invoke tests64               # Win64 only
python -m invoke tests-indydirect      # same suite, Indy Direct host
python -m invoke tests-httpsys         # same suite, HTTP.sys host
python -m invoke tests-isapi           # hosted by IIS Express ISAPI (Win64)
python -m invoke tests-apache          # hosted by Apache 2.4 module (Win64)
python -m invoke tests-all             # classic + Indy Direct + HTTP.sys
python -m invoke tests-all-hosts       # the full host matrix
python -m invoke --list                # everything else

build_tests.bat                        # Win32 unit tests (Delphi 13)

# Manual MSBuild
"C:\Program Files (x86)\Embarcadero\Studio\37.0\bin\rsvars.bat" && msbuild project.dproj /p:Config=DEBUG /p:Platform=Win32

# Packages & Samples
python -m invoke build-core
python -m invoke build-samples [--filter=name]

# Release & Utils
python -m invoke release [--skip-tests] [--skip-build]
python -m invoke generate-nullables
python -m invoke clean
```

**`invoke tests` returns exit code 0 even on BUILD FAIL** — read the output, do not trust the exit status.

## Project Structure

```
sources/              - Framework source (*.pas, *.inc)
packages/             - Packages by Delphi version (d100..d130)
lib/                  - Third-party (loggerpro, swagdoc, dmustache)
samples/              - 40+ sample applications
unittests/            - Tests (TestServer + TestClient)
tools/                - entitygenerator, certificatesgenerator
ideexpert/            - IDE wizard for project creation
  templates/          - TemplatePro .tpro templates
  tests/              - Wizard test suite
```

## Architecture

### The layering that matters

**Everything above the host is host-independent.** A controller, an action, an entity, a middleware does not
know what is listening on the socket. Only the `.dpr` differs between hosts. Keep host-specific code out of
`ConfigureEngine`.

### Server hosts (3.5)

Three backends, one controller stack, selected in the `.dpr` via `TMVCServerFactory` (`MVCFramework.Server.Factory`) returning `IMVCServer` (`.Server.Intf`):

| Host | Constructor | When |
|------|-------------|------|
| **Indy Direct** | `TMVCServerFactory.CreateIndyDirect(lEngine)` | **Default for new projects.** No WebModule, no WebBroker. |
| HTTP.sys | `TMVCServerFactory.CreateHttpSys(lEngine)` | Kernel-mode HTTP. Needs admin or a `netsh http add urlacl`. |
| WebBroker | `TMVCServerFactory.CreateWebBroker(AConfigAction, AEngineConfig)` or `TMVCEngine.CreateForWebBroker(AWebModule, AConfigProc)` | Deploying inside ISAPI / Apache / an existing WebBroker app. Supported indefinitely — **never mark WebBroker APIs deprecated.** |

`IMVCServer`: `Listen`, `Stop`, `IsRunning`, `RunAndWait`. HTTPS is configured **on `IMVCServer`** (certificate properties) — never set up TLS on the Indy component from the caller.
`MVCFramework.Signal` exports `WaitForTerminationSignal`; `EnterInShutdownState` / `IsShuttingDown` live in `MVCFramework`.

**`TMVCListener` is deprecated as of 3.5** (a thin wrapper over `IMVCServer`) and will be removed in 4.0. Do not write new code against it.

### Two routing styles, same engine

- **Controller-based**: attribute-routed methods on a `TMVCController`. Actions are **functions** returning data or `IMVCResponse` — never `procedure` + `Render(...)`.
- **Minimal API** (`MVCFramework.MinimalAPI`): lambda routes on a route group. `lEngine.Root` / `.Prefix(...)` → `TMVCRouteGroup<T>` (**a record — `Use`/`Prefix`/`AsWeb` return a new group; discarding the result is a silent no-op**), then `MapGet`/`MapPost`/`MapMethods`. Handlers are `function(...): IMVCResponse`, max 4 typed args, bound **by argument type**. Register classic middleware **before** the first `MapXxx` — the minimal dispatcher is installed lazily on the first `Map` and short-circuits matching routes.

### Key files

| Area | Files | Key Classes |
|------|-------|-------------|
| Core | `MVCFramework.pas`, `MVCFramework.Router.pas`, `MVCFramework.Commons.pas` | `TMVCEngine`, `TMVCController` |
| Hosting | `MVCFramework.Server.{Intf,Factory,Indy,HttpSys,WebBroker}.pas`, `.Signal.pas` | `IMVCServer`, `TMVCServerFactory` |
| Minimal API | `MVCFramework.MinimalAPI.pas` | `TMVCRouteGroup<T>`, `TMVCRouteHandle`, `TMVCFormFile` |
| Filters | `MVCFramework.Filters.pas` (+ `.Filters.Redis.pas`) | `TMVCEndpointFilter` (per-group), `TMVCHTTPFilter` (engine-wide) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danieleteti/delphimvcframework](https://github.com/danieleteti/delphimvcframework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
