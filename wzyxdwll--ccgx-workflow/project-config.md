---
trigger: always_on
description: > [根目录](../CLAUDE.md) > **skills-v2**
---

# skills-v2 (CCG Multi-Model Collaboration System)

> [根目录](../CLAUDE.md) > **skills-v2**

**Last Updated**: 2026-05-06 (v4.5.2)

---

## 变更记录 (Changelog)

> 完整变更历史请查看 [CHANGELOG.md](./CHANGELOG.md)

### 2026-05-06 (v4.5.2) — ⚡ 体验优化：14 命令切事件驱动等待 + gemini plugin 全 8 处 patch

- ⚡ **14 个多模型协作命令统一"事件驱动等待协议 v2"**：删除主线主动 `TaskOutput({block:true, timeout:600000})` poll，改 spawn 后 turn end + 等引擎 `<task-notification>` 自动唤醒。覆盖 review/analyze/optimize/plan/execute/codex-exec/test/workflow/spec-*/team/debate/spec-impl 全部多模型场景。autonomous 已用此模式 dogfood 验证，现推广全场景。**消除 4-5 min poll interval**，体验对齐 autonomous。
- 🐛 **gemini plugin v1.0.1 全 8 处 spawn windowsHide patch**：新发现 P-4/5/6/7/8（broker daemon / runCommand / taskkill / spawnDetached / binaryAvailable），加 `templates/scripts/repatch-gemini-plugin.mjs` 幂等一键修复脚本（regex 匹配 CRLF/LF 兼容）。
- ✅ 测试 1309/1309 不变（仅文档/模板变更），typecheck pass。

### 2026-05-06 (v4.5.1) — 🐛 Hotfix: launcher path namespace + plugin known-issue 文档化

- 🐛 **launcher path .ccg/ namespace 修正**（commit `79cf8b4`）：v4.5.0 install 验证抓出 8 处文件引用 `~/.claude/scripts/ccg-phase-runner-launcher.mjs` 但 installer 装到 `~/.claude/.ccg/scripts/`（v1.7.75 namespace 隔离）。新机制启用后会 file not found。修正 6 文件含 `DEFAULT_LAUNCHER_PATH` const。抓在 install 阶段，没等 uni-iam dogfood 暴露。
- 📝 **新建 `.ccg-migration/PLUGIN-PATCHES.md`** — 上游 plugin known issue + 本地 patch 持续维护文档。首批 P-1 + P-2 都涉及 `gemini@google-gemini` v1.0.1 Windows spawn 行为：
  - **P-1**: `spawnBackgroundWorker` (line 701) 漏 `windowsHide: true` → spawn 抢焦点（codex plugin 同款代码有，对照参考）
  - **P-2**: 底层 `spawn("gemini", ["--acp"])` (acp-broker.mjs:85 + lib/acp-client.mjs:243) 缺 `shell: process.platform === "win32"` + `windowsHide: true` — gemini 在 Windows 是 `.cmd` 脚本必须 shell:true 才能找到，否则 ENOENT 序列化为 `[object Object]`；本地已 patch（`.bak` 留底），早期发现 v4.5.1 补档
  - 本地 patch 立即缓解，永久路径待上游 PR
- 📊 测试 1309/1309 不变（仅文档 + 字符串路径修正）。

### 2026-05-06 (v4.5.0) — 🚀 phase-runner Bash subprocess + 三层 OS 进程隔离（8 phase / 5 wave dogfood）

> v4.5 把 `Agent(subagent_type="phase-runner")` 主进程内 sidechain spawn 替换为 `Bash(claude -p --agent ccg/phase-runner)` OS-level 子进程。三层进程隔离（主线 → CLI 子进程 → 可选 plugin 进程），治本 v4.4.x main-process RSS leak（uni-iam 实测撞 23GB → v4.5 设计目标 < 8GB）。8 phase / 5 wave 全 completed，wall ~3h（vs 估算 10-15d sequential），双 gate G2 (memory) + G3 (broker tx_id) PASS。

