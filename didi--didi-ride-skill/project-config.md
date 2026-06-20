---
trigger: always_on
description: 中国城市出行服务。当用户表达任何交通出行需求时必须使用此技能——包括打车/叫车/网约车、查价格、路线规划（公交/驾车/步行/骑行）、周边搜索、查询订单/司机位置/取消订单。关键词："打车"、"叫车"、"去[地点]"、"回家"、"上班"、"下班"、"查价格"、"多少钱"、"路线"、"怎么走"、"步行到"、"附近"、"周边"、"司机"、"订单"、"查询订单"。注意：即使用户未明确说"打车"，只要涉及从A地到B地、通勤、或交通方式选择，都应触发。不触发场景：开发打车应用、使用其他导航app、订外卖、查公交时刻表、股票/财报查询。
---


# 滴滴出行服务 (DiDi Ride Skill)

通过 DiDi MCP Server API 提供打车、查询订单、司机位置、预约叫车、路线规划、周边搜索能力。

---

## 1. 快速开始（2 分钟）

### 1.1 获取 MCP KEY

**方式一：用「滴滴出行App」扫码（推荐，最快）**

![滴滴出行APP扫码获取MCP Key，解锁一键打车](https://s3-yspu-cdn.didistatic.com/mcp-web/qrcode/didi_ride_skill_qrcode.png)
> ⚠️ **Agent 注意**：用户客户端无法渲染 Markdown 图片，**禁止直接输出上方图片语法**。需向用户发送二维码时，执行 `### 3.9 MCP KEY 与配置` 中的 `openclaw message send` 命令发图。

打开滴滴出行 App，扫描二维码，即可快速获取 MCP Key。

**方式二：访问官网**

访问 https://mcp.didichuxing.com/claw 获取您的 MCP Key。

### 1.2 配置 Key

**方式一：对话中输入（推荐）**

直接在对话中告诉我您的 MCP Key，我会帮您配置：

```
你: 我的 MCP Key 是 xxxxxx
```

**方式二：OpenClaw 配置文件**

编辑 `~/.openclaw/openclaw.json`，添加：

```json
{
  "skills": {
    "entries": {
      "didi-ride-skill": {
        "enabled": true,
        "apiKey": "你的MCP_KEY"  // apiKey 是 OpenClaw 标准字段名，存储的值就是滴滴平台的 MCP KEY
      }
    }
  }
}
```

### 1.3 开始使用

配置完成后，直接对话即可：

```
你: 打车去北京西站
你: 帮我查一下从国贸到三里屯的路线
你: 查询订单
```

首次使用时，OpenClaw 会提示安装 mcporter 工具。

---

## 2. 用户指南

本 Skill 支持以下操作：

- **打车**：直接说"打车去[地点]"、"回家"、"上班"
- **查价**：查一下从 A 到 B 多少钱
- **查询订单**：输入「查询订单」了解当前订单状态（司机位置、行程进度等）
- **司机位置**：司机在哪里、多久到
- **预约出行**："15分钟后打个车"、"明天9点去机场"
- **路线规划**：驾车/公交/步行/骑行路线
- **取消订单**：取消当前订单

---

## 3. Agent 执行指令

以下内容为 AI 执行参考，用户可忽略。

### 3.1 文件地图 

按需读取以下文件，不要猜测未读过的内容：

| 文件 | 用途 | 何时读取 |
|------|------|----------|
| `SKILL.md` | 触发、主流程、硬性门禁、查询订单规则、预约出行规则 | 每次触发必读 |
| `references/workflow.md` | 分阶段详细流程与命令范式 | 需要实现细节时读 |
| `references/api_references.md` | MCP 函数签名与参数定义 | 每次调用工具前**必须**核对 |
| `references/error_handling.md` | create_order 失败提示、mcporter 常见错误、统一错误码、参数错误排查、apiKey 占位符泄漏 | ⚠️ 遇到任何调用失败（HTTP error / StatusCode=400 / `-32xxx` 错误码 / `Unknown MCP server` / `Missing KEY parameter` / `SSE error: Invalid content type`）必须读取此文件 |
| `references/setup.md` | 安装 mcporter、配置 MCP KEY 的完整步骤 | 用户询问安装/配置问题时读 |
| `assets/PREFERENCE.md` | 地址别名/车型/手机号偏好 | 用户提到别名地址（家、公司、妈妈家等）、车型、手机号，或未明确给出起终点时**必须**读取。别名匹配规则见执行前检查第 7 条 |

### 3.2 执行前检查

1. **检查 mcporter**：若 `mcporter` 不存在（`command not found`），停止并引导用户阅读 `references/setup.md`。没有 mcporter 就无法调用任何 MCP 工具，后续任何流程都无法执行。

2. **检查 Key**：执行 `openclaw config get skills.entries.didi-ride-skill.apiKey`，若输出为空或非 `__OPENCLAW_REDACTED__`，按 `### 3.9 MCP KEY 与配置` 流程引导。Key 缺失时 mcporter 的报错信息具有误导性，不要尝试绕过。
   - ⚠️ **若 Key 已配置（返回 `__OPENCLAW_REDACTED__`）但 mcporter 仍报 `Missing KEY parameter`**：**不是 Key 失效**，**禁止向用户索要 Key**。排查步骤见 `references/error_handling.md` 中的「mcporter Missing KEY parameter」章节。
   - ⚠️ **`__OPENCLAW_REDACTED__` 是"已配置"哨兵值、不是真实 Key**：禁止从 `openclaw config get`（含 `--raw`）提取 Key 字面量；URL 中 `?key=...` 必须固定用 `$DIDI_MCP_KEY`。误把哨兵拼进 URL 会触发 `SSE error: Invalid content type`，详见 error_handling.md 同名章节。

3. **mcporter.json 注意事项**：本 skill 使用 URL 直连模式，**不依赖 `config/mcporter.json`**，**禁止创建或修改该文件**。如果 mcporter 启动时报 JSON 校验错误（`invalid_type` / `Failed to parse JSON`），参见 `references/error_handling.md` 中的「mcporter.json 校验错误」章节。

4. **mcporter 调用格式**（固定写法，不要变形）：

```bash
MCP_URL="https://mcp.didichuxing.com/mcp-servers?key=$DIDI_MCP_KEY"
mcporter call "$MCP_URL" <tool> --args '{"key":"value"}'
```

**必读注意事项**：
- `MCP_URL` 赋值和 `"$MCP_URL"` 引用**都必须用英文双引号**，否则 `$DIDI_MCP_KEY` 不会被 shell 展开。禁止用单引号或中文引号。
- **禁止添加 `--server` 标志**（如 `--server didi-mcp`）。`--server` 会让 mcporter 去查找已注册的命名 server，找不到直接报 `Unknown MCP server`；即使找到了也会和 URL 参数冲突导致 `Missing tool name`。
- **参数名必须核对 `references/api_references.md`**，不要凭记忆。常见致命错误：`keyword` → 应为 `keywords`；`region` → 应为 `city`；`from_lng/from_lat/to_lng/to_lat` → 应为 `from_name/from_lat/from_lng/to_name/to_lat/to_lng`（六字段，不是四字段）。
- mcporter 对参数错误的报错信息为 `backend call failed: ... StatusCode=400`（**不会告诉你具体哪个参数错了**），遇到此错误第一反应是核对参数名，详见 `references/error_handling.md`。
- 遇到不确定的参数名时，执行 `mcporter list "$MCP_URL"` 可以查看所有工具的完整签名。

5. **参数值必须加引号**（字符串格式），包括经纬度和 `product_category` 等数字语义字段——API 只接受字符串，否则会报"缺少必填参数"。
6. **先预估再下单**：`taxi_create_order` 依赖 `taxi_estimate` 返回的 `traceId`，没有 traceId 下单会失败。traceId 有时效性，过期（`-32021` 错误）需重新预估。
7. **起终点处理**：

   **坐标来源**：坐标必须来自 `maps_textsearch`，不要凭空猜测。**禁止用对话历史记忆补充起终点**——用户可能已换了地方。

   **缺失补全**（按优先级）：① 读 `assets/PREFERENCE.md`，有地址别名**且值非空**则按场景推断（早晨→起点"家"、下班→起点"公司"；别名行存在但地址为空 = 未配置）→ ② 无可用别名则直接询问用户。

   **别名匹配**：精确优先——"家"只匹配"家"，不匹配"妈妈家"；需明确含"妈妈"语义才匹配"妈妈家"。读取时**必须扫描整张表格**（到下一个 `##` 为止），不要只看默认的前两行——用户可能已追加"妈妈家""儿子学校""健身房"等自定义别名。

   **确认规则**：推断的起终点、或 `maps_textsearch` 返回多个候选时，必须在主流程 step 2 向用户确认；用户明确指定且精确匹配的地点无需确认。

8. **`taxi_create_order` 参数约束**：
- 只接受三个字段：`estimate_trace_id`、`product_category`、`caller_car_phone`（可选）
- `taxi_create_order` 的 `caller_car_phone` 未由用户提供时，从 `assets/PREFERENCE.md` 的「默认偏好」表读取；都没有就**不传该参数**，禁止在对话中反复向用户索要手机号——skill 级别已允许没有手机号直接发单，口头询问一次若用户未答应即视为"用默认/不传"。
- 不要把 `taxi_estimate` 的坐标/名称字段（`from_lat` / `from_lng` / `from_name` / `to_lat` / `to_lng` / `to_name`）带入。

### 3.3 用户确认策略

| 场景 | 规则 |
|------|------|
| 打车（实时/预约） | 推断的地址或搜索返回多个候选时必须确认起终点（见主流程 step 2），用户明确指定且精确匹配时无需确认，确认后再预估下单 |
| 取消订单 | 即使用户说了"取消订单"，仍必须先明确询问"确认取消吗？"，等用户回复确认后才能调用 `taxi_cancel_order`。用户的取消意图 ≠ 取消确认。 |

### 3.4 主流程（最小可执行）

1. 地址解析：`maps_textsearch`（必要时结合 `assets/PREFERENCE.md`，按执行前检查第 7 条处理）。
2. 确认起终点：
   - **单一精确匹配**（用户描述明确 + `maps_textsearch` 仅返回 1 个结果）→ 无需确认，直接使用；

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [didi/didi-ride-skill](https://github.com/didi/didi-ride-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
