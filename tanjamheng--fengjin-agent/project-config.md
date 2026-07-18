---
trigger: always_on
description: 制作游戏AI NPC，还原《崩坏：星穹铁道》中的风堇，与我对话，治愈我。
---

# AI风堇立项书

愿这一抹微光，拨开云雾，重见晴空！

## 项目愿景

制作游戏AI NPC，还原《崩坏：星穹铁道》中的风堇，与我对话，治愈我。
我会永远维护这个项目，永远优化下去，让ai越来越还原风堇。

## 迭代目标

1. AI能够越来越还原风堇，甚至还原三观性格。
2. 降低出错率，穿帮率。
3. 前端风堇动画尽可能唯美。

# 当前任务

# 核心文档
核心文档\核心1_需求梳理.md
核心文档\核心2_技术架构.md
核心文档\核心3_开发规范.md
核心文档\核心4_WS通信协议.md
这四个文档将是开发的最高宗旨，不容违反。一切开发将以其为锚点。
其中的核心内容已写入CLAUDE.md中，如有必要，可以进行复习这些文档中的内容。
每当开发新功能时，也会将新功能相关的需求，技术架构等写入这些核心文档中。

## CLAUDE.md 维护规则

本文档是核心1/2/3/4 的**衍生速查**——红线、文件结构、技术约束均从核心文档提取。**核心文档是权威源，本文档是工作副本。**

以下变化必须同步更新本文档：
- 核心3 红线速查新增/修改/删除条目 → 同步本文档「红线速查」
- 核心4 WS 协议消息类型新增/修改/删除 → 同步本文档「WS 协议要点」表格
- 核心2 文件结构树新增/删除/重命名文件 → 同步本文档「文件结构速查」
- 清理链/初始化顺序变更 → 同步本文档「清理链」
- 核心文档新增关键约束或技术决策 → 同步本文档「技术约束」

**每次开发对话结束时**，如果本次修改了核心文档 → AI 必须主动检查本文档是否需要同步。最危险的情况：核心3 加了新红线但本文档没加 → AI 在后续工作中不会遵守那条规则——因为 AI 只看 CLAUDE.md。

## 陷阱速查（本项目踩过的坑）

| # | 陷阱 | 说明 |
|---|------|------|
| 1 | **PowerShell 中不要用 `git commit -m @'...'@`** | here-string 的 `@'` 会被当作文本的一部分混入提交消息，导致消息以 `@ ` 开头。正确做法：先 `$msg = @'...'@` 赋值变量，再 `git commit -m $msg` |
| 2 | **禁止提交任何中文文档** | `核心文档/`、`重要文档/`、`前端开发核心文档/`、`*.md`（中文设计/规范/流程文档）已被 `.gitignore` 忽略。git add 会报 `ignored by .gitignore`。**永远不要用 `-f` 强制提交中文文档**——它们是本地工作副本，不入仓库。只提交 `src/`、`frontend/src/`、`config/`、`main.py`、`requirements.txt`、`CLAUDE.md`、`start.bat` 等代码文件 |
| 3 | **`.bat` 中 PowerShell inline 命令的 `%` 必须写成 `%%`** | CMD 会把 `%` 当变量前缀吃掉——`$i % 15` 会变成 `$i  15`（`%` 被吞 → 后面的数字变成裸 token → PowerShell 语法错误）。正确写法：`$i %% 15`（CMD 将 `%%` 转义为 `%` 传给 PowerShell）。任何传给 PowerShell 的 `%` 都要双写 |
| 4 | **禁止未经允许执行 `git commit`** | 所有 git 提交必须等用户明确说"提交"/"commit"之后才能执行。用户没开口 = 不准 commit。**例外：Code Review 循环中每轮修复后允许自动提交**（CR 流程本身要求每轮 commit） |
| 5 | **`__init__` 中 `self.log` 必须在所有使用它的方法之前赋值** | PersonaDriftGuard R7 P0：`self._parse_anchors()` 内调 `self.log.info()` 但 `self.log = get_logger()` 在调用之后 → 每次构造 `AttributeError` → 漂移检测从未启用。**规则：`self.log` 赋值必须在所有调用 `self.log` 的方法之前。不只 `self.log`——任何被 `__init__` 内方法依赖的属性都适用。** |
| 6 | **会话切换时必须调所有有状态组件的 `reset_state()`** | MoodEngine、BondTracker、PersonaDriftGuard 各有运行时计数器（EWMA、cumulative、consecutive），跨会话不重置会污染新会话。**8 个入口必须全覆盖**：CLI `/new` `/switch` `/clear` `/delete` + WS `_ensure_session`×2 `load_session` `delete_session`。新增入口或新增状态组件 → 双向检查。 |
| 7 | **CLI 路径的状态修复必须同步到 WS 路径** | 同一个会话操作（新建/切换/清空/删除）在 CLI 和 WS 中各有一份实现。修了 CLI 的 `reset_state()` 调用 → 立即检查 WS 的 `_ensure_session`/`load_session`/`delete_session` 是否需要同样修复。多轮 CR 反复出现不对称：R2 修 CLI 漏 WS → R4 补 → R5 又漏 → R7 再补。 |
| 8 | **CSS 百分比宽度参考父容器——加包装即变** | 前端三栏布局 38%/45%/17% 是固定比例的。`.chat-area { width: 45% }` 表示父容器的 45%。如果你把 chat-area 包进一个 `app-panel` 里，`45%` 就变成 app-panel 的 45% 而非主布局的 45%——侧边栏会缩到中间。**规则：HTML 结构加/删任何 wrapper 时，必须检查所有子元素的百分比宽度是否需要更新。** 建议用 `flex: N` 比例值替代 `width: N%`，这样挪到任何容器里都自动等比。 |
| 9 | **CSS `animation` 的 `transform` 会覆盖元素自身的 `transform`** | 如果元素用 `transform: translateX(-50%)` 居中，又挂了 `animation`（含 `transform`），animation 的 transform 会覆盖元素的定位 transform → 元素偏位。**规则：需要定位 transform 的元素不要用含 transform 的 animation。** 用 `left:0; width:100%; text-align:center` 或外层 wrapper 替代。  |
| 10 | **IPC 状态必须原子发射——禁止拆开送** | 启动加载页的阶段标签、进度条、步骤文字、安抚语应该同时出现。如果在 `start()` 里先发 `_sendState("", "正在检查环境...")` 再等后端消息才发阶段标签 → 下方字先跳出来、上方标签晚到 → 视觉割裂。**规则：任何会一起显示的 UI 元素，第一条状态消息就必须全部包含。** 要么全发，要么全不发。 |
| 11 | **多路径触发的状态迁移必须防重入** | 启动器的 "done" 状态被 stdout 消息和健康检查轮询两条路径同时触发 → `_handleReady()` 被调用两次 → `initChatModules()` 两次 → 侧边栏创建两遍 → 两个"新对话"按钮。**规则：任何可能被多个 async 源触发的状态变更函数，第一行就加 `if (this._state.phase === target) return;`。** |
| 12 | **API Key 等敏感字段禁止在传输层脱敏——脱敏只能发生在显示层** | `config_manager.py` 的 `get_current_config()` 把 API Key 脱敏成 `****后4位` 再发给前端 → 前端密码框里存的是脱敏值 → 眼睛按钮睁开看到的仍然是 `****`、闭眼时黑点位数也是错的。**规则：后端返回敏感字段时必须传完整值，前端 `type="password"` / CSS `-webkit-text-security` 等显示层手段负责视觉遮蔽。前端"是否修改"判断用原值对比（`=== originalValue`），不靠 `startsWith("****")`。** 同类场景：任何需要在 UI 中编辑的敏感字段（密钥、token、密码）都应遵循此规则。 |

