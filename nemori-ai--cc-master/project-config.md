---
trigger: always_on
description: > 本文是 agent / 贡献者进入 `cc-master` 仓库的**第一站**——通读本文即获得做事所需的最小心智地图：这个插件是什么、目录长什么样、哪些红线不能碰、什么时候该读什么。
---


# cc-master

> 本文是 agent / 贡献者进入 `cc-master` 仓库的**第一站**——通读本文即获得做事所需的最小心智地图：这个插件是什么、目录长什么样、哪些红线不能碰、什么时候该读什么。
> §N "触发式深入阅读" 里的链接**不需要预加载**——只在命中对应触发条件时跳转。这是渐进式披露（progressive disclosure），不是 reading list。
> **运行时的灵魂在 [`plugin/src/skills/master-orchestrator-guide/canonical/SKILL.md`](plugin/src/skills/master-orchestrator-guide/canonical/SKILL.md)（SKILL A）——它是 `SessionStart` hook 每次 compaction 全文重注的常驻手册。本文绝不复述它的七镜头 / 红线 / 决策程序，只给定位与指针。**
> CLAUDE.md = `@AGENTS.md` 一行 include——Claude Code 与 codex 等读同一份真相源。

---

## 1. 这个插件是什么

`cc-master` 是一个 **ship-anywhere 的 agent harness 插件项目**：产品目标是让 Claude Code 或 Codex 等 agent harness 的主会话 agent 都能被初始化成 long-horizon **master orchestrator（总指挥）**。源码结构已按 paragoge 范式重构为 **CLI + source-to-adapter plugin projection**：共享产品语义在 `plugin/src`，Claude Code / Codex 等 host 只通过 adapter 投影成各自可安装产物。

它**不是**：agent framework / library，不是某个 LLM API 的包装，不依赖 agent-teams 或 scheduled routines（见 §3 红线 5）。它的 runtime 是 **commands + 8 skills + hooks + 一个 board 文件**的薄编排层；它的 repo 形态是 **paragoge-style 多 harness adapter 工程**。

**产品愿景 / 北极星（charter）**——cc-master **致力于让** agent 化身 master orchestrator 并具备六项能力：① 异步并行多线程推进、把目标完整落地；② 控制 token 消耗速度；③ 把握自主决策 vs 寻求人类接入的边界；④ 目标的分解 / 管理 / 更新 / 规划；⑤ 资源消耗速度合理前提下最大化实施效率的调度编排；⑥ 按复杂性 / 难度 / 时长选合适的模型。这是**方向目标（aspirational）而非「已全部兑现」**——哪些已落地、哪些 design-only 由 gap 审计度量。**完整六条 charter 的 SSOT 在 [`design_docs/spec.md` §1.0](design_docs/spec.md)**，本段只是摘要回指，不复述。

**深入指引**：
- 用户视角（怎么装、怎么用、三范式对比）→ [`README.md`](README.md)
- 编排方法论（魂）→ [`plugin/src/skills/master-orchestrator-guide/canonical/SKILL.md`](plugin/src/skills/master-orchestrator-guide/canonical/SKILL.md)
- workflow 脚本写法（机制）→ [`plugin/src/skills/authoring-workflows/canonical/SKILL.md`](plugin/src/skills/authoring-workflows/canonical/SKILL.md)
- 完整设计 spec → [`design_docs/spec.md`](design_docs/spec.md)

---

## 2. 仓库形态 + 关键不变式

```
cc-master/
├── AGENTS.md / CLAUDE.md     ← 你正在读 / CLAUDE.md = @AGENTS.md 一行 include
├── README.md / README_zh.md  ← 面向使用者的产品介绍（怎么装 / 怎么用）
├── CONTRIBUTING.md           ← dev loop + before-PR 三道门（红线 SSOT 已外链本文 §3）
├── CHANGELOG.md
├── plugin/                   ← paragoge-style plugin source/projection
│   ├── src/                  ← **单一语义源**：SAP skills + PHIP hooks + host manifest/commands source
│   │   ├── .claude-plugin/   ← Claude Code adapter manifest source（第二 host 出现后可下沉到 host adapter）
│   │   ├── commands/         ← command body source（第一阶段按 Claude Code 投影）
│   │   ├── adapters/         ← cross-surface origin host-native invocation 映射；per-host strategy 投影，状态机/writer 仍归 ccm
│   │   ├── knowledge/        ← skill knowledge graph **repo-only authored/meta** source；point/module/typed edge 是全局事实，accepted composition 是派生 skill artifact；可多 composition 消费同一 SSOT；本目录禁止进入 dist/package/release，能力以 `skill-knowledge contract` 为准
│   │   ├── skills/           ← **SAP**：`<skill>/canonical/` runtime 源 + `<skill>/adapters/<host>/strategy.yaml`
│   │   │   └── _hosts/       ← host-wide skill adapter base（当前 `claude-code`）
│   │   └── hooks/            ← **PHIP**：`_manifest/` contract + `_hosts/<host>/` + `<hook>/implementations/<host>/`
│   └── dist/
│       ├── claude-code/      ← Claude Code adapter 可安装产物；由 `scripts/sync-plugin-dist.sh` 从 `plugin/src` 投影
│       └── codex/            ← Codex adapter 可安装产物；同一 plugin 版本线，release asset 按 harness 拆分
├── .claude/skills/            ← **项目自用** dev/meta skill 源（含需求发现、skill 造/评/治、多 harness plugin 架构/投影/发布工程，**不分发**）
├── .agents/skills/            ← Codex 项目级 skills 投影（由 `scripts/sync-codex-skills.sh` 从 `.claude/skills` 生成；默认 symlink）
├── ccm/                      ← **独立产品/引擎**（ADR-014）：`packages/engine`（`@ccm/engine`·board 引擎 SSOT·TS）+ `apps/cli`（`ccm`·per-OS Node SEA 二进制）+ 未来 desktop。pnpm/Turborepo monorepo（tsdown/biome/changesets）。**独立安装、不随 plugin 分发**——plugin / webview / 未来客户端都是消费方：hooks/skill 脚本经进程边界 shell 调 `ccm`，webview 吃 vendored `@ccm/engine` IIFE。`node_modules`/`dist` gitignored
├── .githooks/                ← 版本化 git hook（`scripts/install-git-hooks.sh` 设置 core.hooksPath）；pre-push 自动检查 `plugin/dist` 与 `plugin/src` 同步
├── scripts/                  ← 带外 **dev-only** 脚本：sync-plugin-dist / check-plugin-dist-sync / sync-codex-skills / skill-knowledge / eval-trigger / eval-benchmark / skill-lint（仅开发本仓用、repo 根调用，**不随 plugin 分发**；裸路径在此正确）。运行时带外脚本（codex-review / view webview）住 `plugin/src/skills/master-orchestrator-guide/canonical/scripts/`（随 skill 分发，见上；旧 cc-usage.sh 已退役 → `ccm usage`）
├── adrs/                     ← 结构性决策快照（ADR-001..022 + AGENTS.md 规约）
├── tests/                    ← hook 测试（bash）；run-tests.sh 编排 hook + content contract
├── design_docs/             ← 设计文档 + eval/ + dogfood-findings.md（plans/ gitignored；`harnesses/` 是 paragoge 迁移后校对版 host 机制资料库）
└── examples/                 ← 可跑样例（sample-orchestration：i18n 场景 walkthrough.md + smoke.sh 冒烟证明）
```

**关键不变式**（每条一句话 + SSOT；硬约束的完整体在 §3 红线）：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nemori-ai/cc-master](https://github.com/nemori-ai/cc-master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
