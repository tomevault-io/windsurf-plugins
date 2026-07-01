---
trigger: always_on
description: This document helps AI agents work effectively in this codebase.
---

# AGENTS.md

This document helps AI agents work effectively in this codebase.

## Project Overview

**Parakeet ASR Server** - A Go-based automatic speech recognition (ASR) server using NVIDIA's Parakeet TDT 0.6B model in ONNX format. Provides an OpenAI Whisper-compatible API for audio transcription.

### Key Technologies

- **Language**: Go 1.25+
- **ML Runtime**: ONNX Runtime 1.25.x (CPU by default; optional CUDA GPU via execution providers)
- **Model**: NVIDIA Parakeet TDT 0.6B (Conformer-based encoder with Token-and-Duration Transducer decoder)
- **API**: REST, OpenAI Whisper-compatible

## Essential Commands

```bash
# Build
make build                  # Build to ./bin/parakeet

# Run
make run                    # Build and run with debug mode
make run-dev                # Run with custom port (5092) for development
./bin/parakeet              # Run binary directly
./bin/parakeet -port 8080 -models /path/to/models -log-level=debug -log-format=json -workers=2

# Download models
make models                 # Download int8 models (default, ~670MB)
make models-int8            # Download int8 quantized models
make models-fp32            # Download full precision models (~2.5GB)

# Test
make test                   # Run tests
make test-coverage          # Run with coverage

# Code quality
make fmt                    # Format code
make vet                    # Run go vet
make lint                   # Run all linters (vet + fmt)

# Dependencies
make deps                   # Download Go dependencies
make deps-tidy              # Tidy dependencies
make deps-onnxruntime       # Install ONNX Runtime library

# Docker
make docker-build-int8      # Build image with int8 models
make docker-build-fp32      # Build image with fp32 models
make docker-run-int8        # Run container with int8 models
make docker-run-fp32        # Run container with fp32 models
make docker-build-cuda      # Build CUDA/GPU image (fp32 models, linux/amd64)
make docker-run-cuda        # Run CUDA/GPU container (needs --gpus all / nvidia-container-toolkit)

# Release
make release                # Build all platforms
make release-linux          # Build Linux binaries (amd64/arm64)
make release-darwin         # Build macOS binaries (amd64/arm64)
make release-windows        # Build Windows binary (amd64)
```

## Project Structure

```
parakeet/
├── main.go                 # Entry point, CLI flags, server initialization
├── internal/
│   ├── asr/
│   │   ├── transcriber.go  # ONNX inference pipeline, TDT decoding
│   │   ├── mel.go          # Mel filterbank feature extraction (FFT, windowing)
│   │   ├── audio.go        # WAV parsing, magic-byte detection, resampling to 16kHz
│   │   ├── ffmpeg.go       # Optional ffmpeg-backed converter for non-WAV inputs
│   │   ├── audio_test.go   # Unit + concurrency tests for audio/ffmpeg logic
│   │   └── provider_test.go # Execution-provider parsing/selection tests
│   └── server/
│       ├── server.go       # HTTP server, route setup, lifecycle management
│       ├── handlers.go     # API endpoint handlers, response formatting
│       └── types.go        # Request/response type definitions
├── models/                 # ONNX models (downloaded separately)
├── bin/                    # Build output directory
├── Makefile                # Build recipes
├── Dockerfile              # Multi-stage container build (CPU)
├── Dockerfile.cuda         # CUDA/GPU image (ONNX Runtime GPU build, fp32, linux/amd64)
├── .agents/                # AI agent documentation
│   ├── AGENTS.md           # This file - codebase guide for agents
│   ├── TODO.md             # Pending tasks and improvements
│   └── DESIGN_DECISIONS.md # Architectural decisions record
├── .github/
│   └── workflows/
│       ├── ci.yaml         # CI pipeline (lint, test, build)
│       └── release.yaml    # Release pipeline (binaries, docker)
└── README.md
```

## Code Organization

### `main.go` (Entry Point)

- Parses CLI flags: `-port`, `-models`, `-log-level`, `-log-format`, `-workers`, `-ffmpeg`, `-ffmpeg-path`, `-ffmpeg-timeout`, `-gpu`, `-gpu-device`
- Configures `slog` global logger (text or JSON handler, four log levels)
- Runs server in background goroutine, listens for SIGINT/SIGTERM
- Graceful shutdown: waits up to 30s for in-flight requests via `http.Server.Shutdown`
- Calls `srv.Close()` after shutdown to release ONNX resources
- Default port: 5092, default models dir: `./models`, default log level: `info`, default log format: `text`, default workers: `4`, ffmpeg fallback enabled by default, ffmpeg timeout: `60s`, GPU provider: `cpu`, GPU device: `0`

### `internal/server/` (HTTP Server Package)

#### `server.go`

- `Config` struct: Port, ModelsDir, LogLevel, LogFormat, Workers, FFmpegEnabled, FFmpegPath, FFmpegTimeout, GPUProvider, GPUDeviceID
- `Server` struct: wraps config, transcriber, `http.Server`, HTTP mux, and API key
- `New()` - Parses the GPU provider via `asr.ParseProvider` (fails fast on unknown values), initializes transcriber with worker pool, execution provider, and optional ffmpeg converter, reads `PARAKEET_API_KEY` env var, and sets up routes
- `Run()` - Starts HTTP listener (blocks until shutdown or error)
- `Shutdown(ctx)` - Graceful HTTP shutdown, waits for in-flight requests to finish

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achetronic/parakeet](https://github.com/achetronic/parakeet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
