---
trigger: always_on
description: > **Status: 1.0.0-rc.5 — V1 release candidate** (May 2026). Multi-AI fleet hub
---

# CLAUDE.md

> **Status: 1.0.0-rc.5 — V1 release candidate** (May 2026). Multi-AI fleet hub
> (Phases (d).1 → (d).16a) is the headline feature. Through rc.5: auto-memory
> writeback, `/memory recent`, `AGENTS.md` cross-CLI scaffold, opt-in
> mid-stream retry, history curation API (Gemini CLI audit reco #3, 3/3
> closed), `Explore` read-only subagent + `disallowedTools` field
> (Claude Code audit Phase A+C), `/subagent` discovery slash, **`/swarm
> <task>`** team-lead UX inspired by Korben, **lessons feature activated**
> + **`<writing_rules>` directive** (Manus AI structured blocks pattern,
> @renschni gist) — completes the persistence trilogy auto-memory +
> lessons + writing discipline. Read
> [`docs/getting-started.md`](docs/getting-started.md) first. See
> [`docs/fleet-guide.md`](docs/fleet-guide.md) and
> [`CHANGELOG.md`](CHANGELOG.md).

Guidance for Claude Code when working in this repo. Keep this file short — it should capture what you *can't* derive by reading the source.

## Build, Test, Lint

```bash
npm install
npm run dev            # Bun dev mode
npm run dev:node       # tsx dev mode
npm run build          # TypeScript build
npm run typecheck
npm run lint
npm run validate       # lint + typecheck + test — run before committing
npm test               # Vitest — full suite is ~26K tests, slow. Prefer a path filter.
npm test -- path/to/file.test.ts
npm run test:run       # one-shot (no watch)
npm run build:gui      # Cowork Electron GUI
buddy install-gui && buddy gui
```

Tests live in `tests/` and in-source `src/**/*.test.ts`. Vitest + happy-dom. `vitest.setup.ts` shims `globalThis.jest` → `vi` so legacy `jest.fn()` works.

## Testing Gotchas

- ESM project (`"type": "module"`). Use `import.meta.url` + `fileURLToPath` for `__dirname`. `@` alias → `./src` (see `vitest.config.ts`). Source imports need `.js` extensions even for `.ts` files.
- Use `logger` (`src/utils/logger.js`) not `console.*` in production — tests spy on `logger.warn`.
- `BashTool` tests: call `ConfirmationService.setSessionFlag('bashCommands', true)` first, and mock every transitive import (`safe-binaries`, `auto-sandbox`, `shell-env-policy`, `bash-parser`, `checkpoint-manager`, `audit-logger`, `command-validator`, `streaming-executor`). `execute()` has async pre-spawn logic, so defer mock process events with `setImmediate()` — don't emit synchronously.
- CLI command tests: Commander `parseAsync()` + `exitOverride()`, mock `console.log` / `process.exit`.
- Channel adapter tests: mock `global.fetch` for health checks, mock dynamic imports via virtual modules.
- `DeviceNodeManager` tests: mock `ssh-transport` / `adb-transport` / `local-transport` and `fs` (prevents `devices.json` bleed between tests). `pairDevice()` is async.
- `AgentRegistry` ships 8 built-in agents: PDF, Excel, DataAnalysis, SQL, Archive, CodeGuardian, SecurityReview, SWE.

## Architecture

Terminal multi-provider AI coding agent (Grok / Claude / GPT / Gemini / Ollama / LM Studio, all via OpenAI-compatible routing). Core is an agentic loop where the LLM autonomously calls tools.

```
User → ChatInterface (Ink/React) → CodeBuddyAgent → LLM provider
                                         │
                                Tool calls (max 50, YOLO 400)
                                         │
                              Execute + confirm → results → loop
```

### Facades (`src/agent/facades/`)

`CodeBuddyAgent` delegates to:
- `AgentContextFacade` — token counting, `ContextManagerV2` compression, memory retrieval
- `SessionFacade` — save/load sessions, checkpoints, rewind
- `ModelRoutingFacade` — model selection, cost tracking, usage stats
- `InfrastructureFacade` — MCP servers, sandbox, hooks, plugins
- `MessageHistoryManager` — message storage, history truncation, export

### Key Entry Points

- `src/index.ts` — CLI entry (Commander), lazy-loaded commands, `--profile` flag
- `src/agent/codebuddy-agent.ts` — main agentic loop, `executePlan()`
- `src/agent/execution/agent-executor.ts` — middleware pipeline, reasoning, tool streaming. **Single source of truth via `runTurnLoop` async generator (task #5 fusion done 2026-04-26).** `processUserMessageStream` is a thin `yield*` wrapper; `processUserMessage` is a thin sequential collector that consumes events and returns the new entries pushed to history. Per-turn injections, transcript repair, output sanitization, and the `__SESSIONS_YIELD__` signal all live in `runTurnLoop` — touch them in one place. Streaming-only events (`ask_user`, `tool_stream`, `token_count`, `reasoning`, `steer`) are silently dropped in the sequential collector (décision #3).
- `src/codebuddy/client.ts` — thin dispatcher (~400 LOC) that delegates to a `Provider` strategy: `GeminiNativeProvider` when the baseURL points at `generativelanguage.googleapis.com`, `OpenAICompatProvider` otherwise. Strategies live under `src/codebuddy/providers/`. Adding a new provider = one new strategy file + a branch in the constructor. `defaultMaxTokens` comes from `getModelToolConfig(model).maxOutputTokens`. Anthropic-specific message hooks (`injectAnthropicCacheBreakpoints`, `injectJsonSystemPromptForAnthropic`) live in `provider-openai-compat-hooks.ts` and are called by both `chat()` and `chatStream()` on the OpenAI-compat strategy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phuetz/code-buddy](https://github.com/phuetz/code-buddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
