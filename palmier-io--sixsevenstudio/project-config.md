---
trigger: always_on
description: sixsevenstudio is a native desktop app for generating and editing AI videos with Sora.
---

# What is sixsevenstudio?

sixsevenstudio is a native desktop app for generating and editing AI videos with Sora.

It lets you brainstorm ideas and storyboards with AI assistant, iterate on Sora videos, and edit the videos, all in one place.

It's built with Tauri, React, Vite, TypeScript, TanStack Query, and a local disk storage.

Key features:
- Bring your own OpenAI API key
- Storyboard AI assistant
- Local storage
- Basic video editing (trimming, stitching, transitions)


### Development

```bash
# TypeScript check + Vite build for production
npm run build

# TypeScript type checking (run automatically during build)
npx tsc --noEmit

# Rust format in src-tauri
cargo fmt
```

You should not run `npm run tauri dev`, you should assume users already are running it in the background with hot-reload.

## Architecture Overview

### High-Level Structure

```
Desktop App (Tauri)
├── Frontend Layer (React + TypeScript)
│   ├── Pages (Home, ProjectPage)
│   ├── Components (UI, Editor, Chat, Storyboard, Videos)
│   ├── Hooks (Business logic, Tauri IPC wrappers)
│   ├── State Management (Zustand stores, React Query)
│   └── AI Integration (Vercel AI SDK)
│
├── IPC Bridge (Tauri invoke)
│
└── Backend Layer (Rust)
    ├── Commands (File system, video, projects, API keys)
    ├── FFmpeg Integration (video processing)
    └── File System Management (project persistence)
```

### Core Concepts

#### 1. **Project Structure (Disk Persistence)**

Projects are stored hierarchically under `~/sixsevenstudio/`:
```
<project_name>/
├── .sixseven/
│   ├── metadata.json
│   └── editor_state.json
├── images/
│   └── scene_*_reference.jpg
├── videos/
│   └── <video_id>.mp4
└── storyboard/
    ├── context.md
    └── scenes/
        ├── index.json
        ├── <scene_id>/
        │   └── scene.md
        ├── <scene_id>/
        │   └── scene.md
        └── ...
```

#### 2. **Key files/folders**

1. src-tauri/src/lib.rs - list of all tauri commands
2. src-tauri/src/commands/video_editor - module to interact with ffmpeg for video editing
3. src-tauri/src/commands/projects - module to interact with filesystem to store project data
4. src/App.tsx - Tauri frontend entry point
5. src/lib/ai-sdk.ts - prompt and tools for ai assistant
6. src/lib/openai/ - openai client to videos/images endpoints. using API key stored in Tauri from user.
7. src/hooks/tauri/ - hooks wrapper on Tauri IPC
8. src/components/ - React components (ui, chat, editor, storyboard, videos, tabs)
9. src/pages/ - Top-level page components (Home, ProjectPage)
10. src/tabs/ - Tabs inside project page (Storyboard, Videos, Editor)


#### 3. **AI Integration Architecture**

**Two-Tier System:**

**Tier 1: Video Generation (Sora 2)**
- OpenAI SDK for API calls
- Status polling via `useVideoStatusStore` (src/stores/)
- Video download and caching
- Support for remixing existing videos

**Tier 2: Storyboard Assistant (GPT-4/GPT-5)**
- Vercel AI SDK for streaming LLM responses
- System prompt: `src/lib/ai-sdk.ts` (detailed guidelines, tool definitions)
- Tool-based approach for scene operations
- Streaming responses via `useChat()` hook
- Model selection: gpt-4.1-mini (fast), gpt-5-mini (auto), gpt-5 (thinking)

#### 4. **State Management Strategy**

| Strategy | Usage | Location |
|----------|-------|----------|
| **Zustand** | Video polling state | `src/stores/useVideoStatusStore.ts` |
| **React Query** | Server state, caching | `src/hooks/use-*.ts` |
| **URL params** | Navigation/tab state | React Router |
| **Component state** | UI state (forms, inputs) | Within components |

#### 5. **FFmpeg Integration**

- Bundled executable in `src-tauri/binaries/`
- Backend commands: `create_preview_video`, `export_video`
- Used for: preview generation, video composition, final export
- Cross-platform (Mac, Windows, Linux)

#### 6. **Rich Text Editing**

- TipTap editor for scene descriptions and context
- Markdown-like interface
- Text alignment support
- WYSIWYG editing for better UX

#### 7. **UI**
- Use shadcn in src/components/ui
- App.css uses Tailwind v4

### Data Flow Examples

#### Creating a New Scene (with AI)

```
1. User inputs scene prompt in ChatMessages
2. Frontend sends to useChat() (Vercel AI SDK)
3. LLM processes with system prompt (ai-sdk.ts)
4. LLM uses "create_scene" tool
5. Tool writes scene to project/scenes/scene-N.md (backend command)
6. Frontend receives response, updates scene list
7. Scene appears in StoryboardTab
```

#### Generating a Video

```
1. User creates scene with image + prompt
2. Frontend calls OpenAI Sora API via openai.beta.videoGenerations.create()
3. Async polling: useVideoStatusStore polls status every 15 seconds
4. When ready, download video to project/videos/
5. Update project metadata with video reference
6. Display in VideosTab with preview options
```

#### Exporting Timeline

```
1. User selects clips in VideoEditorTab
2. Saves editor_state.json via backend command
3. Frontend sends export request with clips + transitions
4. Backend runs FFmpeg to compose clips
5. Outputs final video file
6. Backend handles progress reporting
```

## Coding Style
- Typescript formatting: 4-space indentation, Prettier formatting
- Rust formatting: `cargo fmt`
- Typescript: Do not use `any`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [palmier-io/sixsevenstudio](https://github.com/palmier-io/sixsevenstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