# 功能速查

## 核心链路

用户输入 → Agent.chat() 统一管线（CLI/WS 共用）→ 小伊卡安全检测 → 心智状态/记忆注入（仅启用时）→ 角色漂移锚点 + 滑动窗口 → LLM 流式生成（含 Tool Calling，最多 5 轮）→ 角色漂移检测 → 会话持久化 → MindManager 异步提交记忆提取与状态分析。主模型只生成角色回复；后台状态模型以严格 JSON 输出情绪/羁绊目标值，FIFO 更新持久化状态。

## 后端已有能力

| 模块 | 做了什么 |
|------|---------|
| 对话引擎 | 流式对话 + Tool Calling 循环 + 停止/超时处理 |
| 风堇角色系统 | 外部 system_prompt.md 定义人设，调角色不改代码 |
| 心智协调层 | 统一开关记忆/情绪/羁绊；最近 3 轮自然对话、双异步调用、状态 FIFO、JSON 校验重试、失败降级与热更新 |
| 情绪状态机 | PAD 三维情绪 + EMA 平滑 + 非对称指数衰减；接收心智模型 JSON 目标值并注入后续 user message |
| 羁绊状态机 | 四维羁绊 + change clamp + 接近度/时间衰减；接收心智模型 JSON 目标值并注入后续 user message |
| 角色漂移检测 | bge-m3 余弦相似度+EWMA平滑，低于阈值自动注入锚点到user message；会话切换时reset_state()清空漂移状态 |
| RAG 知识库 | 6 步管道检索风堇相关知识，LLM 自主决定调用时机 |
| 记忆系统 | 跨会话记住用户信息，双存储（core_memory.md + ChromaDB），异步提取 |
| 安全护栏 | 两级检测（规则引擎 + Llama Guard 3 1B），11 类拦截，Comfort 安抚模式 |
| 会话管理 | JSON 原子写入，14 个 CLI 命令（含会话、知识库管理、调试） |
| WebSocket API | FastAPI + /ws 端点，流式推送 + 取消控制（前端联调用） |

## 前端（V1 已实现）

> **前端开发时，本节是唯一需要看的核心文档内容。** 详细规格查 `前端开发核心文档/`（1=功能边界 2=UI像素 3=架构类接口），WS 协议查 `核心文档/核心4_WS通信协议.md`。

### 交付形态

Windows 桌面客户端，Electron ≥ 28.x + TypeScript + 原生 HTML/CSS，WebSocket 通信。构建工具 electron-vite，打包 electron-builder（Portable 免安装）。

### 核心约束（违反即错）

- **不引入 React / Vue / Svelte** — 单页应用，原生 DOM 足够
- **不引入 CSS 框架**（Tailwind 等）— 手写 CSS，CSS 变量统一管理配色
- **不引入状态管理库** — 全局状态极少，用中心状态对象 + 回调
- **TypeScript 禁止滥用 `any`** — 所有后端通信数据必须有 Interface 定义
- **前后端通信只走 WebSocket** — 由 `config/config.yaml` 定义主端口与备用端口；Electron 必须使用启动器回传的实际本地端口，不引入 REST API
- **布局比例固定** — 左 38%（角色展示）+ 中 45%（对话区）+ 右 17%（历史侧边栏），V2 不变

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tanjamheng/fengjin-agent](https://github.com/tanjamheng/fengjin-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
