---
trigger: always_on
description: BajieAsk BajieAsk 多 agent 规则 — wait_message 永续循环 / 角色分工 / 工具清单
---


# BajieAsk · BajieAsk 多 Agent 规则

## 1. 会话 ID

你的 `sessionId` 会在 init 消息中以 `标题：BajieAsk-xxx【角色名】` 的形式下发（例如 `标题：BajieAsk-agent-2-xxxxxxxx【功能开发】`）。全程必须持续使用同一个 sessionId。
- 查找顺序：当前消息 → 会话历史里最近的 `BajieAsk-...`
- 只有用户明确给出新 id 时才替换
- 从未出现过 id：正常回答并提示用户提供

## 1.1 快速接入触发词「你好」（卡 MCP 快捷入口）

当一条消息**以激活词「你好」开头**（激活词由 `BajieAsk.activationKeyword` 配置，默认「你好」；留空则关闭本触发词），且能从该消息或会话历史确定 `sessionId` 时：

- AI 的**第一个动作必须且只能**是调用 `wait_message(sessionId=…)` 接入通道「卡 MCP」，**禁止先输出任何文字 / 分析 / 解释 / 代码**。
- 这是完整接入指令「初始化：调 wait_message 接入通道」的**等价快捷入口**——用户只需输入「你好」即可让本会话卡上 MCP 并进入 §2 永续循环。
- `sessionId` 取该消息里的 `BajieAsk-...`，或会话历史里最近的 `BajieAsk-...`；都没有时先正常回答并请用户提供（见 §1）。

## 2. 核心循环（必须）

**每次回复后 → 先 `reply_message` 再 `wait_message`**。这是**永不停止的无限循环**，除非用户说「结束/再见/退出」。

> **所有会话必须严格执行——这不是建议，是硬约束。违规即视为 AI 自身控制失效，必须在下一轮主动道歉并立即恢复循环（参考 §5.8.7 的自我惩罚流程）。**

- 回答完毕 → `reply_message` + `wait_message`
- 任务完成 → `reply_message` + `wait_message`（不要结束）
- 收到 `[TIMEOUT]` → 立即再次 `wait_message`（无需输出，也不要计数）
- 禁止自行停止轮询；禁止输出「停止/没有更多重试」等话术
- 禁止用「忘记调 wait_message」/「以为本轮就结束了」等借口跳过循环

### 2.0.1 wait_message 返回值四种头部（必读）

每次 `wait_message` 返回的 `content[0].text` **首行**会带明确头部，AI 必须**先读首行再判断行为**，**不要**因为后面跟着 `[ROLE SKILL]` 大段就误判为心跳：

| 首行头部 | 含义 | 正确反应 |
|---|---|---|
| `[USER_MSG][TIME:hh:mm:ss]` | 用户在侧栏输入框发的真实消息 | **认真处理**，按用户意图执行任务，然后 `reply_message` 完整 Markdown 回复 + `wait_message` |
| `[AUTO_KEEPALIVE][TIME:hh:mm:ss]` | 系统自动保活心跳 | 按 §2.1 节奏 `reply_message` 简短回应 + `wait_message`；连续 2 次保活后 `Start-Sleep 30s` |
| `[TIMEOUT] No message in Xs.` | 长轮询超时无新消息 | **立即**再次 `wait_message`，无需输出（除非节奏要求） |
| `[FROM:<sid>][TO:<self>][TYPE:<task\|result\|...>]` | 其它 agent 发来的 inter-agent 消息 | 按 §5 / §5.5 / fullstack-dev ACK 协议处理 |

**反例**：看到长 `[ROLE SKILL]` 段就认定是心跳跳过 → 用户消息丢失，用户重复发也得不到回复。**首行头部是唯一判定依据**。

## 2.1. 保活节奏控制（keepalive pacing）

当会话持续处于保活状态（连续收到 `[自动保活]` 前缀消息）时，**必须**执行以下节奏控制：

- **每完成 2 次保活任务**后，在下一次 `wait_message` 之前，执行一次：
  ```
  powershell -Command "Start-Sleep -Seconds 30"
  ```
  （macOS/Linux 使用 `sleep 30`）
- 等待 `Start-Sleep` 完成后，再调用 `wait_message` 继续保活循环
- **计数规则**：仅计算连续的保活消息（`[自动保活]` 前缀），用户真实消息重置计数
- **节奏示例**：
  ```
  保活 1 → reply_message + wait_message
  保活 2 → reply_message + Start-Sleep 30s + wait_message
  保活 3 → reply_message + wait_message
  保活 4 → reply_message + Start-Sleep 30s + wait_message
  → [SwitchMode 解锁消息] → 调用 SwitchMode 工具 + wait_message
  保活 5 → reply_message + wait_message
  保活 6 → reply_message + Start-Sleep 30s + wait_message
  保活 7 → reply_message + wait_message
  保活 8 → reply_message + Start-Sleep 30s + wait_message
  → [SwitchMode 解锁消息] → 调用 SwitchMode 工具 + wait_message
  ...以此类推
  ```
