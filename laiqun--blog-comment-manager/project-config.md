---
trigger: always_on
description: > 面向 AI 编码代理的项目说明。读完本文件即可上手修改本项目。
---

# AGENTS.md — 博客评论外链管理器

> 面向 AI 编码代理的项目说明。读完本文件即可上手修改本项目。

## 项目概述

这是一个 **Chrome MV3 浏览器扩展**（无构建步骤的纯 JavaScript），一款 SEO 外链建设辅助工具「博客评论外链管理器」。核心业务链路：

1. **收集**：输入同行站点域名，复用用户已打开的 Semrush 工具页（经 dash.3ue.co 共享面板），纯 DOM 抓取反向链接表格（`a[data-test-source-url]` 行，只保留「博客」标签来源），点页面「下一页」按钮自动翻页，随机间隔 3-9 秒。不构造、不重放任何接口请求。
2. **分析**：对收集到的外链逐条访问页面，**纯规则判定**（不经 AI）：要求登录 → 跳过；无评论表单 → 不入库；有评论表单 → 命中入资源库（有验证码标记为 captcha）。来源类型已由收集阶段的 Semrush「博客」标签保证。
3. **发布**：无后台任务队列，由侧边栏「助手」标签页驱动：资源库点 ↗（或助手页「换一个」）把**当前激活标签页**导航到资源页面 → 助手页顶部选定「当前任务」（目标地址/网站介绍/主关键词，可从模板选择/编辑）→ 人工逐步触发 AI 步骤（标题与摘要 / 识别表单 / 生成评论 / 填表）→ 检查后点 Submit / Skip；Submit 成功写 published 表防重复。

原始设计依据在 `docs/插件界面描述.md`（UI 复刻参考文档）。

## 技术栈与运行方式

- **纯 JavaScript ES Module**，无 package.json、无打包器、无 npm 依赖。唯一外部服务是 OpenRouter API。
- 入口清单：`extension/manifest.json`（MV3，`default_locale: zh_CN`，最低 Chrome 111）。
- 权限：`storage / tabs / scripting / alarms / favicon / sidePanel` + `host_permissions: <all_urls>`。
- 安装/调试：`chrome://extensions` 开开发者模式 → 「加载已解压的扩展程序」→ 选 `extension/` 目录。侧边栏页脚的 ↻ 按钮可热重载（`chrome.runtime.reload()`）。
- **无部署流程**：目前是本地开发者模式加载，未发布到 Chrome Web Store。

## 目录结构与模块划分

```
extension/
├── manifest.json            # MV3 清单
├── background/
│   ├── service-worker.js    # 消息路由（switch on msg.type）、状态快照广播、alarms 保活、断点续跑
│   ├── collect.js           # CollectController：DOM 抓取外链 → 翻页 → 「开始分析」逐条访问 + 规则判定入库（不经 AI）
│   └── publish.js           # PublishAssistant：助手页步骤对当前激活标签页执行（识别/生成/填表/提交），「换一个」挑未发布资源
├── content/                 # 由 background 用 scripting 注入，不走 manifest content_scripts
│   ├── analyzer.js          # window.__BCM_ANALYZE__：采集标题/正文/评论表单/评论区信息
│   └── publisher.js         # window.__BCM_PUB__：无 UI 页面操作 detect/fill/submit/cleanup/markForm/locateLink（IIFE，非 ESM）；全框架注入（评论框可能在 iframe 里），不再向页面注入任何浮层 DOM
├── lib/
│   ├── config.js            # ★ 所有「待联调」的选择器、URL 模板、默认值集中在这里，联调只改这个文件
│   ├── storage.js           # chrome.storage.local 内存镜像 + 日志操作 + 旧数据迁移（资源数据全在 IDB）
│   ├── idb.js               # IndexedDB 轻量封装（库 bcm-idb v3，stores: backlinks / analysis / published / templates）
│   ├── openrouter.js        # OpenRouter 客户端 + 四个 AI 角色（classify/formDetect/commentGen/discover）
│   ├── i18n.js              # 中/英字典（MESSAGES）+ applyI18n
│   └── util.js              # URL 处理、CSV（带 BOM）、tab 等待等纯函数
├── sidepanel/               # 侧边栏主界面（panel.html/css/js）：收集/助手/日志/资源库四 Tab，点工具栏图标打开；「助手」Tab 是发布主操作台
├── options/                 # 设置页：API Key / 四模型 / 发布身份 / 语言 / 标题与摘要语言 / AI 超时
└── _locales/                # 仅扩展名称与描述（zh_CN / en）
docs/                        # 设计文档（插件界面描述.md）
test/                        # Node 自带 node:test 单测（见下）
```

