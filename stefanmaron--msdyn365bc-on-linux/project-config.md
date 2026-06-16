---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This project runs the **Microsoft Dynamics 365 Business Central service tier on Linux** by patching it at runtime. BC's NST is a .NET 8 application that Microsoft only ships for Windows; we make it run unmodified on Linux via a `DOTNET_STARTUP_HOOKS` assembly that intercepts Win32 P/Invokes, stubs Windows-only services, and rewrites a handful of methods that hard-depend on Windows. SQL Server runs in a separate Linux container (`mssql/server:2022`).

The code here is **not a fork of BC**. We never recompile Microsoft assemblies — the BC service tier DLLs are downloaded fresh from Microsoft artifact storage at container start, then the startup hook patches them in memory (with a few binary patches written to disk for assemblies the JIT can't reach, e.g. `CodeAnalysis.dll`, `Mono.Cecil.dll`).

## Build, run, test

```bash
# Build + start (first boot ~5–10 min: artifact download + DB restore + extension compile)
docker compose up -d --wait

# Override version/country/type
BC_VERSION=28.0 BC_COUNTRY=de docker compose up -d --wait

# Rebuild image after editing src/StartupHook or src/stubs
docker compose build bc

# Logs (BC writes everything to stderr — entrypoint redirects 1>&2 for unbuffered output)
docker compose logs -f bc

# Tear down (keep artifact cache)
docker compose down
# Tear down + wipe cached artifacts and service dir
docker compose down -v
```

Multiple parallel instances: use `-p <project>` with a unique port offset for every published port (see README.md "Running Multiple Instances"). Forgetting one port causes a bind conflict.

### Running AL tests

```bash
# Publish a test app then run a codeunit range (per-method results require --app)
./scripts/run-tests.sh --app MyTestApp.app --codeunit-range 50000..50100

# Single codeunit
./scripts/run-tests.sh --app MyTestApp.app --codeunit-range 50000
```

`run-tests.sh` is a hybrid OData (suite population + result reading) + WebSocket (test execution via a real client session) flow. The WebSocket step is required because TestPage support needs a `serviceConnection`-style session, which OData can't provide. The test runner extension is in `extensions/TestRunnerExtension/` (AL source under `src/`); the prebuilt `.app` is baked into the image and republished automatically on container start.

**EXPERIMENTAL altool runner (BC 28+ only):** `scripts/run-tests-altool.py` runs tests through the AL dotnet tool's native `al runtests` command (Microsoft.Dynamics.BusinessCentral.Development.Tools, 18.x prerelease — stable 17.x has no `runtests`), which drives the NST's built-in SignalR hub at `/dev/TestRunnerHub`. No TestRunnerExtension, no OData suite, no WebSocket emulation — the server pushes per-method results (status, output, duration) over the hub. Requires the server to advertise Dev API 7.0 (`GET /BC/dev/metadata`), which only exists in BC 28.0+. Caveats: tests do NOT run under an AL test runner codeunit (no AI tests, no test-runner setup/teardown events, isolation from `RequiredTestIsolation`, default Codeunit) — so Microsoft BCApps suites may behave differently than under `run-tests.sh`; the test app must already be published+installed (the script doesn't publish). The reusable workflow's `test_runner` input defaults to `auto`: after BC is healthy it probes `GET /dev/metadata` (via `run-tests-altool.py --probe`, exit 0 = Dev API ≥ 7.0) and uses the altool runner when supported, falling back to the websocket runner otherwise — so 27.x legs and consumers on older versions keep working unchanged. `websocket` forces the legacy flow; `altool` forces the hub and fails hard when unsupported (the regression-detection mode). `altool_version` pins the dotnet tool. Auth comes from `BC_SERVER_USERNAME`/`BC_SERVER_PASSWORD` env vars, which the script sets from `--auth`. The script's stdout deliberately prints the same `N total, P passed, F failed` and `Test codeunits: ...` lines the workflow parser greps — keep that contract if you touch either side.

### Editing the startup hook

The hook is a normal .NET 8 class library:

```bash
cd src/StartupHook && dotnet build -c Release
# Then rebuild the image — nothing on the host runs the hook directly
docker compose build bc && docker compose up -d --wait
```

`kernel32_stubs.c` is compiled to `libwin32_stubs.so` inside the image (gcc is installed in the builder stage). If you add a new exported symbol, also wire it into the `NativeLibrary.SetDllImportResolver` registration in `StartupHook.cs` (Patch #3).

## Architecture

### Layers

1. **`docker-compose.yml`** — two services (`sql`, `bc`). SQL uses a tmpfs for `/var/opt/mssql/data` (4 GB) so first-boot DB restore is fast; the cost is that DB state is wiped on container restart. The `bc` service depends on `sql` being healthy and exposes the dev/OData/API/SOAP/client ports (7045–7089 range).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StefanMaron/MsDyn365Bc.On.Linux](https://github.com/StefanMaron/MsDyn365Bc.On.Linux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
