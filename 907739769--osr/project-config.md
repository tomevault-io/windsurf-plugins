---
trigger: always_on
description: OSR (OpenList STRM Relay) 核心业务层，负责 STRM 生成、文件夹同步、Telegram Bot、企业微信、文件重命名、任务调度、第三方回调、PT 订阅管理、重命名一致性检查等业务逻辑。21 个子包按功能域划分。
---

# OpenList-strm 核心业务模块知识库

## OVERVIEW
OSR (OpenList STRM Relay) 核心业务层，负责 STRM 生成、文件夹同步、Telegram Bot、企业微信、文件重命名、任务调度、第三方回调、PT 订阅管理、重命名一致性检查等业务逻辑。21 个子包按功能域划分。

## STRUCTURE
```
com/osr/openliststrm/
├── api/              # OpenList API 客户端 (网盘操作封装)
├── config/           # 业务配置类 (OpenlistConfig 等)
├── controller/       # REST API 端点 (STRM/同步/任务配置/回调)
├── controller/api/   # 第三方开放 API (qb/callback、企微回调 等)
├── dashboard/        # 首页概览统计
├── enums/            # 业务枚举 (任务状态、类型等)
├── helper/           # 辅助工具 (文件操作、路径处理)
├── mcp/              # MCP 服务端 (端点 /mcp，令牌鉴权 + 工具集，供本地 AI 助理连接)
│   └── tool/         # 五组工具：订阅 / 追剧 / 下载 / 任务 / 运维
├── monitor/          # 任务监控与状态追踪 (MediaRenameProcessor 等)
├── mybatisplus/      # ★ MP 风格数据层 (domain/mapper/service)
├── notify/           # 通知渠道抽象 (INotifier + TG/Webhook/企微实现)
├── openai/           # AI 相关功能 (OpenAIClient)
├── orphan/           # 重命名一致性检查 (孤儿扫描/清理/忽略)
├── pt/               # PT 订阅管理 (downloader/indexer/subscription/media server)
│   ├── autoadd/      # 热门自动订阅 (TMDb 榜单 / RSSHub 豆瓣榜单 → 过滤 → 建订阅)
│   ├── clean/        # 自动删种 (体积区间 + 做种时长分级，辅种整组同删)
│   ├── health/       # 缺集体检 (逾期未入库的分档诊断 + 每日聚合提醒)
│   └── transfer/     # 转移做种 (qB → TR 搬种，IYUU「转移」的自建实现)
├── rename/           # 影视文件重命名 (MediaParser/TitleProcessor/PebbleRenderer)
│   └── cleanup/      # 产物清理 (ArtifactPaths 纯逻辑 + RenameCleanupService 实际 I/O)
├── req/              # 请求 DTO
├── scrape/           # 文件刮削 (ScrapeService 等)
├── service/          # 业务服务层 (IStrmService/ICopyService 等)
├── task/             # 定时任务 + 手动任务执行 (OpenListStrmTask)
├── tg/               # Telegram Bot (StrmBot/TgBotRegister/ResponseHandler)
├── tmdb/             # TMDB 电影/剧集信息查询 (TMDbClient)
├── upload/           # 文件上传处理
└── wecom/            # 企业微信自建应用 (API客户端/回调加解密/指令交互)
```

## WHERE TO LOOK
| 任务 | 位置 | 备注 |
|------|------|------|
| STRM 生成逻辑 | `task/` + `service/` | OpenListStrmTask, IStrmService |
| 文件夹同步 | `service/` | ICopyService, 增量/全量同步 |
| 复制任务监控 | `helper/` | AsynHelper（内存监控链）+ CopyRecoveryTask（重启兜底）+ CopyMonitorRegistry（心跳分工） |
| Telegram Bot | `tg/` | StrmBot (7 个指令), TgBotRegister |
| 企业微信 | `wecom/` + `controller/api/WeComCallbackController` | 收发消息、订阅指令、成员绑定 |
| 通知渠道 | `notify/` | INotifier / NotifierManager / NotifyTarget |
| TMDB 查询 | `tmdb/` | TMDbClient, 元数据获取/增强 |
| 文件重命名 | `rename/` | MediaParser + OpenAI + Pebble 模板 |
| 重命名一致性检查 | `orphan/` | RenameOrphanScanServiceImpl（双向扫描）, OrphanReconciler, OrphanReason |
| 重命名产物清理 | `rename/cleanup/` | RenameCleanupService（purge/purgeRelocated/回收空目录）, ArtifactPaths |
| PT 订阅管理 | `pt/` | Downloader/Indexer/Subscription/MediaServer |
| PT 自动删种 | `pt/clean/` | TorrentCleanService（判定+执行）, TorrentCleanTask（默认每 60 分钟） |
| PT 缺集体检 | `pt/health/` | EpisodeHealthService（纯查询分档+诊断）, EpisodeHealthNotifyService/Task（每 24 小时） |
| 文件刮削 | `scrape/` | ScrapeService, TMDb 刮削/文件删除 |
| 任务监控 | `monitor/` | MediaRenameProcessor 等处理器 |
| 任务配置 | `mybatisplus/domain/` + `controller/` | 所有 *Plus 实体 |
| MCP 服务端 | `mcp/` | McpServerConfig（装配）/ McpAuthFilter（令牌）/ McpCallContext（身份绑定）/ McpToolRegistry（横切）/ tool/（工具声明） |
| 第三方回调 | `controller/api/` | QB 下载完成通知等开放 API |
| MP Mapper | `mybatisplus/mapper/` | BaseMapper 接口 |
| MP Service | `mybatisplus/service/` | IService 接口 + Impl |

