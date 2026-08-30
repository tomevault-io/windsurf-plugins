---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 常用命令

用户通过 `npm install -g omnibot` 安装后，直接使用 `omnibot` 命令：

```bash
# 检查扩展连接状态（CLI 会自动启动 daemon）
omnibot doctor
omnibot status
omnibot tabs

# 安装 skills
omnibot skills install --agent hermes --profile nuwa
omnibot skills install --agent opencode
omnibot skills install --agent claude
omnibot skills install --agent codex

# 浏览器操作
omnibot snapshot -i
omnibot navigate 'https://example.com'
omnibot execute-js "return document.title"
omnibot screenshot -o /tmp/screenshot.png
```

## 架构

omnibot v2 让 LLM 客户端连接**用户正在使用的真实浏览器**，对外能力主要通过本地 daemon 和 CLI 子命令暴露。

**启动链路** → 控制台脚本 `omnibot:main` → `cli.main()` 启动 CLI 命令解析；当执行需要浏览器能力的命令时，CLI 会自动尝试启动或连接本地 daemon。

**`cli.py`** — CLI 命令树：`daemon`（及顶层别名 `start/stop/status/run`）、`tabs`、`snapshot`、`read`、`execute-js`、`batch`、`wait`、`navigate`、`screenshot`、`skills`、`doctor`。

**`daemon.py`** — 本地 daemon 进程，负责保持 `TMWebDriver` 长连接、扩展 WebSocket 状态和浏览器动作调度。

**`daemon_client.py`** — CLI 到 daemon 的客户端层，负责健康检查、自动启动 daemon 和发起 action 请求。

**`actions.py`** — 可复用浏览器动作层，包含核心逻辑：`snapshot`、`read`、`execute_js`、`batch`、`wait`、`navigate`、`screenshot` 等。

**`TMWebDriver.py`** — 核心桥接层。管理通过 WebSocket（`127.0.0.1:18765`）或 HTTP 长轮询（`:18766`）连接的浏览器会话。每个 `Session` 跟踪一个标签页（url、连接类型、活跃状态）。三种连接模式：
- `ws` — 扩展直连 WebSocket
- `ext_ws` — 扩展 WebSocket，带标签页级别的追踪（`tabs_update` 广播）
- `http` — HTTP 长轮询降级方案

也可在 `is_remote` 模式下运行，通过 HTTP 代理到另一 TMWebDriver 实例。

**`simphtml.py`** — 浏览器内 HTML 优化。包含大型嵌入式 JS 代码块（`optHTML()` — 去除脚本/样式/不可见元素并对页面区域分类；`findMainList()` — 检测重复内容列表；`smart_truncate()` — 基于 token 预算的 HTML 截断）。Python 侧用 BeautifulSoup 做后处理（属性清理、列表截断、`execute_js_rich()` 的 DOM 变更 diff）。

**`browser-extension/`** — Chromium 扩展：`background.js`（WebSocket 连接 + 约每 5 秒自动重连）、`content.js`（页面内脚本执行）、`popup.html/js`（扩展工具栏界面）。

**`src/omnibot/skills/`** — omnibot v2 skills 目录，指导 agent 如何使用 CLI 命令读取和操作页面。

**`sop/`** — 扩展 CDP 能力和 Vue3 组件处理的标准操作文档。

## 关键约定

- **日志必须走 stderr**：用 `log()`（在 `TMWebDriver.py` 和 `simphtml.py` 中已定义）输出所有诊断信息。CLI 的结构化结果通过 stdout 输出，诊断信息不要混入 stdout。
- 会话 ID 就是 Chrome 标签页 ID（字符串类型）。每个页面状态命令必须通过 `--tab-id` 显式指定目标标签页，不再有默认目标。
- `execute_js()` 有约 15 秒超时，并跟踪 ACK 确认——能区分"脚本未送达"和"脚本已送达但尚无结果"。
- v2 首版本地优先：CLI 默认通过 `127.0.0.1:18764` 访问 daemon，通过 `127.0.0.1:18765` 连接浏览器扩展。

## 源码验证 daemon 注意事项

- 验证本地源码改动时，不能依赖全局 npm 安装的 packaged daemon。`uv run omnibot ...` 在某些情况下会启动 `/opt/homebrew/lib/node_modules/@omniaibot/omnibot/.../omnibot-macos-arm64`，导致源码新增 action（例如 `read`）返回 `Unknown action`。
- 测试源码内 CLI/daemon 行为前，先停止现有 daemon：
  ```bash
  uv run omnibot stop
  ```
