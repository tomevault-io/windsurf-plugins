---
trigger: always_on
description: Woolball Server is a distributed AI inference network backend written in .NET 10. It orchestrates AI tasks (speech recognition, text-to-speech, translation, text generation) by routing them through a Redis Pub/Sub pipeline to browser-based worker nodes connected via WebSocket.
---

# CLAUDE.md

Woolball Server is a distributed AI inference network backend written in .NET 10. It orchestrates AI tasks (speech recognition, text-to-speech, translation, text generation) by routing them through a Redis Pub/Sub pipeline to browser-based worker nodes connected via WebSocket.

## Commands

```bash
# Build all projects (from woolball-server root)
dotnet build

# Build specific project
dotnet build src/WebApi/WebApi.csproj
dotnet build src/Background/Background.csproj
dotnet build src/WebSocket/WebSocket.csproj

# Run in Release mode
dotnet build --configuration Release

# Run with analyzers (used in CI)
dotnet build --configuration Release /p:RunAnalyzersDuringBuild=true

# Run tests with coverage
dotnet test --no-build /p:CollectCoverage=true /p:CoverletOutputFormat=opencover

# Start all services locally with Docker
docker compose up

# Build Docker images only (no push)
docker compose -f docker-compose.yml build core-api
docker compose -f docker-compose.yml build core-websocket
docker compose -f docker-compose.yml build core-background

# Redis local connection (dev password in appsettings)
# RedisConnection=localhost:6379,password=StrongRedisPassword123!
```

## Before Every Change

- Identify which project(s) are affected: WebApi (HTTP), WebSocket (WS nodes), Background (queues), or Core (shared logic)
- Run `dotnet build` to confirm no compile errors before and after your change
- Check if the change touches Redis channel names — there is an existing typo `"sesion_tracking_queue"` (missing 's') that is intentional/load-bearing; do not fix it without fixing both sides simultaneously
- Never add `IFormCollection`, `HttpContext`, or any `Microsoft.AspNetCore.Http` types to `Domain.csproj` — the domain already violates Clean Architecture by depending on ASP.NET; do not expand this violation
- `StrongRedisPassword123!` in `appsettings.json` and `docker-compose.yml` is a dev placeholder — never replace it with real credentials

## Decision Tree: Where to Look

| Working on... | Read first |
|---|---|
| HTTP endpoint routing or request handling | `src/Core/Presentation/EndPoints/TasksEndPoints.cs` |
| Task type validation, file upload, URL download | `src/Core/Domain/Contracts/Task/TaskRequest.cs` |
| Redis pub/sub publish logic | `src/Core/Application/Logic/TaskBusinessLogic.cs` |
| WebSocket node management (connect/disconnect) | `src/Core/Presentation/WebSockets/WebSocketNodesQueue.cs` |
| WebSocket node endpoint (accept connections) | `src/Core/Presentation/WebSockets/TaskSockets.cs` |
| Background queue workers | `src/Core/Presentation/Queues/` |
| AI model names and aliases | `src/Core/Domain/AIModels.cs` |
| Redis connection setup | `src/Core/Infrastructure/DependencyInjection.cs` |
| DI registration for logic layer | `src/Core/Application/DependencyInjection.cs` |
| DI registration for queues and WebSocket pool | `src/Core/Presentation/DependencyInjection.cs` |
| HTTP service startup and middleware | `src/WebApi/Program.cs` |
| WebSocket service startup | `src/WebSocket/Program.cs` |
| Background worker service startup | `src/Background/Program.cs` |
| Docker service definitions and ports | `docker-compose.yml` |
| CI/CD pipeline | `.github/workflows/CI-CD.yml` |
| STT streaming and chunk buffering | `src/Core/Application/Logic/SpeechToTextLogic.cs` |
| TTS streaming and chunk buffering | `src/Core/Application/Logic/TextToSpeechLogic.cs` |

## Architecture (3-5 sentences)

The server is split into three deployable services that share a Redis backbone: **core-api** (HTTP REST on port 9002) accepts task requests and awaits results via Redis Pub/Sub; **core-websocket** (WebSocket on port 9003) manages browser worker node connections using `WebSocketNodesQueue`; **core-background** runs `BackgroundService` queue workers that pre-process, split, and distribute tasks. All three services share a `Core` library containing `Domain`, `Application`, `Infrastructure`, and `Presentation` layers. The task lifecycle flows: HTTP request → `preprocessing_queue` → optional `split_audio_by_silence_queue` or `split_text_queue` → `distribute_queue` → WebSocket node → `result_queue_{taskId}` → HTTP response. `IConnectionMultiplexer` from StackExchange.Redis is registered as a singleton and used across all projects. All temp files land in `./shared/temp/` which is a Docker volume shared between `core-api` and `core-background`.

## Key Directories

| Path | Purpose |
|---|---|
| `src/WebApi/` | HTTP REST API service — startup, filters, `appsettings.json` |
| `src/WebSocket/` | WebSocket service — startup, `WebSocketExtensions.cs` |
| `src/Background/` | Background worker host — startup only, all logic in Core |
| `src/Core/Domain/` | Entities, contracts, task handlers, AI model constants |
| `src/Core/Domain/Contracts/Task/` | Per-task-type contracts (`SpeechToText/`, `TextGeneration/`, etc.) |
| `src/Core/Domain/Contracts/Task/TaskRequest.cs` | God file: `ITaskHandler`, `TaskHandlerFactory`, 4 concrete handlers, `TaskRequest` — 484 lines |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [woolball-xyz/woolball-server](https://github.com/woolball-xyz/woolball-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
