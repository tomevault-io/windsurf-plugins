---
trigger: always_on
description: 归知是本地优先的 AI 个人知识库。本仓库是 v0.3 起的 Electron 技术栈实现，
---

# 归知 GuiZhi — 开发者与 AI 助手指南

归知是本地优先的 AI 个人知识库。本仓库是 v0.3 起的 Electron 技术栈实现，
应用骨架 fork 自 PromptHub v0.5.9（AGPL-3.0，见 NOTICE），业务域为归知自研。

## 技术栈

- Electron 33（main / preload / renderer 三进程）
- React 18 + TypeScript 5 + Vite 6
- Zustand 5（渲染进程状态）
- Tailwind CSS 3 + Liquid Glass 玻璃拟态令牌（`renderer/styles/globals.css`）
- node-sqlite3-wasm（主进程 SQLite，FTS5；选 wasm 是因为原生模块在 Windows ARM64 上构建不过）
- i18next（zh / en）
- pnpm monorepo：`apps/desktop` + `packages/{core,db,shared}`

## 架构约定

```text
apps/desktop/src/
├── main/        # Electron 主进程：窗口/托盘/快捷键、SQLite、IPC handlers、更新器
│   ├── database/  # @guizhi/db 的桌面包装（路径解析）
│   ├── ipc/       # IPC handlers（settings/security/image/ai + M1 起的知识域）
│   ├── services/  # ai-client、net-safety(SSRF)、network-proxy
│   └── settings/  # 主进程侧设置读取
├── preload/     # contextBridge 白名单：window.api（settings/ai/security）+ window.electron（壳能力）
└── renderer/    # React UI
    ├── components/{app,layout,settings,ui}/
    ├── stores/    # ui.store（AppModule 导航）、settings.store（外观/AI/同步配置）
    ├── services/  # ai.ts（AI HTTP 经主进程代理）
    └── i18n/      # zh.json / en.json
packages/
├── core/        # AI 配置文件服务（config/ai-models.json）、AI 客户端、运行时路径
├── db/          # SQLite 适配器、schema、初始化与进程锁
└── shared/      # 跨进程类型、IPC 频道常量、ai-protocol/network-proxy 工具
```

- 导航：无 react-router。`App.tsx` 的 `PageType`（home/settings）+
  `ui.store` 的 `AppModule`（library/ask/wiki/imports）双层切换。
- IPC：频道常量集中在 `packages/shared/constants/ipc-channels.ts`；
  main 注册于 `main/ipc/index.ts`；preload 白名单暴露。
- 数据：`%APPDATA%/GuiZhi/data/knowledge.db`。知识域表（knowledge_items、
  collections、tags、import_tasks、wiki_*、FTS5 虚表）在 M1 加入
  `packages/db/src/schema.ts`。旧 .NET 版的 `data/guizhi.db` 是只读迁移源。
- 主题：FONT_SIZES（3 档）/ MORANDI_THEMES（6 套）/ 背景图令牌在
  `stores/settings/settings-appearance.ts` 与 `styles/globals.css`。
  不要绕过语义令牌硬编码颜色。
- AI：Provider/模型/路由配置持久化在 `config/ai-models.json`
  （`packages/core/src/ai-config.ts`）；渲染进程经 `ai:httpRequest`/
  `ai:httpStream` IPC 走主进程发起请求（绕过 CORS、支持代理）。
 模型路由：mainText（问答/Wiki/视频总结）、fastText（摘要/标签）、
 visionText（OCR）、embedding（语义检索）、audioText（语音转写）。

## 里程碑路线（v0.3 重构）

v0.3.0-alpha.1（2026-07，内部预发布）：M0 骨架收缩 → M1 核心域
（DB schema + 三栏知识库 + FTS）→ M2 采集管线（导入队列 + 网页抓取 + 去重）→
M3 AI（摘要/标签/问答）→ M4 Wiki（AI 增量编译互链页面网络）→
M5 旧数据迁移（guizhi.db 一键迁入）→ M6 打包发布（CI 构建 + GitHub Release）。

M7 已完成：本地备份/恢复/定时备份 + Markdown 导出（`main/services/backup.ts`、
`export-markdown.ts`）、问答会话持久化（`ask_sessions` 表）、批量多选、
三模块侧栏面板、autoSave 接线、e2e 冒烟。
M8 已完成：embedding 语义检索——`knowledge_embeddings` 表存归一化 Float32 BLOB，
渲染进程分块嵌入（`renderer/services/knowledge-ai/semantic-*.ts`），
主进程余弦 top-k（`main/services/semantic.ts`），问答检索走 FTS+语义 RRF 融合
（`hybrid-search.ts`），未配置 embedding 模型时静默退化为纯 FTS。
M9 已完成：媒体采集——本地图片/音视频资产化导入（`import/media-files.ts`）、
图片 OCR（`knowledge-ai/ocr.ts`，visionText 路由）、音视频转写
（`media/transcribe.ts`，新增 audioText 路由）、yt-dlp 在线视频解析
（`import/video-url.ts`，B站/YouTube，未安装时降级）。抖音与小红书例外，
见下方「抖音不走 yt-dlp」「小红书不走 yt-dlp」。
M10 已完成：Wiki 关系图谱（`WikiGraphView.tsx`，react-force-graph-2d 懒加载）。

