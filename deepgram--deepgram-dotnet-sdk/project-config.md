---
trigger: always_on
description: Instructions for AI coding agents (Claude Code, Cursor, Codex, Copilot) and for humans working with them in this repository. `CLAUDE.md` includes this file.
---

# Agents

Instructions for AI coding agents (Claude Code, Cursor, Codex, Copilot) and for humans working with them in this repository. `CLAUDE.md` includes this file.

## Repository purpose

This is the official .NET SDK for the Deepgram API, published to NuGet as `Deepgram` (the SDK) and `Deepgram.Microphone` (a capture helper). The latest release tag is `7.1.1`. Both packages target `net8.0` and `netstandard2.0`. The SDK is hand-written: there is no code generator, no `fern/` folder, and no `.fernignore`. Edit the source directly.

The package version is not stored in the repository. `Deepgram/Deepgram.csproj` has no `<Version>` element; the CD workflow passes the git tag as `-p:Version=<tag>` at pack time.

Never hardcode API keys or access tokens. Every client constructor and every `ClientFactory.Create*` method takes an optional `apiKey` and falls back to the `DEEPGRAM_API_KEY` environment variable (`DEEPGRAM_ACCESS_TOKEN` for a bearer token). Examples and tests rely on that fallback.

## Repository map

| Path | What lives there |
| --- | --- |
| `Deepgram/` | The SDK project. Top-level `*Client.cs` files are thin public entry points; the implementations live under `Clients/` |
| `Deepgram/Clients/<Product>/<clientVersion>/{REST,WebSocket}` | Client implementations. The folder version is the SDK client version, which is distinct from the API version (see the surfaces table) |
| `Deepgram/Clients/Flux/` | Flux STT (`WebSocket/`, `/v2/listen`) and Flux TTS (`Speak/REST`, `Speak/WebSocket`, `/v2/speak`) clients |
| `Deepgram/Clients/Interfaces/{v1,v2}` | Public client interfaces (`IListenRESTClient`, `IFluxSpeakWebSocketClient`, and so on) |
| `Deepgram/Models/<Product>/<version>/...` | Request schemas and response types; namespaces mirror the path (`Deepgram.Models.Listen.v1.REST`, `Deepgram.Models.Flux.Speak.WebSocket`) |
| `Deepgram/Abstractions/` | `AbstractRestClient` and `AbstractWebSocketClient`, the shared HTTP and WebSocket plumbing |
| `Deepgram/ClientFactory.cs` | Static factory that returns the current client for each product; `Library.cs` configures logging (`Library.Initialize()`) |
| `Deepgram/Constants/Defaults.cs` | `api.deepgram.com`, API version `v1`, and the environment variable names |
| `Deepgram.Tests/` | NUnit 4 unit tests (`UnitTests/`), fakes, and recorded fixtures (`Fixtures/Flux`, `Fixtures/FluxSpeak`) |
| `Deepgram.Microphone/` | The `Deepgram.Microphone` helper package (PortAudio) |
| `examples/` | One console project per scenario, grouped by product |
| `tests/edge_cases/`, `tests/expected_failures/` | Console programs run by hand against the live API to reproduce reconnect, keepalive, timeout, and error paths |
| `extras/live-smoke/` | Dockerized live smokes for Flux STT `ForceEndTurn` and the Voice Agent management endpoints; see its `README.md` |
| `.github/workflows/` | `CI.yml`, `tests-daily.yml`, `CD.yml`, `CD-dev.yml`, `context7.yml` |
| `.github/` | `CONTRIBUTING.md`, `CODE_CONTRIBUTIONS_GUIDE.md`, `GITHUB_WORKFLOW.md`, `BRANCH_AND_RELEASE_PROCESS.md`, `PULL_REQUEST_TEMPLATE.md` |
| `.agents/skills/` | Agent-agnostic skills for using this SDK (speech-to-text, conversational STT, text-to-speech, voice agent, audio intelligence, text intelligence, management API) |

Three solution files exist. `Deepgram.sln` holds only `Deepgram`, `Deepgram.Tests`, and `Deepgram.Microphone`; CI builds and tests it. `Deepgram.DevBuild.sln` holds the two packages and packs them as `Deepgram.Unstable.SDK.Builds` for pre-release tags. `Deepgram.Dev.sln` adds every example and edge-case project (103 projects) for Visual Studio.

## Client surfaces

Every row below was checked against `Deepgram/ClientFactory.cs` and the `UriSegments.cs` files on 2026-09-13.

| Product | Endpoint | Factory method | Implementation | Status |
| --- | --- | --- | --- | --- |
| Speech-to-text, pre-recorded | `POST /v1/listen` | `CreateListenRESTClient` | `Clients/Listen/v1/REST` | Shipped |
| Speech-to-text, streaming (Nova) | `wss /v1/listen` | `CreateListenWebSocketClient` | `Clients/Listen/v2/WebSocket` (SDK client v2 of the v1 API) | Shipped |
| Flux STT (conversational speech-to-text) | `wss /v2/listen` | `CreateFluxWebSocketClient` | `Clients/Flux/WebSocket` | Shipped; `SendConfigure`, `SendForceEndTurn` |
| Text-to-speech, batch (Aura) | `POST /v1/speak` | `CreateSpeakRESTClient` | `Clients/Speak/v1/REST` | Shipped |
| Text-to-speech, streaming (Aura) | `wss /v1/speak` | `CreateSpeakWebSocketClient` | `Clients/Speak/v2/WebSocket` (SDK client v2 of the v1 API) | Shipped |
| Flux TTS, batch | `POST /v2/speak` | `CreateFluxSpeakRESTClient` | `Clients/Flux/Speak/REST` | Shipped |
| Flux TTS, streaming | `wss /v2/speak` | `CreateFluxSpeakWebSocketClient` | `Clients/Flux/Speak/WebSocket` | Shipped; `SendText`, `SendFlush`, `SendInterrupt`, `SendConfigure`, `Stop` |
| Voice Agent | `wss agent.deepgram.com/v1/agent/converse` | `CreateAgentWebSocketClient` | `Clients/Agent/v2/Websocket` | Shipped |
| Voice Agent management | `/v1/projects/{id}/agents`, `/agent-variables` | `CreateAgentManageClient` | `Clients/AgentManage/v1` | Shipped |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deepgram/deepgram-dotnet-sdk](https://github.com/deepgram/deepgram-dotnet-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
