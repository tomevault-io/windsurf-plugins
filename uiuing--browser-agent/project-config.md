---
trigger: always_on
description: Browser Agent is an open-source MV3 Chrome extension: **the agent harness for the browser**. Bring any model; the harness provides the agent loop, context assembly, tools, guardrails, memory and observability — and every claimed page effect is **verified against page truth**. TypeScript strict, pnpm monorepo, no backend.
---

# AGENTS.md — guide for AI coding agents

Browser Agent is an open-source MV3 Chrome extension: **the agent harness for the browser**. Bring any model; the harness provides the agent loop, context assembly, tools, guardrails, memory and observability — and every claimed page effect is **verified against page truth**. TypeScript strict, pnpm monorepo, no backend.

Terminology: **harness** = the whole product. **engine** (`src/engine/`) = the page-action runtime (perception/grounding/execute/verify/failure handling), exposed to the loop as the `page_act` tool.

## Setup & commands

```bash
pnpm install # workspace root; runs `wxt prepare` for the extension
pnpm build # fixtures + extension build, then typechecks bench/e2e
pnpm dev:ext # extension dev server (WXT hot reload)
pnpm --filter @browser-agent/extension compile # tsc --noEmit for the extension only
pnpm --filter @browser-agent/extension zip # release zip → packages/extension/.output/browser-agent-<ver>-chrome.zip
```

Extension build output: `packages/extension/.output/chrome-mv3` (load unpacked in Chrome).

## Tests — run these before claiming done

```bash
pnpm test:engine # fast, headless-DOM checks of the engine (perception/grounding/verify/heal)
pnpm test:e2e # full extension e2e with the scripted MockProvider; needs Playwright Chromium
pnpm shots # screenshot walkthrough of all UI surfaces → shots/
```

Notes for e2e:
- e2e builds the extension itself with `VITE_USE_MOCK=1` (routes to a deterministic scripted mock — no API key needed). Never ship or commit a mock build; `pnpm build` at root always produces the production build.
- Real-model suites (`test:real`, `test:sites` in `packages/e2e`) read `REAL_BASE` / `REAL_KEY` / `REAL_MODEL` env vars (never hardcode keys) and always rebuild in production mode; `SKIP_BUILD=1` reuses the previous build.
- On Windows sandboxes, Playwright may need `PLAYWRIGHT_BROWSERS_PATH` pointing at the local `ms-playwright` cache.
- Chrome 137+ requires `--disable-features=DisableLoadExtensionCommandLineSwitch` to load extensions from CLI (the e2e launcher already handles it).

## Repository layout

```
packages/
├─ extension/ # the product: WXT MV3 extension (React 18, Tailwind v4, Zustand, Zod)
│ └─ src/
│ ├─ kernel/ # agent loop state machine + contracts (session, message, tool)
│ ├─ context/ # layered context assembly: system prompt, environment, history
│ ├─ tools/ # tool registry + built-in packs (page/tabs/browser/skills) + MCP mount
│ │ # ← the community's first extension point: one file = one tool
│ ├─ engine/ # page_act's implementation: perception/grounding/execute/verify
│ │ ├─ contracts/ # Zod schemas: plan, trace, verification, agent decision
│ │ ├─ page/ # runs INSIDE the page: perception, grounding, execute, verify
│ │ ├─ orchestrator/ # agent-loop.ts (closed loop), run.ts (compiled replay), observe.ts
│ │ └─ batch/ # skill extraction/binding, batch runner
│ ├─ guardrails/ # tool-gate.ts (every tool call) + security.ts (page actions)
│ ├─ trace/ # event bus shared by kernel and engine
│ ├─ llm/ # model port: router, providers (openai-compatible / responses /
│ │ # anthropic), chatWithTools + prompted-JSON fallback, scripted mock
│ ├─ messaging/ # chrome-bridge: typed RPC to content script, tab routing, retargeting
│ ├─ storage/ # chrome.storage repos (sessions, providers, runs, skills, batches, audit)
│ ├─ ui/ # sidepanel (chat-first) / options / onboarding (React)
│ └─ entrypoints/ # WXT entrypoints: background, content, *.html
├─ fixtures/ # local test web app (customer/product CRUD) used by tests & bench
├─ e2e/ # Playwright suites: engine-tests, ext-e2e (mock), real-llm, real-sites, shots
└─ bench/ # benchmark runner → docs/benchmark.md
docs/ # architecture.md, benchmark.md
```

## Architecture invariants (do not break these)

1. **Facts beat predictions.** The model's `expect` post-conditions are predictions; `engine/orchestrator/observe.ts` diffs pre/post-action snapshots into observed facts. Reconciliation in `agent-loop.ts` arbitrates: never re-submit an action whose durable effect was observed; never trust a weak prediction that passed while the page shows no change.
2. **The model never grades itself.** A run succeeds only when the final `evidence` conditions verify against the live DOM (`engine/page/verify.ts`). Don't add code paths that mark success from model output alone.
3. **Page effects must come from `page_act`.** Chat replies do not change pages; any page-changing result must carry the verified result of a `page_act` run. The kernel never judges page work itself.
4. **Every tool call passes the guardrails** (`guardrails/tool-gate.ts`): risk tier (`read`/`act`/`dangerous`), per-tool authorization memory, site allow/blocklist, on-demand `chrome.permissions.request`, audit log. Never bypass the registry.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uiuing/browser-agent](https://github.com/uiuing/browser-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
