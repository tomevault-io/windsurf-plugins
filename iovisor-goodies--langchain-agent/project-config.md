---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Portable autonomous agent loop using LangChainGo + Ollama/Gemini. Uses JSON tool calling (not ReAct) for reliability with small models.

## Status

**Working:**
- ✅ Agent loop with tool dispatch
- ✅ SSH tool (remote command execution, ssh-agent + interactive password fallback)
- ✅ Shell tool (local command execution)
- ✅ MCP tool (multiple servers, stdio/SSE/HTTP transport, via mark3labs/mcp-go)
- ✅ Conversation history/memory
- ✅ Tool selection rules in prompt
- ✅ Honest error reporting (no hallucination on failures)
- ✅ Wiki RAG tool (Confluence HTML export with diagram support)
- ✅ Ollama backend (local or remote via `--ollama-url`; default model `qwen2.5:32b`)
- ✅ Gemini backend (Google AI, via `--backend gemini`, requires `GOOGLE_API_KEY`)
- ✅ Edge sensor tools (`edge_temp`, `edge_gpio` — SSH-based, portable across Pi and amd64 Linux, via `--edge user@host`)
- ✅ HTTP webhook listener (`--webhook-port N` — `POST /webhook` runs the agent)

**TODO:**
- ✅ Streaming output
- ✅ Event-driven automation (HTTP webhook; cron/file-watch still open)
- [ ] Domain knowledge improvements (command patterns)
- [ ] `edge_camera` tool (SSH capture via libcamera-still / ffmpeg-v4l2 fallback, scp back) — designed in `PLAN-event-sensor.md`, deferred for now

**Out of scope (by design):** Running the agent *on* the Pi (ARM64-resident autonomous agent). This project is a workstation agent that *operates* edge devices remotely over SSH — the `edge_*` tools are SSH verbs in the multi-hop loop. An agent that runs on the Pi itself would be a separate project.

## Use Cases

```
"ssh to x@y.z and tell me what platform it is"   # → ssh tool
"ssh to x@y.z and see why pods are failing"       # → ssh tool
"list running processes"                          # → shell tool
"check disk space"                                # → shell tool
"use mcp to list files in /tmp"                   # → mcp tool (requires --mcp)
"use mcp to read the file /tmp/test.txt"          # → mcp tool (requires --mcp)
"search wiki for deployment architecture"         # → wiki tool (requires --wiki)
"what does the network diagram show"              # → wiki tool (requires --wiki)
"what is the cpu temperature on the edge box"     # → edge_temp tool (requires --edge)
"read gpio pin 17"                                # → edge_gpio tool (requires --edge)
"what is a container?"                            # → direct answer (no tool)
"is Go faster than Python?"                       # → direct answer (no tool)
```

**Note:** MCP requires explicitly saying "mcp" in the prompt. Tool routing keywords are hardcoded in the system prompt (`llm/ollama.go:BuildSystemPrompt`). The MCP routing line is dynamically generated based on registered MCP tool names (`llm/ollama.go:mcpRoutingLine`).

## Backends

Two LLM backends are supported, selected via `--backend`. Both implement the same `llm.ChatClient` + `llm.StreamingChatClient` interface, so all tools (ssh, shell, mcp, wiki) and the agent loop behave identically across backends.

### Ollama (default, local)

```bash
./langchain-agent                       # default model: qwen2.5:32b
./langchain-agent --model llama3.1      # smaller, reliable floor for tool calling
./langchain-agent --model llama3.2      # smaller/faster, less reliable for tool calling
./langchain-agent --model qwen2.5:32b --ollama-url http://big-tower.local:11434  # remote Ollama host
```
- Requires an Ollama server (default `http://localhost:11434`).
- Default model is `qwen2.5:32b` — it needs a decent GPU, so on a modest local box pass `--model llama3.1` (or point `--ollama-url` at a GPU host that has qwen pulled).
- `--ollama-url` points at a remote Ollama host (e.g. a GPU tower on the LAN). Also honors `$OLLAMA_HOST` when the flag is unset. Ignored for the gemini backend.
- The model must be pulled on the target server and expose the `tools` capability for reliable JSON tool calling — `qwen2.5:32b` and `llama3.1` both do; `llama3.2` (3B) works but is less reliable.
- No API key needed.
- `llama3.1` is the recommended floor for reliable JSON tool calling; `qwen2.5:32b` is the default and most reliable when a GPU is available.

### Gemini (Google AI, cloud)

```bash
export GOOGLE_API_KEY="$(tr -d '[:space:]' < ~/gemini-api-key.txt)"
./langchain-agent --backend gemini                       # default model: gemini-2.5-flash
./langchain-agent --backend gemini --model gemini-2.5-pro
```
- Requires `GOOGLE_API_KEY` env var. The `langchaingo/llms/googleai` package reads it automatically.
- Get a key from https://aistudio.google.com/apikey.
- Verified working end-to-end: knowledge questions, shell tool dispatch, and SSH tool dispatch (including conversation context across turns).

**Known quirks:**
- `gemini-2.0-flash` returns HTTP 404 with langchaingo v0.1.14 even though it's listed in the API's models endpoint. Use `gemini-2.5-flash` (the project default) or newer.
- API keys auto-expire after ~30 days of inactivity. The error message is `"API key expired. Please renew the API key."` even when the AI Studio dashboard doesn't flag the key as expired.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iovisor-goodies/langchain-agent](https://github.com/iovisor-goodies/langchain-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
