---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BiliBili ShadowReplay is a Tauri-based desktop application for caching live streams and performing real-time editing and submission. It supports multiple streaming platforms including Bilibili, Douyin (TikTok China), Huya, Kuaishou, and TikTok.

**Architecture**: Hybrid application with Svelte 3 frontend and Rust backend, using Tauri 2 for desktop integration.

## Development Commands

### Frontend Development
- `yarn dev` - Start Vite development server (frontend only)
- `yarn build` - Build production frontend
- `yarn check` - Run TypeScript and Svelte type checking

### Full Application Development
- `yarn tauri dev` - Start Tauri development with hot reload (recommended for full-stack development)
- `yarn tauri build` - Build production desktop application

### Rust Backend
- `cd src-tauri && cargo check` - Check Rust code without building
- `cd src-tauri && cargo test` - Run all Rust tests
- `cd src-tauri && cargo test <test_name>` - Run specific test

### Platform-Specific Builds
- **Windows CPU**: `yarn tauri dev` (default)
- **Windows CUDA**: `yarn tauri dev --features cuda` (requires CUDA Toolkit and LLVM)
- **macOS**: Requires `SDKROOT` and `CMAKE_OSX_DEPLOYMENT_TARGET=13.3` environment variables
- **Linux**: No special configuration needed

### Documentation
- `yarn docs:dev` - Start VitePress documentation server
- `yarn docs:build` - Build documentation site
- `yarn docs:preview` - Preview built documentation

### Version Management
- `yarn bump` - Run version bump script

## Architecture Overview

### Frontend (Svelte 3 + TypeScript)

**Entry Points**:
- `src/main.ts` - Main application entry
- `src/main_clip.ts` - Clip editing interface
- `src/main_live.ts` - Live streaming interface

**Key Directories**:
- `src/page/` - Page components (Room, Task, AI, etc.)
- `src/lib/components/` - Reusable UI components
- `src/lib/stores/` - Svelte stores for global state management
- `src/lib/agent/` - AI agent implementation using LangChain
- `src/lib/db.ts` - Frontend database interface

**Styling**: Tailwind CSS with Flowbite components

### Backend (Rust + Tauri 2)

**Main Entry**: `src-tauri/src/main.rs`

**Core Modules**:
- `src-tauri/src/recorder_manager.rs` - Main recording orchestration
- `src-tauri/src/handlers/` - Tauri command handlers (frontend-backend bridge)
- `src-tauri/src/database/` - SQLite database operations using sqlx
- `src-tauri/src/subtitle_generator/` - AI-powered subtitle generation with Whisper
- `src-tauri/src/ffmpeg/` - FFmpeg integration for video processing
- `src-tauri/src/progress/` - Progress tracking for recording tasks
- `src-tauri/src/http_server/` - HTTP server for streaming
- `src-tauri/src/migration/` - Database schema migration system

**Custom Workspace Crates**:
- `src-tauri/crates/danmu_stream/` - Danmaku (bullet comment) stream processing library
- `src-tauri/crates/recorder/` - Core recording functionality with platform-specific implementations

**Platform Support** (`src-tauri/crates/recorder/src/platforms/`):
- `bilibili/` - Bilibili live stream recording
- `douyin/` - Douyin (TikTok China) recording
- `huya/` - Huya platform support
- `kuaishou/` - Kuaishou platform support
- `tiktok/` - TikTok international support

### Key Technologies

**Frontend**:
- Svelte 3 with TypeScript
- Vite for build tooling
- Tailwind CSS + Flowbite for UI
- LangChain for AI features (@langchain/core, @langchain/deepseek, @langchain/ollama)
- WaveSurfer.js for audio visualization
- Socket.io for real-time communication

**Backend**:
- Tauri 2 for desktop integration
- SQLite with sqlx (async runtime, WAL mode)
- FFmpeg via async-ffmpeg-sidecar
- Whisper-rs for speech-to-text (with optional CUDA/Metal acceleration)
- M3U8-rs for HLS stream processing
- Tokio async runtime
- Axum for HTTP server
- Socketioxide for WebSocket support

### Database Architecture

- **Primary Storage**: SQLite with Write-Ahead Logging (WAL mode)
- **Location**: `src-tauri/data/data_v2.db`
- **Migration System**: Automatic schema updates via `src-tauri/src/migration.rs`
- **Data Models**: Recording metadata, room configurations, task status, user preferences

### AI Features

**Whisper Integration**:
- Local speech-to-text transcription
- Platform-specific acceleration:
  - Windows: Optional CUDA support via `cuda` feature flag
  - macOS: Metal acceleration enabled by default
  - Linux: CPU-based inference

**LangChain Integration**:
- AI agent for content analysis and summarization
- Support for multiple LLM providers (DeepSeek, Ollama)
- Located in `src/lib/agent/` directory

## Development Guidelines

### Frontend Development

- Use Svelte 3 syntax with `<script>` tags
- Prefer reactive statements with `$:` for derived state
- Use stores from `src/lib/stores/` for global state
- Follow TypeScript strict mode configuration
- Use Tailwind CSS classes for styling

### Rust Backend Development

- Follow workspace structure with custom crates
- Use async/await with Tokio runtime
- Implement proper error handling with thiserror
- Use prepared statements for SQL to prevent injection

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Xinrea/bili-shadowreplay](https://github.com/Xinrea/bili-shadowreplay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
