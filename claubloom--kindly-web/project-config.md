---
trigger: always_on
description: **Kindly Web** — a Chrome extension (Manifest V3) that rewrites hostile comments on Bilibili into kind, rational expressions via a user-configured LLM API. Built with **WXT 0.21 + vanilla TypeScript** (no UI framework). Architecture and design decisions are documented in this file (see Architecture & Data Flow below).
---

# Repository Guidelines

## Project Overview

**Kindly Web** — a Chrome extension (Manifest V3) that rewrites hostile comments on Bilibili into kind, rational expressions via a user-configured LLM API. Built with **WXT 0.21 + vanilla TypeScript** (no UI framework). Architecture and design decisions are documented in this file (see Architecture & Data Flow below).

Security invariant: **LLM requests happen ONLY in the Service Worker** (`entrypoints/background.ts`). The API key never enters page processes; content scripts only receive rewritten text DTOs.

## Architecture & Data Flow

**采集 = 劫持 B 站前端 API**（main-world 内容脚本包装页面 fetch/XHR）：

```
Bilibili page / player iframe ──> bilibili-danmaku.content.ts (world: MAIN, document_start)
   wrap window.fetch + XHR by URL:
   ├─ /x/v2/reply/(main|reply)  → 响应零阻塞放行（页面先渲染原文）→ 提取 rpid/message
   │                               → KW_REWRITE_COMMENTS → SW 改写 → 结果回传
   │                               → bilibili.content.ts (ISOLATED) 按 rpid 定位 DOM 替换
   └─ /x/v2/dm/web/seg.so (protobuf) /x/v1/dm/list.so (xml)  → 先放行（不阻塞播放器）
                                     → 全量分批并发（40/批 × 4 在飞）→ KW_REWRITE_BATCH → SW 批量改写
                                     → 完成后：① 屏上替换（DOM 元素文本匹配，状态角标 改/✓/!/跳）
                                       ② 结果缓存，播放器重载段时直接替换响应
popup / onboarding / options ──KW_SET_ENABLED / KW_CONFIG_CHANGED / KW_TEST_CONNECTION / KW_GET_STATUS──> SW
```

