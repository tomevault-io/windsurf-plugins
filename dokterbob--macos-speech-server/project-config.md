---
trigger: always_on
description: Essential knowledge for AI agents working on this codebase.
---

# Agent Guide

Essential knowledge for AI agents working on this codebase.

## Project overview

A macOS-native HTTP server that exposes OpenAI-compatible speech API endpoints and a Wyoming protocol server for Home Assistant integration, running entirely on-device. Built with Vapor (Swift web framework) and FluidAudio (on-device ASR via Apple's Neural Engine).

- **STT** is fully implemented using FluidAudio's `AsrManager`.
- **TTS** is fully implemented with three engines: `pocket_tts` (FluidAudio PocketTTS, `alba` only), `avspeech` (macOS built-in, 150+ voices), and `kokoro` (FluidAudio Kokoro, 50 voices across 8 languages).

## Tech stack

| Component | Library | Version constraint |
|-----------|---------|-------------------|
| Web framework | [Vapor](https://github.com/vapor/vapor) | 4.76.0+ |
| Speech-to-text / TTS | [FluidAudio](https://github.com/FluidInference/FluidAudio) | 0.12.4+ |
| Multipart parsing | [multipart-kit](https://github.com/vapor/multipart-kit) | 4.0.0+ |
| YAML parsing | [Yams](https://github.com/jpsim/Yams) | 6.0.1+ |
| TCP networking | [swift-nio](https://github.com/apple/swift-nio) | 2.65.0+ |

**Platform:** macOS 14+, Swift 6.2

## Documentation resources

When working on this project, use these MCP tools for up-to-date documentation:

### Context7 (preferred for Vapor)

Use the Context7 MCP to query library docs. Resolve library IDs first, then query.

| Library | Context7 ID |
|---------|-------------|
| Vapor | `/websites/vapor_codes` |
| FluidAudio | `/fluidinference/fluidaudio` |

Example workflow:
1. Call `mcp__context7__query-docs` with `libraryId: "/websites/vapor_codes"` and your question.
2. For FluidAudio: `libraryId: "/fluidinference/fluidaudio"`.

### DeepWiki (recommended for FluidAudio)

FluidAudio is a newer library with less community documentation. Use the DeepWiki MCP to explore its internals:

- `mcp__deepwiki__read_wiki_structure` with `repoName: "FluidInference/FluidAudio"` to browse topics.
- `mcp__deepwiki__ask_question` with `repoName: "FluidInference/FluidAudio"` for specific questions.

## Architecture

### ServerConfig

`ServerConfig` (`ServerConfig.swift`) loads and stores all runtime settings. It is available on `Application` and `Request` via Vapor DI (`app.serverConfig` / `req.serverConfig`).

**Config discovery order** (first match wins):
1. `SPEECH_SERVER_CONFIG` env var — path to a YAML file
2. `./speech-server.yaml` in the current working directory (gitignored — copy from `speech-server.yaml.example`)
3. Built-in defaults (all fields have sensible defaults matching original hardcoded values)

**Struct hierarchy**:
```
ServerConfig
  ├─ logLevel: String              (top-level, CodingKey "log_level")
  ├─ servers: ServersConfig
  │   ├─ http: HTTPConfig          (host, port, uploadLimitMB)
  │   └─ wyoming: WyomingConfig    (host, port)
  ├─ stt: STTConfig                (engine, parakeet settings)
  └─ tts: TTSConfig                (engine, pocket_tts/avspeech/kokoro settings)
```

Access: `config.logLevel`, `config.servers.http.host`, `config.servers.wyoming.port`.

**Engine enums** are exhaustive by design. Adding a new engine requires:
1. Add a `case` to `STTEngine` or `TTSEngine` (the raw value becomes the YAML key, e.g. `"parakeet"`)
2. Add a `XxxSettings` struct with `decodeIfPresent` defaults if the engine has settings
3. Add a `case` in `configure.swift`'s switch to construct and initialize the service
4. Implement the `STTService`/`TTSService` protocol

**`model_version` in `ParakeetSettings`**: mapped to `AsrModelVersion` in `configure.swift` and passed to `FluidSTTService.initialize(modelVersion:)`. Valid values: `"v3"` (Parakeet TDT 0.6B v3, multilingual, 25 languages, default) and `"v2"` (Parakeet TDT 0.6B v2, English-only, higher recall). Invalid values cause a startup error.

**Partial configs work**: all fields use `decodeIfPresent` with defaults, so a minimal `speech-server.yaml` with only `stt:\n  engine: parakeet` is valid.

### Middleware chain (order matters)

1. `RequestLoggingMiddleware` -- logs `METHOD /path STATUS` at NOTICE level
2. `OpenAIErrorMiddleware` -- catches errors, returns OpenAI-format JSON

### Service layer (dependency injection)

Services are registered on `Application.storage` and accessed via computed properties on `Request` and `Application`:

```swift
req.sttService   // -> STTService protocol
req.ttsService   // -> TTSService protocol
app.sttService = FluidSTTService()  // setter on Application
```

Both protocols require `Sendable` conformance.

### Route registration

Routes are registered twice in `routes.swift` -- once at `/audio/*` and once at `/v1/audio/*` for OpenAI API compatibility. Both `SpeechController` and `TranscriptionController` implement `RouteCollection`.

### Transcription upload pipeline

`TranscriptionController` streams the request body directly to a temp file before any
multipart parsing, keeping peak RAM at O(chunk_size) during upload:

1. Body chunks are written via `OutputStream` to `<req.id>.multipart` in the temp directory.
   An in-flight byte counter rejects uploads exceeding **500 MB** with `413 Payload Too Large`.
2. The temp file is mmap-read (`Data(contentsOf:options:.mappedIfSafe)`) and decoded with
   `FormDataDecoder` from MultipartKit.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dokterbob/macos-speech-server](https://github.com/dokterbob/macos-speech-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
