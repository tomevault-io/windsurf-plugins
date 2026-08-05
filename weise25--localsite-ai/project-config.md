---
trigger: always_on
description: Generates a Svelte Playground link with the provided code.
---

You are able to use the Svelte MCP server, where you have access to comprehensive Svelte 5 and SvelteKit documentation. Here's how to use the available tools effectively:

## Available Svelte MCP Tools:

### 1. list-sections

Use this FIRST to discover all available documentation sections. Returns a structured list with titles, use_cases, and paths.
When asked about Svelte or SvelteKit topics, ALWAYS use this tool at the start of the chat to find relevant sections.

### 2. get-documentation

Retrieves full documentation content for specific sections. Accepts single or multiple sections.
After calling the list-sections tool, you MUST analyze the returned documentation sections (especially the use_cases field) and then use the get-documentation tool to fetch ALL documentation sections that are relevant for the user's task.

### 3. svelte-autofixer

Analyzes Svelte code and returns issues and suggestions.
You MUST use this tool whenever writing Svelte code before sending it to the user. Keep calling it until no issues or suggestions are returned.

### 4. playground-link

Generates a Svelte Playground link with the provided code.
After completing the code, ask the user if they want a playground link. Only call this tool after user confirmation and NEVER if code was written to files in their project.

---

## Project Overview

LocalSite-AI generates complete, self-contained HTML/CSS/JS web pages from
natural language prompts using AI. Users pick a provider and model, describe
what they want, and get a live preview with an editable Monaco code editor.

**Tech stack:** SvelteKit 2, Svelte 5 (runes), Deno 2, Tailwind CSS 3,
Vercel AI SDK v5, Monaco Editor, Paneforge, svelte-sonner, @lucide/svelte.

**Runtime:** Deno 2 (`deno task dev/build/check/lint`). `package.json` retains
npm scripts for compatibility but Deno is the primary toolchain.

## Architecture

```
src/
  routes/
    +page.svelte                  Main page: WelcomeView -> GenerationView
    +layout.svelte                Global layout, Toaster
    +layout.ts                    SSR disabled (ssr = false)
    +error.svelte                 Error boundary
    api/
      generate-code/+server.ts    POST: streams NDJSON (text + reasoning)
      get-models/+server.ts       GET: models for provider; POST: list providers
      get-default-provider/+server.ts  GET: default provider from env
  lib/
    state/
      code-generation.svelte.ts   CodeGeneration class (rune-based state)
    server/providers/
      config.ts                   LLMProvider enum, config registry, env helpers
      provider.ts                 9 provider client classes, generateCodeStream()
      prompts.ts                  DEFAULT_SYSTEM_PROMPT, THINKING_SYSTEM_PROMPT
    components/
      [see README for full component list]
    ui/                           8 hand-rolled primitives (Button, Select, etc.)
```

### Data Flow

1. WelcomeView loads providers (POST /api/get-models) and models (GET
   /api/get-models?provider=X). Models cached in sessionStorage.
2. User clicks GENERATE -> CodeGeneration.generateCode() POSTs to
   /api/generate-code.
3. Server streams NDJSON: `{type: "text", content}` and
   `{type: "reasoning", content}` lines.
4. Client accumulates chunks reactively. GenerationView debounces preview
   updates (1s throttle during streaming).
5. PreviewPanel uses double-buffered iframes with z-index/opacity crossfade.

### Key Patterns

- **Svelte 5 runes everywhere.** No stores. State is `$state()`, computed
  is `$derived()`, side effects are `$effect()`, props are `$props()` with
  `$bindable()` for two-way binding.
- **Server/client boundary:** `$lib/server/` files use `$env/dynamic/private`
  for env vars. Never import these from client code.
- **SSR is disabled** (`+layout.ts`). The app renders entirely client-side
  because Monaco, sessionStorage, and streaming fetch require browser APIs.
- **NDJSON streaming.** The generate-code endpoint returns
  `application/x-ndjson`. Each line is a JSON object. The client reads via
  `ReadableStream` reader.

## Development Commands

```bash
deno task dev        # Start dev server (localhost:5173)
deno task build      # Production build to build/
deno task check      # TypeScript + Svelte type checking
deno task lint       # Deno linter
deno task start      # Run production server (build/index.js)
```

## Important Gotchas

These are non-obvious behaviors discovered during development. Read before
making changes in these areas.

### Monaco Editor

- **`executeEdits()` does NOT work in read-only mode.** Monaco silently
  returns `false`. During streaming the editor is always read-only. Use
  `setValue()` for external sync. See `CodeEditor.svelte` `syncEditorContent()`.
- **`setValue()` resets the undo stack.** This is acceptable during streaming
  (read-only) but be careful in edit mode.
- **Monaco loads asynchronously** via dynamic import. Code that arrives during
  loading must be picked up after init — see the `syncEditorContent(code)` call
  at the end of `onMount`.

### Preview Panel

- **Double-buffered iframes** swap via `setTimeout(400ms)` + `onload`. Both
  timeouts MUST be cleaned up in `onDestroy` to avoid leaks.
- **`allow-same-origin` in sandbox is required.** The parent reads

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weise25/LocalSite-ai](https://github.com/weise25/LocalSite-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
