---
trigger: always_on
description: 通用规则在全局 ~/.claude/CLAUDE.md（对话自动加载；项目内镜像 CLAUDE_GLOBAL.md 与其逐字一致，全局更新后同步刷新镜像）。本文件只放 qadmin 项目专属规则，「项目须提供 X」的落地位置在「项目概述」小节；与全局冲突时以本文件为准。
---

# 版本: <260703.7>

通用规则在全局 ~/.claude/CLAUDE.md（对话自动加载；项目内镜像 CLAUDE_GLOBAL.md 与其逐字一致，全局更新后同步刷新镜像）。本文件只放 qadmin 项目专属规则，「项目须提供 X」的落地位置在「项目概述」小节；与全局冲突时以本文件为准。

# qadmin 项目规则

## 项目概述

qadmin 是开源通用管理后台底座。部署即全新：每次发布清空 MySQL + Redis + ClickHouse，从 init.sql + clickhouse_ddl.sql 重建。因此不存在增量迁移——所有表结构、种子数据直接维护在 server_data/shared/mysql/init.sql，新功能的建表/改表/种子一律改 init.sql 本体，禁止另建迁移文件。

- server/：管理后台 API。NestJS 10 + TypeORM + MySQL + ioredis + Bull
- server_admin_ui/：管理后台前端。Vue 3 + Vite + Element Plus + Pinia
- server_go/：管理后台 API（Go 版基座，与 NestJS 同构：system/login/monitor）。Gin + GORM + MySQL + go-redis + viper
- server_rust/：管理后台 API（Rust 版基座，与 NestJS 同构：system/login/monitor）。Axum 0.8 + SeaORM + MySQL + fred + Tokio
- servers_ref/server_py/：管理后台 API（Python 版基座，与 NestJS 同构）。FastAPI + SQLAlchemy + MySQL + redis-py
- servers_ref/server_php/：管理后台 API（PHP 版，全量对齐 NestJS）。Webman(Workerman) + illuminate/database + illuminate/redis(predis)
- servers_ref/server_java/：管理后台 API（Java 版，全量对齐 NestJS）。Spring Boot + MyBatis-Plus + MySQL + Redis
- servers_ref/server_kotlin/：管理后台 API（Kotlin 版，部署上接管全部 cron）。Spring Boot + MyBatis-Plus + MySQL + Redis
- servers_ref/server_csharp/：管理后台 API（C# 版，对齐 NestJS）。.NET 10 + SqlSugar + MySQL + Redis

- 目录分层即优先级：主力 server/、同构基座 server_go/ server_rust/ 留在仓库根；其余降权参考实现（Python/PHP/Java/Kotlin/C#）统一收纳在 servers_ref/ 下。这 5 个后端按 cwd-相对引用共享数据目录时用 ../../server_data/...（比根级后端多一层 ../）

