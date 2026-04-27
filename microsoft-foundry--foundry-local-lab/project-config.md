---
trigger: always_on
description: This file provides context for AI coding agents (GitHub Copilot, Copilot Workspace, Codex, etc.) working in this repository.
---

# Coding Agent Instructions

This file provides context for AI coding agents (GitHub Copilot, Copilot Workspace, Codex, etc.) working in this repository.

## Project Overview

This is a **hands-on workshop** for building AI applications with [Foundry Local](https://foundrylocal.ai) — a lightweight runtime that downloads, manages, and serves language models entirely on-device via an OpenAI-compatible API. The workshop includes step-by-step lab guides and runnable code samples in Python, JavaScript, and C#.

## Repository Structure

```
├── labs/                              # Markdown lab guides (Parts 1–13)
├── python/                            # Python code samples (Parts 2–6, 8–9, 11)
├── javascript/                        # JavaScript/Node.js code samples (Parts 2–6, 8–9, 11)
├── csharp/                            # C# / .NET 9 code samples (Parts 2–6, 8–9, 11)
├── zava-creative-writer-local/        # Part 7 capstone app + Part 12 UI (Python/JS/C#)
│   ├── ui/                            # Shared browser UI (vanilla HTML/CSS/JS)
│   └── src/
│       ├── api/                       # Python FastAPI multi-agent service (serves UI)
│       ├── javascript/                # Node.js CLI + HTTP server (server.mjs)
│       ├── csharp/                    # .NET console multi-agent app
│       └── csharp-web/                # .NET ASP.NET Core minimal API (serves UI)
├── samples/audio/                     # Part 9 sample WAV files + generator script
├── images/                            # Diagrams referenced by lab guides
├── README.md                          # Workshop overview and navigation
├── KNOWN-ISSUES.md                    # Known issues and workarounds
├── package.json                       # Root devDependency (mermaid-cli for diagrams)
└── AGENTS.md                          # This file
```

## Language & Framework Details

### Python
- **Location:** `python/`, `zava-creative-writer-local/src/api/`
- **Dependencies:** `python/requirements.txt`, `zava-creative-writer-local/src/api/requirements.txt`
- **Key packages:** `foundry-local-sdk`, `openai`, `agent-framework-foundry-local`, `fastapi`, `uvicorn`
- **Min version:** Python 3.9+
- **Run:** `cd python && pip install -r requirements.txt && python foundry-local.py`

### JavaScript
- **Location:** `javascript/`, `zava-creative-writer-local/src/javascript/`
- **Dependencies:** `javascript/package.json`, `zava-creative-writer-local/src/javascript/package.json`
- **Key packages:** `foundry-local-sdk`, `openai`
- **Module system:** ES modules (`.mjs` files, `"type": "module"`)
- **Min version:** Node.js 18+
- **Run:** `cd javascript && npm install && node foundry-local.mjs`

### C#
- **Location:** `csharp/`, `zava-creative-writer-local/src/csharp/`
- **Project files:** `csharp/csharp.csproj`, `zava-creative-writer-local/src/csharp/ZavaCreativeWriter.csproj`
- **Key packages:** `Microsoft.AI.Foundry.Local` (non-Windows), `Microsoft.AI.Foundry.Local.WinML` (Windows — superset with QNN EP), `OpenAI`, `Microsoft.Agents.AI.OpenAI`
- **Target:** .NET 9.0 (conditional TFM: `net9.0-windows10.0.26100` on Windows, `net9.0` elsewhere)
- **Run:** `cd csharp && dotnet run [chat|rag|agent|multi]`

## Coding Conventions

### General
- All code samples are **self-contained single-file examples** — no shared utility libraries or abstractions.
- Each sample runs independently after installing its own dependencies.
- API keys are always set to `"foundry-local"` — Foundry Local uses this as a placeholder.
- Base URLs use `http://localhost:<port>/v1` — the port is dynamic and discovered at runtime via the SDK (`manager.urls[0]` in JS, `manager.endpoint` in Python).
- The Foundry Local SDK handles service startup and endpoint discovery; prefer SDK patterns over hard-coded ports.

### Python
- Use `openai` SDK with `OpenAI(base_url=..., api_key="not-required")`.
- Use `FoundryLocalManager()` from `foundry_local` for SDK-managed service lifecycle.
- Streaming: iterate over `stream` object with `for chunk in stream:`.
- No type annotations in sample files (keep samples concise for workshop learners).

### JavaScript
- ES module syntax: `import ... from "..."`.
- Use `OpenAI` from `"openai"` and `FoundryLocalManager` from `"foundry-local-sdk"`.
- SDK init pattern: `FoundryLocalManager.create({ appName })` → `FoundryLocalManager.instance` → `manager.startWebService()` → `await catalog.getModel(alias)`.
- Streaming: `for await (const chunk of stream)`.
- Top-level `await` is used throughout.

### C#
- Nullable enabled, implicit usings, .NET 9.
- Use `FoundryLocalManager.StartServiceAsync()` for SDK-managed lifecycle.
- Streaming: `CompleteChatStreaming()` with `foreach (var update in completionUpdates)`.
- The main `csharp/Program.cs` is a CLI router dispatching to static `RunAsync()` methods.

### Tool Calling
- Only certain models support tool calling: **Qwen 2.5** family (`qwen2.5-*`) and **Phi-4-mini** (`phi-4-mini`).
- Tool schemas follow the OpenAI function-calling JSON format (`type: "function"`, `function.name`, `function.description`, `function.parameters`).
- The conversation uses a multi-turn pattern: user → assistant (tool_calls) → tool (results) → assistant (final answer).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft-foundry/Foundry-Local-Lab](https://github.com/microsoft-foundry/Foundry-Local-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
