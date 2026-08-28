---
trigger: always_on
description: Alear030 — 从零自研的 Python Agent Harness。核心思想：**把模型之外的编排、状态与记忆全部自己实现**。处理工具编排、多 Agent 路由、会话生命周期、事件驱动 Hook、跨会话记忆召回。
---

# CLAUDE.md

## 项目概述

Alear030 — 从零自研的 Python Agent Harness。核心思想：**把模型之外的编排、状态与记忆全部自己实现**。处理工具编排、多 Agent 路由、会话生命周期、事件驱动 Hook、跨会话记忆召回。

**仓库已公开（MIT）。** 代码、注释、文档、commit message、`.claude/skills/` 与 `.cursor/rules/` 都会被陌生人读到——落笔前按「这会被公开」判断，不留本机绝对路径、内部语境或协作过程叙述。

## 运行与验证

所有命令从仓库根目录执行：

```bash
python main.py    # TUI 事件循环；Ctrl+C 由 TUI 收口退出手势，main.py finally 做进程收尾且收尾期间忽略 SIGINT
```

依赖根目录 `.env` 中的三级模型配置（`max_level` / `medium_level` / `low_level`），由 `config.py` 读取。当前没有锁文件，不能把 `pip install -e .` 等命令当成可复现的完整安装方案。

往 bash 里嵌 `python -c` 或 heredoc 时，未转义的反引号会被命令替换吃掉（本项目文案里反引号标识符极密，已踩两次）。含反引号或正则转义的内容一律先落成脚本再执行，不要拼进 bash 命令行。

验证改动是否可用时用 `alear030-verify` 技能——这个项目的验证方式有几个反直觉的坑（`python main.py` 有真实副作用、验证脚本必须用 `python -m` 点号路径调用、`unittest discover` 不能带 `-s test`），别凭经验直接套用通用 Python 项目的验证套路。

## 协作方式

**当前阶段：架构稳定期**。后续改动默认跨模块、机制级，一律走**探索 → 规划 → 执行 → 验收**；「可直接做」仅限无行为影响的纯文本改动。

本项目不走“用户给需求 → Claude 闷头执行”的单向模式。默认节奏是**探索 → 规划 → 执行 → 验收**；仅**无行为影响的纯文本改动**（改错字、加日志、用户明确指定的重命名、注释）可直接做——触碰行为、机制、参数、调用链的改动，即便一句话说得清，也先过规划闸门并说明权威路径与影响面。

1. **探索** — 先读相关代码/资料搞清现状，不改任何东西；产出“现状是 X”
2. **规划** — 提方案并主动点出取舍与风险；较大特性可用 `AskUserQuestion` 反向采访；产出用户能修改、能拍板的计划
3. **执行** — 用户拍板后才动手
4. **验收** — 提供测试输出或跑通结果，不靠“做好了”这句断言

**规模增长协作约定**（2026-08-08 起）：项目体量增大后，在「探索→规划→执行→验收」之上叠加四条节奏约定：

- **切片化工作单元**：跨模块/大特性按「可独立提交、可运行」的切片规划，一次会话一个闭环，不留带已知缺陷的 WIP 半程提交；计划按切片呈现，用户拍板切片边界后开工
- **验证优先**：机制级改动先明确「怎么快速验」；测试/探针尽可能固化进 `test/` 而非随用随删；交付以验证输出为准
- **知识交接**：会话收尾更新「当前主线 + 未完成项」交接物；协作经验与用户偏好落 memory；`@claude` 标记经 scan-claude-markers 定期扫描
- **并行编排**：跨模块改动默认走 workflow 并行探索（Explore 子代理）与并行审查；闸门三级——纯文本直接做 / 单模块机制自验 / 跨模块完整流程

**分工边界**：用户拥有方向、品味、北极星判断和“什么算够好”的最终拍板；Claude 负责读代码、查资料、铺上下文、起草方案、实现、验证并主动指出选项和风险。提出方向、上浮机制根因、指出跨模块影响是 Claude 的职责——**提出不算越界，拍板才算**；任务字面不足以达成目标时，显式提出超范围项交用户拍板。用户在监督位，Claude 在执行位。

**反馈文化**：

- 用户随时可以打断纠偏，越早越好
- 同一问题纠正超过两次 → 停下重开，换根本思路或把已知信息重写成更清楚的需求，不继续叠补丁
- 模糊提问是合法且鼓励的；开放问题用于共创，不要求用户每次先给出成型需求

**机制改动的文档对账**：对机制、触发点、数据流或 `config.py` 常量做改动时，进 CHANGELOG 前顺带核对两处，漂移则同次修正——CLAUDE.md 的「稳定模块地图」「核心运行数据流」「架构核心」相关段，以及 `docs/` 下的三份对外文档（`ARCHITECTURE.md` / `CONFIGURATION.md` / `EXTENDING.md`）。`docs/` 是对外架构事实的权威源，CLAUDE.md 只承载协作时必须立刻知道的约束，两者受众不同、不互为副本；最终架构事实一律以代码为准。

**代码里的 `@claude` 任务标记**：

- 仓库没有提供自动扫描机制,`scan-claude-markers` 技能是现成的可复现入口;想要会话开始自动扫,可另行配置 SessionStart hook,是否随仓库分发取决于配置放在哪里
- 完成标记后将原行改写成 `# done(@claude): <做了什么>`，保留痕迹且避免下次重复扫描
- `# @claude(ignore) ...` 是用户自己的备注，不是 Claude 任务，不要修改

