---
trigger: always_on
description: WellCMS 3.0 是一个**企业级 PHP 内容管理框架**，核心定位是高性能、可扩展的社区/论坛 SaaS 平台。通过插件钩子系统 + IoC 容器实现无侵入式功能扩展，支持千万级数据的分区归档、游标分页、协程安全隔离以及 Swoole/FPM 双运行模式。
---

# WellCMS 3.0 — AI 开发指南

## 项目背景

WellCMS 3.0 是一个**企业级 PHP 内容管理框架**，核心定位是高性能、可扩展的社区/论坛 SaaS 平台。通过插件钩子系统 + IoC 容器实现无侵入式功能扩展，支持千万级数据的分区归档、游标分页、协程安全隔离以及 Swoole/FPM 双运行模式。

- **版权**: Copyright (C) www.wellcms.com
- **许可证**: MIT
- **出口**: `public/index.php` → 定义 `IN_WELLCMS` 全局安全常量
- **DEBUG 分级**: `0`=生产环境（I/O 冻结，零 filemtime 检查），`1`=调试，`2`=开发

## 技术栈

| 层次 | 技术 | 版本要求 |
|------|------|----------|
| 语言 | PHP | 7.2+ (兼容至 8.5) |
| 运行时 | FPM (默认) / Swoole (协程) | — |
| 数据库 | MySQL 5.6+ / MariaDB / PostgreSQL | — |
| 缓存 | APCu / Redis / Memcached / Yac | — |
| 前端 | Tailwind CSS 3.4 + 原生 JS (无框架) | Node 16+ |
| 包管理 | 无 Composer 依赖！自研 Autoload + Compile | — |
| 测试 | PHPUnit (well_store_server 插件) | — |

## 请求生命周期 (Request Lifecycle)

```
public/index.php
  ├─ define('APP_PATH', ...), define('IN_WELLCMS', true)
  ├─ define('DEBUG', 2)
  ├─ require app/Core/Compile.php   → 收集 Hook、构建 Overwrite 缓存、语言包压平
  ├─ require app/Core/Autoload.php  → 前缀映射 classmap 自动加载
  ├─ App\Bootstrap::init($container)
  │    ├─ ConfigServiceProvider       → 注册 app/db/plugin/view/config 等 9 个配置数组
  │    ├─ DatabaseServiceProvider     → PdoDriver + ProxyDriver + Pool 绑定
  │    ├─ ModelServiceProvider        → 所有 Model 绑定（+ 插件钩子注入）
  │    ├─ ApplicationServiceProvider  → 路由、Session、上传、Store、分区管理等绑定
  │    ├─ RouteServiceProvider        → 加载 Routes.php → CompiledRouter
  │    ├─ MetaServiceProvider         → 注册 Meta Resolver 列表
  │    ├─ MiddlewareServiceProvider   → 构造中间件队列
  │    └─ $container->preResolve()   → 实例化所有非延迟服务
  ├─ ServerRequestFactory::createFromGlobals()
  ├─ App\Core\Kernel::run($container, $request)
  │    └─ Pipeline::handle($request)
  │         ├─ ErrorHandlerMiddleware
  │         ├─ RequestProcessorMiddleware   → 处理 URL rewrite 模式、路由参数解析
  │         ├─ LanguageMiddleware           → 注入 locale / LanguageLoader
  │         ├─ SessionMiddleware            → Session 初始化
  │         ├─ RuntimeMiddleware            → 协程上下文快照
  │         ├─ ThrottleMiddleware           → API 限流
  │         ├─ RouterMiddleware             → 路由匹配
  │         ├─ XssFilterMiddleware          → XSS 过滤
  │         └─ MetaDispatcherMiddleware     → 解析路由 meta → 注入 Auth/Csrf/UserPerm 等中间件
  │              ├─ AuthMiddleware (requiresAuth)
  │              ├─ AdminSignInMiddleware (requiresAdminSignIn)
  │              ├─ CsrfMiddleware (requiresCsrf)
  │              ├─ TokenMiddleware (requiresToken)
  │              └─ UserPermMiddleware (requiresUserPerm)
  └─ ResponseSender::send($response)
```

## 目录结构总览

```
wellcms-3.0/
├── public/index.php              # 唯一 Web 入口
├── src/                          # Framework\ 命名空间 — 框架核心层
│   ├── Core/                     # Container, Config, LazyLoadingProxy
│   ├── Database/                 # PdoDriver, ProxyDriver, Query/Builder, Partition, Pool, Sharding
│   ├── Http/                     # PSR-7 实现 (Request/Response/Stream/Uri)
│   │   ├── Middleware/           # Pipeline, MiddlewareFactory, RequestProcessorMiddleware
│   │   ├── Router/               # Router, Route, CompiledRouter
│   │   └── Routing/              # CoreUrlGenerator (UrlGeneratorInterface)
│   ├── Cache/                    # CacheManager + 4 驱动 (Redis/Memcached/APCu/Yac)
│   ├── Scheduler/                # TaskManage, Task, TaskExecutor, RedisTaskQueue
│   ├── Session/                  # SessionInterface + SessionHandler
│   ├── Logger/                   # FileLogger, SysLogger
│   ├── Utils/                    # 16 个工具类 (见 Framework_Utils_Reference.md)
│   ├── Exception/                # Http/Business/Validation/Infra 异常体系
│   ├── Providers/                # ServiceProviderInterface + LoggerServiceProvider
│   └── Config/                   # 9 个 .default.php 默认配置
├── app/                          # App\ 命名空间 — 应用层
│   ├── Bootstrap.php             # 服务注册 + 中间件队列编排
│   ├── Core/                     # Autoload, Compile, Kernel, ExceptionHandler
│   ├── Controllers/
│   │   ├── Base/                 # BaseController, ResponseFormatter, TemplateManager, MessageController
│   │   ├── Frontend/             # 前台控制器 (Index, Auth, User, My, Upload, Manage, Error)
│   │   ├── Admin/                # 后台控制器 (Index, User, Group, Plugin, Theme, Store, Setting 等)
│   │   └── Api/                  # API 控制器 (LinkPreview)
│   ├── Services/                 # 业务服务层
│   │   ├── Auth/                 # UserService, GroupService, SessionService, TokenService
│   │   ├── System/               # CacheService, LogService, MailService, MenuService, IpListService
│   │   ├── Storage/              # UploadService, AttachmentService, FileStorageService, StorageManager
│   │   ├── Content/              # NavigationService, TempContentService, RecycleService
│   │   ├── Upgrade/              # UpgradeService, Downloader, Deployer, ScriptRunner
│   │   ├── Market/               # MarketClient, MarketCircuitBreaker, MarketFallbackService
│   │   ├── Extension/            # ExtensionManager, ExtensionInstaller
│   │   └── Stats/                # RuntimeStats
│   ├── Models/                   # 数据模型 (均继承 BaseModel)
│   ├── Providers/                # 8 个 ServiceProvider
│   ├── Middleware/                # 12 个中间件

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wellcms/wellcmsV3](https://github.com/wellcms/wellcmsV3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
