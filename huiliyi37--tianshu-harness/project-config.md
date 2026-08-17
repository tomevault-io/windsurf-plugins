---
trigger: always_on
description: Terminal coding agent optimized for DeepSeek V4 prefix cache. Node.js 24+ (`engines` pins 24.1.0) / TypeScript strict / 纯 ANSI 终端 UI（`src/tui/engine/`，零 React/Ink 渲染） / node:test。桌面端 `desktop/`（Tauri + React，闭源）与 VS Code/Cursor 插件 `vscode-extension/`（开源，随 sync 进公开仓）均经 `src/server/` sidecar 驱动同一内核。CLI 命令仍为 `rivet`。插件打包/发布见 `docs/VSCODE-EXTENSION-RELEASE.md`。
---

# 天枢 (Tianshu) / Rivet

Terminal coding agent optimized for DeepSeek V4 prefix cache. Node.js 24+ (`engines` pins 24.1.0) / TypeScript strict / 纯 ANSI 终端 UI（`src/tui/engine/`，零 React/Ink 渲染） / node:test。桌面端 `desktop/`（Tauri + React，闭源）与 VS Code/Cursor 插件 `vscode-extension/`（开源，随 sync 进公开仓）均经 `src/server/` sidecar 驱动同一内核。CLI 命令仍为 `rivet`。插件打包/发布见 `docs/VSCODE-EXTENSION-RELEASE.md`。

顶层索引与运行时排查：[`AGENTS.md`](./AGENTS.md)。架构总览：[`docs/architecture-overview.md`](./docs/architecture-overview.md)。

## Build & Test

```bash
npm install && npm run build
npm test          # ~1,133 test files / ~13,000 cases, node:test + node:assert/strict (runner: scripts/run-node-tests.ts，分批 spawn，末尾 tests 行只是最后一批)
npm run typecheck # 跨进程闸门：并发会话检查同一份源码时只跑一次，其余复用（秒回 = 命中缓存，不是没跑）
                  # 指纹 = git HEAD + 脏文件内容 hash；逃生口 typecheck:direct 或 RIVET_TYPECHECK_SHARE=0
```

## Architecture

```
main.ts → AgentLoop (agent/loop.ts)
  ├── RuntimeHookPipeline (agent/runtime-hooks.ts)  ← TUI 2.x 核心
  │     纯分阶段执行器 + 错误隔离（任一 hook 抛错只走 onError，不中断 turn）
  │     5 阶段真实调用点（非概念，已钉在 loop 主路径）：
  │       preTurn / afterPerception → turn-perception.ts
  │       postTool                  → tool-execution.ts（每个工具执行后）
  │       runCompaction             → turn-orchestrator.ts（postTool 后、postTurn 前）
  │       postTurn                  → turn-completion.ts
  │       postSession               → loop.ts（经 turn-orchestrator 调度）
  │     hooks 由 createDefaultRuntimeHooks() 条件装配（create-runtime-hooks.ts），
  │       60+ 模块（src/agent/hooks/），按 deps/开关 gated。默认会话实际激活 ~18+：
  │       常驻 8（base 数组无条件）：perception, signal-consumer, kick,
  │               vigor×2(afterPerception+postTool), theta, stigmergy, radio
 │       advisoryBus 恒构造(loop.ts) → 这批默认全开：self-verify、edit-tool-advisory、
 │               lossy-observation、spec-verify-gate、typecheck-reminder、todo-reminder、
 │               security-pattern(写后正则查危险模式；config agent.securityGuidance=false
 │               或 RIVET_SECURITY_GUIDANCE=0 关)、
 │               context-pressure(需 token getter)、dedup-guard(需 streamedText getter)
  │       deps-gated（默认多为真）：playbook-reflect、anchor-break-shadow、dream/skill-distill、
  │               meridian、telemetry-flush、physarum-file-access、memory-learning、courage/ccr(starSoul)
  │       默认关(opt-in)：songline、constellation、hearth-observe、companion、
  │               dispatcher(自动委派)、anchor-break-scout、mcts-planning/blind-exploration(antiAnchoring)
  ├── AgentSession (messages, usage, turn count)
  ├── EvidenceTracker + FileHistory
  ├── Stores: claim-store, stigmergy-store, playbook-store, trace-store
  └── Tool dispatch → API (SSE streaming) → TUI (纯 ANSI, src/tui/engine/) / Desktop (server SSE)
```

> ⚠️ 旧文档写「9 个固定 hook」是误导性简化。真相是条件装配，开关在 `loop-factory.ts:createRuntimeHooksPipeline` 传入的 deps。改 hook 行为前先看 `create-runtime-hooks.ts` 的 gate 条件，不要假设某 hook 一定在跑。

Key modules: `src/agent/` (loop, hooks/, session, coordinator, star-domain), `src/api/`, `src/tui/`, `src/tools/`, `src/prompt/`, `src/compact/`, `src/cache/`, `src/context/`, `src/server/` (desktop sidecar), `src/repo/`, `src/mcp/`, `src/auth/`, `desktop/`, `vscode-extension/`

## Conventions

- Node.js test runner (`node:test` + `node:assert/strict`), not Vitest or Jest
- ESM with `.js` extension in imports
- Immutable patterns — spread operator, no mutation
- Error classification via `classifyApiError()` — no ad-hoc status code checks in clients
- Tests: `src/**/__tests__/*.test.ts` mirrors source structure

## Known Constraints

- **Prefix cache is the core optimization.** System prompt and early messages must stay stable within a session — avoid rewriting history or injecting before anchor points.
- DeepSeek V4 may emit tool JSON in text content (`hasToolJsonInContentBug` in client config)
- Codex client receives text via both `output_text.delta` and `output_item.done` — `seenTextDelta` dedup handles this
- Agent loop `onTurnComplete(usage, turn, isFinal)` — intermediate turns keep writer alive, only final turn destroys it
- User input during streaming goes to SteerBuffer (not direct interrupt), injected at next tool result
- **星域 `toolWhitelist` 是生效的工具交集过滤器，对当前内置域退化为恒等。** `star-domain.ts` 有 11 个域（天枢/破军/天府/天梁/天权/天机/天璇/辅/文曲/瑶光/华盖）。worker 创建时 `allowedTools = profile.allowedTools ∩ domain.toolWhitelist`（`work-order.ts:toolsForAuthority`）。当前 11 域白名单**逐字相同且是全集**（2026-07-15 起含 `browser_debug` + `computer_use`） → 对内置域交集退化为恒等（no-op），域间行为差异只来自 `systemPromptSuffix`/`volatileBlock`/`courageThreshold`/`decisionStyle`。但机制本身**真实生效**，三处会咬人：①profile 若含白名单外工具（`browser` 等），设了 authority 就被静默削掉；②authority 拼错/域未加载 → **fail-closed 返回 `[]` deny-all**（有意护栏，勿改成回退 profile 全集）；③自定义域（card frontmatter）的 `toolWhitelist` 完全生效，会真实削减工具。改它前先想清楚命中的是哪种场景。
- **Hook 信号经 AdvisoryBus 统一收编后注入**（`59e52394`）。hook 不直接改 prompt，只能用 `RuntimeHookEffects`（`injectUserMessage`/`emitPhaseChange`/`setStrategy`…）；带 priority/ttl/category 的信号优先 `advisoryBus.submit`，降级才 `injectUserMessage`。注入走 system-reminder 通道，不重写 `frozenBase`/`volatileBlock`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huiliyi37/Tianshu-harness](https://github.com/huiliyi37/Tianshu-harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-17 -->
