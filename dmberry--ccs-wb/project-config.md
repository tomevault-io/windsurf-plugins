---
trigger: always_on
description: This file provides context for Claude Code or other AI assistants working on this project.
---

# CCS Workbench - AI Assistant Context

This file provides context for Claude Code or other AI assistants working on this project.

## Project Overview

**CCS-WB** (Critical Code Studies Workbench) is a web application for close reading and hermeneutic analysis of software as cultural artefact. It implements critical code studies methodology based on the work of Mark Marino and David M. Berry.

**Version**: 3.3.0 | CCS Methodology v2.7

## Technology Stack

- **Framework**: Next.js 16 with React 19, TypeScript 5 (strict)
- **Bundler**: Turbopack (Next.js 16 default)
- **Styling**: Tailwind CSS with custom editorial design tokens
- **State**: React Context + useReducer (no external state library)
- **UI Components**: Radix UI primitives, Lucide icons
- **Code Editor**: CodeMirror 6 with custom language modes and annotation extensions
- **AI Integration**: Multi-provider (Anthropic, OpenAI, Google, Ollama, OpenRouter, Hugging Face, OpenAI-Compatible) via Vercel AI SDK
- **Cloud**: Supabase (auth, project storage, real-time sync)
- **PDF Export**: jsPDF

## Key Architecture Decisions

### Entry Modes
The app has three entry modes, each with distinct UI, AI engagement style, and prompts:
- `critique` (Analyze Code) - IDE-style three-panel layout (file tree, code editor, chat). AI acts as expert practitioner.
- `interpret` (Learn Methods) - Exploring CCS methodology and hermeneutic frameworks. AI acts as beginner-friendly guide.
- `create` (Create Code) - "Vibe coding" to understand algorithms by building them. AI acts as intermediate practitioner.

### Session State
All session state flows through `SessionContext.tsx` using useReducer. The `Session` type in `types/session.ts` is the central data structure.

### CCS Methodology
The LLM system prompts are generated from `Critical-Code-Studies-Skill.md` (a skill document loaded at runtime via `lib/prompts/ccs-methodology.ts`). This contains the CCS methodology, conversation phases, and annotation types.

### Cloud Collaboration
Supabase powers cloud projects with OAuth (Google, GitHub, Apple), real-time sync (5-second polling), shareable invite links, and member management. Connection resilience uses an operation queue with IndexedDB for offline support.

### Custom Skins
10 nostalgic visual themes (Atari 2600, BBC Micro, C64, ELIZA, Geocities, HyperCard, Myspace, Teams, Teletext, Vaporwave) managed via `SkinsContext.tsx` and loaded from `public/skins/`.

## Directory Structure

```
src/
├── app/                          # Next.js app router
│   ├── api/                      # API routes
│   │   ├── analyze/              # Code analysis
│   │   ├── chat/                 # Main dialogue API
│   │   ├── export/               # Export operations
│   │   ├── generate/             # Output generation
│   │   ├── literature/           # Literature search
│   │   ├── profile/              # User profile
│   │   ├── skill-document/       # CCS methodology loader
│   │   ├── test-connection/      # AI provider connection test
│   │   ├── upload/               # File upload
│   │   └── version/              # Version endpoint
│   ├── auth/                     # OAuth callback handling
│   ├── conversation/page.tsx     # Main conversation page
│   ├── invite/                   # Shareable invite links
│   └── page.tsx                  # Landing page with mode selection
├── components/
│   ├── auth/                     # LoginModal, UserMenu
│   ├── ccs/                      # CCS guidance panel, method cards, smart hints
│   ├── chat/                     # WorkbenchChatPanel, ContextPreview, MessageBubble
│   ├── code/                     # CodeEditorPanel, CodeMirrorEditor, CodeDiffViewer,
│   │                             # AnnotatedCodeViewer, cm-annotations*, cm-lang-*, cm-theme
│   ├── easter-eggs/              # Clippy
│   ├── layouts/                  # WorkbenchLayout (orchestrator), WorkbenchHeader, WorkbenchModals
│   ├── projects/                 # ProjectsModal, LibraryModal, MembersModal, AdminModal
│   ├── prompts/                  # GuidedPrompts (phase-appropriate questions)
│   ├── pwa/                      # InstallPrompt, AlphaFavicon
│   ├── settings/                 # SettingsModal, AIProviderSettings, AISettingsPanel
│   ├── shared/                   # ConfirmDialog, ConnectionStatus, SaveStatusIndicator, Toast
│   └── ui/                       # Base UI primitives
├── hooks/                        # 24 custom hooks
│   ├── useWorkbenchChat.ts       # Chat state and AI messaging
│   ├── useWorkbenchProject.ts    # Project save/load/export
│   ├── useWorkbenchFileManagement.ts # File add/remove/rename
│   ├── useAnnotation*.ts         # Annotation suggestions, replies, sync
│   ├── useProject*.ts            # CRUD, save, sharing, sync, trash, admin, library, members, modals
│   ├── useCollaborativeSession.ts # Collaborative session management
│   ├── useConnectionHealth.ts    # Connection health monitoring
│   ├── useCCSGuidance.ts         # CCS methodology guidance
│   └── ...                       # AutoSave, CodeFilesSync, LibraryRatings, ReferenceSearch, etc.
├── context/
│   ├── SessionContext.tsx        # Main session state (useReducer)
│   ├── AISettingsContext.tsx      # AI provider config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dmberry/CCS-WB](https://github.com/dmberry/CCS-WB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
