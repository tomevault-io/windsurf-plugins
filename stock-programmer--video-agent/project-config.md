---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **AI-driven video generation SaaS platform** (MVP stage) that enables human-AI collaborative creative video production workflows. The core feature is **Image-to-Video** generation.

**Version Status:**
- **v1.0 (MVP)**: ✅ Completed and deployed (Dec 2024)
- **v1.1**: ✅ Completed (Jan 2025) - Enhanced video generation parameters

**Key characteristics:**
- Single-user assumption (no authentication system in MVP)
- Near real-time state synchronization (draft-like auto-save)
- Flexible third-party API integration (video generation and LLM services)
- Streamlined architecture focusing on core functionality

**Current Status (v1.1):**
- **Frontend**: ✅ Updated with enhanced video generation controls
  - Duration control (5s/10s/15s) with dropdown selector
  - Aspect ratio selection (16:9/9:16/1:1/4:3) with icon buttons
  - Motion intensity slider (1-5 scale) with visual feedback
  - Quality preset selector (draft/standard/high) with time estimates
  - All core components updated for v1.1
  - Backward compatible with v1.0 workspaces
  - Zustand state management with default value handling
  - API and WebSocket client services ready
  - Responsive UI with horizontal scrolling timeline
- **Backend**: ✅ Updated with v1.1 parameter validation and handling
  - All core modules updated for v1.1
  - REST API endpoints with v1.1 parameter validation
  - WebSocket handlers support incremental updates for v1.1 fields
  - Default value handling for backward compatibility
  - Third-party service integrations (Qwen video + Gemini LLM) completed
  - Winston logging configured
  - Integration tests passed (v1.0 + v1.1)
- **Third-party APIs**: ✅ Verified and Integrated
  - Qwen video generation (DashScope wan2.6-i2v) - tested and integrated
  - Google Gemini 3 LLM (gemini-3-flash-preview) - tested and integrated

## Core Architecture

### Frontend
- **Tech Stack**: React 19 + TypeScript + Vite + TailwindCSS 4
- **State Management**: Zustand
- **Data Fetching**: Axios + TanStack React Query
- **Drag & Drop**: dnd-kit
- **Layout**: Horizontal scrolling timeline with multiple workspaces
- **Key Features**:
  - Image upload
  - Video generation form with v1.1 enhancements:
    - Camera movement, shot type, lighting, motion prompts (v1.0)
    - Duration control (5s/10s/15s) (v1.1)
    - Aspect ratio selection (16:9/9:16/1:1/4:3) (v1.1)
    - Motion intensity slider (1-5) (v1.1)
    - Quality preset (draft/standard/high) (v1.1)
  - Video player
  - AI collaboration assistant

**Implemented Components:**
- `Timeline.tsx` - Horizontal scrolling workspace timeline
- `Workspace.tsx` - Individual workspace container
- `ImageUpload.tsx` - Image upload with drag & drop support
- `VideoForm.tsx` - Video generation form with validation
- `VideoPlayer.tsx` - Video playback component
- `AICollaboration.tsx` - AI suggestion interface
- `LoadingSpinner.tsx` - Loading state component
- `ErrorMessage.tsx` - Error display component
- `EmptyState.tsx` - Empty state placeholder

**Frontend Services:**
- `api.ts` - REST API client with Axios
- `websocket.ts` - WebSocket client for real-time sync

**State Management:**
- `workspaceStore.ts` - Zustand store for workspace state management
  - v1.1: Includes default value handling for new parameters
  - v1.1: Debounced WebSocket sync (300ms) for form updates

**Type Definitions:**
- `workspace.ts` - TypeScript interfaces for workspace data
  - v1.1: Added Duration, AspectRatio, MotionIntensity, QualityPreset types
  - v1.1: Added validation functions (isDuration, isAspectRatio, etc.)
  - v1.1: Added default constants (DEFAULT_DURATION, DEFAULT_ASPECT_RATIO, etc.)

### Backend
- **Tech Stack**: Node.js + Express + WebSocket (✅ Implemented)
- **Database**: MongoDB with Mongoose ODM (✅ Implemented)
- **Communication**: WebSocket for near real-time state sync (✅ Implemented)
- **File Storage**: Local filesystem (`uploads/`) for MVP, designed for easy migration to OSS
- **Third-party APIs**:
  - Video Generation: Qwen (DashScope API) - using wan2.6-i2v model (✅ Integrated)
  - LLM Services: Google Gemini 3 (gemini-3-flash-preview) (✅ Integrated)

**Critical Design Principles**:
- **Simple and direct**: No task queues, caching layer, or monitoring services in MVP
- **High cohesion**: Single-file modules (one file = one complete feature)
- **AI-friendly**: No traditional layered architecture (routes/services/models separation)
- **Flexible integration**: Switch third-party providers via config, no code changes needed

**Implemented Modules:**
- **Core Infrastructure**:
  - `server.js` - HTTP + WebSocket server startup
  - `app.js` - Express application setup with middleware
  - `config.js` - Environment configuration management
  - `db/mongodb.js` - MongoDB connection + Workspace model
  - `utils/logger.js` - Winston logging utility

- **REST API Endpoints** (`api/`):
  - `upload-image.js` - Image upload with Multer
  - `get-workspaces.js` - Fetch all workspaces
  - `generate-video.js` - Trigger video generation
  - `ai-suggest.js` - AI collaboration suggestions

- **WebSocket Handlers** (`websocket/`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stock-programmer/video-agent](https://github.com/stock-programmer/video-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
