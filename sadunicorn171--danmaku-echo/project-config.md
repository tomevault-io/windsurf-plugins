---
trigger: always_on
description: Danmaku Echo / 弹幕回声是一个 Chrome / Microsoft Edge Manifest V3 浏览器扩展。
---

# Danmaku Echo — Codex Instructions

## 1. Project

Danmaku Echo / 弹幕回声是一个 Chrome / Microsoft Edge Manifest V3 浏览器扩展。

核心功能：

- 直播弹幕 `+1`
- 回复
- 复制
- 本地收藏
- 收藏快捷面板与轮盘
- 富文本 / Emoji / 平台图片表情处理
- 视频弹幕悬停操作
- 轻量弹幕雷达（高频弹幕 `+1` 提醒）
- 多直播平台统一体验

当前支持：

- Bilibili Live
- Douyu Live
- Huya Live
- Douyin Live

技术栈：

- TypeScript
- Vue 3
- Vite
- Manifest V3
- Vitest
- ESLint
- Oxlint
- Prettier

---

# 2. Communication

默认使用中文与用户交流。

代码、变量、类型、接口、commit message 等继续遵循项目原有语言和命名方式。

回答应直接说明：

- 找到了什么
- 修改了什么
- 验证了什么
- 是否存在限制

不要输出大量与任务无关的教程。

---

# 3. Primary principle

始终优先：

1. 正确理解现有代码
2. 找到问题根因
3. 最小正确修改
4. 保持现有架构
5. 验证修改
6. 避免影响其他平台

不要因为存在更“优雅”的设计，就主动重写已经工作的代码。

---

# 4. Before modifying code

修改前先检查：

```bash
git status
```

如果工作区已经存在未提交修改：

- 不要覆盖它们
- 不要删除它们
- 不要将无关修改混入当前任务
- 不要假设这些修改是 Codex 之前产生的

然后通过搜索和调用关系定位代码。

不要一开始读取整个仓库。

优先搜索：

- 功能名称
- 平台名称
- 类型名
- selector
- message type
- adapter
- sender
- entrypoint
- 测试

---

# 5. Repository map

主要代码边界：

```text
src/core/
```

放跨平台共享类型、文本处理、设置合并等核心逻辑。

```text
src/entries/
```

放浏览器扩展运行时入口。

当前实际构建入口包括：

- `content.ts`
- `service-worker.ts`
- `douyin-bootstrap.ts`
- `douyin-content.ts`
- `douyin-page-hook.ts`

入口旁的通用三平台装配模块是 `content-app.ts`。

`content.ts` 和 `douyin-content.ts` 已是薄启动器；通用三平台装配当前位于
`content-app.ts`，抖音隔离世界装配位于
`src/platforms/douyin/content/content-app.ts`。不要把复杂业务逻辑重新堆回薄入口，
也不要继续扩大装配文件中的平台细节。

```text
src/features/
```

放独立功能。

当前包括：

- `favorites/`
- `repeat-reminder/`

`repeat-reminder/` 是页面内存中的高频弹幕 `+1` 提醒，不是热词、问题、摘要或模型分析系统。

```text
src/platforms/
```

放平台适配代码。

包括：

- `bilibili/`
- `douyin/`
- `douyu/`
- `huya/`
- `live/`

`live/` 用于真正跨平台的直播公共能力。

平台专属行为不要泄漏到公共代码。

当前 `src/platforms/live/adapters.ts` 是允许引用三个平台 adapter 工厂和斗鱼可选 runtime
boundary 的显式装配缝；除该工厂和 entry composition root 外，通用控制器不应直接导入平台实现。

抖音平台内部还分为：

- `src/platforms/douyin/content/`：扩展隔离世界
- `src/platforms/douyin/page/`：页面 MAIN world

---

# 6. Architecture boundaries

优先保持：

```text
Platform-specific page/data
        ↓
Platform adapter / parser / controller
        ↓
shared live behavior（适用时）
        ↓
feature layer
        ↓
UI / send / favorite / reply
```

不要让公共模块通过大量：

```ts
if (platform === 'douyu') ...
if (platform === 'bilibili') ...
```

实现平台行为。

平台差异优先留在：

```text
src/platforms/<platform>/
```

如果多个平台真正共享相同行为，再提取到公共层。

不要为了“未来可能复用”提前抽象。

---

# 7. Platform behavior is not symmetrical

不要假设四个平台实现方式完全相同。

当前代码结构本身就存在明显差异。

修改某个平台时首先阅读该平台已有实现。

不要为了统一目录结构而强制：

- 添加无意义 adapter
- 添加空 wrapper
- 创建统一但复杂的 interface
- 重写成熟的平台实现

统一用户体验不等于强制统一底层实现。

---

# 8. Bilibili

Bilibili 相关逻辑主要位于：

```text
src/platforms/bilibili/
```

修改之前优先检查：

