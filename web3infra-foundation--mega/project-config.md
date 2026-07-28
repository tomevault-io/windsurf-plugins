---
trigger: always_on
description: > 本文档基于 Mega 当前仓库状态、Claude Code 官方能力边界，以及 Anthropic 2026-05-14 发布的《How Claude Code works in large codebases》整理。目标不是一次性建设一套“大而全”的 AI 平台，而是把 Mega 现有工程规范逐步转成可被 AI Agent 稳定消费、执行和审计的仓库资产。
---

# Agent.md - Mega Monorepo 的 AI 落地方案

> 本文档基于 Mega 当前仓库状态、Claude Code 官方能力边界，以及 Anthropic 2026-05-14 发布的《How Claude Code works in large codebases》整理。目标不是一次性建设一套“大而全”的 AI 平台，而是把 Mega 现有工程规范逐步转成可被 AI Agent 稳定消费、执行和审计的仓库资产。

---

## 0. 结论

**总体可落地，但必须拆成两条节奏不同的路线。**

| 路线 | 可落地性 | 原因 | 首个可验收结果 |
|---|---:|---|---|
| Agent harness：`CLAUDE.md` / `AGENTS.md` / `.claude/settings.json` / skills / hooks / subagents | 高 | Claude Code 已原生支持项目级 memory、settings、skills、hooks、subagents；Mega 已有 README、development、contributing 等规范来源 | 从 `ceres/`、`jupiter/`、`moon/apps/web/` 启动 Agent 时能自动说出本目录约束和正确检查命令 |
| `mega-mcp`：把 Mega 自身能力暴露为 MCP tools | 中 | Claude Code 支持项目级 `.mcp.json`，Mega 已有 tree、commit、policy、LFS、Buck upload 等 API/模块；但当前没有跨语言符号索引或稳定 Buck target 覆盖 | 先实现 file tree、commit metadata、policy check、workspace map 等已有能力的 MCP 包装 |
| 跨语言 symbol/cross-ref 搜索、5x 快于 `rg`、完整 Code Attribution | 低到中 | 这些依赖额外索引、LSP/Buck2 集成和归因数据模型，不应放在第一阶段承诺 | 作为 Phase 4 研究项，先写基准和数据模型再实现 |

**关键调整**：当前方案可以做，但不能把所有内容都放进 Phase 1。Phase 1 应只交付低风险、可版本化、可手工回滚的上下文和护栏；MCP 和归因能力应独立成产品化 track。

---

## 1. 当前证据与硬约束

### 1.1 仓库事实

- 当前这个本地工作副本不是标准 Git checkout，而是 Libra 工作区；`git status` 会失败，`libra status --short` 才能看到变更状态。公开贡献流程仍需兼容 `docs/contributing.md` 中的 Git/DCO/PGP 要求。
- 当前 `.gitignore` 第 63-64 行整体忽略 `.claude`，但方案要求 `.claude/settings.json`、`.claude/agents/**`、`.claude/skills/**` 被团队共享。这是 Phase 1 的第一个阻塞项。
- 当前存在 `.claude/settings.local.json`，只放了个人权限覆盖；它应继续作为本地文件，不进入共享配置。
- Rust workspace 当前有 12 个成员，而不是 11 个：`api-model`、`ceres`、`common`、`context`、`io-orbit`、`jupiter`、`jupiter/callisto`、`mono`、`orion`、`orion-server`、`saturn`、`vault`。
- README 已声明 PR 前置检查：`cargo clippy --all-targets --all-features -- -D warnings`、`cargo +nightly fmt --all --check`、`cargo buckal build`，并要求依赖变更后运行 `cargo buckal migrate`。
- `docs/contributing.md` 要求提交包含 `Signed-off-by`，并说明 PGP 签名要求；AI 提交规范不能只写 Conventional Commits 和 `Co-Authored-By`。
- 数据库迁移见 `jupiter-migrate/README.md`；entity 输出目录是 `jupiter/callisto/src`。
- REST API 以 Swagger 为准（`/swagger-ui`）；架构见 `docs/architecture.md`。
- `saturn/` 已有 Cedar schema、policy 和解析/授权代码，是策略审查 agent 的合理边界。
- `mono/src/api/api_router.rs` 已聚合 file tree、commit、Buck、artifacts、permission、reviewer 等 API；MCP MVP 应优先包装这些已有能力，而不是先承诺新的代码索引能力。

### 1.2 Claude Code 能力边界

