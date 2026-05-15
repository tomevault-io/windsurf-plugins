---
trigger: always_on
description: This file provides comprehensive guidance for LLM agents (like Claude Code, Cursor, GitHub Copilot) when working with this codebase. Following these guidelines ensures consistent, high-quality contributions.
---

# AGENTS.md - LLM Agent Guidelines

This file provides comprehensive guidance for LLM agents (like Claude Code, Cursor, GitHub Copilot) when working with this codebase. Following these guidelines ensures consistent, high-quality contributions.

## Quick Start Checklist

Before starting any task:
1. Read this file completely
2. Read README.md for project overview
3. Explore relevant package documentation in `src/[package-name]/`
4. Understand the task requirements completely
5. Create a task plan and get user approval

## Understanding This Codebase

### Project Overview
This is a browser-based video editor that processes videos entirely client-side using modern web technologies (WebAssembly, WebCodecs, WebGPU). The application is built with TypeScript and follows a modular, event-driven architecture.

### Package Organization
The codebase is organized into self-contained packages under `src/`:

- `canvas/` - Canvas rendering and visual effects
- `common/` - Shared utilities, events, and helpers
- `mediaclip/` - Media clip abstractions (audio, video, image, text)
- `medialibrary/` - Media asset management
- `recording/` - Screen and camera recording
- `search/` - Video content search using AI
- `shape/` - Shape and graphics primitives
- `speech/` - Text-to-speech functionality
- `studio/` - Main editor UI and orchestration
- `timeline/` - Timeline management and playback control
- `transcription/` - Audio-to-text transcription
- `video/` - Video encoding/decoding (mux/demux)

### Key Architectural Principles
- **Package Independence**: Each package should be self-contained with minimal coupling
- **Event-Driven Communication**: Packages communicate via events, not direct method calls
- **Browser-First**: No Node.js-specific APIs; everything runs in the browser
- **Class-Based Design**: One class per file, using PascalCase naming
- **Public API Focus**: Only test and document public APIs

### Domain Knowledge: Video Editing Concepts
- **Timeline**: Linear sequence of clips with layers (video, audio, text overlays)
- **Clip**: A segment of media with start/end points and transformations
- **Demux**: Extracting audio/video streams from a container format
- **Mux**: Combining audio/video streams into a container format
- **Codec**: Algorithm for encoding/decoding video (e.g., H.264, VP9)
- **Frame**: Single image in a video sequence
- **Sample Rate**: Audio samples per second (e.g., 44.1kHz, 48kHz)

## Task Execution Plan

### Step 1: Understand the Task
- Read the user's request carefully
- Ask clarifying questions if requirements are ambiguous
- Identify which packages will be affected
- Determine if this is a bug fix, feature addition, refactor, or documentation task

### Step 2: Explore the Codebase
- Use search tools to find relevant files
- Read existing implementations to understand patterns
- Check for similar functionality that can be referenced
- Review tests to understand expected behavior

### Step 3: Create an Execution Plan
**Important**: Always plan the task step by step before writing code. Ask for permission to proceed with the plan.

**Important**: Before proceeding with the plan, create a new file named `tasks/name-of-the-task.md`. Based on the approved plan, list all necessary implementation steps as GitHub-style checkboxes (`- [ ] Step Description`). Use sub-bullets for granular details within each main step.

**CRITICAL**: After you successfully complete each step, you MUST update the `tasks/name-of-the-task.md` file by changing the corresponding checkbox from `- [ ]` to `- [x]`.

Only proceed to the *next* unchecked item after confirming the previous one is checked off in the file. Announce which step you are starting.

### Step 4: Implement with Testing
- Write clean, self-documenting code
- Follow the code style guide (see below)
- Write tests for new functionality
- Run tests to verify behavior
- Update documentation if needed

### Step 5: Verify and Clean Up
- Run `npm run build` to check for TypeScript errors
- Run `npm test` to verify all tests pass
- Remove any debug code or comments
- Ensure no unused imports or variables

## JavaScript and TypeScript Code Style Guide

### Core Principles
1. **One class per file** - Use PascalCase for class names and file names
2. **Self-documenting code** - Avoid comments that describe functionality; write clear, expressive code instead
3. **Private by default** - Use private methods for helpers not intended for external use
4. **No global scope** - Encapsulate all code within classes or modules
5. **Event-driven architecture** - Avoid tight coupling; use events for inter-package communication
6. **Package cohesion** - Keep related functionality together within the same package
7. **YAGNI principle** - Don't add code that isn't needed for the current task

### Specific Rules
- **Language**: TypeScript, browser-first (no Node.js-specific APIs in `src/`)
- **Indentation**: 2 spaces (no tabs)
- **Line width**: ~120 characters maximum
- **File naming**: kebab-case (e.g., `video-export-service.ts`, `audio-clip.ts`)
- **Symbol naming**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apssouza22/web-video-edit](https://github.com/apssouza22/web-video-edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
