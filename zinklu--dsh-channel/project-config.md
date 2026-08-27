---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The message-channel layer for DeepSeek Harness (dsh, a Cordis 4.x plugin framework): a contract package, a shared handler/pure-function kit, and one provider package per platform (Telegram, WeChat, Feishu). npm workspaces monorepo, ESM, TypeScript `NodeNext`, tests on `node:test` via `tsx`. Pinned dsh baseline: `@deepseek-ai/dsh-*@0.1.0-rc.6`. **This repo never patches dsh core** — it only consumes four dsh touchpoints: `session/event`, `ctx.agents.create/resume`, the `approval/request` waterfall, and `ctx.credentials`.

`dsh-channel-design.md` is the single design source (R1–R10 hard constraints in §6, off-track signals in Appendix B). Read the relevant section before changing a contract or mechanism.

## Commands

```bash
npm install
npm run build                      # all five packages → packages/*/lib, in dependency order
npm run test                       # build, then test every workspace
npm run typecheck                  # build, then tsc --noEmit every workspace

npm run build -w dsh-channel-kit   # one package
npm run test  -w dsh-channel-telegram

# single test file / single test (run from the package directory)
cd packages/channel-kit && npx tsx --test test/merge.test.ts
cd packages/channel-kit && npx tsx --test --test-name-pattern='iron rules' test/merge.test.ts
```

Build/test ordering matters because cross-package imports resolve through `node_modules` symlinks to each package's **built `lib/`**, not `src/`:

- `dsh-channel-kit` and `dsh-channel` tests import `../src/*.ts` directly — no build needed.
- Provider tests (`telegram`/`wechat`/`feishu`) have a `pretest` that rebuilds only *that* provider. After editing `channel` or `channel-kit`, rebuild those first (or run root `npm run build`) or the provider tests run against stale `lib/`.
- Root `build` lists packages explicitly because `--workspaces` walks alphabetically and would build `channel-feishu` before `channel-kit`.

Local debugging (see README for details):

```bash
TELEGRAM_BOT_TOKEN='...' node scripts/run-echo-bot.mjs   # echo bot, no model
scripts/run-dev-bot.sh                                     # full agent via the real dsh launcher + scripts/dev-bot.yaml
```

`run-dev-bot.sh` maintains a `dev-bot` profile under `$DSH_HOME` with this repo's packages as `file:` deps (copies, refreshed from `packages/*/lib` each launch) and runs from the gitignored `agent-workspace/`. Two processes long-polling one bot token collide (Telegram 409) — check `ps` for a running `dsh` before starting another. dsh rc.6 has no log sink (`ctx.logger` only fills an in-memory buffer); `scripts/dev-logger.mjs` is the dev-only stderr exporter the dev bot inserts (`DEV_BOT_LOG_LEVEL`, default 3=debug; cordis levels are error 0 < info 1 < warn 2 < debug 3, default threshold 1 drops `warn`).

## Architecture

Dependency direction is strict and one-way (R3: a consumer/policy plugin may only peer-depend on `dsh-channel`; a provider's name never appears in anyone's deps):

```
dsh-channel (contract, zero implementation)
   ▲
dsh-channel-kit (config/ + format/ → policy/ → bridge/ ; plus testing/)
   ▲
dsh-channel-telegram / -wechat / -feishu (transport only)
```

Cross-package deps among dsh packages are `peerDependencies` (+ `devDependencies` for tests); `dependencies` is reserved for third parties.

### `packages/channel` — the contract (`src/index.ts`, one file)

`declare module` for `ctx.channels` + `channel/message` (emit) / `channel/deliver` (waterfall) / `channel/status` (emit); `ChannelRegistry` (a Service, mirrors `LlmRuntime`); `abstract class Channel` (a *plain* abstract class mirroring `LlmAdapter` — deliberately **not** a Service, since many platforms coexist under one ctx key). Platform differences are expressed only as `get` capability facts with conservative defaults (`formatTier`, `streamingMode`, `supportsChoices/Edit/Typing/Media/Reply/Threads/Silent/Reconciliation`, …) plus the small required surface. A method only one platform can implement must never land on `Channel`. Registry keys are `(id, accountId)`; `bindChatKey`/`chatKeyOf` expose sessionId → chat binding for proactive push.

### `packages/channel-kit` — the shared handler

- `config/` — the schemastery fragments (`agentRoutingSchema`/`channelBehaviorSchema`/`allowedUserIdsSchema` + `AgentRoutingConfig`/`ChannelBehaviorConfig`) every provider composes its `Config` from; `BridgeConfig` is derived from these types so handler and schema can't drift.
- `format/` — leaf text/transport shaping, no decisions, no state (`chunkText`, `renderForTier`, `promptHint`, `assertMediaWithinLimit`, `proxiedFetch`).
- `policy/` — decision logic as **pure reducers** `(state, input) → { state, effects }` (`mergeReduce`, `route`, `deliverQueueReduce`, `streamReduce`, `draftThrottleReduce`, `outboundEchoReduce`, recovery/finalization/busy/approval-render/prompt-render/tool-display) and the two policy *seams* with defaults: `PresentationPolicy` and `RecoveryPolicy`. Policies hold no state; state lives in the bridge.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZinkLu/dsh-channel](https://github.com/ZinkLu/dsh-channel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
