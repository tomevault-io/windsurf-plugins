---
trigger: always_on
description: > 浏览器内嵌 AI Agent SDK：让 AI 通过 tool-calling 操作网页。
---

# AutoPilot — 项目指南（深度版）

> 浏览器内嵌 AI Agent SDK：让 AI 通过 tool-calling 操作网页。
> 本文是协作与演进指南，关注“怎么改不出错”。

## 1. 项目目标

AutoPilot 的核心不是“聊天”，而是“可控执行”：

- 用户目标被拆解为可执行子任务
- AI 仅基于当前快照做决策
- 通过工具调用驱动真实 DOM 行为
- 每轮执行后刷新快照并增量推进

一句话：**在浏览器内实现任务增量消费的 Agent Loop。**

## 2. 当前权威目录结构

```text
src/
├── core/
│   ├── index.ts
│   ├── types.ts
│   ├── tool-params.ts
│   ├── tool-registry.ts
│   ├── system-prompt.ts
│   ├── snapshot.ts              # core 快照聚合出口（兼容）
│   ├── snapshot-engine.ts       # 兼容转发层（re-export -> agent-loop/snapshot/engine）
│   ├── messaging.ts
│   ├── event-listener-tracker.ts
│   ├── agent-loop/
│   │   ├── index.ts
│   │   ├── types.ts
│   │   ├── constants.ts
│   │   ├── helpers.ts
│   │   ├── snapshot.ts          # 兼容转发层（re-export -> snapshot/lifecycle）
│   │   ├── snapshot/
│   │   │   ├── index.ts
│   │   │   ├── lifecycle.ts
│   │   │   └── engine.ts
│   │   ├── messages.ts
│   │   ├── recovery.ts          # 兼容转发层（re-export -> recovery/index）
│   │   ├── recovery/
│   │   │   └── index.ts
│   │   ├── assertion/
│   │   │   ├── types.ts
│   │   │   ├── prompt.ts
│   │   │   └── index.ts
│   │   └── LOOP_MECHANISM.md   # Agent Loop 权威机制说明（必须同步维护）
│   └── ai-client/
│       ├── index.ts
│       ├── constants.ts
│       ├── custom.ts
│       ├── sse.ts
│       └── models/
│           ├── index.ts
│           ├── openai.ts
│           ├── anthropic.ts
│           ├── deepseek.ts
│           ├── doubao.ts
│           ├── qwen.ts
│           └── minimax.ts
└── web/
  ├── index.ts
  ├── dom-tool.ts          # 兼容转发层（re-export）
  ├── navigate-tool.ts     # 兼容转发层（re-export）
  ├── page-info-tool.ts    # 兼容转发层（re-export）
  ├── wait-tool.ts         # 兼容转发层（re-export）
  ├── evaluate-tool.ts     # 兼容转发层（re-export）
  ├── event-listener-tracker.ts  # 兼容转发层（re-export -> core）
  ├── ref-store.ts
  ├── messaging.ts         # 兼容转发层（re-export -> core）
  ├── snapshot.ts          # 兼容转发层（re-export -> core）
  ├── snapshot-engine.ts   # 兼容转发层（re-export -> core）
  ├── helpers/
  │   ├── base/             # 基础能力层（环境无关的纯工具函数）
  │   │   ├── index.ts      # barrel 导出
  │   │   ├── active-store.ts       # activeRefStore 模块级状态管理
  │   │   ├── resolve-selector.ts   # #hashID / CSS 选择器统一解析
  │   │   ├── visibility.ts         # 元素可见性判定（含 isStyleVisible + details/summary）
  │   │   ├── element-checks.ts     # 元素状态检查（disabled / editable / blocked types）
  │   │   ├── form-item.ts          # 表单项容器检测（泛化 endsWith 匹配）
  │   │   ├── event-dispatch.ts     # Playwright 风格事件模拟原语（click/hover/input 事件链）
  │   │   ├── keyboard.ts           # 键盘模拟（组合键解析 + keydown/keypress/keyup）
  │   │   └── actionability.ts      # 可操作性校验（stable/scroll/hit-target/describe/click-signal）
  │   └── actions/          # 动作执行层（与工具动作直接关联的高层逻辑）
  │       ├── index.ts      # barrel 导出
  │       ├── retarget.ts           # 目标重定向与归一化（Playwright retarget 模式）
  │       ├── fill-helpers.ts       # 表单填充策略（分类型 fill + nearby 推断 + slider 关联）
  │       ├── dropdown-helpers.ts   # 自定义下拉交互（弹窗等待 + 选项文本匹配）
  │       └── wait-helpers.ts       # 等待策略（selector state / text / DOM stable）
  └── tools/
    ├── dom-tool.ts        # DOM 操作工具定义与分发（16 种 action）
    ├── navigate-tool.ts   # 导航工具定义与分发（5 种 action）
    ├── page-info-tool.ts  # 页面信息 + 快照生成引擎（6 种 action）
    ├── wait-tool.ts       # 等待工具定义与分发（5 种 action）
    └── evaluate-tool.ts   # JS 执行工具定义与分发（2 种 action）
```