- **两个内容脚本，两个 world**：`bilibili-danmaku.content.ts`（`world: 'MAIN'`，劫持层，matches 含 `player.bilibili.com`；`runAt: document_start` 必须在页面业务脚本前完成包装）；`bilibili.content.ts`（`world: 'ISOLATED'`，结果应用层：DOM 替换/气泡/角标）。WXT entrypoint 名冲突规则：两者必须用不同文件名首段（`bilibili.content.ts` vs `bilibili-danmaku.content.ts`）。
- **MAIN world 无 chrome.runtime（已实测，Chrome 137+ Self-XSS 防护）**：main-world 劫持层**不能**直接 `chrome.runtime.sendMessage`（静默失败）。所有 main world ↔ 扩展通信走 `lib/bridge.ts` 的 postMessage 桥：isolated 侧 `startBridge()` 转发（含 ready 握手 + 未就绪消息队列，覆盖页面脚本先于 CS 注入的竞态）；弹幕批结果经桥回传 MAIN。**`startHijack` 必须立即调用一次 `listenFromExtension(() => {})`**——否则 bridgeReady 永不置位，所有评论消息永远排队（已实测：评论静默丢失而弹幕正常）。
- **B 站接口为 wbi 路径（实测 2026-08）**：评论 `/x/v2/reply/wbi/main`、弹幕 `/x/v2/dm/wbi/web/seg.so`。URL 正则必须匹配 `(?:wbi/)?` 段。
- **B 站评论区渲染在 `<bili-comments>` 的 Lit shadow root 内**：多层嵌套（`bili-comments` → `#feed > bili-comment-thread-renderer` → `bili-comment-renderer#comment` → `#content > bili-rich-text` → `#contents`），**评论 DOM 没有 rpid 属性**（实测）→ 结果应用按 **seq**（接口顺序 ↔ `#feed` 直接子 thread 顺序）定位；**置顶评论（`data.top_replies`）渲染在评论区最前**，提取必须纳入并先于 replies 编序（2026-08 实测，遗漏会导致全列表 seq 错位）；**楼中楼**（`/x/v2/reply/reply` 与 main 接口内嵌 `replies[].replies`）按 **path**（`[顶层 thread seq, 楼中楼内索引]`）定位——thread shadow → `div#replies > bili-comment-replies-renderer` (shadow) → `div#expander > div#expander-contents` → `bili-comment-reply-renderer`（亦有 shadow，内部 `div#body > bili-rich-text > #contents`）；main 劫持时建立 rpid→thread seq 映射供 reply 接口定位父线程；**文本匹配必须规范化**（`normalizeCommentText`，`lib/sites/bilibili.ts`：去 `[表情]` 标记 + 压缩空白——DOM 渲染后表情变图片、textContent 缺标记，接口原文与 DOM 文本不一致）；seq 命中后校验文本，不符则按规范化原文在全部线程/楼中楼列表中兜底匹配（优先未登记结果），找不到返回 null 不误替换；"查看全部回复"替换列表导致索引偏移同理。第二层（回复的回复）不提取不改写。rewrite-ui 的 `queryShadowAll`/`findShadowById` 递归穿透 shadow root；fallback observer 递归 observe shadow roots；badge 用 inline style（shadow 内全局 CSS 失效）。结果先于 DOM 渲染的竞态在 shadow 场景同样存在（pendingResults 队列，存 seq/path，15s 窗口）。
- **评论异步语义**：劫持响应**原样放行**（用户立即看到原文）→ 异步改写 → 结果按 **seq**（顶层：接口 replies 顺序 ↔ `#feed` 内 thread 顺序）或 **path**（楼中楼：`[顶层 seq, 子索引]`）定位 DOM 替换（`rpid`/`data-kw-id` 属性为回退路径，含 shadow 穿透）。**表情重建**（实测 2026-08）：B 站把 `[doge]` 等渲染为 `<img alt="[doge]" src="//i0.hdslb.com/bfs/emote/…">`（alt 即标记），替换/恢复/占位统一走 `rebuildTextWithEmotes`（rewrite-ui.ts，collectEmotes 收集原文表情元素、按标记克隆复用），改写文本中的表情标记保持图片显示而非字面文本。**结果可能先于 DOM 渲染到达**（快速模型/本地 mock，已实测触发）→ `pendingResults`/`pendingErrors` 队列重试 15s。若 10s 内未收到 `KW_HIJACK_ACTIVE`（劫持失效，如 B 站改版），isolated CS 回退 MutationObserver 采集（兜底路径仅覆盖顶层评论）。
- **弹幕异步语义**：先放行（弹幕立即显示，绝不等待）→ **弹幕全量送 SW 改写**（阴阳怪气交由 LLM 判断，LLM 侧有"本身友善则原样返回"规则兜底）→ 改写结果**屏上替换**。**全量批管道**（lib/hijack-engine.ts）：段内弹幕按 `DM_BATCH_SIZE=10` 条/批切片（小批 → LLM 响应快），`DM_MAX_INFLIGHT=16` 批并发在飞，SW 全局并发 `MAX_CONCURRENCY=64`、最小请求间隔 50ms、批超时 `BATCH_TIMEOUT_MS=45s`；不设单段条数上限，弹幕密集视频全量分批处理（成本 ≈ 段内条数/批大小 次 LLM 请求）。**弹幕处理上限**（`config.danmakuMaxTotal`，options 下拉 100~10万/无上限，默认 1万）：main-world 劫持 `/x/web-interface/(wbi/)?view`（view 或 view/detail，注意 `data.View.stat.danmaku` 嵌套结构）提取视频弹幕总量 → `KW_VIDEO_META` 上报 SW（per-tab）→ `KW_REWRITE_BATCH` 到达时总量超阈值整批放行原文（`skipped: true` 标记，弹幕打"跳"角标；弹幕量极大的视频通常引战少，跳过省成本）。**SSR 兜底数据源**（实测 2026-08）：B 站新版视频页不再请求 view 接口（信息内嵌 `window.__INITIAL_STATE__.videoData.stat.danmaku`），`startHijack` 时轮询读取（300ms 起 100ms 间隔、5s 超时）经 `KW_VIDEO_META` 上报，与 fetch 劫持路径双路并用以防阈值失效。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClauBloom/Kindly-Web](https://github.com/ClauBloom/Kindly-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
