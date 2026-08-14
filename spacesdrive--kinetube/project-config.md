---
trigger: always_on
description: KineTube is a privacy-first desktop application for downloading YouTube and Instagram media and transcribing it locally with Whisper. It ships as an Electron app for Windows, macOS, and Linux. There is no backend service, no account system, and no telemetry - every request the app makes goes to YouTube, Instagram, or a tool's own release/model host, and every file it produces stays on the user's disk.
---

# KineTube - AI Operating Manual

## Project Identity

KineTube is a privacy-first desktop application for downloading YouTube and Instagram media and transcribing it locally with Whisper. It ships as an Electron app for Windows, macOS, and Linux. There is no backend service, no account system, and no telemetry - every request the app makes goes to YouTube, Instagram, or a tool's own release/model host, and every file it produces stays on the user's disk.

- Primary users: People who want to save YouTube/Instagram media and transcripts without a subscription or a cloud service
- Primary value: One-click download and transcription backed by yt-dlp, FFmpeg, whisper.cpp, and instaloader, auto-installed on first run
- Current platforms: YouTube (video/Shorts/channel/playlist download, MP3 extraction), Instagram (posts/Reels/stories/profile bulk-download), local Whisper transcription
- GitHub repository: https://github.com/spacesdrive/kinetube

---

## Reading Order Before Any Change

Read these in order before writing code. This is not optional - the user has directed that `CLAUDE.md` be read before every task.

1. This file - project identity, rules, and documentation map
2. `docs/WRITING_STANDARDS.md` - typography, icons, writing style, UI copy rules
3. `docs/architecture/OVERVIEW.md` - process topology and request lifecycle
4. `docs/guidelines/JAVASCRIPT.md` - code standards (always apply)
5. The relevant architecture doc - electron, backend, or frontend
6. The relevant feature guide, if one exists in `docs/features/`
7. `docs/philosophy/CROSS_PLATFORM.md` - required reading for anything touching yt-dlp, FFmpeg, whisper.cpp, or instaloader, since today's tool managers are Windows-only (see Known Issues below)

For MCP usage, read `docs/mcp/OVERVIEW.md` before using Context7, Parallel Search, Sequential Thinking, or Filesystem.

---

## Documentation Map

```
docs/
  WRITING_STANDARDS.md        Typography, icons, writing style, UI copy, commit messages
  architecture/
    OVERVIEW.md                Process topology, IPC, request lifecycle, data storage
    electron/
      MAIN_PROCESS.md          main.js responsibilities, window lifecycle, updater, IPC
    backend/
      EXPRESS.md                App structure, middleware, CORS, static serving in prod
      ROUTES.md                 All routes: method, purpose, streaming behavior
      MANAGERS.md               yt-dlp / ffmpeg / whisper / instaloader manager contracts
    frontend/
      REACT_ARCHITECTURE.md    Component tree, state, shadcn usage, SSE consumption
  guidelines/
    JAVASCRIPT.md              Code style, modules, naming, comments policy
    REACT.md                   Component rules, hooks, JSX conventions
    NAMING.md                  File names, function names, variable names
    ERROR_HANDLING.md          Route errors, SSE error events, IPC errors, logging
  workflows/
    FEATURE_DEVELOPMENT.md     Step-by-step feature implementation sequence
    TESTING.md                 Test runner, what is covered, manual verification checklist
    GIT.md                     Commit format, branch rules, changelog policy
    RELEASE.md                 electron-builder targets, versioning, GitHub Actions release flow
  features/
    NEW_API_ROUTE.md           Add a route to the Express backend
    NEW_TOOL_MANAGER.md        Add or extend a managed external binary (yt-dlp-style)
  philosophy/
    ARCHITECTURE.md            Architectural principles and constraints
    CROSS_PLATFORM.md          Windows-only tool bug, what "supporting a platform" requires

DECISIONS.md       Architectural decision log
CHANGELOG.md       Version history and feature changes
ROADMAP.md         Planned features and priorities
```

Every doc referenced above exists in the repository. If you add a new architectural layer, add its doc here in the same pass - do not leave the map pointing at nothing.

---

## Writing and Design Standards

Read `docs/WRITING_STANDARDS.md` for the full specification. The non-negotiable rules:

- Never use emojis, em dashes, or emoticons in code, docs, commit messages, or UI copy (console log lines that predate this policy are legacy - do not add new ones)
- Use Lucide React SVG icons in the UI - never emoji as icons
- Write clear, professional English with no marketing buzzwords or filler text
- UI copy names things from the user's perspective ("Download folder", not "outputDir")
- Error messages explain what went wrong and what to do - never "Something went wrong"
- One comment per non-obvious logic block, maximum - never restate what the code does
- Every page feels like it belongs to the same app: consistent padding, shadcn cards, consistent toast usage (Sonner)

---

## Engineering Standards

Act like a senior engineer shipping a desktop application that runs unattended on someone else's machine, with no server to patch remotely except through the auto-updater.

Every implementation must be:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spacesdrive/kinetube](https://github.com/spacesdrive/kinetube) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