- Claude Code 官方建议通过 `CLAUDE.md`、settings、skills、MCP servers 组成 harness；大仓库并不要求先建 embedding/RAG 索引。
- 项目级 subagents 放在 `.claude/agents/`，项目级 skills 放在 `.claude/skills/`，都可以随仓库共享。
- 项目级 MCP 配置应使用仓库根目录 `.mcp.json`，不是 `.claude/mcp/*.json`。
- Hooks 可以阻止工具调用或让 Stop 事件继续工作，但 command hooks 以当前用户权限运行，必须非常保守；不要在 Stop hook 里默认跑长时间全量构建。
- Read-only subagent 如果只授予 Read/Grep/Glob/Bash，就不能同时要求它写 `.agents-scratch/<task>.md`。要么返回结构化摘要，要么明确给它受保护的写权限并用 hook 限制路径。

---

## 2. 设计原则

1. **先上下文，后自动化**  
   先让 Agent 读到正确入口、模块边界和检查命令，再逐步增加 hook、skill、MCP。不要一开始就把重型质量门禁塞进 Stop hook。

2. **共享配置必须能进版本控制**  
   `.claude/settings.json`、agents、skills、commands、hooks 如果是团队资产，就必须解除 `.gitignore` 中对 `.claude` 的整体忽略；个人 override 继续放 `.claude/settings.local.json`。

3. **`permissions.deny` 只用于安全边界，不用于普通噪声控制**  
   不要 deny `Cargo.lock`、`pnpm-lock.yaml`、`tests/**` 这类开发中经常需要读取的文件。噪声控制靠 `CLAUDE.md` 指引、文件建议、命令作用域和 agent 习惯，安全控制才用 deny。

4. **重命令用显式 skill/command，轻检查用 hook**  
   `fmt --check`、路径保护、配置变更审计适合 hook；`clippy --all-targets`、`cargo buckal build`、`pnpm -C moon lint` 应放到 `/pre-pr-check` 或 skill，由用户或 Agent 在收尾阶段显式运行。

5. **MCP MVP 只包装已有事实，不承诺不存在的索引**  
   第一版 `mega-mcp` 包装 tree、commit、LFS、policy、workspace map、Buck upload/session 等已有接口。`search_symbol`、`cross_ref`、“比 `rg` 快 5x”必须等索引方案和基准存在后再进入验收。

6. **Agent 输出必须可审计**  
   AI 提交至少保留 Conventional Commit、`Signed-off-by`、PGP 签名要求、`Co-Authored-By`；后续再把这些 trailer 与 Mega 的 commit binding / attribution 模型打通。

---

## 3. 改进后的目标架构

```text
mega/
├── CLAUDE.md                         # Claude Code 根 memory：指针 + 跨模块硬规则
├── AGENTS.md                         # Codex/Cursor 等通用 Agent 入口，内容与 CLAUDE.md 同步但不必逐字相同
├── .mcp.json                         # 项目级 MCP 配置，Claude Code 官方共享位置
├── .claude/
│   ├── settings.json                 # 团队共享设置：权限、env、轻量 hooks
│   ├── settings.local.json           # 个人覆盖，必须继续忽略
│   ├── agents/
│   │   ├── rust-explorer.md
│   │   ├── frontend-explorer.md
│   │   ├── schema-policy-reviewer.md
│   │   └── monorepo-impact-analyzer.md
│   ├── skills/
│   │   ├── pre-pr-check/
│   │   ├── rust-workspace-change/
│   │   ├── sea-orm-migration/
│   │   ├── cedar-policy-edit/
│   │   └── conventional-commit/
│   ├── commands/
│   │   └── impact.md                 # 保留少量手动命令；复杂流程优先做 skill
│   └── hooks/
│       ├── deny-protected-paths.sh
│       ├── post-edit-rustfmt.sh
│       └── stop-preflight.sh
├── .agents-scratch/                  # 可选，临时报告目录，必须忽略
├── api-model/CLAUDE.md
├── ceres/CLAUDE.md
├── common/CLAUDE.md
├── context/CLAUDE.md
├── io-orbit/CLAUDE.md
├── jupiter/CLAUDE.md
├── jupiter/callisto/CLAUDE.md
├── mono/CLAUDE.md
├── moon/CLAUDE.md
├── moon/apps/web/CLAUDE.md
├── moon/apps/sync-server/CLAUDE.md
├── orion/CLAUDE.md
├── orion-server/CLAUDE.md
├── saturn/CLAUDE.md
├── vault/CLAUDE.md
└── docs/agent.md
```

### 3.1 `.gitignore` 必须同步调整

把当前整体忽略 `.claude` 改成“共享配置可提交、个人/临时内容忽略”：

```gitignore
# Claude Code shared project assets
!.claude/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [web3infra-foundation/mega](https://github.com/web3infra-foundation/mega) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
