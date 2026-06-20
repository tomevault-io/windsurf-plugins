---
trigger: always_on
description: > **用途**：这是给 **AI Agent (Claude/Cursor/Pi/Codex)** 阅读的操作指南。
---

# AIRP MCP Server — Agent Skill Guide

> **用途**：这是给 **AI Agent (Claude/Cursor/Pi/Codex)** 阅读的操作指南。
> 它教你如何通过 MCP 协议使用 AIRP 来管理和推进角色扮演(RP)会话。
> **AIRP 只做数据管理；推理、叙事推进、角色演绎都由你（Agent）完成。**
>
> **本手册可选**：它是这些工具的*操作范例*，不是运行前提。server 独立工作——你可以只挑几个工具用、
> 走自己的流程，完全不照本文做。它也**不依赖、不教你拼接**任何其他 AIRP 仓库（Core / Gateway）；
> 那些是可选的独立项目。

---

## 0. 哲学约束 — 请先阅读

> **AIRP MCP 的全部内容 (Tools / Resources / Prompts / 本文档) 均作为参考建议，
> 不具备强制性。最终决定权和使用权完全归 Agent。**

### 为什么？

不同的 MCP Client 背后是不同的 LLM——Claude、GPT、Gemini、DeepSeek——
它们的推理能力、上下文窗口、思考风格、遵循指令的倾向各不相同。
一套固定的 "标准工作流" 不可能适合所有 Agent。

### 这意味着什么？

```
✅ 你可以做的:
  - 只使用你觉得有用的工具，忽略其余
  - 改写、简化、扩展本文档中的工作流
  - 用你自己的方式推进叙事，不必遵循 6 阶段 / 三幕结构
  - 跳过你认为不必要的步骤（如每轮都读 state）
  - 根据你的上下文窗口大小调整 parallel/sequential 策略

❌ 你不必做的:
  - 照搬文档中的每一个步骤
  - 在不需要验证的时候调 validate_card
  - 每轮都调 apply_lorebook（如果已知世界书不变）
  - 严格遵守 20-30 轮封档的建议

💡 AIRP 的立场:
  AIRP 提供的是「工具箱」，不是「操作手册」。
  你选择哪些工具、按什么顺序、以什么节奏使用——全部由你决定。
  本文档中的一切示例、策略、工作流都是建议，不是规则。
```

---

## 0.5 使用前·优先评估清单（决策提示，非强制）

> 下面不是规则，是**决策提示**。AIRP 无法、也不会强制你执行任何动作。
> 但请在动手前**先权衡得失**——这里给出「为何要先想」和「不先想的代价」，
> 让你基于后果**知情地自行决断**，而非被命令。

### ① 沉浸式 RP 文笔质量
- **建议**：用 `export_context_bundle` 产出成品上下文，交给**隔离的 subagent** 书写；不要在编排主上下文里直接写。
- **为何要先想**：主 Agent（你）的上下文塞满编程身份、工具 schema、harness 提醒；直接写 RP 时这层编程腔会压扁文笔。
- **不先想的代价**：即使预设文风已成功注入，输出仍**僵硬、出戏**（"死人化"）。这是文笔质量问题，不是指令遵循问题——靠改提示词救不回来。
- 你仍自行决断：单模型客户端、轻量短场景可跳过，直接写。

### ② 用户嫌文风僵硬 / 出戏
- **建议**：用 `tune_preset` 改**预设源头**；不要靠重生成或对输出做后处理。
- **为何要先想**：风格其实**已经成功迁移**（特供预设会盖过宿主模型本味）。问题在预设**内容**，常是跨模型的压制脚手架在当前模型上过度压制。
- **不先想的代价**：重生成/后处理只是**洗稿**（治标），且每轮**多烧一遍 token**；源头不改，下轮照样僵。
- 你仍自行决断：若预设本就适配当前模型，可不动。

### ③ 上下文 / Token 纪律
- **建议**：分块、按需读（`get_recent_context` 的 n 取小；用 `apply_lorebook` 关键词扫描，别整文件倒进上下文）；长会话用 `seal_volume` 归档并清空。
- **为何要先想**：AIRP 的数据（卡/预设/历史/世界书）可能很大；插件 blob/KV 也可能巨大。
- **不先想的代价**：把整文件/全历史灌进上下文 → **token 预算瞬间烧光**。读取已有 32 KiB/100KB 上限兜底，但**调用方的读取策略**才是第一防线。
- 你仍自行决断：短会话、小数据可直接全读。

### ④ 世界书：关键词触发，不整本预载
- **建议**：每轮用 `apply_lorebook(text)` 按关键词取相关条目；不要把整本 lorebook 塞进 system prompt。
- **为何要先想**：世界书天生是**关键词门控**的背景库。
- **不先想的代价**：双重损失——**token 浪费** + 角色**「知道太多」**（一次倒出全部设定）→ 破坏「逐步揭示」的沉浸感。
- **适用范围**：④ 的懒加载指**单角色回合的增量 top-up**。**多角色场景**由 `build_scene_system_prompt` 在装配时**全量前载 union 合并世界书**（整个 ensemble 须共驻，剧情才靠角色互相反应推进）——**别用 ④ 去剥它**。
- 你仍自行决断：世界书极小或剧情需要全量背景时可整载。

