---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Always start each question by looking up relevant documentation sections in the Svelte MCP.
Always run "npm run format" followed by "npm run checks" before finishing a task, and fix any linting issues.

## Project Overview

This is a browser-based AI chat and transcription application that runs entirely client-side without sending data to external servers. The application uses Svelte 5, SvelteKit, WebAssembly, the Wllama library for chat completions, and @transcribe/transcriber for audio transcription.

## Development Commands

### Core Commands

- `npm run dev` - Start development server
- `npm run build` - Build for production (with OPFS enabled by default, uses adapter-node)
- `npm run build:static` - Build static version for production (uses adapter-static)
- `npm run preview` - Preview production build
- `npm run check` - Type check with svelte-check
- `npm run check:watch` - Type check in watch mode
- `npm run lint` - Run ESLint and Prettier checks
- `npm run lint:fix` - Fix ESLint and Prettier issues automatically
- `npm run format` - Format code with Prettier
- `npm run checks` - Run format, lint, and check in sequence
- `npm run copy-wasm` - Manually copy WASM files from npm packages

### Testing Commands

- `npm run test` - Run unit tests once
- `npm run test:unit` - Run unit tests in watch mode

### Translation Commands (Wuchale)

- `npm run wc` or `npx wuchale` - Extract translatable strings and update .po files
- `npm run wcs` or `npx wuchale status` - Check translation status
- `npm run wcc` or `npx wuchale --clean` - Clean translation files

### Docker Commands

- `docker build -t sveltekit-local-ai .` - Build Docker image
- `docker run -p 3000:3000 sveltekit-local-ai` - Run container

## Architecture & Key Concepts

### Application Structure

- **Routes**: Main pages in `src/routes/` with route groups for apps `(apps)/`
- **Components**: Reusable UI components in `src/lib/components/`
- **State Management**: Persisted stores using `svelte-persisted-store` in `src/lib/stores.ts`
- **AI Integration**: Wllama configuration and models in `src/lib/wllama-config.ts`

### WebAssembly Integration

- Uses Wllama library for local LLM inference
- Models are downloaded and cached in browser (OPFS when supported)
- Cross-Origin headers required for WebAssembly: `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp`

### Svelte 5 Features

- Uses `$state()` for reactive variables instead of legacy syntax
- Component props use the `$props()` rune
- Snippets used for component children patterns

### Model Management

- Available models configured in `AVAILABLE_MODELS` array in `src/lib/wllama-config.ts`
- Models are downloaded with progress tracking and cached locally
- Support for both single-threaded and multi-threaded WebAssembly builds

### State Persistence

Key persisted stores:

- `messages`: Chat message history
- `inferenceParams`: AI model parameters (temperature, context length, etc.)
- `whisperModel`: Selected Whisper model for transcription

### Component Organization

- **Chat components**: `src/lib/components/chat/` (ModelSelector, ChatMessages, MessageInput, Message)
- **Whisper components**: `src/lib/components/whisper/` (file upload, model selection, transcription)
- **TTS components**: `src/lib/components/tts/` (voice selection, speed control, sample rate, WebGPU toggle)
- **Background Remover components**: `src/lib/components/background-remover/` (upload, progress, results)
- **Common components**: `src/lib/components/common/` (LoadingProgress, ErrorDisplay, ProgressBar, CardInterface, etc.)

## Key Technical Details

### AI Model Configuration

- Default chat template uses ChatML format (`<|im_start|>` tokens)
- Inference parameters: 4096 context, temperature 0.2, auto-threading
- Models served from external CDN (configured via BASE_MODEL_URL in src/lib/config.ts)

### Browser Compatibility

- Requires modern browsers with WebAssembly and SharedArrayBuffer support
- OPFS (Origin Private File System) used for model caching when available
- Fallback storage mechanisms for browsers without OPFS

### Development Notes

- TypeScript used throughout the codebase
- ESLint configuration includes Svelte-specific rules
- Prettier for code formatting
- Supports both adapter-node (default) and adapter-static (via ADAPTER env var)
- Express server with health check endpoint at `/_health` (adapter-node only)
- Bundle strategy set to 'single' in svelte.config.js for optimized output

### ONNX Runtime Web Integration

- ONNX Runtime Web is used for TTS models (kitten-tts, piper-tts, kokoro-tts) and background removal
- WASM and bundle files are automatically copied from npm packages to ensure version compatibility
- Use `npm run copy-wasm` to manually update files from `onnxruntime-web` and `@huggingface/transformers`
- Files are located in `static/onnx-runtime/` (for TTS) and `static/transformers/` (for background removal)
- Both `dev` and `build` scripts automatically copy files before starting
- Static ONNX Runtime directories are gitignored since they're generated from npm packages

### Internationalization (i18n)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [khromov/nook](https://github.com/khromov/nook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