- `adapter.ts`
- `candidate-rules.ts`
- `dom-config.ts`
- `rich-emoji.ts`
- `sender.ts`
- `rich-message-sender.ts`
- `direct-emoticon-send.ts`
- `emoticon-metadata.ts`
- `native-send-observer.ts`
- `overlay-motion.ts`
- 对应 `__tests__/`

Bilibili 富弹幕和图片表情存在官方输入面板与后备发送逻辑。

不要随意把图片表情退化为普通文字。

不要绕过现有资源校验直接发送房间资源。

修改发送逻辑时必须同时考虑：

- 普通文字
- Unicode Emoji
- 图片表情
- 文字与表情混排
- 当前直播间
- 官方输入路径
- 后备发送路径

---

# 9. Douyin

Douyin 是项目中最特殊的平台。

相关入口：

```text
src/entries/douyin-bootstrap.ts
src/entries/douyin-content.ts
src/entries/douyin-page-hook.ts
```

相关模型：

```text
src/platforms/douyin/
```

当前分层包括：

- 根目录：barrage/track model、chat message、emoji catalog/token、rich data、
  protocol、own message、input order 和 rich message sender
- `content/`：侧聊解析、富内容恢复、发送者索引、悬停、操作分发、编辑器、发送、
  本人消息、雷达采集、page bridge、装配和生命周期
- `page/`：Renderer 类型、MAIN world page bridge、Canvas Hook、Worker/MessagePort Hook、
  官方弹幕内容解析器、CSS 像素内容测量器、Renderer 实例注册表、纯轨道运动模型和
  无 DOM 频道调度器、两阶段提交的 DOM Renderer、页面胶囊与单条悬停控制器，
  本人弹幕匹配器、页面诊断控制器、MAIN world 总生命周期 runtime 和页面装配工厂

`douyin-content.ts` 和 `douyin-page-hook.ts` 均已完成拆分并删除 `@ts-nocheck`。
`douyin-page-hook.ts` 只负责重复加载保护、创建并启动 runtime；MAIN world 装配位于
`page/page-app.ts`，总生命周期由 `page/page-runtime.ts` 统一持有。DP-01 至 DP-15 已完成。

当前抖音视频弹幕采用安全 DOM 接管架构。

除非用户明确要求改变架构：

不要将其重写为：

- 主动拦截 WebSocket
- 复制原生 Canvas 像素
- 直接替换官方 Worker
- 依赖私有 WebSocket 发包

修改抖音代码时必须考虑：

- `live.douyin.com`
- `www.douyin.com` SPA 进入直播
- 切换直播间
- Worker 生命周期
- Canvas 恢复
- DOM 接管失败回退
- 全屏
- 页面销毁
- 重复初始化

出现异常时，扩展应尽可能恢复官方原生行为，而不是留下隐藏 Canvas 或失效弹幕层。

---

# 10. Douyu

Douyu 相关逻辑位于：

```text
src/platforms/douyu/
```

优先检查：

- `adapter.ts`
- `message-content.ts`
- `native-capsule.ts`
- `native-hover.ts`
- `native-motion-fallback.ts`
- `rich-emoji.ts`
- `rich-message-sender.ts`
- `sender.ts`

斗鱼自身已经存在原生弹幕交互能力。

修改扩展胶囊时注意区分：

- Danmaku Echo 胶囊
- 斗鱼原生胶囊

不要因为隐藏扩展元素而误删、破坏或永久修改斗鱼原生功能。

---

# 11. Huya

Huya 相关逻辑位于：

```text
src/platforms/huya/
```

优先检查：

- `adapter.ts`
- `candidate-config.ts`
- `rich-emoji.ts`
- `rich-message-sender.ts`
- `sender.ts`

虎牙仍以 selector adapter 为基础，平台层比 Bilibili、抖音轻量。

不要为了与 Bilibili 或 Douyin 对齐而人为增加复杂层级。

如果现有 adapter 已能表达行为，优先继续扩展 adapter。

---

# 12. DOM

直播网站是动态页面。

不要假设：

```text
DOMContentLoaded
```

之后 DOM 不再变化。

必须考虑：

- SPA
- 切房
- React/Vue 重渲染
- 播放器重新创建
- 聊天容器替换
- 全屏容器变化

MutationObserver 应尽可能监听最小范围。

创建 Observer、Listener、Timer 后必须考虑清理。

禁止通过大量短周期轮询扫描整个 document。

---

# 13. Content script vs page context

牢记：

```text
Content Script
≠
Page JavaScript Context
```

不要假设 content script 可以直接访问页面内部对象。

需要 page context 数据时：

优先复用项目已有：

- page hook
- script injection
- postMessage / event channel

不要为了一个新功能新增第二套平行通信协议。

---

# 14. Manifest V3

不要假设：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SadUnicorn171/danmaku-echo](https://github.com/SadUnicorn171/danmaku-echo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
