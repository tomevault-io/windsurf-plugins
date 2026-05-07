---
trigger: always_on
description: A Windows port of [clicky](https://github.com/farzaa/clicky) by @farzaa. The original is a macOS menubar app — an AI companion that lives near your cursor, sees your screen, and responds via voice. This repo is a faithful Windows reimplementation using WPF/.NET 8, keeping the same push-to-talk UX and pipeline.
---

# Clicky Windows — Agent Context

## What this project is

A Windows port of [clicky](https://github.com/farzaa/clicky) by @farzaa. The original is a macOS menubar app — an AI companion that lives near your cursor, sees your screen, and responds via voice. This repo is a faithful Windows reimplementation using WPF/.NET 8, keeping the same push-to-talk UX and pipeline.

**This repo is the code-only, developer-friendly version.** It uses a Cloudflare Worker proxy to avoid embedding API keys in the binary. There is a separate GUI/app repo (`clicky-windows-app`) that removes the proxy and adds a first-run setup screen for end users.

---

## Architecture

### Full pipeline (one interaction cycle)

```
User holds Ctrl+Alt
  → GlobalHotkeyMonitor fires HotkeyPressed
  → AudioCaptureService starts capturing mic (PCM 16kHz mono)
  → AssemblyAiTranscriptionService opens WebSocket, streams audio chunks
User releases Ctrl+Alt
  → AssemblyAI sends Terminate, waits for final transcript
  → ScreenCaptureUtility captures all monitors as JPEG
  → ClaudeApiClient sends screenshots + transcript to Claude (SSE streaming)
  → Claude response is parsed for [POINT:x,y:description:screenName] tags
  → OverlayWindow animates the cursor buddy to the pointed coordinates
  → ElevenLabsTtsClient POSTs text to TTS, plays MP3 via NAudio
```

### Key files

| File | Role |
|------|------|
| `clicky-windows/CompanionManager.cs` | Central state machine. Owns the full pipeline. Exposes `VoiceState`, `StreamingResponseText`, `LastTranscript` via `INotifyPropertyChanged`. |
| `clicky-windows/OverlayWindow.xaml.cs` | Full-screen transparent click-through overlay. Hosts animated cursor buddy, response bubble, waveform. |
| `clicky-windows/TrayManager.cs` | System tray icon + context menu (toggle cursor, model picker, quit). |
| `clicky-windows/CompanionPanelWindow.xaml.cs` | Side panel showing state/transcript (not always visible). |
| `clicky-windows/App.xaml.cs` | Entry point. Instantiates CompanionManager, OverlayWindow, TrayManager. |
| `clicky-windows/Services/ClaudeApiClient.cs` | Sends vision requests to Claude via Cloudflare Worker proxy (`/chat`). Streams SSE response. |
| `clicky-windows/Services/AssemblyAiTranscriptionService.cs` | WebSocket client for AssemblyAI v3 real-time streaming. Fetches short-lived token via Worker (`/transcribe-token`). |
| `clicky-windows/Services/ElevenLabsTtsClient.cs` | POSTs text to ElevenLabs via Worker (`/tts`), plays MP3 with NAudio. |
| `clicky-windows/Services/AudioCaptureService.cs` | Captures microphone as PCM 16kHz mono chunks using NAudio. |
| `clicky-windows/Services/ScreenCaptureUtility.cs` | Captures all monitors, returns JPEG bytes + labels (e.g. "Screen 1 (Primary)"). |
| `clicky-windows/Services/GlobalHotkeyMonitor.cs` | Low-level keyboard hook for Ctrl+Alt push-to-talk. |
| `clicky-windows/app.manifest` | Sets UAC level to `requireAdministrator` so the overlay can capture elevated windows (Task Manager etc). |
| `worker/` | Cloudflare Worker source that proxies requests to Anthropic, AssemblyAI, and ElevenLabs. |

---

## Key design decisions and gotchas

### Cloudflare Worker proxy
All three API keys live in the Worker's environment variables, not in the app binary. The app only knows `WorkerBaseUrl` (set in `CompanionManager.WorkerBaseUrl`). This is the code-repo design — the app repo (`clicky-windows-app`) removes the proxy and calls APIs directly.

### POINT tag regex
Claude outputs pointing coordinates in tags like:
```
[POINT:247:19:Screen 1 (Primary)]
[POINT:247,19:cursor description:Screen 1 (Primary)]
```
The regex must handle both `:` and `,` as the x/y separator, and the description field is optional:
```csharp
@"\[POINT:(\d+(?:\.\d+)?)[:,](\d+(?:\.\d+)?):(?:([^:\]]+):)?([^\]]+)\]"
// Group 1: x, Group 2: y, Group 3: optional description, Group 4: screen name
```
Do not simplify this regex — Claude's output format varies.

### WPF animation hold problem
After a `BeginAnimation` call, WPF holds the property value and silently ignores direct `Canvas.SetLeft`/`Canvas.SetTop` calls. To resume manual positioning after the fly animation, you MUST call:
```csharp
CursorCanvas.BeginAnimation(Canvas.LeftProperty, null);
CursorCanvas.BeginAnimation(Canvas.TopProperty, null);
```
This is done in `OverlayWindow.cs` inside the `settleTimer.Tick` handler after the pointing animation completes.

### Cursor follow after POINT
After pointing, the cursor buddy should stay at the pointed location until the user moves their actual cursor more than 4 pixels. The pattern:
1. After fly animation settles, capture `GetCursorPos` into `_cursorPosAtPointEnd`
2. In `OnCursorFollowTick`, if `_cursorPosAtPointEnd` is set and cursor hasn't moved 4+ pixels → return early (don't follow)
3. Once cursor moves, clear `_cursorPosAtPointEnd` and resume normal follow

### DPI awareness
`GetCursorPos` returns physical pixels. Convert to WPF logical units using:
```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shreshth-s/clicky-windows](https://github.com/shreshth-s/clicky-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
