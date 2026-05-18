---
trigger: always_on
description: Handles camera access and video recording.
---

# Freewrite - Technical Documentation for AI Agents

> **⚠️ IMPORTANT FOR AI AGENTS**: This file (`AGENTS.md`) and `CLAUDE.md` are clones and must be kept in sync.
>
> Update both files when changes are **substantial** and meaningfully affect future agent understanding (for example: architecture, data flow, storage/permissions, threading behavior, user-facing workflows, or major bug fixes).
>
> For minor tweaks (small UI polish, copy edits, tiny refactors), updates are optional. Do **not** churn these docs on every small code change.
>
> If one file is updated, mirror the same change in the other file immediately.

## Product Vision & User Experience

### What is Freewrite?

Freewrite is a **distraction-free writing environment** for macOS designed around the concept of stream-of-consciousness writing and video journaling. The core philosophy is to remove barriers between thought and writing by creating a minimalist, opinionated interface that prioritizes the act of writing over formatting, organization, or editing.

### User Experience Philosophy

**Core Principles:**
1. **No Backspace (Optional)**: Users can disable the backspace key to encourage forward-thinking writing without self-editing
2. **Timed Sessions**: Built-in timer (default 15 minutes) creates focused writing sprints
3. **Auto-Everything**: Auto-save, auto-new-entry, auto-timestamp - the app manages logistics so users can focus on writing
4. **Local-First**: All data stays on the user's machine in plain markdown files they can access directly
5. **Minimal UI**: Most UI elements hide during timed sessions, leaving only the text

### Use Cases

**Primary Use Case - Stream of Consciousness Writing:**
- Users open the app and start writing immediately (no "New Document" dialog)
- The app creates a new entry automatically at the start of each day
- Writing is saved continuously with no manual save action
- Timer creates urgency and prevents over-editing
- Backspace disable forces forward momentum in thinking

**Secondary Use Case - Video Journaling:**
- Quick video capture for visual thoughts/ideas
- Video entries stored alongside text entries in chronological history
- One-click recording with built-in timer
- Local storage ensures privacy for personal video journals

**Tertiary Use Case - AI-Assisted Reflection:**
- "Chat" button sends writing to ChatGPT or Claude
- Prompts designed to help users reflect on and understand their writing
- AI provides feedback, questions, or analysis of stream-of-consciousness text

**Hidden Power Feature - Long-Form Writing:**
- Despite minimalist interface, supports full markdown
- Entries can be exported as PDFs
- Font and size customization for comfort during long sessions
- Dark mode for night writing

## Overview

Freewrite is a native macOS writing application built with SwiftUI that allows users to write text entries and record video entries. All data is stored locally in `~/Documents/Freewrite/`.

## Architecture

### Technology Stack
- **Framework**: SwiftUI (macOS)
- **Minimum macOS Version**: 14.0
- **Language**: Swift 5.0
- **Build System**: Xcode
- **Media**: AVFoundation for camera/video recording

### Project Structure

```
freewrite/
├── freewrite.xcodeproj/          # Xcode project file
├── freewrite/
│   ├── freewriteApp.swift        # App entry point
│   ├── ContentView.swift         # Main view (1400+ lines)
│   ├── VideoRecordingView.swift  # Video recording interface
│   ├── VideoPlayerView.swift     # Video playback interface
│   └── freewrite.entitlements    # App permissions
├── CLAUDE.md                     # This file
└── AGENTS.md                     # Duplicate of this file
```

## Data Model

### Entry Types

```swift
enum EntryType {
    case text
    case video
}

struct HumanEntry: Identifiable {
    let id: UUID
    let date: String              // Display format: "MMM d" (e.g., "Feb 20")
    let filename: String          // Format: [UUID]-[YYYY-MM-DD-HH-mm-ss].md
    var previewText: String       // First 30 chars or "Video Entry"
    var entryType: EntryType      // .text or .video
    var videoFilename: String?    // Format: [UUID]-[YYYY-MM-DD-HH-mm-ss].mov
}
```

### File Storage

**Location**: `~/Documents/Freewrite/`

**Text Entries**:
- Format: Markdown (.md)
- Naming: `[UUID]-[YYYY-MM-DD-HH-mm-ss].md`
- Content: Plain UTF-8 text
- Example: `[6910BBDE-75FC-415C-ABB9-C76644B037B2]-[2026-02-20-08-01-04].md`

**Video Entries**:
- Format: QuickTime Movie (.mov)
- Naming: `[UUID]-[YYYY-MM-DD-HH-mm-ss].mov`
- Metadata: Corresponding `.md` file with "Video Entry" text in `~/Documents/Freewrite/`
- Storage layout: `~/Documents/Freewrite/Videos/[UUID]-[YYYY-MM-DD-HH-mm-ss]/`
- Directory contents:
  - `[UUID]-[YYYY-MM-DD-HH-mm-ss].mov`
  - `thumbnail.jpg`
  - `transcript.md` (optional; speech transcript for that recording)
- Example directory: `~/Documents/Freewrite/Videos/[6910BBDE-75FC-415C-ABB9-C76644B037B2]-[2026-02-20-08-01-04]/`

## Key Components

### ContentView.swift

The main view containing all UI and business logic.

#### State Variables (Selection)

```swift
@State private var entries: [HumanEntry] = []           // All loaded entries
@State private var text: String = ""                    // Current text editor content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farzaa/freewrite](https://github.com/farzaa/freewrite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
