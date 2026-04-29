---
trigger: always_on
description: <!-- This is the single source of truth for all AI coding agents. CLAUDE.md is a symlink to this file. -->
---

# TipTour - Agent Instructions

<!-- This is the single source of truth for all AI coding agents. CLAUDE.md is a symlink to this file. -->
<!-- AGENTS.md spec: https://github.com/agentsmd/agents.md — supported by Claude Code, Cursor, Copilot, Gemini CLI, and others. -->

## Overview

macOS menu bar companion app. Lives entirely in the macOS status bar (no dock icon, no main window). Clicking the menu bar icon opens a custom floating panel with companion voice controls. Uses push-to-talk (ctrl+option) to capture voice input, transcribes it via AssemblyAI streaming, and sends the transcript + a screenshot of the user's screen to Claude. Claude responds with text (streamed via SSE) and voice (ElevenLabs TTS). A blue cursor overlay can fly to and point at UI elements Claude references on any connected monitor.

All API keys live on a Cloudflare Worker proxy — nothing sensitive ships in the app.

## Architecture

- **App Type**: Menu bar-only (`LSUIElement=true`), no dock icon or main window
- **Framework**: SwiftUI (macOS native) with AppKit bridging for menu bar panel and cursor overlay
- **Pattern**: MVVM with `@StateObject` / `@Published` state management
- **Voice Mode**: Gemini Live only. Single-model realtime WebSocket to `gemini-3.1-flash-live-preview` — bidirectional voice (PCM16 16kHz in, PCM16 24kHz out), vision (JPEG screenshots), text transcription, AND tool calling all in one streaming connection. Two tools exposed: `point_at_element(label)` for single-click asks, `submit_workflow_plan(goal, app, steps)` for multi-step walkthroughs. Gemini produces workflow plans itself inside its tool call — no separate planner model. API key fetched from Worker's `/gemini-live-key` endpoint.
- **Legacy voice code** (Apple Speech STT → Claude → ElevenLabs) remains compiled but has no user-visible UI to select it. The code paths are candidate for deletion in a future cleanup.
- **Screen Capture**: ScreenCaptureKit (macOS 14.2+), multi-monitor support
- **Voice Input**: Gemini Live mode streams mic audio directly over the WebSocket. Hotkey is a listen-only CGEvent tap so modifier-only shortcuts (ctrl+opt) work reliably in background.
- **Element Pointing**: The LLM calls one of two tools. `ElementResolver` resolves the `label` argument to pixel positions via a three-tier lookup:
    1. **macOS Accessibility tree** — pixel-perfect, ~30ms. Works on native Mac apps, most Electron apps, and anywhere a well-formed AX tree is exposed.
    2. **YOLO + OCR visual detection** — fallback for apps without good AX support (Blender, games, some web content). Uses step 1's optional `x,y` hint as proximity anchor.
    3. **Raw LLM coordinates** — absolute last resort.
- **Element Detection**: On-device via `NativeElementDetector.swift` — CoreML YOLO model (`UIElementDetector.mlpackage`) for UI element bounding boxes + Apple Vision framework (`VNRecognizeTextRequest`, accurate mode) for OCR text detection. No external server, no Python, no OmniParser dependency.
- **Accessibility Tree**: `AccessibilityTreeResolver.swift` walks the user's target app's AX tree via `ApplicationServices`, matches elements by title/description/value, returns exact pixel frames in global AppKit coordinates. Uses a snapshot of `NSWorkspace.frontmostApplication` taken at hotkey press time so the query targets the app the user was actually looking at, not TipTour's own menu bar.
- **Concurrency**: `@MainActor` isolation, async/await throughout
- **Analytics**: PostHog via `TipTourAnalytics.swift`

### API Proxy (Cloudflare Worker)

The app never calls external APIs directly. All requests go through a Cloudflare Worker (`worker/src/index.ts`) that holds the real API keys as secrets.

| Route | Upstream | Purpose |
|-------|----------|---------|
| `GET /gemini-live-key` | — (returns secret) | Returns the Gemini API key so the app can open a direct WebSocket to Gemini Live. Cloudflare Workers can't proxy Gemini's WebSocket so we expose the key to trusted clients. |
| `POST /generate-guide` | `generativelanguage.googleapis.com/.../gemini-2.5-flash` | Gemini guide generation from YouTube transcripts |
| `POST /transcript` | `youtube-transcript` (npm package) | Fetches YouTube video transcripts |

Worker secret: `GEMINI_API_KEY`

Removed previously: `/chat` + `ANTHROPIC_API_KEY` + `ELEVENLABS_API_KEY` + `/tts` + `/chat-fast` + `OPENROUTER_API_KEY` + `ELEVENLABS_VOICE_ID` (Gemini-only shipping default, the Claude + ElevenLabs + OpenRouter legacy pipelines are no longer reachable from the UI); `/plan` (Gemini now plans inside its own tool call); `/transcribe-token` + `ASSEMBLYAI_API_KEY` (Gemini Live does STT in-stream).

### Key Architecture Decisions

**Menu Bar Panel Pattern**: The companion panel uses `NSStatusItem` for the menu bar icon and a custom borderless `NSPanel` for the floating control panel. This gives full control over appearance (dark, rounded corners, custom shadow) and avoids the standard macOS menu/popover chrome. The panel is non-activating so it doesn't steal focus. A global event monitor auto-dismisses it on outside clicks.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [milind-soni/tiptour-macos](https://github.com/milind-soni/tiptour-macos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
