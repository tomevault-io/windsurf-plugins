---
trigger: always_on
description: **与用户沟通时一律使用简体中文**。包括但不限于：解释方案、复述需求、报告执行结果、追问澄清、insight 注释中的解释文字。代码、命令、文件名、英文专有名词保持原样不翻译。
---

# AGENTS.md

## 沟通语言

**与用户沟通时一律使用简体中文**。包括但不限于：解释方案、复述需求、报告执行结果、追问澄清、insight 注释中的解释文字。代码、命令、文件名、英文专有名词保持原样不翻译。

## 仓库性质

本目录是一个 **Claude Code / Codex Skill**(`api-test-E10`)，**项目级**，固定安装在 `test-automation` 仓库的 `.claude/skills/api-test-E10/` 路径下，随仓库一起分发。它的工作是驱动同一仓库内 `E10自动化/接口自动化测试/` 的接口测试编写。绝大多数"源代码"是 Markdown 规范，只有 `tools/`、`capture/`、`skill_utils/`、`hooks/` 目录里有 Python。

用户触发本 skill 时，期望你把 `SKILL.md` 当作执行手册严格遵循，而不是去重构它。本仓库内的"代码改动"主要形式是文档编辑。

## 文档层级（加载顺序很重要）

文档刻意拆分，并非每次激活都全部加载：

- `SKILL.md` —— **每次会话必加载**。保留前置门禁的**新增 / 维护分流读取要求**、方式分流、核心原则纲领。AI 行为的入口权威来源；详细执行手册按任务类型 Read `doc/preflight_gates_new.md` 或 `doc/preflight_gates_maintenance.md`，核心原则按需 Read `doc/core_principles.md`，方式细则按需 Read `doc/mode_*.md`。
- `README.md` —— 面向用户的快速指南。与 `SKILL.md` 部分内容重叠。修改策略时**先改 `SKILL.md`**，再把用户可见的部分同步到 `README.md`。
- `python tools/scan_page_api.py`：刷新 `tools/page_api_index.sqlite3` 接口索引。
- `doc/preflight_gates_new.md` —— **新增任务按需加载**，新增用例/接口方法的前置门禁详细执行手册（5 项必填、四选一菜单等）。
- `doc/preflight_gates_maintenance.md` —— **维护任务按需加载**，维护已有用例/接口方法的前置门禁详细执行手册（2 项必填、四选一菜单等）。
- `doc/core_principles.md` —— **按需加载**，核心原则 1-5 的详细规则（查重流程、索引维护、pytest 闭环细节等）。SKILL.md 触发对应原则时按需 Read。
- `doc/mode_capture_driven.md` / `doc/mode_reference_case.md` / `doc/mode_curl_manual.md` / `doc/mode_java_controller_source.md` —— 新增任务的四种接口自动化编写方案，选定方式后必须按需读取对应文件。
- `doc/maintenance_prompt_context.md` + `doc/mode_maintenance_*.md` —— 维护任务专用上下文与四种维护方案，其中方式4为 pytest 报错驱动维护。
- `doc/coding_style_guide.md` —— **按需加载**，仅在编写接口方法 / 用例代码前 Read。
- `doc/high_frequency_experience.md` —— 仅在踩到对应坑（Codex apply_patch、`show_list`、参数化与断言同步等）时加载。
- `flow_chart/flow.md` 与同目录 PNG —— Mermaid 源码与导出图。当前导出图覆盖前置 hook、主流程、前置门禁、新增总览、新增四方式、维护总览、维护四方式、pytest 闭环。`flow.md` 实质变化时才更新 PNG。
- 第三方依赖 Skill：维护方式4默认优先使用 `/test-fixing`；只有 `/test-fixing` 无法解决、维护遇到困难或前后接口/调用栈信息不明确时，才使用 `/Debugging` 断点调试辅助定位。

避免在多个文档间复制策略。这种拆分的目的就是让 `SKILL.md` 保持每次会话固定加载、其余按需拉取。

## 项目根定位架构（最不显眼的要点）

本 skill 物理位置固定为 `<project>/.claude/skills/api-test-E10/`。**项目根直接由 skill 自身位置推导**：`SKILL_ROOT/../../..`。统一封装在 `skill_utils/project_root.py`，被 `capture/capture_addon.py`、`tools/match_captures.py`、`tools/analyze_java_controller.py`、`tools/scan_page_api.py`、`tools/preflight_check.py` 复用。

校验：推导出的项目根下必须存在 `E10自动化` 子目录，否则报错并拒绝继续工作（防御 skill 被复制到错误位置）。

不再使用 `config.json.project_path` 字段，也不再有「CWD 向上 10 层搜索 `E10自动化`」的 fallback 逻辑。`config.json` 仅保留 `baseurl` / `apiDataUpdateDate` / `multi_capture` 等运行时配置。

剩余两条仍然成立：

