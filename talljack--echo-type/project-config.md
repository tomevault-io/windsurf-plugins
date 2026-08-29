---
trigger: always_on
description: > Core knowledge base for AI agents working on this project.
---

# EchoType — AGENTS.md

> Core knowledge base for AI agents working on this project.

## Project Overview

EchoType is an English learning SaaS with four core modules: Listen, Read, Speak, Write, plus AI Chat and Review. Built with Next.js 16 App Router.

## Tech Stack

- **Framework:** Next.js 16 (App Router) + TypeScript + React 19
- **Styling:** Tailwind CSS v4 + shadcn/ui (New York style)
- **State:** Zustand (20 module-specific stores)
- **Local DB:** Dexie.js (IndexedDB) — 9 tables (contents, records, sessions, books, conversations, favorites, favoriteFolders, lookupHistory, translationCache)
- **Cloud:** Supabase (auth + cloud sync)
- **AI:** Vercel AI SDK — 15+ provider support with tool calling
- **Speech:** Web Speech API (SpeechRecognition + SpeechSynthesis), server-side STT fallback
- **TTS:** Browser SpeechSynthesis, Fish Audio, Kokoro, Google Cloud TTS
- **Audio:** wavesurfer.js for waveform visualization
- **Icons:** Lucide React (SVG only, no emojis)
- **Animation:** Framer Motion
- **Sound:** use-sound (Howler.js wrapper)
- **Desktop:** Tauri v2 (Rust backend, sidecar Next.js standalone)
- **Linter:** Biome (primary) + ESLint (Next.js-specific rules only)
- **SRS:** FSRS algorithm for spaced repetition

## Design System

| Token | Value |
|-------|-------|
| Primary | `#4F46E5` (Indigo) |
| Secondary | `#818CF8` |
| CTA/Success | `#22C55E` (Green) |
| Background | `#EEF2FF` |
| Text | `#312E81` |
| Error | `#EF4444` |
| Heading Font | Poppins |
| Body Font | Open Sans |
| Style | Glassmorphism |

## Core Modules

### 1. Listen (`/listen`)
- Play English content via multi-source TTS (Browser, Fish Audio, Kokoro, Google Cloud)
- Content types: articles, phrases, sentences, words
- Interactive transcript — click word to hear it
- Speed control: 0.5x–1.5x
- wavesurfer.js waveform for uploaded audio
- Translation overlay with sentence-level alignment
- AI recommendations panel

### 2. Read (`/read`)
- Shadow reading with speech recognition feedback
- Color-coded live feedback: green (correct), yellow (close), red (wrong)
- Levenshtein distance for word-level accuracy
- Pronunciation assessment via AI
- Translation support
- AI recommendations panel

### 3. Speak (`/speak`)
- Scenario-based conversation practice with AI
- Free conversation mode with topic suggestions
- Voice input via Web Speech API (with server STT fallback for Tauri)
- Real-time AI streaming responses
- Per-message translation toggle
- AI recommendations panel

### 4. Write (`/write`)
- Typing practice inspired by qwerty-learner
- `useReducer` state machine for typing engine
- Character-level comparison: green/red/gray states
- Error pattern: word shakes → 300ms delay → input resets (forces re-type)
- Hidden input captures keystrokes (NOT controlled input)
- Timer: starts on first keystroke, WPM = (words / seconds) × 60
- Error word review loop with auto-retry
- Sound effects: correct keystroke, error beep, word complete
- AI recommendations panel

### 5. AI Chat (Global floating panel)
- Vercel AI SDK with tool calling (library search, pronunciation, translate, etc.)
- Floating button (bottom-right) → slide-up chat panel
- Multi-provider: OpenAI/Claude/DeepSeek/GLM/Groq/OpenRouter and more
- Context-aware: knows current module + content being practiced
- Conversation history stored in Dexie
- Configurable maxOutputTokens (global + per-provider)

### 6. Review (`/review/today`)
- FSRS-based spaced repetition review queue
- Daily plan with progress tracking
- Rating buttons for review feedback
- Review forecast analytics

## Shared Data Model

```typescript
interface ContentItem {
  id: string;           // nanoid
  title: string;
  text: string;
  type: 'article' | 'phrase' | 'sentence' | 'word';
  category?: string;
  tags: string[];
  source: 'builtin' | 'imported' | 'ai-generated';
  difficulty?: 'beginner' | 'intermediate' | 'advanced';
  createdAt: number;
  updatedAt: number;
}

interface LearningRecord {
  id: string;
  contentId: string;
  module: 'listen' | 'speak' | 'read' | 'write';
  attempts: number;
  accuracy: number;
  wpm?: number;
  mistakes: MistakeEntry[];
  nextReview?: number;
  fsrsCard?: FSRSCard;   // FSRS scheduling data
  updatedAt: number;
}
```

## Dexie Schema (v15)

```typescript
contents: 'id, type, category, source, difficulty, createdAt, updatedAt, *tags'
records: 'id, contentId, module, lastPracticed, nextReview, updatedAt'
sessions: 'id, contentId, module, startTime, completed'
books: 'id, title, source, createdAt'
conversations: 'id, updatedAt, createdAt'
favorites: 'id, normalizedText, type, folderId, sourceContentId, targetLang, nextReview, autoCollected, createdAt, updatedAt'
favoriteFolders: 'id, sortOrder, createdAt'
lookupHistory: 'text, count, lastLookedUp'
translationCache: 'key, createdAt'
mediaBlobs: 'contentId, createdAt'
alignmentCache: 'cacheKey, createdAt'
weakSpots: 'id, module, weakSpotType, normalizedText, lastSeenAt, resolved, [module+weakSpotType+normalizedText]'
collections: 'id, category, source, difficulty, createdAt, updatedAt, *tags'
journals: 'id, lessonDate, source, updatedAt, *tags'
```

## Routing Structure

```
app/
├── layout.tsx                  # Root: fonts, providers, floating chat
├── page.tsx                    # Landing page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Talljack/echo-type](https://github.com/Talljack/echo-type) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
