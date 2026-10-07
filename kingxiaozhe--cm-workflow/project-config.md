---
trigger: always_on
description: Codex-native、spec-driven 的双运行时工作流分发包：需求文档 → 开发规格 →
---

# cm-workflow

Codex-native、spec-driven 的双运行时工作流分发包：需求文档 → 开发规格 →
实现 → 独立审查 → QA → 文档同步，并支持存量功能只读测试与可选外部专家研究。Codex Skills 与
`runtime/` 是权威流程，
Claude Code 跨平台直接使用 `/cm-*` Skills；`compat/claude-commands/` 保存
macOS/Linux 的历史 `/cm:*` 别名包装。

## 技术栈

- 语言: Markdown（prompt 资产主体）+ Bash（安装与可视化脚本）+ PowerShell（Windows 安装器与自检入口）+ Python（标准库校验/夹具）+ JavaScript（Node.js 18+；Playwright 可选）
- 框架: Pi/BYZ 原生 package + Codex plugin + Agent Skills；Claude Code commands/agents 兼容层
- 包管理: 根 `package.json` 保存 Pi/BYZ manifest 和 npm 安装命令入口，无 npm 依赖或 lifecycle scripts；Codex npm 入口复用 `install-codex.sh`，Claude Code 用 `install.sh` / `install.ps1`
- 版本控制: remote
- 交付形态: Pi/BYZ package + Codex/Claude Code 本地开发者工具（无构建产物）
- 业务地图: 跳过（开发者工具分发仓库，不是 `src/` 风格业务应用）；本地扫描产物不提交，公开架构见 `docs/architecture.md`

## 常用命令

- 安全扫描：`$cm-security` / `/cm-security`，分支差异加已跟踪未提交修改；`--all` 扫全部已跟踪文件。工具缺失保留缺口，不自动修复或安装；定点验证 `node --test scripts/cm-security.test.mjs`。
- 分支影响分析：直接 `$cm-test` / `/cm-test`；默认比较当前提交与主分支，输出业务影响和回归重点，再检查单测覆盖率；“补齐单测”连续补测、重跑、审查。显式目标保留原模式。

- JS 修复入口: `node scripts/cm-fix-host.mjs --help`；同仓specs可显式配置`protectSpecs:true`，原权限/审查不变，宿主只回文本提案，由固定沙箱写入；QA子配置复用，详见`docs/js-workflow-control.md`

- cm-ai 进度：当前任务真实阶段更新 `.cm-status.json`；单／批驱动检查边界及阻塞说明写 stderr。可选 guidance 只解释原恢复前提，不授予重试权限；保护模式不转发原始命令输出，并行状态卡旧完成不能抢占，详见 `docs/js-workflow-control.md`。

- CM 收尾：有文档能力的新 run 默认在原 documentation_inspect 做只读资料核对；原 workflow-config 可用 knowledgeCloseout:false 关闭，旧 run 沿快照，报告不代替 Review/QA。见 `skills/cm-ai/references/knowledge-closeout.md`。

- 双运行时容灾: 断点交接 `node scripts/cm-failover.mjs status|handoff|probe --specs {SPECS_DIR}`（只读，不标记完成）；启动前选路 `cm-ai-host.mjs serve ... --failover`（显式 opt-in，只定起跑运行时，切换必播报）。运行时声明 `runtimes.available` 与四个预设见 `templates/cm-workflow.yml`、`runtime/workflow-config.md`（声明不等于派发）。主从与边界见 `docs/runtime-failover.md`；定点测试 `node --test scripts/cm-failover.test.mjs scripts/cm-runtime-failover.test.mjs`。

- 运行时声明：`$cm-runtime` 先查看并默认保留，主动修改才进入预设向导；高级模型入口只读说明。`show|preview <preset>|set <preset>|set --user <preset>|unset --user`；向导确认 `[y/N]`，`set --expect-preview <sha256>` 拒绝过期预览。项目 > `~/.cm-workflow/runtimes.yml` > 未声明；只影响新 run。安装器可跳过配置且默认保留，`--yes` / `-Yes` 或非 TTY 跳过。
- 外部模型：`node scripts/cm-model-setup.mjs configure --provider codex|claude`，一组模型／可选强度；`--external-models` 显式启用，宿主不切模型，恢复只读原快照，未知调用不重派。范围及回滚见 `docs/external-models.md`。

