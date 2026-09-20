---
trigger: always_on
description: 面向 AI 编码助手与新加入的开发者。阅读本文即可掌握项目定位、目录结构、关键约定与雷区。
---

# AGENTS.md — 115-Station 项目总览

面向 AI 编码助手与新加入的开发者。阅读本文即可掌握项目定位、目录结构、关键约定与雷区。
用户向文档见 [README.md](README.md) 与 [USAGE.md](USAGE.md)。
**动 115 接口前先看 `docs/115-station-notes/REFERENCES.md`**（仓库内，但 `/docs/` 已 gitignore，不会提交）—— 外部参考项目清单与已验证的接口事实。

---

## 1. 这是什么

**115-Station** 是一个 Go 单体服务：把 115 网盘的媒体库映射成本地 STRM 文件供 Emby/Jellyfin 刮削入库，
播放时以 302 重定向让播放器直连 115（服务器不转发流量），并在同一个 Web 后台里完成
同步 / 整理 / 洗版 / 重命名 / 元数据回传 / 消息机器人的闭环。

**本仓库是 [DaisyYijin/STRMhub](https://github.com/DaisyYijin/STRMhub) 的二次开发版本。**
功能性改动包括：**移除 123 云盘、夸克网盘、阿里云盘支持**，只保留 115 链路；**移除 MetaTube、成人影片番号识别及其专属分类、刮削、重命名和通知功能**（AV1、AVC 等普通视频编码支持保留）；**移除播放账号功能**（小号播放 / 多端播放 / 账号池），播放统一走主号直链。

### ⚠️ 许可证约束（改动前必读）

上游仓库**没有 LICENSE 文件**，按 GitHub ToS 与著作权法通行规则默认为「保留所有权利」。因此：

- **不要**给本仓库添加 LICENSE 文件、SPDX 头或任何开源授权声明；
- **不要**在文档里声称本项目是 MIT / Apache / GPL 等许可；
- **不要**建议发布预构建二进制或公共镜像；
- README 的「许可证与再分发声明」章节是刻意这样写的，修改前先与维护者确认。

### 不入库的维护者文档

`REFERENCES.md`（外部参考项目与 115 接口事实）和 `INCR-SYNC-UPGRADE.md`
（增量同步改造记录）放在 `docs/115-station-notes/`，整个 `/docs/` 目录被 `.gitignore`
排除，刻意不提交：它们含本机绝对路径、逆向结论与内部开发流水，对使用者无意义。
本文里引用到它们的地方都指的是那个目录下的副本。

**怎么读**：直接读工作区里的文件，不需要联网——

```bash
cat docs/115-station-notes/REFERENCES.md        # 外部参考项目清单：哪个项目解决哪类问题
cat docs/115-station-notes/INCR-SYNC-UPGRADE.md # 增量同步改造全过程
```

`REFERENCES.md` 顶部写着参考项目在本机的位置（目前是 `D:\Code\115strm\` 下的
`p115client-main` / `115driver-main` 等），115 接口的字段含义、错误码、调用形态到那里
`grep` 最快。**读它们、不要抄它们**——理由见上面的许可证约束，用到某个做法时在代码注释里写明出处。
如果那些副本不在本机，按 `REFERENCES.md` 里的项目名去上游仓库看同名文件。

但**第三方组件的许可证义务是独立的**，不受上述限制，也不要删：

- [`THIRD-PARTY-NOTICES.md`](THIRD-PARTY-NOTICES.md) 与 [`licenses/`](licenses/) 目录
  是嵌入字体（OFL 1.1）、CodeMirror / Mermaid（MIT，压缩时许可头被剥掉）、
  ECharts（Apache-2.0）要求随附的版权声明与许可证副本。新增任何 vendor 进来的
  第三方文件时，同步在这两处登记。
- `.github/workflows/docker.yml` 的 `PUBLISH` 开关控制是否推 ghcr，现在是 `true`：
  push master 与 `v*` tag 都会产出 `ghcr.io/pancras-loe/115-station`（amd64 + arm64）。
  README「许可证与再分发声明」一节已说明镜像同样适用上游未授权的状况。

---

## 2. 技术栈

| 层 | 选型 |
|---|---|
| 语言 | Go 1.25（module 名与二进制名都是 `115-station`） |
| Web 框架 | Gin（`gin.New()`，**不是** `gin.Default()`） |
| ORM / DB | GORM + SQLite（纯 Go 驱动 `glebarez/sqlite`，`CGO_ENABLED=0`） |
| 认证 | JWT（`golang-jwt/v5`）+ 环境变量管理员账号 |
| 115 客户端 | `SheltonZhu/115driver`（Cookie 通道）+ 自研 OpenAPI 客户端 |
| 前端（现役） | Vue 3 + TypeScript + Vite + Naive UI（`webui/`），详见 [webui/README.md](webui/README.md) |
| 前端（已停用·保留备查） | 原生 HTML/CSS/JS（`web/`），`WEBUI=legacy` 可切回 |
| 外部依赖 | ffmpeg/ffprobe（镜像内）、可选 Emby/Jellyfin |

---

## 3. 目录结构

```
.
├── main.go                     # 启动、日志轮转、Gin 装配、TLS 明文自动跳转、优雅退出
├── internal/
│   ├── api/                    # 全部业务逻辑（~33k 行，41 个测试文件）
│   ├── config/                 # 环境变量配置、配置文件读写、TLS 自签证书
│   └── model/                  # GORM 实体与建表/默认数据初始化
├── webui/                      # 管理后台前端·现役（Vue3 + TS + Vite + Naive UI）
├── web/                        # 管理后台前端·旧版，已停用，保留供对照实现（WEBUI=legacy 可切回）
├── wiki/index.html             # 完整版使用 Wiki（单文件）
├── .github/workflows/docker.yml# CI：测试门禁 → 多架构镜像构建
├── Dockerfile                  # 多阶段交叉编译 → alpine + ffmpeg
└── docker-compose.yml
```

### `internal/api/` 模块地图

文件很多但命名规律清晰，按职责分组：

| 分组 | 文件 | 说明 |
|---|---|---|
| **路由与认证** | `routes.go` | `Handler{DB, Config}` + 全部路由注册 + 登录防爆破 + 备份/日志接口 |
| **115 基础设施** | `115.go` `115crypto.go` `http115.go` `open115.go` `files115.go` `ops115.go` `dir.go` `ratelimit.go` | Cookie 通道、ECC 加密、专用 HTTP 客户端（处理缺 SAN 证书）、OpenAPI（PKCE + 刷新）、文件/目录操作、**全局节流器** |
| **同步** | `full115.go` `incr115.go` `incrdeps.go` `life115.go` `panpath.go` `incrstatus.go` `share.go` `upload115.go` `orphan115.go` `cron.go` `suppress.go` | 全量 / 增量（生活事件，只管外部变更）/ 分享转存 / 上传与监控回传 / 失效 STRM 检测 / 调度 / 整理自产事件抑制。**增量这条链分了四层**：`life115.go` 拉事件（游标 + 405 降级 + 开关门禁）、`panpath.go` 解析 cid→路径（祖先链 + `PathCache` 缓存）、`incr115.go` 消费事件落盘、`incrstatus.go` 对外报状态；`incrdeps.go` 是它们之间的注入接口，主流程靠它才能整体单测 |
| **整理流水线** | `organize.go` `org115.go` `orgstrm.go` `orgrecord.go` `emptydir.go` `resource.go` `rename.go` `wash.go` `enrich.go` `scrape.go` `tmdb.go` `airecognize.go` | 识别 → 分类 → 洗版 → 重命名 → 搬移 → **写 STRM / 下附属 → 刮削 → 刷 Emby**（一条龙，见 §6.8）；`resource.go` 是文件名结构化解析的核心，`orgstrm.go` 是落盘出口，`orgrecord.go` 是整理记录与「重新整理」，`airecognize.go` 是 TMDB 全部搜索策略都落空后的 AI 兜底（OpenAI 协议，界面「AI 增强识别」） |
| **播放链路** | `proxy.go` `offlineplay.go` `embyproxy.go` `embylibrary.go` `emby_notify.go` | 302 代理、边下边播、Emby 反代与建库 |
| **资源站** | `guanying.go` `pansou.go` `mukaku.go` `re0.go` `tgsearch.go` `tgsub.go` | 四个转存页签 + TG 抓取与关键词订阅 |
| **通知** | `notify.go` `notify_extra.go` `medianotify.go` `wecombot*.go` `wecomcrypto.go` | 企微双向机器人（AES 验签）、TG / 飞书 / OneBot / QQ 官方、入库通知防抖聚合 |
| **其他** | `dashboard.go` `offline.go` `dllink.go` `covergen.go` `checkin115.go` | 仪表盘、离线下载、**下载记录**、媒体库封面生成、115 签到 |

### 数据模型（`internal/model/model.go`）

19 个实体，关键的几个：`Storage`（网盘账号凭据）、`StrmFile`、`SyncTask` / `SyncEvent` / `SyncedFile`（同步台账）、
`CategoryRule` / `WashRule` / `ScrapeRule`（YAML 规则）、`Setting`（键值配置）、`MediaEnrich`（ffprobe 结果）、
`MediaLibrary`、`UploadMark`、`OrganizeRecord`（整理流水，一次动作一条）、`EventSuppress`（整理自产事件抑制）、
`PathCache`（115 目录 id → 网盘绝对路径）、`DownloadLink`（下载记录，见下）。

> `MediaLibrary` 与 `OrganizeRecord` 不是一回事：前者「一部影视一条」（去重 upsert，仪表盘用），
> 后者「一次整理动作一条」且失败与未识别同样留痕（记录页与「重新整理」用）。

> `DownloadLink`（`dllink.go`）是下载记录：磁力/ed2k/HTTP 离线与 115 分享转存提交时落一行，
> 内容整理入库后由 `orgSink.note` 里的 `dlLinkClaim` 把识别结果（片名 / 年份 / TMDB id /

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pancras-loe/115-station](https://github.com/pancras-loe/115-station) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
