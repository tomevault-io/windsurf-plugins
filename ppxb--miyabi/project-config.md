---
trigger: always_on
description: 接入 115 网盘的 JAV 媒体库管理工具。单二进制部署，Go 后端嵌入 React 前端。
---

# Miyabi

接入 115 网盘的 JAV 媒体库管理工具。单二进制部署，Go 后端嵌入 React 前端。

## 核心原则

- 单用户自用工具，不做多租户、不做权限系统。
- 代码简洁优先：不写防御性兜底，错误直接向上返回，由 API 层统一转成响应。
- 成熟库能用就用，不自己造轮子。
- 网盘目前只有 115，包名直接叫 `pan`，不带 115 前缀。
- 115 是唯一媒体文件来源；JavDB 是唯一发现、元数据和磁力来源，不接入 DMM、JavBus 或其他刮削源。
- JavDB 接入参考 `javdb-cli` 的实现与测试，但 Miyabi 本地实现所需协议，不直接依赖其 SDK，也不以子进程调用 CLI。
- JavDB 的剧情字段覆盖率和准确性不足，项目不采集、不存储、不展示 `plot`，也不把标题或其他字段伪装成剧情。
- 每层职责单一，禁止跨层调用（handler 不碰 ent；`javdb`、`pan` 不碰数据库）。

## 技术栈

| 层 | 选型 | 理由 |
|---|---|---|
| HTTP | `github.com/gin-gonic/gin` | 参数绑定、校验、JSON 响应、文件上传、SSE 开箱即用 |
| ORM | `entgo.io/ent` | schema 即代码，查询类型安全，多对多关系清晰 |
| 数据库 | SQLite (`modernc.org/sqlite`) | 纯 Go 无 CGO，开 WAL 模式 |
| 任务队列 | 自维护 goroutine 池 + ent 任务表 | 单机足够，重启可恢复 |
| 实时推送 | SSE | 单向进度推送，比 WebSocket 简单 |
| 通用 HTTP 客户端 | `github.com/go-resty/resty/v2` | 115、图片下载等普通 HTTP 请求 |
| JavDB Transport | `github.com/bogdanfinn/tls-client` | App API 已验证使用 Chrome TLS 指纹；只在 `internal/javdb` 内使用 |
| 图片处理 | `github.com/disintegration/imaging` | 海报裁剪、缩略图 |
| 限速 | `golang.org/x/time/rate` | 115 与 JavDB 分别限速 |
| 配置 | `github.com/knadh/koanf/v2` | 启动参数：监听地址、数据目录、日志级别、代理。文件 + 环境变量覆盖。服务端运行时设置存 Setting 表；NSFW 由 Zustand persist 存 localStorage，主题由 next-themes 管理 |
| 日志 | `log/slog` | 标准库 |
| 前端 | Vite 8 + React 19 + TypeScript | |
| 前端状态 | TanStack Query + Zustand | 服务端状态与 UI 状态分离 |
| 前端 UI | shadcn/ui + Tailwind CSS 4 | 组件可拷贝可改 |
| 播放器 | Artplayer + hls.js | m3u8 与直链都支持 |
| 前端路由 | TanStack Router | 类型安全 |
| 前端 lint/格式化 | oxlint + oxfmt | 不用 ESLint/Prettier |

## 目录结构

