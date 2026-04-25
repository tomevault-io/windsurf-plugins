---
trigger: always_on
description: A macOS-native personal assistant that syncs emails and calendar from Apple Mail/Calendar,
---

# Giva - Generative Intelligent Virtual Assistant

A macOS-native personal assistant that syncs emails and calendar from Apple Mail/Calendar,
runs local LLM inference via MLX, and provides a CLI + REST API + SwiftUI menu bar app.

## Quick Reference

```bash
# Install (editable, with dev deps)
pip install -e ".[dev]"

# Run CLI
giva

# Run API server (port 7483)
giva-server

# Run tests
pytest

# Lint
ruff check src/ tests/
```

## Architecture

```
src/giva/
├── cli.py              # Interactive REPL (prompt-toolkit + rich)
├── server.py           # FastAPI REST + SSE API, _SpecialTokenFilter + _ThinkParser
├── config.py           # TOML config: config.default.toml → ~/.config/giva/config.toml → GIVA_* env
├── bootstrap.py        # Server-side bootstrap state machine (checkpoint Markov chain)
├── models.py           # HuggingFace model discovery, recommendation, download management
├── hardware.py         # Mac hardware detection (chip, RAM, GPU cores) for model sizing
├── benchmarks.py       # Live LLM benchmark fetching (Open LLM Leaderboard, LMArena)
├── db/
│   ├── models.py       # Dataclasses: Email, Event, Task, UserProfile, Goal, VlmTask
│   ├── store.py        # SQLite + FTS5 data layer (WAL mode, schema v10)
│   └── migrations.py   # Schema versioning + ALTER migrations
├── sync/
│   ├── mail.py         # Apple Mail sync via JXA, chunked headers + LLM filter
│   ├── calendar.py     # EventKit (fast, needs TCC grant) or AppleScript fallback
│   └── scheduler.py    # Background sync via threading.Timer
├── llm/
│   ├── apple_adapter.py # Apple Foundation Model adapter (on-device ~3B)
│   ├── engine.py       # MLX ModelManager: dual-model, extract_thinking, strip_special_tokens
│   ├── prompts.py      # All prompt templates
│   ├── structured.py   # Pydantic models for structured LLM output
│   └── vlm.py          # VLM inference placeholder (mlx-vlm integration)
├── intelligence/
│   ├── queries.py      # NL query → FTS5 context retrieval → streamed LLM response
│   ├── tasks.py        # Task extraction from emails/events via LLM
│   ├── profile.py      # SQL-first user profile analytics + LLM topic extraction
│   ├── proactive.py    # Priority suggestion engine
│   ├── onboarding.py   # Multi-step conversational onboarding with profile extraction
│   ├── goals.py        # Goal CRUD, strategy generation, progress tracking
│   ├── agents.py       # Post-chat agent pipeline: intent detection + action routing
│   ├── context.py      # Budget-aware context assembly + conversation memory
│   ├── daily_review.py # Daily goal review + reflection
│   ├── task_review.py  # Post-extraction task dedup, classify, route
│   └── mcp_observations.py # MCP source observation gathering for onboarding/context
├── agents/
│   ├── base.py         # Agent Protocol, AgentManifest, AgentResult, BaseAgent
│   ├── registry.py     # AgentRegistry: auto-discovers agents in giva.agents.*
│   ├── router.py       # Two-stage routing: keyword pre-filter → LLM classification
│   ├── queue.py        # Thread-safe priority queue for background agent jobs
│   ├── orchestrator/
│   │   ├── agent.py    # OrchestratorAgent: plan → validate → execute → QA → synthesize
│   │   ├── planner.py  # Assistant-model plan generation + structural validation
│   │   ├── executor.py # Sequential subtask execution with QA checks
│   │   └── prompts.py  # Orchestrator-specific prompt templates
│   ├── email_drafter/
│   │   ├── agent.py    # EmailDrafterAgent: drafts emails using assistant model + history
│   │   └── prompts.py  # Email drafting prompt templates
│   ├── mcp_agent/
│   │   ├── agent.py    # MCPAgent: wraps MCP servers as Giva agents (no LLM)
│   │   ├── config.py   # MCPServerConfig: parsed MCP server definitions
│   │   ├── lifecycle.py # MCPConnection: stdio/SSE process management
│   │   └── _compat.py  # MCP SDK version compatibility shim
│   └── web_orchestrator/
│       ├── agent.py    # WebOrchestratorAgent: plan → write VLM subtasks → return
│       └── prompts.py  # Web task decomposition prompt templates
├── audio/
│   ├── tts.py          # Qwen3-TTS via mlx-audio, per-sentence synthesis
│   ├── stt.py          # Lightning Whisper MLX speech-to-text
│   └── player.py       # Threaded audio playback queue for streaming TTS
└── utils/
    ├── applescript.py   # osascript/JXA runner helpers
    ├── email_parser.py  # MIME parsing utilities
    ├── power.py         # Battery, thermal, memory pressure monitoring (cached)
    └── recents.py       # Spotlight-based recently-used file discovery

GivaApp/                        # SwiftUI macOS app (Xcode project)
├── GivaApp.swift               # App entry point (bootstrap → main UI)
├── Models/
│   └── APIModels.swift         # Codable structs + ServerPhase enum
├── Services/
│   ├── APIService.swift        # URLSession wrapper + SSE streaming
│   ├── APIServiceProtocol.swift # Protocol for dependency injection + testing
│   ├── AgentActionHandler.swift # Shared agent action parsing (used by both ViewModels)
│   ├── BootstrapManager.swift  # First-run: venv creation, pip install, launchd daemon
│   ├── ServerManager.swift     # Connects to launchd-managed daemon (health polling)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cognivix) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