## CONVENTIONS
- **按功能域分包**，非按层分包 (tg/, tmdb/, rename/, orphan/, pt/ 各自独立)
- **数据层**: 使用 MyBatis-Plus (BaseMapper + IService +ServiceImpl)，XML Mapper 在 `resources/mapper/mybatisplus/`
- **Controller 只负责** 参数接收、调用 Service、返回响应，不写业务逻辑
- **枚举优先**: 任务状态、类型等使用 enum，不用魔法数字
- **FastJSON2**: 所有 JSON 序列化/反序列化统一使用 FastJSON2
- **异步任务**: 使用虚拟线程 (Java 25 preview) 处理并发 IO
- **孤儿判定**: `orphan/OrphanReconciler` 纯逻辑无 I/O，方便单测覆盖；`RenameOrphanScanServiceImpl` 负责实际 I/O
- **「删产物」与「删记录」是两件不同的事，任何入口都不许合并成一个按钮**。`rename_detail` 同时是三处判据的事实来源，只删数据库行是「失忆」操作：①孤儿正向扫描以它为遍历起点，删了就再也发现不了那些文件；②`ScrapeService` 的 `hasSiblingInSameSeason/Show` 靠它计数，行少了会让别的记录删刮削时误删还在用的 `tvshow.nfo`/`season.nfo`/剧集图；③`MediaRenameProcessor#processOnce` 的 `processedKeys` 也来自它，删了记录后手动执行任务会把源文件当成没处理过、重新复制一份出来，用户会看到「明明删了怎么又冒出来」。删产物走 `rename/cleanup/RenameCleanupService#purge`，前端确认框必须把「只删记录」的这三条后果写出来
- **清理顺序是硬要求：先删文件、后删记录**。删完记录就没有 `new_path`/`new_name` 可用了，而且兄弟判定要靠这些行还在才算得对。**批量清理必须把整批 id 一次性传给 `ScrapeService.DeleteOptions#excludeDetailIds`**：逐条调用时兄弟计数是边删边变的，不排除整批的话前几条会认为"还有兄弟"而跳过共享元数据，只靠最后一条兜底——中途任何一条失败就留下没人认领的 `tvshow.nfo`
- **预览与执行必须走同一份判定**：`ScrapeService#resolveScrapeFiles` 解析路径、`deleteScrapeFiles` 遍历它去删，`RenameCleanupService#preview` 也调它。分叉一次就会出现"确认框里列的"和"真正删掉的"对不上，那比不给预览更糟
- **空目录回收必须锚定 `电影`/`电视剧` 那一层，找不到锚点就一个都不删**（`ArtifactPaths#mediaRootOf`）。这两个顶层目录名是 `MediaRenameProcessor#buildDestPath` 硬编码产出的，因此一定出现在每条产物路径里，可以拿来当可证明的下界；删到它或它的祖先，Emby/Jellyfin 的媒体库根目录会直接失效。回收只删 `Files.newDirectoryStream` 为空的目录，**绝不用递归删除**
- **重命名换位（改标题重试导致落到另一部剧）必须调 `RenameCleanupService#purgeRelocated`，且必须在改写记录的 `new_path`/`new_name` 之前调**。旧实现只删旧主文件，旧目录里的单集 NFO、`season.nfo`、`tvshow.nfo` 和七张剧集图原样留下，Emby 会扫出一个只有元数据没有视频的鬼剧集。**`keepDir`/`keepShowRoot` 不能省**：此刻记录的 `new_path` 还是旧值、兄弟判定又会排除自己，不传的话"同剧还有没有别的记录"会答成"没有"，把新位置正要用的 `tvshow.nfo` 一起删掉

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [907739769/OSR](https://github.com/907739769/OSR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
