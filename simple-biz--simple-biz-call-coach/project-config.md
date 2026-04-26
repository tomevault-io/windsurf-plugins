---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Simple.Biz Call Coach is a Chrome extension that provides real-time AI-powered call coaching for CallTools agents. It captures audio from CallTools.io tabs, transcribes it using Deepgram's WebSocket API, and displays live transcriptions and coaching tips in a side panel.

## Development Commands

### Build and Development
- `npm run dev` - Start Vite development server with hot reload
- `npm run build` - Build extension for production (runs TypeScript compiler + Vite build)
- `npm run preview` - Preview production build

### Loading the Extension
1. Build the extension with `npm run build`
2. Open Chrome and navigate to `chrome://extensions/`
3. Enable "Developer mode"
4. Click "Load unpacked" and select the `dist` folder

## Architecture

### Chrome Extension Components

This is a Manifest V3 Chrome extension with the following components:

1. **Background Service Worker** (`src/background/index.ts`)
   - Central state management hub
   - Coordinates between all components via message passing and port connections
   - Manages tab capture lifecycle and offscreen document
   - Handles state persistence to chrome.storage.local
   - Generates basic coaching tips based on transcription patterns
   - Key state includes: `isRecording`, `isOnCall`, `deepgramStatus`, `transcriptions`, `coachingTips`

2. **Content Script** (`src/content/index.ts`)
   - Injected into `*://*.calltools.io/*` pages
   - Detects active calls using multiple DOM detection methods:
     - Timer element with format `HH:MM:SS`
     - "On a Call" status text
     - Red hangup button visibility
   - Uses port-based connection with background for reliable messaging
   - Implements debouncing (3 confirmations for active, 5 for inactive) to avoid false positives
   - Auto-reconnects if port disconnects

3. **Offscreen Document** (`src/offscreen/index.ts`)
   - Required for tab audio capture (Manifest V3 restriction)
   - Receives audio data via MessagePort from WebRTC Bridge (alternative to tab capture)
   - Manages Deepgram WebSocket connection for live transcription
   - Converts audio to linear16 PCM format for Deepgram
   - Monitors audio levels and broadcasts to UI
   - Handles Deepgram reconnection on connection failure
   - Supports dual audio channels: caller (remote) and agent (local)

3a. **WebRTC Interceptor** (`src/injected/webrtc-interceptor.ts`)
   - Injected into page context (not isolated content script context)
   - Proxies native `RTCPeerConnection` constructor to intercept WebRTC streams
   - Captures both remote (caller) and local (agent mic) audio tracks
   - Hooks `ontrack`, `addEventListener('track')`, and `addTrack` methods
   - Exposes `__getInterceptedStreams()` function to window for content script access
   - Posts messages to window for content script detection (cross-context communication)

3b. **WebRTC Bridge** (`src/content/webrtc-bridge.ts`)
   - Runs in content script context (has chrome API access)
   - Receives intercepted streams from injected script via window messages
   - Creates AudioContext to process both caller and agent audio separately
   - Uses ScriptProcessor nodes to convert Float32 to Int16 PCM
   - Establishes MessageChannel for direct port-based communication with offscreen
   - Streams processed audio chunks to offscreen document with speaker labels ('caller' or 'agent')
   - **Audio Loopback**: Connects processors to `audioContext.destination` for playback

4. **Popup** (`src/popup/`)
   - Browser action popup for quick controls
   - Login/logout functionality (stores user email)
   - Detects if current tab is CallTools.io
   - Shows call status and initiates coaching sessions
   - Polls call state every 2 seconds when on CallTools tab

5. **Side Panel** (`src/sidepanel/`)
   - Main UI for viewing transcriptions and coaching tips
   - Displays live transcriptions with interim/final states
   - Shows Deepgram connection status
   - Audio level visualization
   - Auto-scrolls to latest transcription
   - Persists and syncs state with background via chrome.storage.local

### State Management

- **Zustand Store** (`src/stores/call-store.ts`): React state management for UI components
  - Syncs with chrome.storage.local for persistence
  - Automatically loads persisted state on initialization
  - State includes: `callState`, `audioState`, `deepgramStatus`, `transcriptions`, `coachingTips`, `audioLevel`

- **Settings Store** (`src/stores/settings-store.ts`): User configuration
  - Deepgram API key
  - n8n webhook URL
  - Audio sensitivity
  - Theme preferences

### Message Flow Architecture

The extension uses a hybrid messaging approach:

1. **Port-based connections**: Content script ↔ Background (persistent, auto-reconnect)
2. **Runtime messages**: Background ↔ Popup/SidePanel/Offscreen (broadcast pattern)
3. **Storage events**: State synchronization across components

Key message types:
- `CALL_STARTED` / `CALL_ENDED` - Call lifecycle from content script
- `START_CAPTURE` / `STOP_CAPTURE` - Audio capture control to offscreen
- `GET_WEBRTC_STREAMS` / `STOP_WEBRTC_STREAMING` - WebRTC stream control to bridge

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Simple-biz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
