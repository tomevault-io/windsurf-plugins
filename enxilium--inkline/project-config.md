---
trigger: always_on
description: **Inkline** is a desktop application for authors, featuring rich text editing and local/cloud AI generation tools.
---

# Project: Inkline - AI-Powered Storyteller Editor

## 1. Project Overview & Tech Stack

**Inkline** is a desktop application for authors, featuring rich text editing and local/cloud AI generation tools.

### Tech Stack

- **Core**: Electron, TypeScript
- **Frontend**: React, Zustand (State), **Tiptap** (Core Rich Text Editor)
- **Backend/Persistence**: Supabase (Auth, DB, Storage), Local JSON (Structure)
- **AI**: Gemini (API), ComfyUI (Local Edge AI - Parler-TTS, XTTS, Flux)

---

## 2. Architecture Map (Clean Architecture)

The project follows strict **Clean Architecture** principles. The dependency rule is paramount: **Source code dependencies must point only inward, toward higher-level policies.**

```
src/
├── @core/                      # THE INNER CIRCLE (Business Logic)
│   ├── domain/                 # Enterprise Business Rules
│   │   ├── entities/           # Pure Data Objects (e.g., Chapter, Character)
│   │   ├── repositories/       # Interfaces for data access (e.g., IChapterRepository)
│   │   └── services/           # Domain Services interfaces
│   └── application/            # Application Business Rules
│       ├── use-cases/          # Orchestrators (e.g., CreateChapter, GenerateImage)
│       ├── ports/              # Input/Output ports (if strict ports/adapters used)
│       └── daos/               # Data Access Objects (DTOs)
│
├── @infrastructure/            # THE OUTER CIRCLE (Frameworks & Drivers)
│   ├── db/                     # Supabase client, Local File System adapters
│   ├── ai/                     # OpenAI client, ComfyUI WebSocket client
│   └── logging/                # Logger implementations
│
├── @interface-adapters/        # THE GLUE (Controllers & Gateways)
│   ├── controllers/            # Handles IPC events, calls Use Cases
│   └── preload/                # Exposes safe API to Renderer
│
├── main/                       # Electron Main Process Entry Point
└── renderer/                   # React Frontend (Presentation Layer)
```

---

## 3. Rules of Engagement (Strict Enforcement)

### 🔴 @core/domain

- **ALLOWED**: Pure TypeScript.
- **FORBIDDEN**: `axios`, `fs`, `electron`, `react`, or any infrastructure libraries.
- **PURPOSE**: Define _what_ the app is (Entities) and _how_ we access data (Interfaces).
- **NOTE**: `ScrapNote` represents any additional files the user wants to create (e.g. world background, poems, clues) that aren't characters, locations, or chapters.

### 🟠 @core/application

- **ALLOWED**: Imports from `@core/domain`.
- **FORBIDDEN**: `axios`, `fs`, `electron`, UI libraries.
- **PURPOSE**: Orchestrate the flow of data. Contains specific business rules (e.g., "When a chapter is created, ensure it has a unique title").

### 🟡 @interface-adapters

- **ALLOWED**: Imports from `@core/application` and `@core/domain`. Electron `ipcMain`.
- **PURPOSE**: Receive input from the UI (via IPC), convert it to Use Case input, and return formatted results.

### 🟢 @infrastructure

- **ALLOWED**: Imports from `@core/application` and `@core/domain`. External libraries (`@supabase/supabase-js`, `axios`, `ws`).
- **PURPOSE**: Implement the interfaces defined in Domain. Talk to the outside world.

### 🔵 renderer

- **ALLOWED**: React, Zustand, **Tiptap** (Primary Editor).
- **FORBIDDEN**: Direct Node.js APIs (`fs`, `path`), Direct Electron imports (`ipcRenderer` - use `window.api`).
- **PURPOSE**: Display data and capture user intent.

### ⚪ Simplicity & IDs

- Keep every API/request DTO to the bare minimum fields needed for the use case. If a parameter can be derived (IDs, titles, storage paths), derive it instead of accepting it from callers.
- All generated IDs (chapters, assets, conversations, etc.) must be globally unique across projects so collisions can never occur, even between separate projects.
- Entities only store relationship identifiers. Never embed another entity or asset directly (e.g., prefer `character.voiceId`, `location.galleryImageIds`, `project.authorId`).

### 🟣 World Entities

- Characters track `currentLocationId`, `backgroundLocationId`, and a primary `organizationId`. Locations keep cached `characterIds` and `organizationIds` arrays; whenever a character or organization moves, is deleted, or is reassigned, these caches must be updated to stay in sync (e.g. `SaveCharacterInfo` and `SaveOrganizationInfo` must push/pull IDs on both sides).
- Organizations maintain their own `locationIds` array that lists every place they have a presence. Always validate that any referenced location belongs to the same project before saving, and clear both sides of the relationship when deleting.
- Deleting a character, location, or organization must also delete any gallery images, voices, BGMs, or playlists tied to that entity via `IAssetRepository`/`IStorageService` so no orphaned files remain.

### 📝 Behavioral Decisions

- `ChatMessage` objects purposely remain ID-less; ordering is derived solely from their position within a conversation and no per-message mutations are required.
- Chapter creation must accept the caller-provided insertion index so the UI can insert immediately after the focused chapter (e.g., splitting chapter 31 produces a new chapter 32 without an extra move step).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enxilium/inkline](https://github.com/enxilium/inkline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
