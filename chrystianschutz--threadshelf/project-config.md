---
trigger: always_on
description: Guidance for AI coding agents (Claude Code, Codex, Cursor, etc.) working in this
---

# AGENTS.md

Guidance for AI coding agents (Claude Code, Codex, Cursor, etc.) working in this
repository. Humans should read this too — it is the short, accurate map of the
project. `CLAUDE.md` simply points here so Claude Code picks up the same rules.

## What this project is

**ThreadShelf** — a local, offline semantic-search and backup tool for AI chat
exports (Google AI Studio, OpenRouter, OpenAI/ChatGPT, Anthropic/Claude, LM Studio,
Grok/xAI). It
parses exported JSON into normalized turns, chunks and embeds them locally
(Xenova/Transformers.js), stores vectors in LanceDB, and exposes search through a
React web UI, an HTTP API, and an MCP stdio server.

Parsing, embeddings, storage, and search run on the user's machine. **By default,
no chat data leaves the device.** The embedding model may be downloaded on first use.
Treat all real chat exports as private.

The optional conversation-generation layer is **Experimental Beta**. Its
primary `llama.cpp` engine is local and loopback-only. OpenRouter is an explicit,
opt-in external exception: picking the OpenRouter provider tab sends selected
user/assistant thread content, the optional master prompt, and the new prompt
off-device; archived thinking is excluded. There is no longer a per-send consent
checkbox — the `off-device` chip on the model button and the composer hint carry
that signal. New chats are saved locally by default in the protected
`threadshelf_conversations` collection and semantic index. The explicit
ghost-icon private mode is tab-scoped and never persisted.

The **master prompt** is a small collection of user-written system prompts. They
are hand-written and expected to last, so they are stored server-side in
`.threadshelf/master-prompts.json` (`src/generation/master-prompts.ts`, atomic
write, mode 0600, `MASTER_PROMPTS_PATH` override) and served by the loopback-only
`/api/generation/prompts` CRUD routes. The client holds no copy beyond the
react-query cache (`['master-prompts']`). The active prompt is sent as
`systemPrompt` on every generation request and prepended as a leading `system`
message; it is never persisted into a stored thread, so re-reading a chat never
replays a prompt the user has since changed.

## Repository layout

```
src/                Server + core logic (TypeScript, ESM, run via tsx)
  server.ts         Express app entrypoint
  env.ts            Startup loader for the optional, gitignored root `.env`
  load-env.ts       Testable `.env` loading helper (explicit process env wins)
  cli.ts            `npm run parse` CLI
  parser.ts         Provider detection + export -> normalized turns
  chunking.ts       Turn -> embeddable chunks
  embedding.ts      Local Xenova embeddings
  model-label.ts    Portable model labels (strip private local filesystem prefixes)
  ingest.ts         Parse -> chunk -> embed -> store pipeline
  watch.ts          Watch-folder mode (fs.watch + debounced re-ingest)
  store.ts          LanceDB access
  validation.ts     Turn/types + input validation
  routes/           HTTP routes (health, search, thread, collections, files, ingest, insights)
    stream-abort.ts        Shared "client went away" AbortController for streamed routes
  services/         search, thread, collections, insights business logic
  generation/       Experimental Beta provider plugins, config, model discovery, llama wrapper
    downloader.ts          Shared resumable, hash-verifying downloader (runtime + models)
    model-catalog.ts       Read-only Hugging Face GGUF browser (public API, no token)
    model-download.ts      Plans and fetches catalog models into the download directory
    hardware.ts            Accelerator/RAM detection and the model "will it fit" verdict
    quick-setup.ts         One-screen setup plan (runtime + model), fingerprint, runner
    master-prompts.ts      User system prompts on disk (.threadshelf/master-prompts.json)
    error-log.ts           Optional rotating generation errors (.threadshelf/generation-errors.log)
    filesystem-browser.ts  Loopback-only, directory-only model-root browser
client/             React + Vite + TypeScript web UI (npm workspace)
  src/              Components, pages, store (zustand), queries (react-query)
    components/ModelCombobox.tsx  Searchable generation models + local favorites
    components/ModelCatalogModal.tsx  Hugging Face model browser (search, gating, VRAM fit)
    components/QuickSetupPanel.tsx    One-confirmation llama.cpp + model install
    components/NumberCombobox.tsx Typeable token-budget dropdown (presets + free entry)
    components/MasterPromptMenu.tsx  Master-prompt editor (server-stored, sent with every request)
    components/NotFound.tsx       Router `defaultNotFoundComponent` for unknown URLs
mcp/server.ts       MCP stdio server exposing local search
test/               Node test runner unit tests + fixtures/
  e2e/              API + MCP end-to-end tests (boot a real server)
  playwright/       Browser E2E (see "Known gaps")
docs/               Architecture, getting started, MCP, etc.
public/             Built UI output — GENERATED, do not edit by hand
```

The repo is an **npm workspaces** monorepo: the root and `client/` share a single
`package-lock.json` and a hoisted `node_modules/`. A plain `npm install` at the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrystianSchutz/ThreadShelf](https://github.com/ChrystianSchutz/ThreadShelf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
