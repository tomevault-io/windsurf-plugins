---
trigger: always_on
description: This file is for AI coding assistants (and any contributors) working in this repo. It explains the project structure, commands, conventions, and the security boundaries that must not be broken.
---

# AGENTS.md

This file is for AI coding assistants (and any contributors) working in this repo. It explains the project structure, commands, conventions, and the security boundaries that must not be broken.

## Project overview

`botler-agent` is a **general-purpose, lightweight personal agent framework**: it receives messages from Telegram / Feishu / WeChat and uses `@earendil-works/pi-agent-core` + `@earendil-works/pi-ai` (the pi SDK) to autonomously complete short tasks within each data subproject under the data root (`DATA_ROOT`), then sends the results back.

Key design principles (understand before changing, do not break):

1. **The framework only defines boundaries, it does not hardcode business logic**: the operable directories (first-level subdirs of `DATA_ROOT`), the toolset (read/write/edit/run + schedule, where run is limited to existing in-project scripts and schedule only writes the fixed externalized `schedules.json`, neither is an arbitrary shell), the post-write JSON-validity fallback, and automatic git commit. Business schemas and rules are self-described by each data subproject's root `AGENTS.md`, concatenated into the system prompt at runtime.
2. **Each message = a brand-new `Agent` instance**, with no cross-task memory. Therefore any "retry/fix" must be a **self-contained instruction** (pointing to the file + location + how to change it), not relying on the previous conversation.
3. **App / data separation**: this repo (including `.env`) and the `DATA_ROOT` data directory are two separate locations. The data directory contains only the projects being operated on — no source code or secrets.
4. **Config externalized**: the system prompt, the real `.env`, and `providers.json` (custom model providers) live in `~/.botler-agent/` (overridable via `BOTLER_CONFIG_DIR`), reused across clones / machines. The source-dir `.env` is only a dev fallback.
5. **Path allowlist is a security boundary**: the agent's tools may only read/write first-level subdirs of `DATA_ROOT`. Do not relax `safePath`, do not add `bash`-class tools to the agent unless explicitly requested and the consequences are understood.

## Commands

```bash
npm install            # install dependencies
npm run init           # initialize ~/.botler-agent/ (.env template + providers.json template + system-prompt.md template)
npm start              # run: no args starts the persistent channel; a message arg enters CLI mode
npm start -- "message" # CLI mode: process a single message directly (local debugging)
npm run typecheck      # tsc --noEmit type check (must pass before committing)
npm test               # node:test suite (scheduler cron + store)
```

> Environment variables are in `.env.example`; the real config goes in `~/.botler-agent/.env`. `.env` is gitignored — do not commit real credentials.

## Architecture and data flow

```
channel (telegram.ts / feishu.ts)
   → dispatcher.ts  dispatch(text, {id, source})
       ├─ dedup (5-minute window, by id)
       ├─ sequential queue (Promise chain, serializes writes)
       └─ runner.ts  runTask(text)
             ├─ resolveModel() (provider/model resolution + cache)
             ├─ loadSystemPrompt() (externalized first, fallback to built-in default; injects __DATA_ROOT__ / __PROJECTS__)
             ├─ new Agent({ systemPrompt, model, tools: fileTools })
             ├─ agent.prompt(text)
             ├─ take the body from the last assistant message in state.messages
             └─ split out markdown image refs (`![](…)`), each validated by safePath
   → write task: validateState() (all data JSON valid) → commitIfChanged() commits
   → return `{ text, images }` to the channel (only the WeChat channel sends images; others use text only)

scheduler (scheduler/engine.ts) → dispatch(schedule.message, {id, source:"scheduler", projectHint})
   → if entry has a recipient: deliver({text, images}, recipient) pushes the result back
       → primary channel first, then fallback telegram → feishu → wechat (configured + recorded contacts only)
```

### Module responsibilities

| File | Responsibility |
|------|----------------|
| `src/index.ts` | Entry: positional args → CLI; otherwise start Telegram / Feishu per `.env` |
| `src/init.ts` | Initialize `~/.botler-agent/` (.env + providers.json + system-prompt.md templates); existing files not overwritten |
| `src/config.ts` | Two-level `.env` loading (user-level > source-level) + providers.json loading + `CONFIG` construction + `USER_CONFIG_DIR` |
| `src/dispatcher.ts` | Dedup, sequential queue, validation retry, commit orchestration (**never rejects**) |
| `src/runner.ts` | Build Agent, run task, extract final reply, decide whether it mutated; greeting short-circuit (`greeting.ts`) + image-aware routing |
| `src/providers.ts` | Build a custom provider config into a pi-ai `Provider` (openai-completions) |
| `src/prompts/system-prompt.ts` | Built-in generic default prompt + `loadSystemPrompt()` (externalized first + placeholder injection) |
| `src/tools/paths.ts` | **Security boundary**: `safePath()` allowlist check + `projectOf()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crossoverJie/botler-agent](https://github.com/crossoverJie/botler-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