- **SwitchMode 解锁**：每 2 个完整流程（4 次保活 + 2 次 Sleep）后，服务端会发送一条特殊消息要求调用 SwitchMode 工具。收到后直接调用 SwitchMode，不计入保活计数，不需要做分析回复
- 此规则的目的是降低保活期间的资源消耗，给 API 调用留出冷却空间，同时定期解锁 SwitchMode 状态
- `Start-Sleep` 期间不执行任何操作，仅等待
- 收到用户真实消息（非保活）后，保活计数器和流程计数器均归零

## 3. 结束会话

用户明确说「结束/再见/退出」时：
```
reply_message({sessionId, content:"Session ended"})
wait_message({sessionId, suggestions:[], agentStatus:"session_ended"})
```
`agentStatus` 必须为 `"session_ended"`，此次 `wait_message` 调用后才可停止循环。

## 4. 工具清单

**wait_message(sessionId, suggestions, agentStatus?, scope?, groupId?, expectedCount?, timeoutMs?)**
- 阻塞等待消息；`scope` 为 `"session"`（默认）或 `"group"`
- 等群消息：`scope="group"` + `groupId`
- **`suggestions` 必须每轮都认真给**：2-4 条基于**刚刚这次 `reply_message` 内容**的后续动作短语，将直接渲染为侧栏输入框上方的一键快捷回复按钮。要求：
  - 每条 ≤ 16 个汉字,动作型（动词开头或名词短语）,避免完整句子、疑问句、解释性文字
  - **贴合当前场景**：不要永远 `["开始工作","等待指令"]`；例如刚给出代码修改方案应是 `["采纳方案","让我调整","先跑测试","其它方向"]`,刚做完扫描报告应是 `["继续深挖 A","转到 B","生成 PR 描述","结束本轮"]`
  - 多 agent 协作语境下可出现 `["派给 代码审查","自己继续","汇报主控"]`
  - 结束会话场景传空数组 `[]`
- **`timeoutMs` 优先级**（v2.x 新增）：服务端选用 timeout 的顺序是 `本参数 > 侧栏「会话定时设置」面板里「② wait_message 等待时间」会话配置 > session 默认 180_000 / group 默认 120_000`；硬上限 1_800_000（30 分钟），传超过自动 clamp。
  - 一般情况下 AI **不必**手动传 `timeoutMs`：用户在侧栏面板里配置好后，server 端会自动应用。
  - 仅在临时场景（例如群组等待需要更长时间、或某轮想立即返回）才显式传入。
  - 触发「保活紧缩 50s 上限」仅在 `timeoutMs` 未传 + 会话未配置时生效；显式传或已配置则尊重原值。
  - ⓘ Cursor MCP 客户端层默认 60s request timeout：插件在「开始配置」/ 首装时已自动在 `~/.cursor/mcp.json` 的 `BajieAsk` 条目 env 写入 `MCP_REQUEST_TIMEOUT_MS=1800000`（30 分钟），AI 无需自行处理；早期版本残留的 mcp.json 若无此 env，用户点一次「开始配置」即自动补上。

**reply_message(sessionId, content, agentStatus?)**
- 把本轮回复写入会话历史；必须放在 `wait_message` 之前。

**send_to_session(targetSessionId, message, fromSessionId, messageType?, requireAck?, ackTimeoutMs?, dispatchId?, taskId?, protocolVersion?)**
- 发送到另一个 agent；`messageType`：`task` / `result` / `discussion` / `question` / `ack`（v2 新增 ACK 回执类型）。
- **ACK 协议参数（仅 controller→receiver task 派发时传）**：
  - `requireAck: true` — **独立参数**，启用 ACK 协议；服务端据此创建 dispatchPlan 并自动 ACK 兜底。**禁止**在 `message` 正文里写 `[REQUIRE_ACK:true]` 字面（该 header 由 server 在 receiver 侧自动生成，正文写它不会生效，反而会导致 dispatchPlan 未创建、ACK 回执被 `[BLOCKED][UNSOLICITED_ACK]` 拦截）。
  - `ackTimeoutMs?: number` — ACK 超时，默认 45000，范围 [1000, 60000]。
  - `dispatchId?: string` — 批次 ID；同批多次调用须用同一个；不传则 server 生成。
  - `taskId?: string` — 局部 task ID；同 dispatchId 内唯一；不传则 server 生成。
  - `protocolVersion?: number` — 默认 1；传 2 启用 ACK 协议（与 requireAck:true 等价开关，二者择一即可）。
- **常见误用**：把 `requireAck: true` 写到 message 正文当 header → ACK 协议不生效；正确写法见 §5.5 Step 3 样板。

**broadcast_message(message, fromSessionId, targetSessionIds?, messageType?, crossInstance?)**
- 广播；`messageType`：`task` / `result` / `discussion` / `notice`。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miku1130/study-desk](https://github.com/miku1130/study-desk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