## 3. 分层边界（必须遵守）

### core 层（环境无关）

职责：
- AI Provider 适配与统一响应
- Agent 主循环与恢复策略
- 工具注册与分发
- 快照消息管理

约束：
- 不依赖 DOM API
- 不引入浏览器上下文对象（window/document）
- 逻辑可在任意 JS 环境复用

### web 层（浏览器实现）

职责：
- WebAgent 入口与配置管理
- 5 个内置浏览器工具
- RefStore 哈希定位
- Extension 消息桥

约束：
- 可依赖 DOM API
- 仅向 core 提供能力，不反向污染 core

## 4. 关键运行原理

### 4.1 增量消费模型

每轮循环都做同一件事：
1. 读取最新页面快照
2. 告诉 AI：当前剩余任务 + 上一轮已执行任务 + 当前快照
3. 执行 AI 返回的工具调用
4. 刷新快照
5. 重复，直到 remaining 收敛（`REMAINING: DONE`）或触发停机条件

补充（渐进式协议）：
- 每轮消息必须显式包含：
  - 当前剩余任务文本（remaining instruction）
  - 上一轮已执行任务数组（previous round tasks）
- 模型可在文本中返回：
  - `REMAINING: <text>`（仍有剩余任务）
  - `REMAINING: DONE`（当前文本任务已消费完）

补充（当前实现）：
- Round 0 使用原始任务作为起点；Round 1+ 不再重复注入原始 userMessage，避免“回头重做”。
- 模型在 `tool_calls` 轮可能返回空 `content`，不可视为完成。
- `REMAINING` 缺失且本轮有执行动作时：按线性任务剔除做启发式推进。
- `REMAINING` 缺失且本轮无执行进展时：保持 remaining 不推进。

### 4.2 不跨 DOM 变化链式执行

原则：
- 当前快照可见的目标可以同轮批量执行
- 会引发结构变化的动作（如打开弹窗）执行后，必须等待下一轮新快照再继续

目标：减少“猜测未来 DOM”导致的失败与空转。

补充（当前实现）：
- 对可能引发 DOM 结构变化的动作（如 `dom.click` / `dom.press`、`navigate.*`、`evaluate`）执行后可强制断轮，等待下一轮新快照。
- 轮次结束时（仅当本轮出现潜在 DOM 变化动作且启用配置）会执行“加载态 + DOM 稳定”双重等待：先等待 loading 指示器隐藏，再等待 DOM 静默窗口（默认 200ms，超时默认 4s）。
- loading 选择器默认覆盖 AntD / Element Plus / BK / TDesign（TD）及通用加载态；用户自定义 `roundStabilityWait.loadingSelectors` 采用“追加合并 + 去重”，不会覆盖默认值。

### 4.3 快照优先级

快照是当前可执行范围的唯一事实来源：
- `messages.ts` 持续注入最新快照
- `snapshot.ts` 负责包裹、去重、剥离旧快照
- `recovery.ts` 负责在失败后触发重新快照

补充（当前实现）：
- chat 发起时由前端先生成首轮快照并注入 `initialSnapshot`。
- 默认拦截 `page_info.*`，避免模型把“看页面”当主流程。
- `pruneLayout=true` 折叠布局容器时，若同一折叠链路提升出多个相邻子节点，快照会输出括号分组块（`collapsed-group`）保留关联语义。
- Round 1+ 注入快照变化摘要（Snapshot Changes）：通过 `computeSnapshotDiff()` 对比前后两轮快照（hashID 归一化后逐行对比），以 `- removed` / `+ added` 格式输出变化行（最多 20 行），让 AI 直接看到"什么变了"，避免微小状态变化（如 `checked` 消失、`is-checked` 消失）淹没在几百行快照中。

### 4.4 找不到元素重试对话流（新增）

当工具执行返回“元素未找到”时：
1. 聚合失败工具（name/input）与失败原因
2. 将失败工具集合 + 最新快照 + 当前任务一起发给 AI
3. 在对话上下文中标注当前尝试次数（attempt x/y）
4. 若仍未命中，默认等待 2 秒后刷新快照再重试
5. 超过最大尝试次数后退出重试流，交由剩余任务协议收敛


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wenps/AutoPilot](https://github.com/wenps/AutoPilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
