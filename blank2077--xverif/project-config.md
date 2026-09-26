---
trigger: always_on
description: 本文件是本仓库的 agent 工作规则入口。所有 agent 先读本文件，再按任务读取更细的外部材料。
---

# AGENTS.md

本文件是本仓库的 agent 工作规则入口。所有 agent 先读本文件，再按任务读取更细的外部材料。

## 基本沟通

- 必须使用中文和用户沟通。
- 回答要直接、具体、证据驱动；不把猜测当事实。
- 当结论依赖仓库状态、schema、测试输出或环境行为时，先检查真实文件和命令结果。
- 用户明确要求实现时，直接完成实现、验证和交付说明；用户要求计划、评审或只读探索时，不越界修改。

## 执行前确认

- 除非用户明确要求直接执行，否则每次执行前至少给出三个可选方案，并交由用户选择。
- 计划阶段和实现阶段必须分清：计划阶段不修改 repo；实现阶段按已确认计划落地。
- 如果用户已经明确给出 `PLEASE IMPLEMENT THIS PLAN`、`开始实现`、`提交`、`推送` 等指令，可按该指令执行，不再重复询问同一决策。

## 权限与环境

- 所有 NPI、VCS 仿真、VIP、真实 license、真实 LSF、真实 EDA 工具动作，默认在沙箱外运行。
- 遇到进程通信、网络端口、文件系统、license、UDS/TCP/file transport、MCP stdio-loop 等问题，先判断是否为沙箱差异，再判断产品、SDK 或代码问题。
- 沙箱内失败不能直接当作产品回归；需要时做 sandbox-vs-host 对照，并在结果里说明执行位置。
- 不打印 access token、refresh token、cookie、完整唯一 ID 或其它敏感凭据。

## Fallback 规则

- 除非用户明确要求，不允许私自 fallback。
- 如果确实需要 fallback，必须先向用户说明原因、风险和替代路径，并等待确认。
- 不能因为某个环境动作失败就静默切换 transport、后端、数据源、测试层级或工具入口。

## Git 规则

- git commit 信息必须使用中文，并写清楚动机、范围和验证情况。
- 提交前必须运行 `git status --short`，确认只包含本次相关文件。
- 不使用 `git add .` 盲目打包；优先显式列文件，或在只提交已跟踪改动时使用 `git add -u`。
- 不回滚用户或其它进程产生的无关改动。
- 用户要求推送远端时，提交后推送当前目标分支，并回报 commit id 和推送结果。

## 项目概述

`xverif` 是面向芯片验证工作的工具集合，提供 debug、coverage、bit 计算、日志定位、协议/断言辅助和 agent/MCP 集成能力。

- `xdebug/`：统一的设计数据库、波形数据库和 combined debug 查询工具，提供 JSON action、schema、session、engine、log、transport 和测试体系。
- `xcov/`：coverage database 查询与报告工具，面向 VCS/Verdi coverage 数据。
- `xbit/`：确定性 bit、SystemVerilog literal、slice、mask 和表达式计算工具。
- `xentry/`：entry、descriptor、header、fragment 等结构化字段解析工具。
- `xloc/`：压缩日志位置 ID 与源码位置之间的还原、统计和标注工具。
- `xsva/`：SVA 解析、IR 生成和语义解释工具。
- `xverif_mcp/`：把 xverif 工具暴露给 MCP client 的 server、adapter 和测试。
- `skills/`：面向 Codex/Claude 等 agent 的工具使用说明、reference、脚本和可安装 skill。
- `doc/`：项目级报告、计划、架构说明和临时交付文档。

## 测试要求

- 一旦修改源码，在提交 git 前必须把关联测试全部跑通。
- 文档-only 修改可只做内容、链接、格式和引用检查；不需要运行源码测试。
- 测试命令必须来自当前仓库的 Makefile、README、pytest 配置或脚本，不凭旧记忆猜命令。
- 如果测试因 license、EDA 环境、真实数据、LSF 或沙箱限制无法运行，必须在最终说明和提交说明中写清楚阻塞原因。

常用入口：

- 全仓快速门禁：`pytest --xverif-gate fast`
- 全仓确定性回归（沙箱外）：`XVERIF_TEST_EXECUTION_ENV=host pytest --xverif-gate regression -n auto`
- 全仓 nightly（沙箱外）：`XVERIF_TEST_EXECUTION_ENV=host pytest --xverif-gate nightly -n auto`
- focused suite：在对应 gate 后追加 `--xverif-suite <catalog-id>`
- 显式准备数据库：`pytest --xverif-prepare <fixture-id>` 或 `all-generated`
- 全量 Fixture 校验：`pytest --xverif-fixture-validation --xverif-all-fixtures`
- 查看选择计划：`pytest --xverif-gate <gate> --xverif-plan`
- 前置依赖检查：`XVERIF_TEST_EXECUTION_ENV=host python tools/check_test_environment.py --gate <gate>`