- 后端结构：src/modules/（框架功能：system/login/monitor/cache）、src/modules_biz/（业务模块，如 nav）、src/common/、src/shared/
- Go 基座结构：internal/base/（system/login/monitor 基座）、internal/app/（装配）、pkg/、config/、cmd/server（入口）
- Rust 基座结构：crates/{core,infra,middleware}（基础设施）、crates/{mod_system,mod_monitor}（功能模块）、crates/app（装配入口）
- PHP 结构：app/common/（契约+PHP8 attributes）、app/middleware/（反射读 attribute 的守卫链）、app/process/（worker 注册 + cron 调度常驻进程）、app/modules/（system/login/monitor + 额外模块）、app/biz/nav、config/
- 多套后端（NestJS/Go/Rust/Python/PHP）同构，共库部署。Go/Rust 与 server_php 一样全量对齐 NestJS（含 tenant/sms/mail/social/notify/i18n/sensitive_word/file/api_log/gen/nav）
- [人工决策-2026-06-19 14:02:41] Go/Rust 由「剥离业务的纯基座（只保 system/login/monitor）」改为全量对齐 NestJS 所有功能，推翻 [人工决策-2026-06-17 22:41:57] 中关于 Go/Rust 为纯基座的定位（该决策的目录分层部分仍有效）
- Redis key 文件：server/src/modules/sys_redis_keys.ts（NestJS）；servers_ref/server_php/app/common/RedisKeys.php（PHP）
- 时区配置：后端环境变量 BIZ_TIMEZONE（默认 Asia/Shanghai），前端环境变量 VITE_BIZ_TIMEZONE（默认 Asia/Shanghai）。各端读取位置——NestJS: server/src/common/env.ts env.bizTimezone + formatBizTime()；Go: server_go/config/config.go app.biz_timezone + pkg/utils/time.go BizLoc/FormatBizTime()/ParseBizTime()；Rust: server_rust/crates/core/src/config.rs app.biz_timezone + pretty_log.rs init_biz_timezone()；前端: server_admin_ui/src/shared/libs/env.js env.bizTimezone(←VITE_BIZ_TIMEZONE)、时间库 luxon 在 shared/libs/lib.js(utcToBiz 做 UTC→业务时区显示)。业务时区「日期范围」→ UTC 转换（纯日期查整天、datetime 精确、begin/end 各自可选、右闭、DST 安全）：NestJS server/src/common/env.ts bizDateRangeWhere()（返 TypeORM where 操作符），Go server_go/pkg/utils/time.go BizBeginToUtc()/BizEndExclusiveToUtc()，Rust server_rust/crates/core/src/time_utils.rs biz_begin_to_utc()/biz_end_exclusive_to_utc()。部署时前后端统一配置同一个 IANA 时区值
- 配置与时间库入口：业务配置单一入口 NestJS server/src/common/env.ts（process.env 只在此读，dotenv 引导在 main.ts、进程身份 SHARD_ID 用 NODE_APP_INSTANCE/pid 属例外）、前端 server_admin_ui/src/shared/libs/env.js（import.meta.env 只在此读）；时间库统一 luxon，server 从 src/common/libs.ts 导入 DateTime/luxonFrom、前端从 shared/libs/lib.js 导入 DateTime 与 formatDateTime/formatDateTimeShort/formatDate/formatUtc 助手，禁止直接 import luxon
- [人工决策-2026-06-14 04:46:46] server_php 共存与架构决策：运行时 Webman(Workerman) 常驻 worker；所有后端共享同一套 Redis key（admin:/sys:/captcha: 用于 session/缓存/在线，qadmin: 用于协同层 worker 注册+cron 锁）；定时任务全端从 Bull 改分布式锁调度(方案 A，锁 key qadmin:lock:cron:{jobId}:{unixMinute} NX PX90s，分钟粒度，NestJS 已去 Bull 用法)；代码生成器产出 Vue+PHP(Webman) 模板
- [人工决策-2026-06-28 19:17:54] Redis key 前缀统一为 qadmin: 全前缀，推翻 [人工决策-2026-06-14 04:46:46] 中"session/缓存/在线用裸 admin:/sys:/captcha:"的部分（该决策的 cron 锁/worker 注册/方案A/代码生成器部分仍有效）。原文字与 NestJS 真值源代码(sys_redis_keys.ts 早已全 qadmin: 前缀 + snake_case 段名，如 qadmin:admin:token/qadmin:admin:nick_name/qadmin:sys:config/qadmin:sys:dict/qadmin:captcha:{uuid})矛盾，导致 java/php/csharp(裸前缀阵营)与 NestJS/go/rust/kotlin(qadmin:阵营)共库时登录态互不可见。已统一 java/php/csharp 对齐 NestJS（含散落硬编码点：online strip/cache 监控分组 pattern/config·dict SCAN pattern；注意 clearCacheAll 须只清 qadmin:admin/captcha/sys 排除 workers/lock 运行态）
- 开发环境用 docker-compose 启动 MySQL、Redis 和 ClickHouse，见 README.md
- 各后端 dev 启动命令：NestJS cd server && npm run dev（:3000）；Go cd server_go && make dev（:9711，需 air）或 make run（编译后运行）；Rust cd server_rust && just dev（:9811，需 cargo-watch）或 just run（单次运行）。Rust APP_ENV 默认 development 无需显式指定
- 包管理分栈：后端 server/（及 llysc/pi 等同构后端）用 bun 装依赖（bun add/bun remove，不是 pnpm）；前端 server_admin_ui/ 用 pnpm。需安装/卸载的依赖照旧告诉用户、由用户自行执行
- 前端 CSS 优先用原子 CSS（Tailwind）：能用原子类（flex、mb-4、text-sm、任意值类 text-[var(--xxx)] 等）就用原子类直接写在模板上，不新增 <style scoped>。仅当原子类表达不了（复杂选择器、::v-deep、关键帧动画、:deep() 穿透组件库样式）才写 scss；改现有 scss 文件时跟随现有风格
- 定时任务显式时区（通用规则「定时任务」的本项目落地）：NestJS @Cron({ timeZone })、Spring @Scheduled(zone)；多端同一 cron 抢锁细节见「多后端架构与协同契约」

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SunSeekerX/qadmin](https://github.com/SunSeekerX/qadmin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
