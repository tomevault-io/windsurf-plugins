---
trigger: always_on
description: MarkuprPlus is a macOS/Windows menu bar app and CLI/MCP tool that intelligently captures developer feedback. It records your screen and voice simultaneously, then uses an intelligent post-processing pipeline to correlate transcript timestamps with the screen recording -- extracting the right frames at the right moments and stitching everything into a structured, AI-ready Markdown document. The output is purpose-built for AI coding agents: every screenshot placed exactly where it belongs, every i
---

# CLAUDE.md - MarkuprPlus

## Project Overview

MarkuprPlus is a macOS/Windows menu bar app and CLI/MCP tool that intelligently captures developer feedback. It records your screen and voice simultaneously, then uses an intelligent post-processing pipeline to correlate transcript timestamps with the screen recording -- extracting the right frames at the right moments and stitching everything into a structured, AI-ready Markdown document. The output is purpose-built for AI coding agents: every screenshot placed exactly where it belongs, every issue clearly documented.

As of v2.5.0, MarkuprPlus also ships as:
- **CLI tool** (`npx markuprx analyze ./recording.mov`) -- headless video analysis pipeline
- **MCP server** (`npx --package markuprx markuprx-mcp`) -- Model Context Protocol server for AI coding agents (capture screenshots, analyze video, start/stop recordings)
- **GitHub Action** (`eddiesanjuan/markuprx-action@v1`) -- CI/CD visual feedback on PRs

**Version:** 3.0.0
**License:** MIT (Open Source)

## Tech Stack

- **Framework:** Electron + React + TypeScript
- **Build:** electron-vite + Vite (desktop), esbuild (CLI/MCP)
- **Transcription:** Local Whisper (default), OpenAI Whisper-1 API (optional cloud)
- **AI Analysis:** Anthropic Claude API (BYOK or premium tier)
- **Testing:** Vitest (unit, integration, e2e)
- **Package:** electron-builder (DMG, NSIS, AppImage)
- **Styling:** Tailwind CSS
- **Schema Validation:** Zod
- **MCP:** @modelcontextprotocol/sdk

## Architecture

```
src/
├── main/                   # Electron main process
│   ├── index.ts            # Entry point, window management, IPC orchestration
│   ├── SessionController.ts # 7-state FSM with watchdog timer
│   ├── CrashRecovery.ts    # Auto-save every 5s, crash detection, recovery dialog
│   ├── ErrorHandler.ts     # Centralized error handling
│   ├── HotkeyManager.ts    # Global hotkey registration
│   ├── MenuManager.ts      # Application menu
│   ├── TrayManager.ts      # Menu bar tray icon and status
│   ├── PermissionManager.ts # macOS permission checks
│   ├── AutoUpdater.ts      # Auto-update via electron-updater
│   ├── ai/                 # AI analysis pipeline (Claude)
│   │   ├── AIPipelineManager.ts  # Orchestrates AI analysis (free/byok/premium tiers)
│   │   ├── ClaudeAnalyzer.ts     # Claude API integration
│   │   ├── ImageOptimizer.ts     # Screenshot optimization for API
│   │   ├── StructuredMarkdownBuilder.ts # AI-enhanced markdown output
│   │   └── types.ts              # AI pipeline types
│   ├── analysis/           # Feedback analysis and categorization
│   ├── audio/              # Microphone capture, VAD
│   ├── capture/            # Screen capture via desktopCapturer
│   ├── ipc/                # IPC handler registration and routing
│   ├── output/             # Document generation and export
│   │   ├── MarkdownGenerator.ts  # llms.txt-inspired markdown output
│   │   ├── ExportService.ts      # Multi-format export (MD, PDF, HTML, JSON)
│   │   ├── ClipboardService.ts   # Clipboard bridge (copies file path)
│   │   ├── FileManager.ts        # Session file management
│   │   └── sessionAdapter.ts     # Type conversion utilities
│   ├── pipeline/           # Post-processing pipeline
│   │   ├── PostProcessor.ts      # Pipeline orchestrator (transcribe -> analyze -> extract -> generate)
│   │   ├── TranscriptAnalyzer.ts # Heuristic key-moment detection
│   │   └── FrameExtractor.ts     # ffmpeg-based video frame extraction
│   ├── platform/           # Platform-specific code (Windows taskbar)
│   ├── settings/           # Persistent settings with secure API key storage
│   ├── transcription/      # Transcription tier management
│   │   ├── TierManager.ts       # Tier selection (Whisper, timer-only)
│   │   ├── WhisperService.ts     # Local Whisper integration
│   │   └── ModelDownloadManager.ts # Whisper model download from HuggingFace
│   └── windows/            # Window management (popover, taskbar)
├── cli/                    # Headless CLI tool
│   ├── index.ts            # CLI entry point (commander-based)
│   ├── CLIPipeline.ts      # Video analysis pipeline (ffmpeg + Whisper + markdown)
│   ├── WatchMode.ts        # Watch directory for new recordings
│   ├── doctor.ts           # System dependency checker (ffmpeg, Whisper, etc.)
│   └── init.ts             # Project config scaffolding (.markuprx.json)
├── mcp/                    # MCP server for AI coding agents
│   ├── index.ts            # MCP entry point
│   ├── server.ts           # MCP server setup
│   ├── tools/              # MCP tool implementations
│   │   ├── analyzeScreenshot.ts  # Analyze a screenshot with Claude
│   │   ├── analyzeVideo.ts       # Analyze a video recording
│   │   ├── captureScreenshot.ts  # Capture a screenshot
│   │   ├── captureWithVoice.ts   # Capture with voice narration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hashfunction/MarkuprPlus](https://github.com/hashfunction/MarkuprPlus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
