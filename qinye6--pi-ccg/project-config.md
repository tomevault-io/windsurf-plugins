---
trigger: always_on
description: **Last Updated**: 2026-08-25 (v3.2.8)
---

# CCG for Pi CLI

**Last Updated**: 2026-08-25 (v3.2.8)

> 当前架构为 Pi-only。历史变更见 [CHANGELOG.md](./CHANGELOG.md)。

## 项目定位

CCG 为 Pi CLI 提供动态、有上限的智能开发工作流。Pi 是唯一 supervisor，不再由 Claude 主控，也不再通过 Codex/Gemini wrapper 充当固定外部角色。

标准流程：

```text
ccg-project-scout
→ ccg-planner
→ supervisor contract relay + START approval
→ Pi 动态 builder fanout（N frontend + M backend，按 component/profile/wave）
→ builder FINISH handoff
→ ccg-test-runner
→ ccg-reviewer
→ componentId 定向修复（最多 2 轮）
```

## 六个 Pi role templates

| Role template | 责任 |
|---|---|
| `ccg-project-scout` | 只读项目/组件发现 |
| `ccg-planner` | 组件拆分、文件边界、ownership、依赖、waves、测试计划 |
| `ccg-backend-builder` | 通用后端、服务、API、数据与基础设施组件 |
| `ccg-frontend-builder` | 通用前端组件；Web、管理后台、小程序/微信等通过 `componentProfile` 区分 |
| `ccg-test-runner` | 测试、typecheck、lint、build |
| `ccg-reviewer` | 独立正确性、质量、安全审查 |

`ccg-miniprogram-builder` 已退休，不属于当前 active runtime。小程序/微信只作为 frontend `componentProfile`，由通用 `ccg-frontend-builder` 实例处理。

模型映射：

```text
frontendModel → generic ccg-frontend-builder instances
backendModel  → generic ccg-backend-builder instances
reviewModel   → ccg-reviewer + ccg-test-runner
scout/planner → Pi subagents.defaultModel
```

## 动态 fanout 上限

```text
effectiveDevParallelism = min(
  devAgentCap,
  globalConcurrencyLimit,
  parallel.concurrency,
  parallel.maxTasks
)

requiredSpawns = 2 + (N_frontend + M_backend) + 1 + 1
```

默认：

```text
devAgentCap = 4
globalConcurrencyLimit = 4
maxSpawnsPerSession = 24
maxSubagentDepth = 1
```

所有 CCG agents 使用 `defaultContext: fresh`；supervisor 以 `context: "fresh"` 调用。跨 run 的 plan/build/test context 必须内联进 task string，不能依赖另一个 run 的相对 `reads`。

## Supervisor 协调规则

- `ccg-planner` 输出稳定 `componentId`、component/profile、文件 ownership、依赖 waves、测试计划和修复路由。
- Pi supervisor 在 builder 写入前 relay contract，并等待 supervisor `START` approval。
- Pi 按 wave 派生 builder 实例；同一 wave 的有效并发不得超过 caps。
- 每个 builder 只修改自己 ownership 范围内的文件；跨组件变更必须回报 supervisor。
- 每个 builder 完成时必须返回 `FINISH` handoff，包含 `componentId`、变更文件、验证、假设和风险。
- test-runner/reviewer 的失败必须携带 `componentId`，供 Pi 定向回派 owning builder。
- componentId 定向修复最多两轮。

## Pi slash commands 与 durable board

安装后的 Pi prompt command family：

```text
/ccg         主入口：确认需求后由 leader 自主 plan/build/test/review/repair
/ccg-board   只读看板
/ccg-replay  只读复盘
/ccg-resume  从 durable leader checkpoint 恢复，child 仍 fresh
/ccg-go      兼容入口
```

这些是 Pi prompt commands；Claude harness 的 `/ccg:go` 属于不同命名空间。fresh install 缺失命令时运行 `ccg init`，已有 metadata 但资产缺失时运行 `ccg update`，磁盘 prompts 更新后需要重启/重新加载 Pi 才会刷新 `/` 菜单。

