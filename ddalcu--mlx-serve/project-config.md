---
trigger: always_on
description: Native Zig server that runs MLX-format LMs on Apple Silicon and exposes OpenAI-compatible and Anthropic-compatible HTTP APIs. No Python.
---

# mlx-serve – project context for AI

Native Zig server that runs MLX-format LMs on Apple Silicon and exposes OpenAI-compatible and Anthropic-compatible HTTP APIs. No Python.

## Stack

- **Zig** 0.15+
- **mlx-c** (Apple) via Homebrew; FFI in `src/mlx.zig`.
- **Jinja engine** (lib/jinja_cpp): llama.cpp's C++17 Jinja2 implementation with nlohmann/json. Pre-compiled as `libjinja.a` (rebuild: see comment in `build.zig`).
- **stb_image** (lib/stb_image.h): JPEG/PNG decoding for vision pipeline
- **libwebp** via Homebrew: WebP image decoding for vision pipeline
- **safetensors** for weights; BPE tokenizers (SentencePiece / byte-level)

## Layout

| Path | Role |
|------|------|
| `src/main.zig` | Entry, CLI (`--model`, `--serve`, `--host`, `--port`, `--prompt`, `--max-tokens`, `--temp`, `--ctx-size`, `--timeout`, `--reasoning-budget`, `--no-vision`, `--log-level`, `--version`, `--help`) |
| `src/mlx.zig` | mlx-c FFI |
| `src/model.zig` | Config + safetensors loading; see **Supported Architectures** below |
| `src/tokenizer.zig` | BPE tokenizer |
| `src/transformer.zig` | Forward pass (embedding, attention, MLP, MoE, GatedDeltaNet); architecture dispatch |
| `src/generate.zig` | Autoregressive generation, sampling (temperature, top-k, top-p, repeat penalty, presence penalty, logprobs) |
| `src/chat.zig` | Chat template formatting (ChatML, Gemma turns, Llama-3, Jinja2 via llama.cpp engine); thinking/reasoning tags; tool call parsing |
| `src/vision.zig` | Vision encoder (Gemma 4 SigLIP): patch embedding, 2D RoPE, clipped linears, position pooling, embedding projection |
| `src/server.zig` | HTTP server: `/health`, `/v1/models`, `/v1/chat/completions`, `/v1/completions`, `/v1/messages`, `/v1/responses`, `/v1/responses/compact`, plus a WebSocket transport on `/v1/responses` (OpenAI Chat + Responses + Anthropic Messages, stream + non-stream, tool calling, KV cache, vision) |
| `src/responses.zig` | OpenAI Responses API: input-item parser (incl. `compaction` items), tool-shape translation, output-item builders, in-memory `ResponseStore`, `encodeCompactionBlob` (HTTP/streaming live in `server.zig`) |
| `src/ws.zig` | RFC 6455 WebSocket framing + handshake (server-side only). Generic over a `Conn`-shaped type so it stays test-friendly without depending on `server.zig`. |
| `src/status.zig` | TUI status bar (CPU, memory, GPU metrics) |
| `src/log.zig` | Leveled logging (error, warn, info, debug) |
| `build.zig` | Zig build; links mlx-c, libjinja.a, libwebp, stb_image |

### MLX Core (Swift macOS app)

| Path | Role |
|------|------|
| `app/Package.swift` | Swift package; `MLXCore` executable + `MLXCoreTests` test target |
| `app/Sources/MLXServe/MLXServeApp.swift` | App entry, menu bar + Chat/Browser windows |
| `app/Sources/MLXServe/AppState.swift` | Global state, chat session management, persistence |
| `app/Sources/MLXServe/Models/ChatModels.swift` | `ChatMessage`, `ChatImage`, `SerializedToolCall`, `ChatSession` |
| `app/Sources/MLXServe/Models/AgentModels.swift` | `AgentToolKind`, `AgentPlan`, `StepResult` |
| `app/Sources/MLXServe/Services/APIClient.swift` | HTTP + SSE streaming client for mlx-serve |
| `app/Sources/MLXServe/Services/AgentPrompt.swift` | System prompt, tool definitions (10 tools), `SkillManager` (prompt-based skills from `~/.mlx-serve/skills/`) |
| `app/Sources/MLXServe/Services/AgentEngine.swift` | Shared agent logic: history building, tool execution, repetition tracking, token estimation, overflow management |
| `app/Sources/MLXServe/Services/ToolExecutor.swift` | Tool handlers: shell, cwd, readFile, writeFile, editFile, searchFiles, listFiles, browse, webSearch, saveMemory |
| `app/Sources/MLXServe/Services/ImagePreprocessor.swift` | Image preprocessing for vision encoder (resize, float32 CHW conversion) |
| `app/Sources/MLXServe/Services/BrowserManager.swift` | WKWebView (headless, created eagerly for background browsing) |
| `app/Sources/MLXServe/Services/ServerManager.swift` | mlx-serve process lifecycle, stderr capture (`serverLog`), auto-start |
| `app/Sources/MLXServe/Services/TestServer.swift` | Embedded HTTP server (port 8090) for test automation — uses AgentEngine for shared logic |
| `app/Sources/MLXServe/Services/AgentMemory.swift` | Agent context memory (recent dirs, commands) |
| `app/Sources/MLXServe/Views/ChatView.swift` | Chat UI + `runAgentLoop()` + image attachment + context monitor |
| `app/Sources/MLXServe/Views/StatusMenuView.swift` | Menu bar UI, server log viewer, Claude Code launcher |
| `app/Sources/MLXServe/Views/BrowserView.swift` | Browser window (uses shared WKWebView) |

## Testing

- Always add tests, for anything you do, and update them as needed
- Unit tests are fine, but also add integration tests with real models, these are the real tests
- Make sure tests account for all the suported model architecture types, not just one.
- After a big feature, always test by building mlx-serve and mlx core.app, then run the .app bundle with TestServer.swift enable and test agentic harness
- `zig build test` — unit tests (chat, server, generate, model, log, tokenizer)
- `cd app && swift test` — Swift unit tests (agent harness, SSE parsing, serialization, history)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ddalcu/mlx-serve](https://github.com/ddalcu/mlx-serve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
