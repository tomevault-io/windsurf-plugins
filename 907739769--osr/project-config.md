---
trigger: always_on
description: 影视 STRM 管理系统。Java 25 (Spring Boot 4.0.6) + Vue 3 + Vuetify 3，Docker 双容器部署。核心功能：STRM 文件生成、文件夹同步、Telegram Bot 控制、TMDb 刮削/重命名、第三方回调自动化。
---

# OSR (OpenList STRM Relay) 项目知识库

## OVERVIEW
影视 STRM 管理系统。Java 25 (Spring Boot 4.0.6) + Vue 3 + Vuetify 3，Docker 双容器部署。核心功能：STRM 文件生成、文件夹同步、Telegram Bot 控制、TMDb 刮削/重命名、第三方回调自动化。

> 本文件是本项目唯一的 AI 知识库，Claude Code 与 opencode 共用。根目录 `CLAUDE.md` 仅做引用，改动请直接改本文件。

## STRUCTURE
```
├── osr-admin/          # 启动模块 (Spring Boot main)，端口 6895
├── osr-common/         # 通用工具 (annotation, utils, exception, mybatisplus)
├── osr-framework/      # 框架配置 (security, config, websocket)
├── osr-system/         # 标准系统管理模块 (user/role/menu/config domain)
├── osr-quartz/         # 定时任务 (job scheduler)
├── osr-openliststrm/   # ★ 核心业务，新功能几乎都写在这里 (21个子包，见下)
├── osr-web/         # Vue 3 前端 (Vite + Pinia + Vuetify 3 + PWA)
├── Dockerfile.backend    # Java 25 JRE + --enable-preview
├── Dockerfile.frontend   # Node 20 build → Nginx Alpine
├── docker-compose.yml    # MySQL 8.0 + backend + frontend
└── nginx.conf            # SPA + API proxy + WebSocket proxy
```

`osr-openliststrm` 按功能域分包（21 个）：
`api/ config/ controller/ dashboard/ enums/ helper/ monitor/ mybatisplus/ notify/ openai/ orphan/ pt/ rename/ req/ scrape/ service/ task/ tg/ tmdb/ upload/ wecom/`

## WHERE TO LOOK
| 任务 | 位置 | 备注 |
|------|------|------|
| STRM 生成 | `osr-openliststrm/src/main/java/com/osr/openliststrm/` | task/, helper/, tmdb/, rename/ |
| 文件夹同步 | `osr-openliststrm/src/main/java/com/osr/openliststrm/` | api/, upload/, service/ |
| Telegram Bot | `osr-openliststrm/src/main/java/com/osr/openliststrm/tg/` | bot commands & handlers |
| 企业微信 | `osr-openliststrm/src/main/java/com/osr/openliststrm/wecom/` | 自建应用 API、回调加解密、订阅指令交互 |
| 通知渠道 | `osr-openliststrm/src/main/java/com/osr/openliststrm/notify/` | INotifier 抽象 + TG/Webhook/企微/Bark/Gotify 五个实现；路由由 `notify_route` 表决定 |
| 刮削 | `osr-openliststrm/src/main/java/com/osr/openliststrm/scrape/` + `tmdb/` | TMDb 刮削、文件删除 |
| 定时任务 | `osr-openliststrm/src/main/java/com/osr/openliststrm/task/` + `osr-quartz/` | 自定义 task + job |
| 重命名一致性检查 | `osr-openliststrm/src/main/java/com/osr/openliststrm/orphan/` | 双向孤儿扫描、清理、忽略 |
| 重命名产物清理 | `osr-openliststrm/src/main/java/com/osr/openliststrm/rename/cleanup/` | 删主文件+刮削+回收空目录，重命名换位时清旧位置 |
| PT 订阅管理 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/` | downloader/indexer/subscription/media server |
| PT 自动删种 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/clean/` | 按体积区间+做种时长分级删种，辅种整组同删 |
| PT 转移做种 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/transfer/` | 把 qB 上做够时长的种子搬到 TR 继续做种（IYUU「转移」的自建实现，不含辅种） |
| PT 热门自动订阅 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/autoadd/` | TMDb 榜单 + RSSHub 豆瓣榜单，按规则过滤后自动建订阅 |
| 追剧日历 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/calendar/` | 播出日期同步 + 按日期区间查排播 |
| 缺集体检 | `osr-openliststrm/src/main/java/com/osr/openliststrm/pt/health/` | 逾期未入库的分档诊断 + 每日聚合提醒 |
| 安全/认证 | `osr-framework/src/main/java/com/osr/framework/security/` | Spring Security + JWT（无 Shiro，早期文档写的 shiro/ 目录并不存在） |
| 登录防爆破 | `osr-framework/src/main/java/com/osr/framework/security/LoginAttemptService.java` | 账号桶 + IP 桶双计数，超阈值临时锁定 |
| 健康检查 | `osr-admin/src/main/java/com/osr/web/controller/api/HealthApiController.java` | `/api/health`，匿名、只探数据库，供 Docker healthcheck 用 |
| STRM 任务级覆盖 | `osr-openliststrm/src/main/java/com/osr/openliststrm/service/StrmSettingsFactory.java` | 全局配置 + `openlist_strm_task.strm_override` JSON 合并 |
| 第三方回调 | `osr-openliststrm/src/main/java/com/osr/openliststrm/controller/` | 开放 API 端点 |
| 前端页面 | `osr-web/src/views/` + `views-mobile/` | PC + 移动端 |
| 前端 API 层 | `osr-web/src/api/` | axios 封装 + 模块 API |
| 前端路由 | `osr-web/src/router/index.ts` | 动态路由 |
| 前端状态 | `osr-web/src/stores/` | Pinia (app, user, permission) |
| DB 脚本 | `osr-common/src/main/resources/sql/` | 初始化 + 升级脚本 |
| MyBatis Mapper | `osr-system/src/main/resources/mapper/system/` + `osr-openliststrm/src/main/resources/mapper/mybatisplus/` | XML 映射 |

前端另有独立知识库 `osr-web/src/AGENTS.md`，改前端前先读。

## CONVENTIONS
- **包命名**: `com.osr.{module}.{layer}` — controller/service/mapper/domain 分层
- **OpenList-strm 模块**: 按功能域分包 (tg/, tmdb/, rename/, helper/, monitor/, orphan/, pt/, mybatisplus/)
- **MyBatis-Plus**: `osr-openliststrm` 使用 MP 风格 (BaseMapper + IService)，`osr-system` 使用传统 XML Mapper
- **Spring Security + JWT**: 无状态认证（`SessionCreationPolicy.STATELESS`），`JwtAuthenticationFilter` 解析 token，放行路径靠 `@Anonymous` 注解被 `PermitAllUrlProperties` 扫出来
- **Java 25 Preview**: 编译/测试/运行均带 `--enable-preview` (虚拟线程/结构化并发)
- **FastJSON2**: 统一使用 FastJSON2 做 JSON 序列化
- **密码加密**: 使用 Cipher 加密存储敏感配置 (DB_PASSWORD 等)；密钥与连接信息走 `.env` (见 `.env.example`)，不要硬编码或提交进仓库
- **前端**: unplugin-auto-import + unplugin-vue-components 自动导入，`@` 指向 `src/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [907739769/OSR](https://github.com/907739769/OSR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