- 运行时产物（抓包产生的 `latest.jsonl`、match 产生的 `capture_selection.md`、Java 源码分析产生的 `java_sourceCode_analysisResult.md`）写入**项目根**的 `api_test_dwp_temp/`，**绝不**写入 skill 自身目录。
- 与之相反，`tools/page_api_index.sqlite3` 与本 skill 全部源码一起**纳入版本管理**。

排查"工具找不到文件"问题时，先确认 skill 仍处在 `<project>/.claude/skills/api-test-E10/` 路径下，其上 3 层即项目根，并且项目根下确实有 `E10自动化` 子目录。

## 复用代码必须放在 skill_utils/（硬规则）

**任何被两个及以上模块复用的基础函数 / 常量 / 工具类，统一放在 `skill_utils/` 下，不得在调用方文件内复制粘贴**。这条规则覆盖：

- 跨模块共用的纯函数（如项目根定位、路径规范化、URL 解析等）
- 跨模块共用的配置常量（如 `REPO_MARKER = "E10自动化"`、`TEMP_DIR_NAME = "api_test_dwp_temp"`）
- 与运行时环境无关的解析 / 校验逻辑

当你发现某段逻辑要在第二处使用时：

1. 立刻把它抽进 `skill_utils/` 下合适的模块（按职责命名，如 `skill_utils/project_root.py`、`skill_utils/url_parse.py`），**不要在新位置复制一份**。
2. 调用方通过 `sys.path.insert(0, _SKILL_ROOT)` + `from skill_utils.xxx import yyy` 引用——`_SKILL_ROOT` 由 `os.path.abspath(__file__)` 向上推算两层，**不要依赖 CWD**。
3. 涉及日志输出的工具函数（如读 config 失败时的 warn），通过 **callback 注入**（`on_warn` / `on_info` 参数）解耦——`skill_utils` 本身不依赖 `mitmproxy.ctx.log` 或 `print(sys.stderr)`，由调用方各自传 lambda 适配。
4. 不在 `skill_utils` 内引入任何重量级第三方依赖；只用标准库 + 项目已直接依赖的库。

发现仓库内现有重复实现却没放进 `skill_utils` 的，按上述规则补救。

## 任何编写工作前的两道强制门禁

由 `SKILL.md` 强制执行，不得绕过：

1. **新增任务前置门禁**：按 `doc/preflight_gates_new.md` 执行，必须提供 `[接口方法文件]` / `[接口方法位置]` / `[接口用例文件]` / `[接口用例位置]` / `[用例名]` 5 项，并选择新增四种方式之一。
2. **维护任务前置门禁**：按 `doc/preflight_gates_maintenance.md` 执行，只强制提供 `[接口用例文件]` / `[接口用例位置]` 2 项，`[接口用例位置]` 必须能定位具体待维护的单个或多个用例，并选择维护四种方式之一。方式4按 pytest 报错驱动维护时，默认先用 `/test-fixing`，必要时再用 `/Debugging`。

纯查询 / 诊断 / 工具状态查询类任务可豁免；只要触及接口方法或用例代码即必须走门禁。

## 常用命令

以下命令默认在本 skill 目录（`.claude/skills/api-test-E10/`）下执行：

```bash
# 抓包服务（mitmproxy 监听 12138）—— 方式①使用
capture/start.bat                     # 启动（前台）
capture/stop.bat                      # 停止
capture/restart.bat                   # 停止 12138 后等待 1 秒并重启
python tools/check_capture_server.py  # 退出码 0=RUNNING、1=NOT_RUNNING、2=PORT_OCCUPIED

# 入口前置：接口数据时效检查（skill 触发后第一步必跑）
python tools/preflight_check.py

# 扫描 page_api URL（写入 tools/page_api_index.sqlite3）
python tools/scan_page_api.py         # 自动模式：库空全量重建（id 从 1 起）；库非空时按 Create Date 近 30 天 + (api_url, method) diff 增量追加
python tools/scan_page_api.py --full  # 强制全量重建（清空 + 重写，id 重新从 1）

# 用抓包 JSONL 与索引比对，生成勾选草稿
python tools/match_captures.py
python tools/match_captures.py --jsonl path/to/latest.jsonl

# 用 Java Controller 源码 / Jacoco 报告与索引比对，生成可编辑源码分析草稿
python tools/analyze_java_controller.py --source path/to/Controller.java
python tools/analyze_java_controller.py --source http://host/Controller.java.html#L76
```

`scan_page_api.py` 已覆盖三类 HTTP method 写法：`requests.<verb>(...)` / `requests.request("VERB", ...)` / `self.send_msg("get"|"post", ...)`。method 在入库前统一 `upper()`。

`preflight_check.py` 仅做日期比对：
- 当前时间与 `config.json.apiDataUpdateDate` 相差 ≤ 7 天 → 提示一周内最新
- > 7 天 → 自动调 `scan_page_api.py`，并写回 `apiDataUpdateDate=今天`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buer2233/ai-api-test-skill](https://github.com/buer2233/ai-api-test-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
