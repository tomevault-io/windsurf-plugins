---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Briefly** is a Chrome extension (Manifest V3) that generates AI-powered summaries of YouTube videos by extracting transcripts and processing them with OpenAI's GPT models. This codebase was reverse-engineered from minified production code and rebuilt with modern TypeScript, React 19, and Vite 7.

## Build Commands

```bash
# Install dependencies
npm install

# Development build with hot reload
npm run dev

# Production build (outputs to dist/)
npm run build

# Preview production build
npm run preview
```

**After building**, load the extension in Chrome:
1. Navigate to `chrome://extensions/`
2. Enable "Developer mode"
3. Click "Load unpacked" and select the `dist/` folder
4. After code changes, click the reload icon on the extension card

## Architecture

### Chrome Extension Structure

This is a **Manifest V3** extension with three isolated execution contexts that communicate via Chrome's message passing API:

1. **Background Service Worker** (`src/background/index.ts`)
   - Orchestrates transcript extraction and OpenAI summarization
   - Manages per-tab badge state and system notifications
   - Tracks in-progress summaries in `chrome.storage.local`

2. **Content Script** (`src/content/index.ts`)
   - Injected into all `youtube.com` pages
   - Has direct DOM access to the YouTube page
   - Handles two types of messages:
     - `GET_TRANSCRIPT_DATA`: Clicks transcript button and extracts transcript from DOM
     - `SEEK_VIDEO`: Controls video playback (seek, play, scroll)
   - Cannot access Chrome extension APIs or popup state

3. **Popup UI** (`src/popup/`)
   - React application shown when clicking extension icon
   - Starts summaries by sending `START_SUMMARY` to the background worker
   - Listens for `SUMMARY_PROGRESS`, `SUMMARY_DONE`, and `SUMMARY_ERROR` messages
   - Has access to Chrome APIs but NOT to page DOM
   - Persists settings and viewed-summary state to `chrome.storage.local`

### Message Passing Pattern

Communication between popup and content script uses Chrome's message passing:

```typescript
// Popup sends message to content script
chrome.tabs.sendMessage(tabId, {
  type: 'SEEK_VIDEO',
  time: timeInSeconds
});

// Content script receives and responds
chrome.runtime.onMessage.addListener((message, _sender, sendResponse) => {
  if (message.type === 'SEEK_VIDEO') {
    // Manipulate video element
    sendResponse({ success: true });
  }
  return true; // Keep channel open for async response
});
```

### Data Flow

1. **User clicks "Summarize This Video"** in popup
2. **Popup** extracts video ID from active tab URL
3. **Popup → Background**: popup sends `START_SUMMARY` with `{ videoId, videoTitle, tabId, openaiApiKey, comfortableLanguages }`.
4. **YouTube Transcript Extraction** (`src/utils/youtube.ts` + `src/content/transcriptDom.ts`):
   - Sends `GET_TRANSCRIPT_DATA` message to content script
   - Content script clicks "Show transcript" button on YouTube
   - Waits for transcript panel to load
   - Extracts transcript segments from DOM (`ytd-transcript-segment-renderer` elements)
   - Parses timestamps (MM:SS or H:MM:SS format) and text
   - Returns `TranscriptEntry[]` with timestamped text
5. **OpenAI Summary Generation** (`src/utils/openai.ts`):
   - Formats transcript with timestamps
   - Calls OpenAI Chat Completions API (GPT-4o-mini)
   - Returns markdown-formatted summary with clickable timestamps
6. **Background → Storage & Badge**:
   - Stores summary under `summary-{videoId}` and clears `summaryInProgress`
   - Sets a green per-tab badge (`✓`) and optional notification when summary is ready
7. **Background → Popup**:
   - Sends `SUMMARY_PROGRESS`, `SUMMARY_DONE`, or `SUMMARY_ERROR` messages to any open popup
8. **Popup**:
   - Reads summaries from `chrome.storage.local` for the active video
   - Clears the `✓` badge for the active tab the first time the user views that summary
9. **User clicks timestamp** in summary
10. **Popup** sends `SEEK_VIDEO` message to content script
11. **Content script** seeks video, plays if paused, scrolls into view

### State Management

No external state library - popup state is in `App.tsx` using React hooks:
- `settings`: API keys loaded from and persisted to `chrome.storage.local`
- `summary`: Summary for the active video (includes `videoId`, title, content, timestamp, `viewedByUser`)
- `loading` / `loadingMessage`: Progress indicators driven by background messages
- `error`: String for error messages derived from structured error codes
- `showSettings`, `isExpanded`, `isYouTubeVideo`: Local UI state
- Background worker maintains `summaryInProgress` and per-video summaries in `chrome.storage.local`

## Key Implementation Details

### YouTube Transcript Extraction

**Critical**: The extension extracts transcripts by interacting with YouTube's native transcript UI:

1. Background worker calls `getYouTubeTranscript(videoId)` in `src/utils/youtube.ts`.
2. That helper sends `GET_TRANSCRIPT_DATA` to the content script.
3. Content script uses `src/content/transcriptDom.ts` to search for transcript entry points using DOM selectors:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alekko-dev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