## 架构要点（改代码前必读）

- **持久化分两层**：
  - `chrome.storage.local`（key `bcm_store`）存小状态：settings / collectState / assistantTask / publishRuntime / lastPublish / logs。`lib/storage.js` 的 `state` 是它的内存镜像，**变更后必须 `save(...keys)`**；`save` 是合并写入（先 get 再展开），不要绕过它直接写 storage。旧版的任务队列（`tasks` / `activeTaskId`）已随「发布」Tab 移除，`load()` 里有一次性清理。
  - **IndexedDB（`bcm-idb` v3）是大数据表的唯一持久层**，不进 chrome.storage、不进内存态：`backlinks`（主键 `[targetDomain, url]`）、`analysis`（同主键；命中结论 ready/captcha 的记录即「可用资源」，含 `enabled` 启停标记，默认启用）、`published`（已发过的外链，主键 `[url, targetUrl]`，Submit 成功时写入——不要求页面在资源库中，手动打开的资源页同样落库；记录的 `targetDomain` 同行网站域名取助手页「定位同行网站」输入框的值，空则回落会话 refDomain；资源库「已发布」标记与「换一个」的筛选都按当前任务判定——published 表中存在 `[url, 当前任务目标地址]` 记录才算已发布，目标地址缺省回落到设置里的身份网址）、`templates`（「当前任务」模板，主键 `name`，助手页顶部可存/选/删，同名覆盖即编辑）。绑定页面时若同 `[url, targetUrl]` 已发布过会在助手页状态行提示。资源库 UI 直读 `analysis` 表，行内删除按钮（`deleteLibraryResource`）同时删 analysis 与 backlinks 记录（只删 analysis 的话该外链会回到「队列中」，下次分析重跑回库），published 发布历史保留；v1 的 `resources` 表已废弃（升级时已发布记录自动迁入 `published` 后删表），旧版 chrome.storage 双写的 backlinks 由 `storage.js` 里的一次性迁移函数搬到 IDB。
- **MV3 service worker 随时被回收**：所有状态落盘后才能丢；`chrome.alarms`（`bcm-tick`，30 秒）负责唤醒续跑收集队列，并把回收途中卡住的 `stopping` 状态收尾为 `idle`；`onInstalled`/`onStartup` 做断点续跑。发布无后台循环，回收不影响——publishRuntime（助手会话）落盘后重开面板照样续上。
- **UI ↔ 后台通信**：sidepanel 用 `sendMessage` RPC（`getSnapshot`、`startCollect`、`setAssistantTask`、`setSettings` 等；助手页交互 `pub:step`/`pub:decision`/`pickNextResource` 也来自 sidepanel 的「助手」Tab）；**「定位同行网站」例外**：它是只读视觉辅助，由面板直接向当前标签页主框架注入 publisher.js 执行 `locateLink`（不经后台，避免 SW 回收丢响应把按钮卡死在「正在定位」，带 `LIMITS.locateTimeoutMs` 超时兜底；域名为空时面板直查 IDB analysis 表按 URL 取同行域名，与后台 bind 口径一致）。后台状态推送走 `chrome.runtime.sendMessage` 单向广播（`stateChanged` 快照），**不用 port 长连接**——无连接状态，SW 回收重启后新实例照样送达，面板关着时静默丢弃、打开时由 `init()` 的 `getSnapshot` 追平。**设置的唯一写入口是 `setSettings` 消息**，options 页也不得直接写 storage（会被后台内存态覆盖）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [laiqun/blog_comment_manager](https://github.com/laiqun/blog_comment_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
