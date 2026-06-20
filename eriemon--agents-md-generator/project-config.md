---
trigger: always_on
description: Use when creating, updating, verifying, or reviewing AGENTS.md files and AI coding-agent rule files; when a project or current work folder root lacks AGENTS.md; when root AGENTS.md version metadata is missing or mismatched; when the user explicitly mentions AGENTS.md, agent rules, or scoped AGENTS.md; or when the user is talking about the current workspace/current repository/current work folder and says 计划, 规划, or 准备, which must first trigger a root AGENTS.md check.
---


# AGENTS.md Generator

Create operational context files for AI coding agents. Use facts from the repository first, ask only for missing human policy, and verify before claiming the draft is ready.

当用户明确提到 `AGENTS.md`、`agent rules`、`scoped AGENTS.md` 时，直接进入本技能。

当用户在当前工作区、当前工程、当前仓库、或当前工作文件夹语境里说“计划”、“规划”或“准备”时，也要进入本技能，但第一步不是直接设计，而是先检查当前工作文件夹根 `AGENTS.md` 状态。

当前工作区类请求除了检查当前工作文件夹根 `AGENTS.md`，还要检查全局 `~/.codex/AGENTS.md` 或 `$CODEX_HOME/AGENTS.md` 是否存在且包含受管的全局基线区块。这个全局文件只负责入口规则：要求代理先读取当前工作文件夹根 `AGENTS.md`，而不是替代仓库根文件中的本地约束。

全局 `~/.codex/AGENTS.md` 基线还负责跨工作区的轻量任务入口习惯：优先复用现有工具、库、模板、仓库内相似实现和成熟开源项目；不要每个任务都询问难度或规模，而是先运行或遵循 `task_rating_gate.py` 的启发式判定。只有脚本建议 `ask_user_rating=true` 时，才向用户确认难度和规模；地狱或噩梦难度默认推荐严格规划、颗粒度对齐和可复用开源/相似代码调研。

入口路由要显式区分 4 类任务：`read_only`（解释、分析、规划、现状检查、只读 review）、`design`（收集/修订答案但尚未申请写入）、`write`（正式生成或更新 AGENTS 控制档案）、`governance_high_risk`（release/merge 前治理审查）。只有 `write` 和 `governance_high_risk` 可以触发审查子智能体；`read_only` 明确禁止。

当根 `AGENTS.md` 通过 `default_language` 把默认自然语言回复锁定为某种语言时，这个约束也必须覆盖 Plan Mode：若代理输出 `<proposed_plan>`，标签内的方案正文必须使用该默认语言，除非用户显式切换语言；`<proposed_plan>` 标签本身保持原样，代码、命令、日志、原始错误文本和专有名词可保留原文。

如果用户明确要求创建、更新、审查或修复 `AGENTS.md` / `agent rules` / `scoped AGENTS.md`，先检查根 `AGENTS.md`，然后继续进入完整设计访谈或更新流程；不要因为根文件健康就提前停止。只有当前工作区类“计划 / 规划 / 准备”探测入口在根 `AGENTS.md` 正常时才只报告“检查通过”。

根 `AGENTS.md` 缺失或缺少版本元数据时，先报告异常原因，然后进入完整设计访谈。只有根 `AGENTS.md` 的 `agents_version` 或 `generator_version` 与当前已安装 `agents-md-generator` 版本不一致，并且当前工作文件夹已经有落地内容时，才允许进入最小 takeover 兼容流程。

如果全局 `.codex/AGENTS.md` 缺失、为空、或缺少受管基线区块，不要静默忽略；要明确报告“全局入口规则未落盘”，并给出 `python skills/agents-md-generator/scripts/manage_docs.py sync-global-codex-agents . --write` 作为修复命令。

如果是“已有内容但无根 `AGENTS.md`”的工作文件夹，除了结构检查之外，还要读取精确匹配当前工作目录的 Codex sessions：只接受 `.codex/sessions` 中 `session_meta.payload.cwd` 规范化后与当前工作目录完全一致的会话。先用这些会话和现有落地文件补建 `docs/experience/history_experience/` 历史经验，再生成最新的 `docs/experience/*.md`。

如果用户明确要求进行 Codex Token 用量统计，例如直接要求“统计 Codex token usage / Token 用量 / token 消耗”，走只读工具分支：直接运行 `python skills/agents-md-generator/scripts/codex_token_usage_review.py --hours 48`，按用户要求决定是否附加 `--verbose` 或 `--json`，不要进入 AGENTS 设计访谈。这个分支只在当前环境可解析到 `$CODEX_HOME/sessions` 或 `~/.codex/sessions` 且目录存在时执行；如果检测不到 Codex sessions 目录，要明确拒绝执行并说明原因。`--sessions-root` 只允许等于或位于当前 Codex sessions 根目录之下，用于测试或诊断时也不能跳出当前 Codex sessions 树。泛化的成本、优化、会话健康、/status 建议等问题不算显式触发。

外部工作区生成的 AGENTS/docs 治理命令必须调用已安装的 `agents-md-generator` 运行时，例如 `python <codex-home>/skills/agents-md-generator/scripts/manage_docs.py ...`。不要把这些 governance runtime 脚本写成目标项目自己的 `scripts/manage_docs.py` / `scripts/manage_dirs.py`，也不要为了让命令可执行而把本 skill 的脚本复制进目标工作区。只有 `agents-md-generator` owner repo 自身允许继续使用 repo-local `python skills/agents-md-generator/scripts/...` 自举路径。

## Pipeline

1. **Detect**
   - Run `python skills/agents-md-generator/scripts/inspect_project.py <project>` to gather language, framework, package manager, CI, AI configs, files, and directories.
   - If `root_agents_md_exists` is false, or the root `AGENTS.md` is missing `agents_version` or `generator_version`, or either version does not match the current local installed `agents-md-generator` version, treat the workspace as trigger-required/rebuild-required.
   - Inspect the global `.codex/AGENTS.md` baseline status too. If it is missing, empty, unmanaged, or outdated, report the exact reason and recommend `python skills/agents-md-generator/scripts/manage_docs.py sync-global-codex-agents . --write`.
   - When the user says `计划`, `规划`, or `准备` in the current workspace/current repository/current work folder context, do this root-AGENTS check first. These are trigger entries for inspection routing, not a bypass around the AGENTS.md check.
   - If the root check passes during a current-workspace/current-repository/current-work-folder `计划` / `规划` / `准备` trigger, report that the current work folder root `AGENTS.md` check passed and stop. If the user explicitly asked to create, update, review, or repair AGENTS governance, continue into the full design/update flow instead of stopping.
   - If the root check fails because the root `AGENTS.md` is missing, missing `agents_version`, or missing `generator_version`, report the exact abnormal reason and continue into the full design interview.
   - If the root check fails because `agents_version` or `generator_version` is version-mismatched in an old workspace with landed content, switch to takeover handling instead of the full design interview: minimize identity questions to project type, project/skill name, and default conversation language, but do not skip the structured directory-contract interview. Takeover must still collect local structure, remote structure, feature-placement rules, and any required remote runtime policy before AGENTS.md can be written.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eriemon/agents-md-generator](https://github.com/Eriemon/agents-md-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