- 安装依赖: 无 npm 安装步骤（`package.json` 无依赖；可视化工具按需使用外部 Playwright）
- 开发运行: 不适用（直接维护 Markdown 与脚本）
- 构建: 不适用（无构建产物）
- 测试: `bash scripts/test-shell-compat.sh && node --test scripts/cm-ai-admission.test.mjs scripts/cm-workflow-config.test.mjs scripts/cm-log-event.test.mjs scripts/cm-task-gate.test.mjs scripts/validate-test-cases.test.mjs && python3 scripts/test-task-gate.py`
- Lint/安全: `python3 scripts/validate-public-repo.py && python3 scripts/scan-public-safety.py`
- Bash 语法: `find . -type f -name '*.sh' -print0 | xargs -0 -n1 /bin/bash -n`
- Pi/BYZ package: `pi install git:github.com/kingxiaozhe/cm-workflow`
- Codex 安装: `./install-codex.sh`（装完新开会话跑 `$cm-check`）
- Claude 安装: `./install.sh`（核心运行时原子更新/失败回滚；可选更新器 best-effort，装完跑 `/cm-check`）
- Windows 安装: `powershell -ExecutionPolicy Bypass -File install.ps1`（同样原子更新）
- 一致性自检: `./scripts/cm-check-runtime.sh`；完整当前会话机械/语义接线：`node scripts/cm-check-host.mjs --help`
- 完整 `cm-check` 默认先调用 `scripts/cm-check-update.mjs` 查询并升级已管理安装，再检查返回目录；低层检查仍只读。定点测试：`node --test scripts/cm-check-update.test.mjs scripts/cm-check-host.test.mjs`。
- 全局日志夹具: `./scripts/cm-check-runtime.sh --log-fixtures`
- API 用量报告: `python3 scripts/cm-usage-report.py --last 10`
- OpenAI 兼容调用夹具: `python3 scripts/test-cm-openai-compatible-call.py`
- 审批 manifest: `python3 scripts/cm-spec-manifest.py {specs}`
- PRD 单轮恢复夹具: `python3 scripts/test-cm-prd-review-gate.py`
- PRD JS 变更/会话恢复: `node scripts/cm-prd-host.mjs --help`；定点夹具 `node --test scripts/cm-prd-completion.test.mjs`
- 公开包检查: `python3 scripts/validate-public-repo.py`
- 安全扫描: `python3 scripts/scan-public-safety.py`
- JS runtime 兼容夹具: `node --test experiments/js-orchestration/*.test.mjs`（当前完整套件要求 macOS + Node.js 24.14+，含原生 SQLite；源码随 runtime 分发不等于 host 已激活）
- JS 单任务入口: `node scripts/cm-ai-host.mjs --help`；默认当前会话，Codex同仓specs可显式选`--protected-config`并按轮授权真实开发/审查，原workflow配置接受保护QA与审前文档；限制见`docs/js-workflow-control.md`，安装与真实模型验收另验
- JS 多任务入口: `node scripts/cm-ai-batch-host.mjs --help`；复用原batch/单任务宿主，Codex/Claude可选`--protected-conversation-config`同仓文本提案/沙箱，Review按feature/task/轮次授权
- 当前会话执行证据：四驾驶员可显式配置 `liveEvidence`，合同与不可用恢复见 `docs/live-evidence-drivers.md`；静态文件不能代替实际执行。
- 第二轮 blocked 后的人工补正：明确人审范围后使用一次性的 `cm-task-gate.mjs prepare-human-correction` / `publish-human-correction-review`；保留旧证据，补正独立 Review，详见 `docs/human-correction.md`。
- cm-fix 原根因拒绝恢复：standalone `rediagnose` 需 `--allow-rediagnosis`；原 run 一次新诊断/fresh r2，历史不覆盖，第二次拒绝停机；详见 `skills/cm-fix/references/js-host.md`。
- JS 重构: `node scripts/cm-refactor-host.mjs --help`；协议见 `skills/cm-refactor/references/js-host.md`，轻量/批量、判官准备、恢复、Learning/规则/备忘共用原门禁；真实双端使用与Git交付另验
- Claude JS 接线: 单/批任务 `--runtime claude` 已接原开发/Review/QA链，已安装CLI通过本机回环配置诊断；真实模型Review及Skill实装仍缺，不等于双端验收
- 只读任务提案: `scripts/cm-task-gate.mjs` 的 `prepare-mark-done` / `verify-mark-done-plan`；参数与私有输出契约见 `runtime/task-gates.md`，不是完成授权
- 插件验证: `python3 ~/.codex/skills/.system/plugin-creator/scripts/validate_plugin.py .`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kingxiaozhe/cm-workflow](https://github.com/kingxiaozhe/cm-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
