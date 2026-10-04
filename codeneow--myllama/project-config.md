---
trigger: always_on
description: MyLlama is a cross-platform local LLM management app built on **Wails v3** (v3.0.0-beta.16, pinned in `go.mod`; Go 1.25 backend) — one Go + Vue codebase targeting **Windows** (WebView2), **Linux** (WebKitGTK — GTK4 stack by default, `gtk3`-tagged variant for the released `.deb`s), **macOS** (universal binary, Metal on arm64) and **Android** (system WebView, phone-tier UI). The frontend uses **Vue 3 + TypeScript + Vite 5** (no third-party UI library, hand-written CSS variable theming). The infere
---

# MyLlama Development Guidelines

## Project Overview

MyLlama is a cross-platform local LLM management app built on **Wails v3** (v3.0.0-beta.16, pinned in `go.mod`; Go 1.25 backend) — one Go + Vue codebase targeting **Windows** (WebView2), **Linux** (WebKitGTK — GTK4 stack by default, `gtk3`-tagged variant for the released `.deb`s), **macOS** (universal binary, Metal on arm64) and **Android** (system WebView, phone-tier UI). The frontend uses **Vue 3 + TypeScript + Vite 5** (no third-party UI library, hand-written CSS variable theming). The inference engine is **llama.cpp**: desktop platforms run llama-server in router mode (lazy load, multi-model, one OpenAI endpoint); Android runs direct mode — a single resident model started via the `StartServerWithModel` binding with GPU-specific flags stripped by `core/preset.go`'s `modelDirectArgs`.

Core pipeline: scan `LLM-Models/` for GGUF files → configure per-model inference parameters → generate llama-server model presets (INI) → launch an OpenAI-compatible service (default `127.0.0.1:8080`).

An optional API-route (headless) mode (Windows) relaunches the app as a pure background process — Go backend + system tray + llama-server, no GUI — with the llama-server process kept running uninterrupted across GUI ↔ headless switches (see `core/headless.go` / `core/handover.go`).

## Common Commands

Run these from the repository root:

```bash
go install github.com/wailsapp/wails/v3/cmd/wails3@v3.0.0-beta.16   # Wails v3 CLI (keep the pinned version in sync with go.mod)
wails3 task dev                 # Dev loop: Go backend + Vite frontend (:5173) hot-reload; flow defined in build/config.yml
wails3 task build               # Production desktop build; output at build/bin/MyLlama.exe (build/ is gitignored)
wails3 task build:frontend      # Frontend only: regenerates bindings, then vue-tsc type-check + vite build into frontend/dist
wails3 task android:package     # Android arm64 APK into build/bin/ (run build:frontend first; needs JDK 17 + Android SDK/NDK)
cd frontend && npm run build    # Frontend only (vue-tsc type-check + vite build; does NOT regenerate bindings)
cd frontend && npm run dev      # Vite only (without the Wails runtime, backend calls fail at fetch time; see below)
cd frontend && npm run dev:mock # Vite with the in-repo mock runtime: full UI in a plain browser, no backend needed
```

Quality gates (run as needed after changes; see "Pre-commit Test Tiers"):

```bash
go build ./...                                   # Backend compilation
go test ./...                                    # Backend unit tests (standard library testing)
gofmt -l .                                       # Go formatting check; must produce no output
golangci-lint run                                # Go static analysis (govet / ineffassign / unused)
cd frontend && npm run build                     # Frontend type-check + build (vue-tsc --noEmit zero errors)
cd frontend && npm test                          # Frontend unit tests (vitest)
make check                                       # Combined quality gate (POSIX)
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\check.ps1  # Combined quality gate (Windows)
```

> The frontend artifact `frontend/dist` is a compile-time dependency embedded via `go:embed` (gitignored). If the frontend has not been built locally, run `npm run build` or `make check-frontend` before running backend quality gates.

## Architecture and Code Navigation

| File | Responsibility |
| --- | --- |
| `main.go` | Wails v3 entry point: `application.New` with the bound `core.App` service and the embedded `frontend/dist` asset handler, single frameless window "main" (1200×800, min 900×600), tray icon embed exposed to core via `core.TrayIcon`, `--headless` / `--gui` mode flags (API-route headless mode vs forced GUI) and the single-instance mutex; lifecycle lives in `core.App.ServiceStartup` / `ServiceShutdown` (the v3 equivalents of OnStartup / OnShutdown) |
| `main_android.go` | Android entry hook (`//go:build android`): its `init()` registers `main()` with the Wails Android runtime (`application.RegisterAndroidMain`) — a `-buildmode=c-shared` build never calls `main()` on its own, the Java bridge invokes the JNI `nativeInit` export instead |
| `Taskfile.yml` + `build/config.yml` | Wails v3 task runner: `dev` (flow defined in build/config.yml: build:go:dev → dev:frontend → wait:frontend gate → run:dev), `build` (bindings + frontend + production go build with windowsgui ldflag and the branded `.syso` icon), `generate:bindings`, and the `android:*` tasks (NDK clang c-shared `libwails.so` + gradle APK packaging); the Vite dev port comes from the `WAILS_VITE_PORT` env var, default 5173 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CodeNeow/MyLlama](https://github.com/CodeNeow/MyLlama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