```
miyabi/
├── cmd/miyabi/main.go          入口：加载配置、初始化依赖、启动 HTTP
├── internal/
│   ├── api/                    gin 路由与 handler，只做参数绑定与响应
│   │   ├── router.go
│   │   ├── movie.go
│   │   ├── discover.go
│   │   ├── task.go
│   │   ├── pan.go
│   │   ├── play.go
│   │   ├── setting.go
│   │   └── sse.go
│   ├── service/                业务编排，唯一允许同时调用 ent / pan / javdb 的层
│   │   ├── library.go          扫描入库、文件与影片关联
│   │   ├── discover.go         JavDB 发现、搜索、分类与本地状态投影
│   │   ├── scrape.go           JavDB 元数据入库、封面下载与 NFO 写入
│   │   ├── offline.go          JavDB 磁力提交、115 离线任务轮询
│   │   └── play.go             获取播放地址
│   ├── ent/                    ent 生成代码（schema/ 手写，generate.go 触发生成）
│   │   ├── generate.go
│   │   └── schema/
│   │       ├── mixin.go        created_at / updated_at
│   │       ├── movie.go
│   │       ├── actor.go
│   │       ├── tag.go
│   │       ├── file.go
│   │       ├── task.go
│   │       └── setting.go
│   ├── pan/                    115 客户端，对外只暴露 Client
│   │   ├── client.go           鉴权、限速、通用请求
│   │   ├── auth.go             开放平台 OAuth / 设备码
│   │   ├── file.go             列目录、搜索、移动、重命名、上传
│   │   ├── offline.go          离线下载增删查
│   │   └── play.go             直链与 m3u8
│   ├── javdb/                  JavDB App API，本地最小协议实现
│   │   ├── client.go           公共参数、响应 envelope、错误处理与限速
│   │   ├── transport.go        tls-client、超时、代理与 context
│   │   ├── signature.go        jdsignature
│   │   ├── route.go            startup、动态域名解密、选线与故障重选
│   │   ├── wire.go             私有 API JSON wire 类型
│   │   ├── model.go            对 service 暴露的强类型模型
│   │   ├── movie.go            搜索、发现、影片详情
│   │   ├── entity.go           演员、系列、厂牌、导演
│   │   ├── tag.go              标签 taxonomy
│   │   └── magnet.go           磁力列表与排序
│   ├── nfo/                    Kodi 格式 .nfo 读写
│   ├── codeid/                 番号识别与规范化，纯函数，表驱动测试
│   ├── image/                  封面存储路径、裁剪、缩略图
│   ├── worker/                 任务池：取任务、执行、写进度、广播事件
│   └── config/
├── web/                        React 项目
│   ├── src/
│   │   ├── routes/             TanStack Router 文件路由
│   │   ├── api/                fetch 封装 + TanStack Query hooks
│   │   ├── components/
│   │   └── stores/
│   └── dist/                   构建产物，由 Go embed
├── embed.go                    //go:embed web/dist
├── Makefile
└── AGENTS.md
```

## 数据持久化原则

115 是唯一媒体文件事实来源，JavDB 是外部目录与元数据来源。本地只有索引和缓存，媒体库可从 115 中的文件和 NFO 完整重建。

- **SQLite**：索引。影片、演员、文件关系、任务状态。
- **本地图片目录**：缓存。封面、海报、缩略图，用于列表页快速渲染。直接从 115 读图需要换取时效直链且受限速，不适合高频小图请求。
- **115 影片目录**：元数据完成后写入 `<code>.nfo`、`poster.jpg`、`fanart.jpg`，与视频同目录。NFO 不写剧情。目录结构兼容 Emby/Jellyfin。
- **JavDB 缓存**：不做全量镜像。taxonomy、发现列表和详情只做有 TTL 的按需缓存；缓存失效不影响已有媒体库。
- **发现与媒体库分离**：发现结果不写入 Movie。只有 115 中实际出现视频文件并完成扫描后，才创建或关联 Movie。
- **重建**：扫描时若目录已有 `.nfo`，直接解析入库并下载图片到缓存，不再请求 JavDB。换机器或删库后重新扫描即可恢复。

## 数据模型

- **Movie**：`code`（唯一，规范化番号）、`javdb_id`（可空唯一）、`title`、`release_date`、`duration`（分钟）、`director_id/name`、`maker_id/name`、`series_id/name`、`rating`、`cover`、`poster`、`fanarts`（JSON）、`scrape_status`（pending/done/failed）。没有 `plot` 字段。
- **Actor**：`javdb_id` 唯一、`name`、`name_zht`、`gender`、`avatar`，与 Movie 多对多。JavDB 的演员数组包含男性演员，因此不使用 Actress 模型；UI 可默认只展示女性演员。年龄由生日计算，不持久化。
- **Tag**：`javdb_id` 唯一、`name`、`name_zht`、`category_id`，与 Movie 多对多。标签名称不作为唯一键。
- **File**：`file_id`（唯一）、`pick_code`、`sha1`、`name`、`size`、`parent_id`，可选关联 Movie。
- **Task**：`type`、`status`（queued/running/done/failed）、`payload`（JSON）、`progress`、`error`、`created_at`、`updated_at`。
- **Setting**：`key` 唯一、`value` JSON。

## JavDB 协议约定

JavDB 当前没有官方公开 API。Miyabi 使用经 `javdb-cli` 验证的 Android App 1.9.28 私有 JSON API，但只实现产品需要的匿名只读子集。私有协议没有稳定性保证，wire 类型、签名、设备参数和线路细节必须封装在 `internal/javdb`，不能泄漏到 handler、ent schema 或前端；`javdb_id` 等业务需要的稳定来源标识除外。

### 已验证接口

- 搜索：`GET /api/v2/search`
- 发现与分类浏览：`GET /api/v1/movies/tags`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ppxb/miyabi](https://github.com/ppxb/miyabi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