v0.4.0（2026-07，首个公开发布）在此之上重构了知识库界面：卡片 / 列表双视图、
表格列配置与分页（`library/ItemTableView.tsx`、`item-table-config.ts`）、
标签选择浮层（`TagPickerPopover.tsx`）、详情全屏弹窗（`ItemDetailModal.tsx`）。

v0.5.0（2026-07）是一轮集中整改：知识库列表改服务端分页（`page`/`pageSize`
进 store 直落 SQL）、AI 请求打通取消链路（`ai:httpCancel` + signal 贯穿到
`runScenarioChat`）、Wiki 编译不再整体覆盖未进上下文的页面并新增
`wiki_page_revisions` 快照、补上 `schema_migrations` 执行器与 `user_version`
版本戳、Electron 侧加 CSP 与 `will-navigate` 拦截、settings 双向字段白名单。

v0.6.0（2026-07）围绕抖音采集：抖音脱离 yt-dlp（见下），图文作品补成完整能力
（配图入资产库 + 逐图 OCR + `ImageLightbox` 查看器 + 正文拆成文案/图片/图中文字
三个标签，`shared/utils/image-note.ts`）。顺带修了两个影响全应用的渲染缺陷——
`.prose` 没恢复被 preflight 清掉的 `list-style`（列表没序号），以及 rehype-sanitize
与 react-markdown 的 `urlTransform` 双双拦掉 `local-image://`（正文图是破图）。

侧栏「平台」分区（知识库与标签之间）：按采集来源分组，抖音 / 哔哩哔哩 / 小红书 /
YouTube / V2EX 各一行，外加网页与本地文件两个兜底桶——少了后两个，通用抓取与本地
导入的条目在这个分区里一条都点不到。判定收敛在
`shared/utils/source-platforms.ts` 的 `resolveSourcePlatform`，采集落库
（`import-service.ts` 写 `source_records.platform`）与老库回填（迁移
`0009-source-platform`）共用它，且它内部直接复用连接器分流用的
`detectVideoPlatform` / `detectForumPlatform`：采集当时走了哪条抽取路径，事后回填
就归到哪个平台，两边算不出不同的结果（B 站专栏页不是视频页，两边一致地落进网页桶）。
该函数回传的是这两个检测函数的枚举值而非就地写死的字面量，新增平台时忘了加进
`SOURCE_PLATFORMS` 会直接编译不过。
`platform` 这一列建表时就在，但采集管线从来没写过，全库皆为 NULL；唯一写过的是旧版
.NET 迁移，落进去的是老应用自己的一套取值。回填因此是**全部重算**而不是「只补
NULL」：留着老取值会在分区里多出几个用户认不出来的分组，而这一列此前没有任何读取方，
重算不弄丢任何在用的数据。来源是 1:N（旧版迁移可能给同一条目带进多条记录），
过滤用 EXISTS、计数用 `COUNT(DISTINCT item_id)`，换成 JOIN 会让条目在列表里重复
出现、总数虚高。分区只列有条目的平台且按常量表的固定顺序排（不按数量），
从不用抖音的人不必盯着五个 0 找自己那一行，采集一条也不会让侧栏重排。
平台是派生分组、不可增删改名，所以行上没有「更多」菜单；它与知识库、标签同为互斥的
导航轴，四条轴的复位逻辑收在 store 的 `navigateTo` 里（此前三处各抄了一遍）。
表格视图另有一列「来源」（默认显示，紧挨「类型」）：光有筛选轴不够，在「全部」
视图里仍然分不出哪条来自哪儿。它需要 `platform` 进 `KnowledgeItemListEntry`，
由 list() 的相关子查询带出（与 get() 取 source_uri 同一形状，走 `idx_sources_item`）。
查表用 `getSourcePlatformMeta` 而不是直接索引：取值来自数据库，新版本写入的平台
在旧版本里查不到，`getItemTypeMeta` 当年就是为这个加的兜底——那一次抛异常的后果是
整个知识库列表白屏。
这两处的图标是各平台的真品牌 logo（`ui/PlatformLogos.tsx`），不是形态近似的通用
图标。此前抖音取音符、B 站取电视、小红书取书，得先读懂旁边的文字才对得上号，
等于没帮上忙，而这一列的全部用处就是不看文字也能扫出哪行是哪个平台。
路径取自 Simple Icons v16（CC0-1.0，商标归各平台，NOTICE 里有声明），**内联**而不
装 `@icons-pack/react-simple-icons`：只用得上五个 glyph，为此引一个 3400 图标的依赖
不划算，何况这里要按主题换色，包给的组件也帮不上忙。lucide 靠不住——v1 已经把
全部品牌图标删了（法务压力），官方迁移指南指向的正是 Simple Icons，我们那个
`YoutubeIcon` 是锁在 0.460 才还在。
抖音用 TikTok 的音符：字节这两个产品共用同一个符号，Simple Icons 也只收了一份

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Couleur-Share/GuiZhi](https://github.com/Couleur-Share/GuiZhi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
