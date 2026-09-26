---
trigger: always_on
description: 用户说“创建一个 XXX agent / 智能体”时按下面链路执行，不要只拼一个 `agent create`。
---

# Agent

## 创建 Agent SOP

用户说“创建一个 XXX agent / 智能体”时按下面链路执行，不要只拼一个 `agent create`。

1. `arkcli auth status --format json`，确认登录、profile、project、API key。
2. 如果用户没有给精确模型，先完整查询候选：`arkcli agent model list --format json`。用户明确提出上下文/模态/能力要求时，才按[具体版本 metadata](#按具体版本筛选)补查并筛选；无此要求不额外逐模型查询。把返回的 `items[].model` 原样作为 `--model`。随后必须先完成 0/1/N 分支：0 个候选时报告无候选并停下；1 个候选时复述其 ID 后才进入步骤 3；多个候选时展示真实候选并**立即结束当前回合**，用户选定前禁止执行步骤 3 及后续步骤。
3. 模型已经唯一确定后，才把用户意图扩展成 skill 选择上下文。例：数据分析 -> `数据分析 Excel CSV 表格 BI SQL`；代码助手 -> `代码 编程 repo bash`；文档写作 -> `文档 写作 总结 Markdown`。
4. 创建 Agent 默认优先从本账号已有 custom skill 中选择，即使用户没有显式说“使用 custom skill”：按需执行 `arkcli agent skill list --source custom --limit 100 --format json`。由 AI agent 读取这一页全部 `Items`，按名称、描述、能力和版本判断；没有合适候选时，将响应中的 `NextPage` 原样传给 `--page` 继续拉下一页，直到命中或没有下一页。不要只调用 `search --source custom "<query>"` 后选第一条；用户明确要求完整清单或需要离线分析时才使用 `--page-all`。
5. custom skill 分页查完仍没有合适候选，或用户明确要求 market/SkillHub skill 时，再搜索：`arkcli agent skill search "<query>" --limit 10 --format json`。根据名称、描述、能力标签、版本选择 skill。不要臆造 `SkillId`；搜不到时可创建基础 agent，并说明未找到匹配 skill。
6. 组装参数：补领域化 system prompt，按服务端 `ModelMeta` 和用户需求选择 `--thinking`、`--reasoning-effort`、`--service-tier`；不要默认注入旧版 `speed=standard`，也不要把 `speed` 映射成新字段。线上测试资源名使用 `arkcli` 前缀；用户没给名字时生成 `arkcli-<domain>-agent-<YYYYMMDDHHMMSS>`，如 `arkcli-data-agent-20260707153000`。
7. 先执行同一条 `agent agent create ... --dry-run --format json`，读取零网络 `preview.v1`，向用户复述 `DisplayName`、`Model`、`Skills`、默认 `Tools`、`McpServers` 和 `unresolved`。裸 custom Skill 省略 `Version` 表示始终使用服务端最新版本，不需要在线补版本；`--skill-zip` 上传结果仍是实际执行时才能解析的占位符。用户确认后去掉 `--dry-run` 再真实创建。
8. 真实创建时，CLI 会先检查 Managed Agent 能力和模型开通状态：模型未开通会走共享模型开通确认链路；非交互环境不会自动开通，返回 `model_activation_required`。Managed Agent 产品/能力未开通时，TTY 下会提示用户确认并调用前端同款 `OpenChargeItems(ResourceType=DataManagedAgentSum, ResourceNames=[sandbox, web_search])`，非交互环境返回 `managed_agent_activation_required`，不会自动开通。
   - 如果已在对话中拿到用户明确确认，非交互调用可重跑原命令并同时加 `--yes` 和环境变量 `ARKCLI_ALLOW_HEADLESS_ACTIVATION=1`。不要在没有用户确认时设置该环境变量。
9. 真实创建后立刻 `agent agent get <agent-id> --format json` 确认落库。对用户回显时必须展示服务端最终配置，不要只展示“已创建”或单独摘要某个字段。
10. 用户要求端到端验证时，再创建 env/session，发送一条最小消息，拉 events/thread/resources。除非用户明确要求清理，不要删除创建出的资源。

只有模型已经由用户明确给出，或白名单过滤后唯一确定时，skill 与 MCP provider 候选才可以并行查。模型仍有多个候选时不得运行下面的 Skill/MCP 查询，也不得用 `agent agent create --help`、preview 或参数准备来提前推进创建流程：

```bash
arkcli agent model list --format json
arkcli agent model list --query "<capability-query>" --format json
arkcli agent skill search "<capability-query>" --limit 10 --format json
arkcli agent vault oauth-provider list --limit 100 --format json
```

## 模型选择

配置模型的 `speed/thinking/reasoning_effort/service_tier` 时，先按[模型参数 metadata](model-config.md)查询目标模型的可选值与默认值；不要把其他模型的选值当作通用枚举。

`agent agent create --model` 必须传精确可用的模型 ID。不要凭印象写裸模型名或展示名。

默认完整列出可用候选：

```bash
arkcli agent model list --format json
```

`--query` 模式仍以 ArkModels 白名单为主表，不从模型目录反向生成候选。它会调用 `models search` 拿详细信息并增强白名单模型：命中的白名单模型会带 `detail` 字段并排在前面；未命中的白名单模型仍保留，只是没有 `detail`。`detail` 字段包含用于判断适配度的信号，例如 `display_name`、`description`、`context_window`、`input_modalities`、`output_modalities`、`capabilities`、`lifecycle_status`。

需要额外详情和相关度排序时，可选用（不是默认步骤）：

```bash
arkcli agent model list --query "数据分析 Excel CSV SQL agent" --format json
```

选择规则：

- 默认只从 `agent_support=true` 的结果里选；`agent model list` 默认已经过滤非 Agent 模型。
- 主模型和工具模型查询都不默认加 `--primary-only`，也不在本地默认排除 `primary_version=false` 的候选；只有用户明确要求只看主版本时才启用。主版本不等于最新版本，同名多版本时不要跨条目混拼。
- 创建时传用户最终选中条目的 `model` 字段，不要传返回里的 `id`，也不要把列表第一项当成默认选择。
- 先只按用户明确给出的模型族、模态、上下文长度、能力等硬约束，以及所选用途的 `agent_support` / `tool_model_support` 过滤；`primary_version` 仅在用户明确要求主版本时作为过滤条件。查询排序、展示顺序、`router_baseline_support` 或 Agent 对“性能更强 / 更合适”的主观判断都不能把多个候选变成唯一候选。
- 过滤后只有 1 个候选时复述其 `model` 后继续；只要硬约束过滤后仍有多个候选，就必须使用宿主结构化选择能力展示本轮结果中实际存在的 `model`、`name/display_name`、`version`、`context_window`、`capabilities`、`lifecycle_status/status` 等区分字段，并停在模型选择阶段。候选输出就是当前回合的最终输出，之后不再调用任何工具。可以标注推荐项和依据，但用户选定前不得继续查询 Agent Skill/MCP，不得查看创建命令的 `--help`，也不得执行 `agent create/update`、`+new-agent`、`+iterate` 或它们的 `--dry-run`。不要从模型记忆补选项；宿主没有结构化选择能力时退化为精简编号列表。
- 用户明确要求某个模型族时，用 `--name <keyword>` 缩小 Agent 白名单范围：

```bash
arkcli agent model list --name doubao-seed-2-0-pro --format json
```

- 需要排障或确认为什么某模型不可选时，加 `--include-all`，查看 `agent_support=false` 的条目。
- 有明确硬指标时按[具体版本 metadata](#按具体版本筛选)筛选；list 不支持 `--size`、`--modality`、`--input-modality`、`--output-modality`、`--multimodal`、`--min-context-window`、`--capability`、`--strict-filter`。这不改变通用 `models search` 的参数。
- 用户已经给了完整模型 ID 时可以直接用，但如果创建失败提示模型不支持 / 不存在，回到 `agent model list` 重新选择。

### 按具体版本筛选

仅当用户明确提出如“上下文至少 200000 tokens”“支持图片输入”“支持 function calling”等要求时执行：

1. 先 `agent model list`（工具用 `--usage tool`）获取完整资格候选；保留每条的 `name`、`version`、`model`。名称约束可先用 `--name`，不默认只看主版本。
2. 若已有同一 `name` + `version` 的 metadata，直接复用。否则按需查询 `ListModelMetaDatas`。`models get <name> --version <version>` 可查产品详情；需要原始 metadata 时无专用产品命令，使用 [API Explorer](../../arkcli-api-explorer/SKILL.md)，先确认 registry 和本地 preview，再实际只读调用：

```bash
arkcli api model.list_model_meta_datas \
  --params '{"FoundationModelName":"<items[].name>","FoundationModelVersion":"<items[].version>","Keys":["context_window"],"PageNum":1,"PageSize":1000}' \
  --format json
```

占位符替换为候选的真实字段。只查用户要求的 keys，可在同一次请求批量传多个 key：

| 用户约束 | metadata key / 判定 |
| --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [volcengine/ark-cli](https://github.com/volcengine/ark-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