**协作经验落盘**：协作中形成的长期工作方式写入用户 memory（feedback 类），不要让它随会话消失。

**Claude Code 执行子代理**：仅在当前会话实际提供 `alear-executor` 类型时可用。它用 Sonnet 承接上下文可自包含、会产生大量一次性噪音的执行任务。节奏是 **Opus 主对话规划与拍板 → 主动推荐派发 → 用户拍板 → Sonnet 执行**；不得自动委派。派发指令必须自包含，它不适合需要边写边理解模块耦合的模糊实现。

项目中有三套不同的 Agent 概念，不能混用：

- `agent/agents.yaml`：项目进程内的 5 个常驻配置 Agent（main/slice/summary/plan/memory）
- `subagent_create`：项目运行时按任务临时构造的 Subagent，随机唯一名 `subagent_{uuid8}`，并注册进 agents 容器供按名路由
- `alear-executor`：Claude Code 层面的执行子代理；仅当前会话提供时可用

## 稳定模块地图

> 以下描述反映当前实现；机制改动时需同步，漂移以代码为准（见"协作方式 > 机制改动的文档对账"）。

| 路径 | 职责 |
|------|------|
| `main.py` | 进程入口、高层对象装配与退出收尾（交互循环已移至 `tui/`） |
| `config.py` | 模型级别、数据路径及运行常量 |
| `loop/` | ReAct `Loop` 与独立的 plan 编排器 `PlanRunner` |
| `agent/` | YAML 常驻 Agent 定义、实例化与工具授权 |
| `prompt/` | Prompt 分块注册、组合与启动时上下文快照 |
| `tool/` | 工具注册、schema、授权和具体工具实现 |
| `hook/` | Hook 注册、发现及同步/后台调度 |
| `session/` | 当前 session 的消息、切片、摘要、压缩和 plan 状态 |
| `memory/` | 跨 session 的切片分类、去重、用户画像与持久化 |
| `local_model/` | embedding worker 进程（client/worker/`_EmbeddingProxy` 门面）+ 已跟踪模型元数据 |
| `mcp_client/` | MCP 客户端：`mcp.json` 配置、asyncio 隔离 supervisor、远端工具接入工具体系（详见"MCP 客户端"） |
| `skill/` | 项目运行时技能资源 |
| `tui/` | TUI 界面：channel 路由、widget 注册体系、接收 loop 流式事件（详见"TUI 架构"） |

给某个模块补对外机制文档时放 `docs/modules/<module>.md`（文件名对齐本表路径），设计取舍类叙事放 `docs/design/<topic>.md`，两者分层逻辑与已有条目见 [docs/index.md](docs/index.md)。

`workspace/`、`z_ccstudy/`、`z_old_code/` 不参与主项目分析。

`.cc_file/`（已 `.gitignore`）是非项目内容的存放位置（笔记、复盘、外部材料等），不参与项目代码分析，不受"外科手术式改动"等代码规范约束。

## TUI 架构

`tui/tui_core.py` 是 TUI 入口（当前主线）。

**事件流**：loop 边跑边发流式事件 → TUI 按 agent_name 找 channel → `append_stream` → `build_widget` 渲染。

```text
loop.emit (Alear030TUI.__init__ 里 self.loop.emit = self.receive_loop_emit；main.py 构造 TUI 时传入 loop)
  → tui 按 agent_name 找 TuiChannel（subagent 已写入 agents 供按名路由铺路；本版 channel 仅登记 main）
  → call_from_thread 送回 UI 线程 → channel.append_stream → tuiwidgets.build_widget 渲染
```

- 用户可见 harness 错误走 `Loop.emit(event='SystemError', content={'message':...})`；工具错误走 `_error_result` / `tcr` extra_info，不另开通知总线
- `tui/tui_widget/` 是 widget 注册体系：`@widget_register` 注册类，`tuiwidgets.build_widget(type, content)` 按类型构造；消息类 widget 首段读 `widget_content`，`update_widget` 整段替换（累积在 loop 侧，不全在 widget）
- Textual 8 自定义 Widget 不写 `height` 默认填满父容器；消息/条目类 widget 的 css 第一条写 `height: auto`（内置 Static/Markdown 的 DEFAULT_CSS 自带 auto）
- `App.__init__` 的 `css_path` 是关键字参数（第 1 位置参数是 driver_class）；`set_focus` 传 widget 对象，不传 CSS 选择器字符串
- `call_from_thread(func, *args)` 传函数引用；带括号 = 当前线程立即执行，不会送回 UI 线程。widget 更新必须在 UI 线程：worker 线程迭代流式 → emit → `call_from_thread` 送回 UI 再 mount/update
- `tool_call_extra_info` 的 css 走 `ExtraInfoHandler._widget_css_handler`：写 `margin-*`/`padding-*` 子属性会被聚合为 `margin`/`padding` 元组（Textual 内联只认复合属性，`setattr` 对连字符属性名不生效）；单词属性（color/width/height）直接生效

## 核心运行数据流

> 以下描述反映当前实现；机制改动时需同步，漂移以代码为准（见"协作方式 > 机制改动的文档对账"）。

### 启动、顶层轮次与退出


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Alear030/Alear030](https://github.com/Alear030/Alear030) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
