---
trigger: always_on
description: Build and ship a **Rork-like AI Mobile App Builder** — a web tool where users describe a mobile app in natural language, an AI agent autonomously builds complete Expo/React Native code, and users can preview, edit, export, sync to GitHub, and build for app stores. Deployed on **Vercel** connected to **Supabase**.
---

# CLAUDE.md — Rork AI Mobile App Builder

## Goal

Build and ship a **Rork-like AI Mobile App Builder** — a web tool where users describe a mobile app in natural language, an AI agent autonomously builds complete Expo/React Native code, and users can preview, edit, export, sync to GitHub, and build for app stores. Deployed on **Vercel** connected to **Supabase**.

---

## Instructions

- **Repo**: `https://github.com/ungden/rork-ai-builder.git` — cloned at `/Users/alexle/Documents/mobileai/rork-ai-builder`
- **Never rewrite from scratch** — always read existing code first, identify gaps, fix precisely
- **Always read files before editing**
- **Stack**: Next.js 16 (Turbopack), Tailwind v3 (downgraded from v4), Supabase SSR, `@ai-engine/core` workspace package, pnpm + Turborepo monorepo
- **AI providers**: Gemini (`GEMINI_API_KEY`) is the default model; Claude (`ANTHROPIC_API_KEY`) optional for full 11-tool `RorkAgent` loop
- **Deployment**: Vercel (connected to `ungden/rork-ai-builder` GitHub repo), Supabase project ID `wpfufmolfgcjnkxjgpqt`
- **Design direction**: Dark theme, clean/minimal like rork.com — centered text, proper spacing, Inter font
- **Do NOT commit `.env.local`** — credentials go in Vercel dashboard env vars only
- **`vercel.json`** configured with `buildCommand: "pnpm build:web"`, `outputDirectory: "apps/web/.next"`, `maxDuration: 300` for agent routes
- **Supabase env vars on Vercel**: `NEXT_PUBLIC_SUPABASE_URL` = `https://wpfufmolfgcjnkxjgpqt.supabase.co`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `GEMINI_API_KEY` — all confirmed set
- **Supabase schema**: Deployed to live project (user confirmed)
- **Signup**: Only email + password (Name field removed)
- **Next.js 16 uses `proxy.ts`** not `middleware.ts` — file is `src/proxy.ts` with `export async function proxy`

---

## Key Architecture

### How the Landing → Editor Flow Works
1. `HeroPromptBox` (landing page) saves prompt to `sessionStorage('rork_pending_prompt')`
2. If logged in: creates project via `/api/projects` → redirects to `/editor/{id}`
3. If not logged in: redirects to `/signup`, after auth → dashboard → `PendingPromptHandler` auto-creates project → redirects to editor
4. Editor picks up prompt from sessionStorage → passes as `initialPrompt` to `ChatPanel`
5. ChatPanel auto-sends prompt to AI agent after 800ms delay (via `useRef` pattern, NOT `useCallback`)

### Preview Panel (Expo Snack SDK)
- Uses **Expo Snack SDK** (`snack-sdk@6.6.0`) to run real Expo code in an iframe (web preview) and on devices via QR code
- `useSnack` hook (`apps/web/src/hooks/useSnack.ts`) manages the Snack session — creates instance, goes online, provides `webPreviewRef`, `webPreviewURL`, `expoURL`, `isBusy`, `connectedClients`
- `SnackPreview` component (`apps/web/src/components/editor/SnackPreview.tsx`) renders the iframe and handles runtime errors + "Fix with AI"
- Editor page wires `project-files-changed` events to `snack.updateFiles()` — files push to Snack in real-time as AI generates them
- Initial project files are synced to Snack on load via `snackSetAllFiles()`
- `isGenerating` is wired to `agentStore.isRunning` (not projectStore)
- Files apply immediately via `addGeneratingFile()` as each SSE event arrives — real-time preview updates

### AI Agent Pipeline
- **Gemini path**: `GeminiProvider.streamCode()` — uses `write_file` tool declaration, multi-turn tool loop (up to 20 rounds), yields file events via SSE
- **Claude path**: `RorkAgent` with 11 tools (create_plan, write_file, patch_file, search_files, verify_project, delete_file, read_file, list_files, run_test, fix_error, complete)
- System prompt is assembled from 5 modules in `packages/ai-engine/src/prompts/`: `expo-sdk54.ts`, `navigation.ts`, `styling.ts`, `components.ts`, `expo-knowledge.ts`
- All prompts target real Expo SDK 52 code: `expo-router` (file-based routing), `@expo/vector-icons` (Ionicons), `react-native-reanimated`, `react-native-safe-area-context`, `expo-image`, `expo-blur`, `expo-haptics`, `AsyncStorage`, CSS `boxShadow`
- `@react-three/fiber` for 3D graphics (Rork Max capability)

### State Management
- `projectStore` (Zustand + Immer): files, messages, activeFile, isGenerating, selectedModel
- `agentStore` (Zustand + Immer): isRunning, phase, plan, files, progress, processEvent

---

## Discoveries / Gotchas

### Tailwind v4 → v3 downgrade
- Tailwind v4's `@theme {}` hex tokens don't support opacity modifiers and its CSS layer ordering broke basic utilities on Vercel
- Downgraded to Tailwind v3 with `tailwind.config.ts`, standard postcss plugin, and `@tailwind` directives

### Supabase URL was wrong on Vercel
- Originally pointed to `ekeicbetveuvazfgmfii.supabase.co` (wrong) — corrected to `wpfufmolfgcjnkxjgpqt.supabase.co`

### Auto-send bug (FIXED in d749dea)
- `handleAgentRun` was wrapped in `useCallback` with `[files]` in deps → stale closure → prompt never sent
- **Fix**: Replaced with `useRef` pattern. `handleAgentRunRef.current` always points to latest function


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ungden) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