leader 是唯一 agent 指派者、状态迁移者和 `.pi/ccg/` 写入者。任务投影位于 `.pi/ccg/tasks/<taskId>/board.json`、`events.jsonl`、`summary.md`，schema 为 `ccg.taskBoard.v1` / `ccg.taskEvent.v1`。看板是 `pi-subagents` lifecycle/FleetView 的有界脱敏投影，不是第二执行引擎；uninstall 默认保留复盘历史。builder FINISH 只交 leader，leader 再启动 fresh test-runner/reviewer；test/reviewer 不修改产品代码。


```text
ccg
ccg init | ccg i
ccg style <name>                 # 切换 leader 输出风格；default 恢复默认
ccg update [--install-dir <path>]
ccg extensions [--install-dir <path>]
ccg doctor [--install-dir <path>] [--project-dir <path>]   # 检查 Pi CLI、必需 runtime、agents、caps、模型、扩展与 MCP 配置存在性
ccg status [--install-dir <path>] [--project-dir <path>]
ccg uninstall
```

`init` 是十三阶段状态机；其中 thinking 阶段配置角色组推理强度，persona 阶段选择 leader 的输出风格：

```text
language → environment → extensions → scope → provider → frontend
→ backend → review → thinking → limits → persona → entry → summary
```

可选 persona 为 `default`、`engineer-professional`、`nekomata-engineer`、`laowang-engineer`、`ojousama-engineer`，以及四种 abyss 风格：`abyss-cultivator`、`abyss-concise`、`abyss-command`、`abyss-ritual`。persona 只影响 `/ccg` 与 `/ccg-go` leader 的 prose；child contract、JSON、测试、审查、board、凭据与协调协议不变。用户自管的 `SYSTEM.md` / `APPEND_SYSTEM.md` 不被修改。

交互式 `ccg init` 在 persona 阶段选择风格；非交互式安装使用 `--persona <name>`。`ccg style <name>` 可切换已安装风格，`ccg style default` 恢复默认风格。当前选择写入 CCG metadata，`ccg update` 保留该选择；style 操作不改变 package lifecycle。

thinking 阶段按四组配置 `planningThinking`、`frontendThinking`、`backendThinking`、`reviewThinking`，对应 scout/planner、frontend builder、backend builder、reviewer/test-runner。合法值为 `off | minimal | low | medium | high | xhigh | max`；undefined 表示继承，不写 override。非交互 flags 为 `--planning-thinking`、`--frontend-thinking`、`--backend-thinking`、`--review-thinking`。配置写入 `settings.json -> subagents.agentOverrides` 并由 metadata/update 保留；exact preset 做 capability 校验，unknown model 只由 doctor warning，不猜测 provider-specific wire schema。

## 入口与模块

| 路径 | 作用 |
|---|---|
| `bin/ccg.mjs` | npm executable |
| `src/cli.ts` | CAC CLI 入口 |
| `src/cli-setup.ts` | Pi-only command 注册 |
| `src/commands/init.ts` | 十三阶段安装向导、扩展/thinking/persona 选择 |
| `src/commands/extensions.ts` | 扩展 catalog 状态、确认与 package lifecycle |
| `src/commands/update.ts` | metadata 驱动更新；不执行第三方 package 操作 |
| `src/commands/doctor.ts` | doctor/status |
| `src/commands/menu.ts` | Pi-only 菜单 |
| `src/utils/installer.ts` | Pi install/uninstall |
| `src/utils/pi-extensions.ts` | 扩展 catalog、selection、ownership、package/config execution plan |
| `src/utils/pi-extension-config.ts` | `pi-web-access` JSON 三态检查与安全 create/merge |
| `src/utils/pi-provider-presets.ts` | exact model capability presets 与 provenance |
| `src/utils/pi-redaction.ts` | provider/config/diagnostic 集中脱敏 |
| `src/utils/pi-runtime.ts` | Pi package inventory 与安全 lifecycle 命令 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qinye6/pi-ccg](https://github.com/qinye6/pi-ccg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