- ✨ **Phase 1 (P1a, `e1f0fab`)**: phase-runner spawn 改 Bash 直调路径。`buildPhaseRunnerBashCommand(phase, brief, jobId)` helper + `autonomous.md` Step 4.2-4.3 模板改写；stream-json 流式输出落 `.context/jobs/<job-id>/progress.jsonl`。
- ✨ **Phase 2 (P1b, `20fb5fe`)**: process supervisor + recovery。`src/utils/process-tree.ts`（Windows `taskkill /T /F` + POSIX `setsid` 进程组）+ `templates/scripts/ccg-phase-runner-launcher.mjs`（包装 `claude -p`，原子写 state）+ `cancel.md` cooperative + grace + kill-tree。
- ✨ **Phase 3 (P1c, `1086aca`) — G2 PASS**: nested RSS stress gate。pilot 2 矩阵（trivial-single N=3 / plugin-single N=2）。**关键发现**：codex C1 200-333MB linear 推导被实测推翻——marginal post-warmup **5-15 MB/nested**，4-outer worst case ~1.1GB（远低于 codex 4-6.7GB）。`MAX_NESTED_PER_PHASE = 3`。
- ✨ **Phase 4 (P1d, `285b2ac`) — G3 PASS**: broker tx_id isolation + 20-way stress。`src/utils/broker-log.ts`（writer + reader + 8 字段强 schema，tx_id via `crypto.randomUUID`）+ launcher 注入 `CCG_BROKER_TX_ID` env。**100k spawn 0 碰撞 / 227ms**；**2000 spawn 4-outer × 5-nested concurrent 0 misattribution / 79s**。
- ✨ **Phase 5 (P1e, `c722d08`)**: cost/cache real-workdir benchmark。`tests/poc/prompt-cache-bench.ts` + 2 repo × rapid TTL × 5 spawn = 10 真实 claude CLI 数据点。**D3 budget 经实测无需修订**（fast=$1 / triple=$2 / debate=$5）；warm cache 86% reduction 验证 PoC T3 cold→warm 27× 投影。autonomous milestone real-cost: triple warm $10-15 / cold $15-27。
- ✨ **Phase 6 (P1f, `097cda7`)**: nested G-plan opt-in + launcher wiring。`phase-runner.md` 删除 v4.0.1 "引擎层硬约束"段（CLI 模式下 T9 实测失效）+ 新增 "Nested rescue delegation"；`quality-router.ts` 加 `nestedRescue` field + `--nested=on|off` flag + `buildPhaseRunnerLauncherCommand` helper；autonomous Step 4.0c + Step 4.2-4.3 wire launcher。**默认 `--nested=off` + `useLauncherWiring=false` 100% 等价 v4.5 v1**（baseline `285b2ac`，单测 §7 验证）。
- ✨ **Phase 7 (P2, `614d742`)**: `/ccg:status` v2。dashboard + `--tail` 双模式 + `stream-renderer.ts` + `stuck-detector.ts`（3 类警告：相同 tool_call ×3 / single tool >30s / stream stalled >5min）；ASCII-7 progress bar regex enforced（Windows cp936 安全）。
- 🔄 **PoC D3 fast budget 升级 0.5 → 1.0**（commit `359ea8f`）：T1 实测大 CLAUDE.md 场景 0.5 会 truncate，1.0 留 2.4-7.4× buffer。
- 📊 **架构数字**：测试 1100 → **1309**（+204）/ 命令 28 不变（status 内功能扩展）/ 新 helpers + 4：`process-tree.ts` / `broker-log.ts` / `stream-renderer.ts` / `stuck-detector.ts` / 新 launcher script + 1：`ccg-phase-runner-launcher.mjs`。
- 🐛 **已知未验证项**：(1) **uni-iam 真实环境 RSS < 8GB 验证**留用户在 install + 新会话执行（chicken-and-egg）。Release entry criteria 待办：5+ phase autonomous，全程 claude.exe RSS < 8GB。(2) **跨平台 broker.log 验证**：Phase 4 单机器跑 stress，跨 Windows + Linux 一致性留 Phase 8 dogfood 自然覆盖。(3) **debate / impl 路径 silent fallback** 完全消除需 openclaw 路线 → v4.6+ 候选。
- 📋 详见 [CHANGELOG.md](./CHANGELOG.md#450---2026-05-06) + 迁移指南 [.ccg-migration/v4.4-to-v4.5.md](./.ccg-migration/v4.4-to-v4.5.md)。

### 2026-05-05 (v4.4.3) — 🔒 silent fallback 治理收尾（verify path 补齐 + debate retry protocol 硬约束）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wzyxdwll/ccgx-workflow](https://github.com/wzyxdwll/ccgx-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
