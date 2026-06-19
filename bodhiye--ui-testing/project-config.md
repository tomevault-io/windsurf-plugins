---
trigger: always_on
description: 基于 Chrome DevTools MCP 的零脚本 UI 自动化测试 Skill：输入 URL 自动遍历页面并沉淀/复用 Playbook，执行 E2E 并输出标准化报告（MCP 优先，支持 CDP Proxy 降级）。
---


# ui-testing

## 作用

面向 Web 站点的 UI 自动化测试 Skill。基于 Chrome DevTools MCP 直连浏览器会话，无需编写测试脚本：接收单个待测 URL 后，按“先广后深”的方式遍历站点页面，梳理端到端测试用例并沉淀为站点专属 Playbook；在已有 Playbook 的情况下，直接复用用例执行测试，记录关键步骤、截图、报错统计，并输出标准化测试报告。

## Use when

当用户有以下需求时使用本 Skill：

* 测试某个 Web 站点或某个 path 下的 UI 功能；
* 为站点自动梳理端到端测试用例；
* 更新某个站点或 path 的 Playbook；
* 复用既有 Playbook 执行自动化测试；
* 查询 Playbook、报错明细、历史测试报告；
* 导出本次测试报告。

典型触发语：

* “用 ui-testing 测试 <https://example.com>”
* “测试这个站点并更新用例”
* “查询 example.com 的 Playbook”
* “查询本次测试的报错明细”
* “导出本次测试报告”

## Do not use when

以下场景不要使用本 Skill：

* 用户要测试的是原生 App、桌面应用、接口、数据库或非 Web 页面；
* 用户只需要静态代码审查，而不是浏览器中的 UI 行为验证；
* 用户要求执行恶意、破坏性、越权或非法操作；
* 用户要求收集、保存或泄露密码、Cookie、个人隐私等敏感信息；

## 本地浏览器连接方式

本 Skill **优先**通过 `chrome-devtools-mcp` 提供的 MCP 工具直接操作浏览器。

当 Agent 无法使用 MCP 工具（如 MCP 未接入、工具调用失败、或 Agent 不支持 MCP）时，允许**降级**到内置的 Node.js CDP Proxy（ `scripts/cdp-proxy.mjs` ）来完成同等的基础浏览器操作能力（navigate/eval/click/screenshot/snapshot 等）。

约束：

* 允许使用本仓库内置的 `cdp-proxy.mjs` 作为兜底代理（常驻服务形态）

### 前置条件

* Node.js >= 20.19（MCP 模式）
* Node.js >= 22（CDP Proxy 降级模式，使用内置 WebSocket）
* 本机已安装 Google Chrome
* （推荐）`chrome-devtools-mcp` 已作为 MCP Server 注册到 Agent 的 MCP 配置中

### MCP Server 配置（必须）

`chrome-devtools-mcp` 必须在 Agent（如 Trae、OpenClaw、Claude Code、Cursor 等）的 MCP 配置中注册，否则 Agent 无法调用浏览器操作工具。

在配置文件的 `mcpServers` 中添加：

```json
{
  "chrome-devtools": {
    "command": "npx",
    "args": ["-y", "chrome-devtools-mcp@latest", "--browser-url=http://127.0.0.1:${PORT}"]
  }
}
```

其中 `--browser-url` 的端口号需要与 Chrome 实际的 remote debugging 端口一致。

### 连接验证清单

执行测试前，按以下顺序验证：

01. **Chrome 进程**：确认 Chrome 已启动且带 `--remote-debugging-port` 参数
02. **端口连通**：使用实际端口（以启动脚本输出为准），例如：`curl -sS "http://127.0.0.1:${PORT}/json/version"` 返回版本信息
03. **MCP 工具可用**：调用 `list_pages` 能返回当前标签页列表
04. 若第 3 步失败，说明 MCP 配置未生效，需检查 Agent 的 MCP 配置并重启

### Chrome 启动方式

一条命令完成 Chrome 的启动：

```bash
node ./scripts/chrome-devtools-mcp.mjs
```

脚本会自动：
01. **优先复用**：扫描 `9222..PORT_SCAN_END` 范围内已运行的 Chrome DevTools 端口，若存在则直接复用
02. **否则新启动**：选择空闲端口并启动带远程调试端口的 Chrome
03. 等待 Chrome 就绪并自检

也可通过环境变量指定固定端口：

```bash
PORT=9222 node ./scripts/chrome-devtools-mcp.mjs
```

### 连接自检

```bash
# 以脚本输出的端口为准；若需固定端口可在启动时指定 PORT
curl -sS "http://127.0.0.1:${PORT}/json/version"
curl -sS "http://127.0.0.1:${PORT}/json/list"
```

### 操作方式要求

**优先**使用 MCP 工具直接操作浏览器，可用工具包括：