### ⑤ 首轮注入「思考模式」指令（社区公认质量第一杠杆）
- **建议**：RP 首轮注入一段**思考模式指令**，控制模型推理过程（`<think>`）的形态——**角色沉浸内心独白** 或 **纯分析**；只注一次，后续靠历史持续生效。
- **为何要先想**：这比文风**更上游**。不注入，模型的思考过程不进入角色状态，正文再调也容易「翻译腔/出戏」。
- **不先想的代价**：文笔从根上「死」——思考层没入戏，下游文风补不回来；还常出现「思考全英文/输入味」。
- **怎么落**：内容是**你的数据**（模型相关，如 DeepSeek `<think>` 规则），AIRP 不代写不解析。喂 subagent 时用 `export_context_bundle` 的 `thinking_mode_text` 槽（原样置于 context.md 最前）；或存为 style 工件首轮发送。
- 你仍自行决断：不带 `<think>` 的模型、或已稳定入戏可省。

> 另：导入**不可信第三方卡/预设**前，建议先 `validate_card` / `validate_preset`（详见 §15.5）——孤儿代码/破损宏会漏进 prompt 或被误删。情境性较强，故不列入每会话首要清单。

> 原则：决策提示**抬高优先级与显著性**，但**不剥夺你的选择权**。约束来自「得失」，不来自「命令」。

---

## 1. 核心概念

```
AIRP 的角色：
  ┌─────────────────────────────────────────────┐
  │  数据管理 (AIRP)        │  推理演绎 (YOU)    │
  │─────────────────────────│──────────────────│
  │ 角色卡导入/存储          │ 读取卡 → 构建人格  │
  │ 会话消息持久化           │ 写 user/assistant │
  │ 世界书关键词扫描         │ 决定何时触发      │
  │ 状态追踪 (HP/MP/...)    │ 根据剧情更新状态  │
  │ 预设文风/正则过滤        │ 遵守文风输出      │
  │ 卷封档                  │ 判断封档时机      │
  └─────────────────────────────┴────────────────┘
```

你用 AIRP 的工具读写数据，用你自己的推理能力推进故事。

---

## 2. 快速开始：三步进入 RP

### Step 1: 导入角色卡
```
import_card(png_path="./card/角色.png")    # 推荐：服务端读盘，base64 不进上下文
```
> ⚠️ **别自己 Read PNG 再 base64**：一张 10 MiB 卡 ≈ 13 MiB base64 文本灌进上下文，**烧光 token**（社区实测「蓝屏级卡死」）。用 `png_path` 让 AIRP 服务端读+解析；仅当文件路径不可达时才退回 `png_base64`。

如果没有角色卡，用 `list_characters()` 查看已有卡片。

### Step 2: 启动会话
```
start_session(character_id="<id>", preset_id="<可选预设>")
```
返回 session_id，记住它！后续所有操作都需要。

### Step 3: 构建上下文并开始对话
```
1. 用 build_system_prompt(character_id, preset_id) 获取系统提示词
2. 用 get_recent_context(character_id, session_id, n=10) 获取历史
3. 开始角色扮演——每个回合：
   a. 读 context
   b. 用 apply_lorebook(text, character_id) 扫描世界书
   c. 生成角色回复
   d. 用 append_message 保存用户和助手的消息
```

---

## 3. 角色卡管理

| 工具 | 用法 | 何时用 |
|:--|:--|:--|
| `import_card` | `import_card(png_path)` 推荐 / `png_base64` 退回 | 导入新角色卡（`png_path` 服务端读盘，不烧 token） |
| `list_characters` | 无参数 | 查看所有已导入角色 |
| `get_character` | `get_character(character_id)` | 查看角色详情 |
| `delete_character` | `delete_character(character_id)` | 删除角色及所有数据 |

---

## 4. 会话与消息管理

### 创建会话
```
start_session(character_id, preset_id?, session_id?)
→ 返回: Session created: <session_id> for character: <name>
→ 同时加载: preset 配置 + lorebook 统计 + state 字段列表
```

### 消息操作
```
每轮对话的标准操作：
  1. append_message(character_id, session_id, role="user", content="...")
  2. 你生成回复
  3. append_message(character_id, session_id, role="assistant", content="...")
```

### 获取上下文
```
get_recent_context(character_id, session_id, n=10)
→ 返回最近 N 条消息的 JSON 数组（含 role, content, timestamp）
```

---

## 5. 世界书/Lorebook — 动态世界知识注入

世界书是**关键词触发**的背景知识库。对话中提及关键词时自动注入相关设定。

### 读取世界书
```
读 Resource: airp://characters/{id}/world/lorebook
```

### 关键词扫描
```
apply_lorebook(character_id, text="我们在天剑阁门前停下")
→ 返回匹配的 lorebook 条目（如 "天剑阁" → 门派背景/历史）
```

### 在对话中正确使用
```
每轮对话前：
  1. 调 apply_lorebook(text=用户最新消息, character_id)
  2. 如果返回条目非空 → 将这些信息纳入你的角色知识

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GhostXia/AIRP-MCP-Server](https://github.com/GhostXia/AIRP-MCP-Server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
