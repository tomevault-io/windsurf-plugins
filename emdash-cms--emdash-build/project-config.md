---
trigger: always_on
description: A chat-based site builder where users describe the site they want and an AI agent builds it in a Cloudflare Sandbox, with a live preview alongside the chat.
---

# EmDash Build

A chat-based site builder where users describe the site they want and an AI agent builds it in a Cloudflare Sandbox, with a live preview alongside the chat.

## Blank-builder prototype

New projects use the local `prototype/builder-cloudflare` scaffold. It provides EmDash, Astro, Tailwind 4, and source-owned accessible Astro primitives but no content schema or site design; the agent creates those from the brief. Public generated routes must remain Astro/vanilla JavaScript even though React stays configured for the EmDash admin. Existing persisted projects recover their own scaffold from Artifacts and are not replaced.

Read `SPEC.md` for product context. Its blank-builder note and the architecture here describe the current prototype; its designer-template selection, worked examples, and tool table describe earlier designs.

## Stack

- **Worker**: Hono for HTTP, `agents` SDK for the BuilderAgent DO, `@cloudflare/sandbox` for containers
- **LLM**: Workers AI binding, streaming via the Vercel AI SDK (`ai` package)
- **Frontend**: React 19 + Tailwind CSS 4 + `@cloudflare/kumo` component library
- **Build**: Vite + `@cloudflare/vite-plugin` (single build for Worker + SPA)
- **Test**: Vitest 4.1 + `@cloudflare/vitest-pool-workers`
- **Formatting**: oxfmt (tabs for indentation)

## Directory Structure

```
src/
  worker/           # Worker entrypoint + BuilderAgent DO
    index.ts        # Hono app, sandbox proxy, agent routing
    agent.ts        # BuilderAgent DO + provisionSite + onChatMessage interview/build phases
    tools.ts        # Sandbox file/shell/unsplash tools
    prompts.ts      # buildInterviewPrompt + buildBuildPrompt (composes markdown sections)
    prompts/        # Prompt prose as markdown files (imported with ?raw)
      build-blank.md # Blank builder schema/design system prompt
      interview.md  # Shared interview behaviour
      interview-blank.md    # Domain-neutral schema/design intake
  client/           # React SPA
    components/     # Chat UI, preview panel, tool cards
  vite-env.d.ts     # Ambient declaration for "*.md?raw" imports
```

## Key Architecture Decisions

1. **The agent runs in a Durable Object**, not the Worker. The DO owns the conversation state (persisted in SQLite), the sandbox instance, and the agent loop. The Worker routes requests to the right DO.

2. **Standalone blank scaffolding.** The Sandbox image contains one installed archive built from `prototype/builder-cloudflare`; each new project extracts it into `/home/user/site`. No monorepo, public-template checkout, `GH_TOKEN`, or workspace links. EmDash itself is on npm (`emdash`, `@emdash-cms/cloudflare`).

3. **There is no template-selection phase.** Provisioning begins immediately from `builder-cloudflare` while the agent runs a domain-neutral interview. Once MCP is connected, the agent creates the schema and frontend from the brief.

   The agent runs in two phases: an **interview phase** (first turn, only the optional `ask_questions` tool, domain-neutral intake from `prompts/interview-blank.md`) and a **build phase** (once provisioning is ready and any questions have been answered or skipped, full tools and rules from `prompts/build-blank.md` plus the scaffold's own `AGENTS.md` appended as `## Template-specific guidance`).

   The opening brief anchors a durable `initialGeneration` state. Interview, setup, and build assistant messages carry its ID, so the client shows one evolving activity card and an ordered timeline. Verified form submissions keep structured answers alongside model-readable text; freeform replies remain visible in chat. Stop, retry, and recovery retain this identity until a validated first site is ready. Later edits remain separate turns.

4. **Sandbox tools:** `read_file`, `read_files`, `write_file`, `write_files`, `edit_file`, `edit_files`, `exec`, `refresh_types`, `validate_site`, `search_unsplash`, `upload_media`, `view_preview`, and `offer_clone`. CMS operations (schema, content, taxonomy, settings) come from the EmDash MCP server at `/_emdash/api/mcp`, wired in at runtime in `agent.ts` against an allowlist (`ALLOWED_MCP_TOOLS`). `upload_media` does its `fetch`/download and `POST /_emdash/api/media` entirely **in the Worker** (via the public preview URL), so the bearer API token never enters the sandbox where the LLM has shell access.

   Builder-managed files (`src/worker.ts`, `src/live.config.ts`, `wrangler.jsonc`, `.dev.vars`, `AGENTS.md`) are refused by `write_file`/`edit_file` (`.dev.vars*` also by `read_file`/`read_files`), and `exec` runs inside `guardProtectedFiles`, which restores any it changed. The guard catches mistakes, not a determined model, so the scaffold guidance that enters the system prompt is pinned in the agent's SQLite rather than re-read from the sandbox.

   The model has no deployment tool. User-initiated static WfP publishing remains behind the server-owned `ENABLE_PUBLIC_PUBLISHING` release flag and account/project authorization.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emdash-cms/emdash-build](https://github.com/emdash-cms/emdash-build) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-28 -->