* `navigate_page` — 页面导航
* `take_snapshot` — 获取页面 a11y 树快照（优先于截图，用于定位元素）
* `take_screenshot` — 截图保存到指定路径
* `click` — 点击元素（通过 uid 定位）
* `fill` — 填充表单输入框
* `type_text` — 键盘输入文字
* `evaluate_script` — 执行 JavaScript（处理复杂交互）
* `handle_dialog` — 处理 alert/confirm 弹窗
* `wait_for` — 等待页面文本出现
* `list_pages` / `select_page` / `close_page` — 多标签页管理与清理

当 MCP 工具不可用时，使用 CDP Proxy 模式（见下文）作为降级兜底。

### 驱动选择与降级

执行任何测试前，先做“驱动探活”，选择 MCP 或 CDP 两种驱动之一。后续所有浏览器操作都必须走同一套“统一接口”，只切换底层驱动实现。

#### 1) Driver Preflight

01. **尝试 MCP**：调用 `list_pages`。
02. 若 MCP 成功：使用 **MCP Driver**（直接调用 MCP 工具）。
03. 若 MCP 失败：启动/复用 **CDP Proxy**，并通过 `GET /health` 或 `GET /list_pages` 探活；成功则使用 **CDP Driver**。
04. 驱动选择仅限 **MCP Driver** 或 **CDP Driver**；禁止切换到 `Playwright`、Puppeteer 或任何独立 testing browser 作为兜底。
05. 若 MCP 与 CDP 都不可用：终止执行，进入 Failure handling（记录原因与环境信息）。

#### 2) Unified Interface Mapping（MCP ↔ CDP Proxy）

以下“统一接口”在 Skill 内部语义保持一致；实现层根据驱动不同映射到 MCP 工具调用或 CDP Proxy HTTP 调用：

| 统一接口 | MCP Driver | CDP Driver（HTTP） |
| --- | --- | --- |
| 列表页 | `list_pages()` | `GET http://127.0.0.1:${PROXY_PORT}/list_pages` |
| 选页 | `select_page(pageId/targetId)` | `POST /select_page` |
| 新建页 | `new_page(url)` | `POST /new_page` |
| 关页 | `close_page(pageId)` | `POST /close_page` |
| 导航 | `navigate_page({type:'url', url})` | `POST /navigate_page` |
| 执行脚本 | `evaluate_script({function, args})` | `POST /evaluate_script` |
| 快照 | `take_snapshot({verbose})` | `POST /take_snapshot` |
| 点击 | `click({uid})` | `POST /click` （支持 `uid` 或 `selector` ） |
| 截图 | `take_screenshot({fullPage,filePath,uid?,selector?})` | `POST /take_screenshot` （支持 `uid` 或 `selector` 元素截图） |

#### 3) CDP Proxy Mode（兜底）

启动命令（示例）：

```bash
CHROME_PORT=${PORT} PROXY_PORT=8888 node ./scripts/cdp-proxy.mjs
```

说明：

* `CHROME_PORT`：Chrome remote debugging 端口（由 `chrome-devtools-mcp.mjs` 输出）
* `PROXY_PORT`：CDP Proxy HTTP 监听端口（默认 8888）
* CDP Proxy 需要 Node.js 22+（使用内置 WebSocket）

#### 4) 临时执行脚本产物

当 Agent 在 `CDP Driver` 模式下，因执行复杂流程需要临时生成辅助执行脚本（例如 `run_test.mjs` ）时，按以下规则处理：

* 临时脚本只能放在本轮运行目录 `{SKILL_ROOT}/playbooks/{domain}/{YYYYMMDDHHmm}/` 下
* 临时脚本默认**保留**，作为本轮测试产物的一部分，便于复现、排查和二次执行
* 若用户明确要求“清理临时文件”或“只保留最终产物”，才额外删除临时脚本
* 最终运行目录允许保留 `report.md`、`results.json`、截图文件（`*.png`）、临时执行脚本（如 `run_test.mjs` ）以及其他明确对用户有价值的日志或附件
* 若执行过程中生成了多个临时脚本，应优先复用并收敛，避免在单个运行目录内留下无意义的重复脚本
* 若用户要求清理但删除失败，应在最终结果中明确说明失败原因与残留路径

## Inputs

### 必填输入

* `url: string`
  + 待测试 URL。
  + 必须是完整的 `http://` 或 `https://` URL。

### 可选输入

* `updateCase: boolean = false`

  + 是否更新用例。
  + `true`：重新遍历页面、重建并覆盖对应 Playbook。
  + `false`：优先读取现有 Playbook；若不存在则自动创建。
* `command: string`

  + 辅助指令，可选值包括但不限于：
    - `queryPlaybook`
    - `exportReport`
    - `stopTest`
    - `queryErrors`
    - `editCase`
    - `deleteCase`
* `instruction: string`

  + 用户的自然语言补充要求，例如：
    - “只更新用例，不执行测试”
    - “只查询 Playbook”
    - “删除 example_com_001 用例”
    - “修改 example_com_002 的预期结果”

## Core principles

01. **先校验，后执行**：先做 URL 格式校验与可达性检测，不通过则终止。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bodhiye/ui-testing](https://github.com/bodhiye/ui-testing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
