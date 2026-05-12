---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Development Commands

### Essential Commands
- `pnpm install` - Install dependencies (requires pnpm v10.18.0, Node.js >=20.0.0)
- `pnpm run dev` - Start Remix Vite development server
- `pnpm run build` - Build for production
- `pnpm run start` - Run production build locally with Wrangler Pages (uses bindings.sh)
- `pnpm run preview` - Build and run production preview locally
- `pnpm run deploy` - Build and deploy to Cloudflare Pages
- `pnpm run setup` - Automated Supabase database setup with guided instructions

### Testing & Code Quality
- `pnpm test` - Run Vitest test suite once
- `pnpm test:watch` - Run tests in watch mode
- `pnpm run typecheck` - Run TypeScript type checking (ALWAYS run before committing)
- `pnpm run lint` - Run ESLint with caching
- `pnpm run lint:fix` - Run ESLint and auto-fix issues
- `pnpm run typegen` - Generate TypeScript types using Wrangler

### Utility Commands
- `pnpm run clean` - Clean build artifacts and cache directories
- `pnpm run clean:cache` - Remove all cache files (.vite, .remix, node_modules/.cache)
- `pnpm run clean:build` - Remove build outputs (dist, build, public/build)

### Environment Setup
Create `.env.local` with required variables:
```bash
# Required - Anthropic (default model)
ANTHROPIC_API_KEY=sk-ant-api03-xxxxx

# Required - Supabase (authentication & persistence)
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your-anon-key
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key

# Optional - Additional AI Providers (19+ models supported)
OPENAI_API_KEY=sk-xxxxx
GOOGLE_API_KEY=xxxxx
DEEPSEEK_API_KEY=xxxxx
XAI_API_KEY=xxxxx
MISTRAL_API_KEY=xxxxx

# Optional - Debugging
VITE_LOG_LEVEL=debug
```

**Setup Database**: Run `pnpm run setup` for guided Supabase database setup, or manually execute `scripts/schema.sql` in Supabase SQL Editor.

**Important:** Chrome 129 has known issues with Vite local development. Use Chrome Canary for development testing. Production builds work fine in Chrome 129.

## Architecture Overview

### Tech Stack
BoltDIY V2.0 is an AI-powered in-browser IDE built with:
- **Framework:** Remix (React) with Vite and TypeScript
- **Deployment:** Cloudflare Pages + Workers
- **Styling:** Tailwind CSS with UnoCSS compatibility
- **State:** Nanostores (lightweight reactive state)
- **AI:** Multi-model support (19+ models from 6 providers via Vercel AI SDK):
  - **Anthropic**: Codex Sonnet 4.5, Codex Sonnet 4
  - **OpenAI**: GPT-5, GPT-4.1, o3, o4-mini, GPT-4o
  - **Google**: Gemini 2.5 Pro, Gemini 2.5 Flash, Experimental
  - **DeepSeek**: V3.2, Reasoner
  - **xAI**: Grok Code Fast 1, Grok 3, Grok 4
  - **Mistral**: Codestral 25.08, Large, Small
- **Runtime:** WebContainer API (browser-based Node.js environment)
- **Editor:** CodeMirror 6 with multi-language support
- **Terminal:** xterm.js with fit and web-links addons
- **Auth/DB:** Supabase (PostgreSQL with RLS policies)
- **UI Components:** Radix UI primitives with custom theming

### Core Concepts

#### WebContainer Runtime
- Provides full-stack Node.js sandbox environment in the browser
- Manages filesystem, package manager, dev server, and terminal
- No native binaries (uses web-based implementations)
- Python available but standard library only (no pip)
- Git is NOT available in WebContainer
- Entry point: `app/lib/webcontainer/index.ts`
- Working directory: Controlled by `WORK_DIR_NAME` constant

#### AI Integration Layer
Located in `app/lib/.server/llm/`:
- `prompts.ts` - System prompts and AI instruction templates (modify here for AI behavior changes)
- `provider-factory.ts` - Factory pattern for creating provider instances based on model selection
- `model-config.ts` - Configuration for all 19+ AI models with capabilities, context limits, and pricing
- `stream-text.ts` - Streaming response handler with multi-provider support
- `constants.ts` - Model limits (MAX_TOKENS: 8192, MAX_RESPONSE_SEGMENTS: 2)
- `providers/` - Individual provider implementations:
  - `anthropic.ts` - Codex models configuration
  - `openai.ts` - GPT and o-series models
  - `google.ts` - Gemini models
  - `deepseek.ts` - DeepSeek models
  - `xai.ts` - Grok models
  - `mistral.ts` - Mistral and Codestral models

**Multi-Model Architecture:**
- Each chat can use a different model
- Model selection persists per chat session
- Provider factory dynamically loads the appropriate SDK based on model choice
- Streaming works consistently across all providers

The AI has complete control over the WebContainer environment including filesystem, terminal, and package manager.

#### State Management
Nanostores in `app/lib/stores/`:
- `workbench.ts` - Primary workbench state and WebContainer orchestration
- `files.ts` - File system operations and tree management
- `editor.ts` - Code editor state and content
- `terminal.ts` - Terminal state and command execution
- `chat.ts` - AI conversation state
- `previews.ts` - Preview iframe management
- `theme.ts` - Theme preferences and lightning theme support
- `settings.ts` - User settings and preferences

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Stijnus/bolt.diy_V2.0](https://github.com/Stijnus/bolt.diy_V2.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
