---
trigger: always_on
description: ClawBench is a mobile-first AI workstation wrapping AI CLI tools (CodeBuddy, Claude Code, OpenCode, Codex, Qoder CLI, VeCLI, CodeWhale, MiMo-Code, Pi, Cline, Copilot, Kimi) into a web-accessible platform. Go backend shells out to CLI tools and streams JSON output via WebSocket; Vue 3 frontend renders the streamed events in real time. Supports ACP (Agent Client Protocol) stdio transport for agents with native or bridge-adapter support, providing structured mode switching, slash commands, and perm
---

# AGENTS.md

## Project Overview

ClawBench is a mobile-first AI workstation wrapping AI CLI tools (CodeBuddy, Claude Code, OpenCode, Codex, Qoder CLI, VeCLI, CodeWhale, MiMo-Code, Pi, Cline, Copilot, Kimi) into a web-accessible platform. Go backend shells out to CLI tools and streams JSON output via WebSocket; Vue 3 frontend renders the streamed events in real time. Supports ACP (Agent Client Protocol) stdio transport for agents with native or bridge-adapter support, providing structured mode switching, slash commands, and permission management. Also supports SSH tunnel-based port forwarding for remote/mobile access and a scheduled task (cron) system for recurring AI execution.

## Build & Run Commands

```bash
./build.sh                # Full build (Go binary + Vue frontend)
./build.sh --windows      # Cross-compile: Windows amd64
./build.sh --linux        # Cross-compile: Linux amd64
./build.sh --darwin       # Cross-compile: macOS arm64

./dev-server.sh           # Dev mode (Vite HMR proxy to production backend's dev HTTP port)
./dev-server.sh --fg      #   foreground
./dev-server.sh --stop    #   stop
./dev-server.sh --restart #   restart

./clawbench               # Run directly (foreground, default port 20000)
./clawbench --port 8080   #   specify port
./clawbench --data-dir /data/.clawbench  #   custom data directory

go build -o clawbench ./cmd/server   # Go binary only
go test ./...                        # All Go tests
go test ./internal/ai/...            # Package-specific
npm test                             # Vitest (all frontend tests)

# Coverage gate (CI 合入门槛)
./scripts/check-go-coverage.sh              # Go: run tests + check per-package coverage
./scripts/check-go-coverage.sh --skip-test   # Go: reuse existing coverage.out
./scripts/check-go-coverage.sh --update      # Go: auto-update baseline after coverage improvement
./scripts/check-frontend-coverage.sh              # Frontend: run tests + check per-dir coverage
./scripts/check-frontend-coverage.sh --skip-test   # Frontend: reuse existing coverage data
./scripts/check-frontend-coverage.sh --update      # Frontend: auto-update baseline after improvement
./scripts/check-android-coverage.sh              # Android: run tests + check per-class coverage
./scripts/check-android-coverage.sh --skip-test   # Android: reuse existing JaCoCo report

# Android APK (requires JDK 17)
cd android && JAVA_HOME=/usr/lib/jvm/jdk-17.0.12 ./gradlew assembleDebug    # Debug APK
cd android && JAVA_HOME=/usr/lib/jvm/jdk-17.0.12 ./gradlew assembleRelease  # Release APK
```

## Architecture

### Backend (Go)

**Entry point:** `cmd/server/main.go` — config → port → LoadAgents → SyncDiscoverAgents → SyncDiscoverModels → MergeDiscoveredData → AsyncRefreshModelCache → scheduler init.

**Packages:**
- `internal/handler/` — HTTP endpoints. All `/api/` routes use `middleware.Auth` (localhost bypass for CLI). Chat streaming via WebSocket (`/api/ai/events/ws`).
- `internal/service/` — Business logic: chat persistence, auto-summary, scheduler, SQLite, versioned schema migration, agent store (DB-backed), API key encryption (AES-256-GCM), default project persistence (`is_default` column in `recent_projects`). `SessionExecutor` unifies AI session execution for both chat and scheduled tasks.
- `internal/ai/` + `internal/ai/backends/` — AI backend abstraction and plugin system. `AIBackend` interface → `CLIBackend` (CLI args + LineParser) → `AutoResumeBackend` (ExitPlanMode → cancel → resume) → `ACPBackend` (JSON-RPC over stdio, connection pool). 12 backend sub-packages (claude, cline, codebuddy, copilot, codex, qoder, vecli, deepseek, kimi, mimo, opencode, pi), each registering via `ai.RegisterBackend()` in `init()`. `all.go` aggregates imports for `main.go`. ACP mapping wired by `backends/acp_wire.go`. `acpStdoutFilter` fixes JSON-RPC protocol violations (string-number ID mismatch, non-JSON stdout lines). CodeWhale (registered as `"deepseek"`, CLI: `codewhale`) has ACP support with remaps and stdout filter. Pi has ACP bridge support via `@touchtechclub/pi-acp`. `BackendSpec.AltCmd` for fallback CLI detection.
- `internal/model/` — Data models, `BackendRegistry` (backend specs + model discovery), `ProviderRegistry` (28 LLM providers).
- `internal/cli/` — AI agent self-service: `task`, `rag`, `migrate`.
- `internal/middleware/` — Auth, request logging, panic recovery, request ID.
- `internal/platform/` — Cross-platform path resolution, shell detection, Windows CLI utilities.
- `internal/speech/` — TTS: MiniMax, Edge TTS (native Go), Piper/Kokoro/MOSS-Nano.
- `internal/summarize/` — Text summarization for auto-summary, TTS, task summaries.
- `internal/ssh/` — SSH tunnel server. Publishes `tunnel_status` via EventBus.
- `internal/proxy/` — HTTP reverse proxy + port forwarding. Rewrites Host header for virtual-host backends.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawbench-dev/clawbench](https://github.com/clawbench-dev/clawbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