补充约束：

- `--xverif-fixture-validation` 内部以 `rebuild=True` 调用 prepare，会**强制重建全部选中 fixture**；只想消费缓存时不要运行它。
- 只重建失效 fixture：先用 `--xverif-fixture-validation --xverif-changed <ref>` 或比对每个 fixture 的当前指纹与 `current.json` 预判，再对确认失效的 id 单独 `--xverif-prepare`。
- 同一工作树的正式 pytest gate/suite 默认串行启动。

Makefile 不再提供测试 target；裸 `pytest` 是 usage error。普通 regression/nightly 只消费缓存，cache miss 不自动仿真、不降级、不把 required 变成 SKIP。

## Skill 维护

- `skills/<name>/` 是 Codex/Claude skill 的唯一 source of truth；安装目录不是编辑源。
- 修改 CLI、MCP tool、action/schema、session 生命周期、输出合同、SDK-free wrapper 或测试入口时，必须同步检查对应 skill 的 `SKILL.md`、references 和 `agents/openai.yaml`。
- 公共参数不允许接受后静默忽略；实现不支持的参数必须从公开 schema 删除或返回明确错误。
- skill 修改必须通过对应 `skills.*` catalog suite，至少检查 Markdown 链接、可复制 JSON 示例、action/tool 覆盖和附带脚本。
- repo skill 提交并通过测试后，使用 Makefile 安装目标同步到 `~/.codex/skills` 与 `~/.claude/skills`，并逐 skill 执行 `diff -qr` 验收。
- SDK-free UDS readiness 以 server 成功进入 `listen()` 为准；禁止用 socket 文件存在、固定 sleep 或静默 connect 重试替代 ready 合同。
- 仅修改 skill 文档时不要求真实 NPI、编译或仿真；涉及真实 NPI/FSDB/VDB 的 skill 验证仍按本文件权限规则在沙箱外执行。

## Schema 维护

- `xdebug/specs/actions/actions.yaml` 是 action 名称、状态、handler、required args、required target、schema 路径和 example 路径的目录级 source of truth；修改公共 action 合同时必须先核对这里，不能只改 handler 或单个 JSON schema。
- runtime request 的允许参数集合、共享语义说明和 action-specific 补充参数维护在 `xdebug/tools/sync_runtime_request_schemas.py` 与 `xdebug/specs/action_contracts.py`。同名参数不得靠另一个 action 的既有 schema 推断业务语义；新增、删除或改名参数时必须同步 handler、`actions.yaml`、该生成脚本、checked-in schema 和 request example，禁止只手改生成后的 schema。
- 跨 action 的复用业务对象必须在共享合同组件中定义，再由生成器投影到各 action；例如 reset 一律为 `{"signal":"<one-bit waveform path>","polarity":"active_low|active_high"}`。不得重新引入 `rst_n`、裸 string reset、表达式 reset 或默认极性；外部 config 文件、持久化配置、runtime response 和 request schema 必须使用同一对象。
- 所有公开 action（包括 AXI）的 response schema 统一由 `xdebug/tools/sync_response_schemas.py` 生成；AXI `summary/data`、transaction、config、finding 等业务对象维护在 `xdebug/specs/non_sampling_response_contracts.py`，再由统一生成器投影并关闭未知字段，禁止直接手改 checked-in AXI response schema。
- schema 的 AI-facing purpose、使用场景和参数说明由 `skills/xverif/references/xdebug/action-reference.md`、`actions.yaml` 和 `xdebug/tools/sync_action_schema_hints.py` 同步；需要修改提示时先改 source，不在生成 schema 中单独维护漂移副本。
- 所有公开 request 顶层和 `args` 默认使用 `additionalProperties: false`；`query`、`output`、`time_range`、`match` 等嵌套对象也必须显式列出属性并关闭未知字段，除非合同明确要求可扩展对象。
- handler 接受的每个公共参数都必须出现在 action-specific schema 中并实际生效；schema 中公开但实现不支持的参数必须删除或返回明确错误，禁止接受后静默忽略。参数名、enum、默认值、required/conditional-required 语义必须在 native CLI、MCP、schema、example 和 skill 中一致。
- request/response schema 与 `examples/requests`、`examples/responses` 必须成对维护。response 不得在 `summary` 和 `data` 重复同一事实；时间只发布一个 canonical 带单位字符串，截断必须区分完整分析计数与返回行数，并提供 `truncated`、`truncation_scope` 或对应完整性字段。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BLANK2077/xverif](https://github.com/BLANK2077/xverif) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