- 然后显式用源码启动 daemon：
  ```bash
  uv run python -m omnibot --api-port 18764 --ws-port 18765 daemon run
  ```
- 另开终端执行验证命令，例如：
  ```bash
  uv run omnibot doctor
  uv run omnibot read --screens 3 https://x.com/home
  ```
- 若 `doctor` 显示 daemon 正常但 action 仍是旧行为，检查 `ps` 中监听 18764 的进程路径，确认不是 packaged binary。

## 测试

### 发版前测试

标准发版 gate：

```bash
python3 tests/release/preflight.py
```

该 gate 执行：

```bash
uv run python -m pytest tests/unit -q
python3 tests/e2e/feature_matrix_test.py --no-playwright
python3 tests/e2e/full_workflow_test.py --no-playwright
```

其中 `tests/e2e/feature_matrix_test.py` 按 Omnibot 子功能拆分测试；`snapshot` 暂不作为单独 case，只作为其他 case 的观察和 ref 获取前置能力。

### 点击功能测试

多维度持久性点击功能测试，验证 omnibot 在不同场景下的点击稳定性。

**测试维度：**
- 基础点击：按钮、链接、输入框
- 动态内容：热搜"换一换"等动态加载内容
- 多标签页：新标签页打开验证
- 边界条件：快速连续点击
- 持久性：重复迭代测试

**运行测试：**

点击功能覆盖已合并到 feature matrix（见上方"发版前测试"）。如需单独回归点击维度：

```bash
python3 tests/e2e/feature_matrix_test.py --case snapshot_ref_click --case dom_cua_click --no-playwright
```

**测试报告：**
- JSON 报告：`tests/reports/comprehensive_test_report_<timestamp>.json`
- 文本报告：`tests/reports/comprehensive_test_report_<timestamp>.txt`

### snapshot + click 闭环测试

v2 核心交互路径：`snapshot` 拿 a11y tree + `@eN` 引用 → `click @eN` 触发操作 →
`snapshot` 验证状态变化。验证 `omnibot snapshot` / `omnibot click` /
`omnibot dblclick` 在真实浏览器 + 扩展下的端到端可用性。

**测试维度：**
- Snapshot 输出：@eN 引用唯一性、`-i`/`-c`/`-d`/`-s`/`-u` flag
- click @eN：button / link / input
- click 选择器回退：CSS / `text=` / `xpath=`
- Round-trip：dynamic refresh / navigation / modal
- Ref 失效与跨 tab 隔离
- `dblclick` + `--new-tab`
- 持久性 / 延迟 / 失败路径
- 弹窗控件：snapshot 自动追加 DOM Popup Controls，并可用生成的 @e 引用点击取消/关闭按钮

**运行测试：**

```bash
# 运行 snapshot + click 闭环测试（需要浏览器扩展连接）
python3 tests/e2e/feature_matrix_test.py --case snapshot_ref_click --no-playwright
```

**测试报告：**
- JSON 报告：`tests/reports/comprehensive_snapshot_click_test_report_<timestamp>.json`
- 文本报告：`tests/reports/comprehensive_snapshot_click_test_report_<timestamp>.txt`

### Upload 单元测试

Upload 命令的单元测试，覆盖 CDP `DOM.setFileInputFiles`、`DOM.querySelector` nodeId 回退、以及 JS `DataTransfer` fallback 三条上传路径，无需浏览器扩展连接。

**测试用例：**
- `test_upload_action_request_maps_to_upload_action` — CLI 参数到 action 映射
- `test_upload_action_sets_file_input_files_with_cdp` — CDP Runtime.evaluate objectId 路径
- `test_upload_action_falls_back_to_dom_node_id` — CDP DOM.querySelector nodeId 回退路径
- `test_upload_action_falls_back_to_js_file_assignment` — JS DataTransfer fallback 路径

**运行测试：**

```bash
uv run python -m pytest tests/unit/test_cli_contract.py -k upload -q
```

### 弹窗控件端到端测试

弹窗/模态框 DOM Popup Controls 扫描 + 点击闭环测试，使用本地 HTML fixture 页面验证 5 种弹窗类型的控件检测和 @eN 引用点击。

**测试用例：**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denni281/omnibot](https://github.com/denni281/omnibot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
