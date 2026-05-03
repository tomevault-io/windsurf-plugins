---
trigger: always_on
description: 这是一份关于 SonicLens 项目的长期记忆清单，整合了项目架构、核心逻辑、开发规范及关键“陷阱”规避方案。AI Agent 在进行任何代码变更前应阅读本清单。
---

# ⚡ SonicLens Quick Memory

这是一份关于 SonicLens 项目的长期记忆清单，整合了项目架构、核心逻辑、开发规范及关键“陷阱”规避方案。AI Agent 在进行任何代码变更前应阅读本清单。

---

## 0. 长期记忆管理协议 (Memory Protocol)

- **核心索引文件**：[memory_index.md](./memory_index.md) 是全站开发的特性历史索引。
- **更新守则**：**AI Agent 在完成重大特性开发、架构重构或核心逻辑修复后，必须执行以下操作**：
    1. 在 `memory/YYYY-MM-DD/` 线下创建详细的 `feature_manifest.md` 特性清单。
    2. 将该清单挂载到 `memory_index.md` 的顶部。
    3. 同步审查并更新本 `GEMINI.md` 文件，确保“核心业务记忆”章节反映最新的逻辑现状。
- **公众号文章生成规约**: 详情见 [wechat_article_generation_constraint.md](./output/wechat_article_generation_constraint.md) 

---

## 0.1 约束抽象母规则

以下四条是当前 SonicLens 的高频母规则；后续新增细则时，优先判断自己是在补充哪一条，而不是继续堆叠零散规则。

- **结构化事实优先**：专辑/曲目身份、展示文案、反馈状态、分析结果默认版本等，都必须优先消费结构化字段或显式契约；禁止回退到从标题字符串、隐式顺序或页面状态里二次猜测事实。
- **高频状态专仓化**：高频播放态、收藏态、资料库查询态、连接恢复态必须收口到共享 store / projection / coordinator；`AppStore`、页面容器和密集列表只消费派生值，不直接承载高频广播。
- **异步事件主链 + 查询对账**：WebSocket / 推送 / 异步任务是前台主通道，GET 查询是恢复和对账通道；长任务、播放态、连接态都要避免重新退回“单次长请求即真相”的旧模型。
- **生成物目录化 + 兼容入口化**：文章、分享物料、导出资源等生成物应落到独立目录，根目录只保留稳定入口、约束文件和兼容层；不要再把真实产物直接散落到仓库公共出口目录。

---

## 1. 项目架构蓝图与模块索引

### 1.1 核心模块树 (Module Tree)

- **`main.go`**: 应用总入口。负责初始化配置、日志、数据库连接及启动 API Server。
- **`cmd/`**: 独立命令行工具集，用于同步、回放和维护任务。
- **`common/`**: 通用枚举、转换与基础工具，供全项目复用。
- **`core/`**: 基础设施与外部能力适配层，负责日志、数据库、缓存、WebSocket、歌词、AI、播放器/第三方服务集成等底座能力。
- **`internal/`**: 核心领域层，负责 DAO、业务编排、播放器适配与后台同步任务。
- **`api/`**: Gin 接口层，负责参数绑定、权限判断、响应编码与 WebSocket 接口暴露。
- **`soniclens-bridge/`**: SwiftUI 客户端工作区，包含三端 target、共享核心层、ViewModels、Views 与客户端文档。
- **`templates/`**: Web 模板层，承载历史 Web UI 与页面脚本入口。
- **`static/`**: Web 静态资源层，存放 CSS、图片与前端素材。

**快速定位梗概**

- 数据库 CRUD 与事务入口看 `internal/model/`；不要在 `api/` 或 `logic/` 直接散落 GORM 细节。
- 业务流程编排看 `internal/logic/`；播放器状态接入看 `internal/scrobbler/`；批处理/回放/补写任务看 `internal/sync/`。
- 实时协议与播放态广播看 `core/websocket/`；歌词解析与 LRC 同步判定看 `core/lyrics/`；AI 结构化 schema 看 `core/ai/`。
- 读接口缓存优先在 `api/` 层按路由挂载 Redis middleware，默认 TTL 5 分钟，可按接口覆盖；空结果走 3 秒短 TTL 负缓存，命中时要回写 `ETag` / `Cache-Control`，Redis 不可用时必须透明降级。
- Web 端页面逻辑主要落在 `templates/*.html`；后台总入口已轻量拆分为 `templates/admin.html` shell、`templates/admin/*.html` partial、`static/admin/admin.css` 和 `static/admin/*.js`，`/` 与 `/admin` 共用该入口。其中 `/api/dashboard/*` 仍表示“仪表盘统计域”，不要和后台入口命名混淆。Bridge 共享能力看 `soniclens-bridge/SoniclensCore/`，端侧容器与交互看 `soniclens-bridge/SoniclensBridge/ViewModels` 和 `soniclens-bridge/SoniclensBridge/Views`。
- iPhone 分享能力已收口到 `soniclens-bridge/SoniclensBridge/ShareKit/`：`Builder` 负责 payload 装配，`Template/iPhone` 负责版式，`Render` 负责长图/分页导出，`Action` 负责相册保存与系统分享，禁止继续在详情页里直接拼接快照导出逻辑。
- iPhone 分享预览的公共外壳已收口到 `SharePosterShell`：顶部公共标题、背景、玻璃拟态容器、封面式 hero、底部居中品牌信息统一复用，`TrackInfoPosterView`、`LyricsLongPosterView`、`InsightLongPosterView` 只允许保留各自正文内容。
- `SharePosterHeader` 的 hero 样式固定为“左侧封面、右侧歌名/艺人专辑、左下位置/指标、右下收藏态”结构，位置标签不再放在顶部。
- 涉及客户端边界、API 映射与构建验证时，优先查 `soniclens-bridge/Docs/`，不要只凭页面代码反推架构。

### 1.2 模块调用拓扑
`Main` -> `API` -> `Internal/Logic` -> `Internal/Model` -> `Core/DB`
`Scrobbler` -> `Internal/Logic` -> `Internal/Model`
`Bridge App` -> `ViewModels` -> `SoniclensCore` -> `API/WebSocket`

---

## 2. 开发规范与最佳实践 

### 2.1 后端编码规约 (Go)
- **命名与风格**: 严格遵循 Uber Go 风格。变量/函数使用 `PascalCase` 或 `camelCase`，包名全小写。
- **枚举规范**: 对外可复用的枚举必须统一放在 `common` 包（优先 `common/enum.go`）；枚举需声明独立类型并使用类型化常量，底层类型限定为 `string`、`int8` 或 `uint8`，禁止在业务文件内分散定义裸常量枚举。
- **注释要求**: **所有注释必须使用中文**。导出的函数、类型必须有阐述“为什么”的注释。
- **日志记录**: 使用结构化日志 `log.Info/Error(ctx, "msg", zap.Field)`。错误日志必须携带 `zap.Error(err)`。
- **日志打印要求**: **所有日志必须使用中文**。打印不同级别的日志（具体使用什么级别看紧急程度，不要滥用）, 关键的函数要出入口要打印。
- **异步协程红线**: 禁止直接写裸 `go func` / `go xxx(...)`；统一使用 `core/telemetry.GoSafe`、`GoSafeDetached` 或 `GoOnlySafe`，分别处理“新建异步 span”“脱离取消的异步 span”“仅 recover 不起长期 span”的场景，避免 panic 打崩进程并避免长循环 trace 失真。
- **可观测性红线**: 面向 SigNoz 的 tracing / metrics 统一走 `core/telemetry` 提供的全局 tracer/meter provider；HTTP 入站走 `otelgin`，Redis 走 `redisotel`，GORM 走 `gorm opentelemetry tracing plugin`，`database/sql` 连接池指标走 `otelsql.RegisterDBStatsMetrics`，不要再在这些标准链路旁边叠加手写重复 span。
- **错误处理**: 禁止忽略错误。使用 `%w` 进行错误包装以保留调用链。
- **测试边界**: 默认 `go test ./...` 必须可在无本地音乐文件、无私有配置、无外部服务凭据环境下稳定运行；依赖真实文件系统、真实第三方 API 或本地私有配置的测试统一使用 `integration` build tag 隔离。


### 2.2 数据库设计指南 (GORM)
- **代码位置规范**：**所有数据库 CRUD 操作必须定义在 `internal/model/` 下对应的表 `go` 文件中**。严禁将原生的数据库查询/更新逻辑散落在各个业务模块（Logic 层）里。
- **复用性原则**：优先封装可复用的模型方法（如 `GetOrCreateAlbum`），减少重复的 SQL 逻辑，确保数据访问层（DAO）的纯粹性。
- **事务边界**：多表事务允许由 Logic 层负责编排，但**事务入口必须由 `internal/model/` 提供**（例如 `InTx`）；Logic 层在事务闭包中只能调用 DAO，不能直接书写 `tx.Where/First/Save/Create/Updates`。
- **DAO 形态**：需要参与事务的 DAO 优先提供 `ctx` 入口与 `Tx` 入口两套能力，公开接口复用事务内核，避免同一 SQL 在上下文版和事务版之间漂移。
- **接口文档约束**：任何新增或修改 `/api/*` 路由都必须同步维护 `api/api.md`，至少更新对应的功能表和关键调用链；如果是重大接口能力变更，还要同时检查相关 `memory/YYYY-MM-DD/*.md` 特性清单是否需要补充。
- **上下文绑定**: 所有数据库操作必须使用 `.WithContext(ctx)` 确保链路可追踪。
- **并发控制**: 重要更新（如 `PlayCount` 增加）应实现基于 `version` 字段的**乐观锁**机制。
- **索引原则**: 复合索引遵循最左前缀原则。新系统必须包含 `created_at` 和 `updated_at`。

### 2.3 客户端规约
- **日志打印要求**: **所有日志必须使用中文**。打印不同级别的日志（具体使用什么级别看紧急程度，不要滥用）, 关键的函数要出入口要打印。
- **当前现状**: Bridge 是独立的 SwiftUI 客户端体系；新增能力优先下沉到 `SoniclensCore`、`ViewModels` 与专题文档，禁止把共享逻辑散落在端侧页面里临时兜底。`soniclens-bridge` 当前包含 `SoniclensBridgeMac`、`SoniclensBridgePad`、`SoniclensBridgePhone` 三个产品线，端差异应停留在容器与交互层，禁止将 macOS `AppKit` 窗口语义泄漏到 iPad/iPhone。详细边界见 `soniclens-bridge/Docs/CLIENT_MODULE_BOUNDARY.md`。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentchyu/sonic-lens](https://github.com/vincentchyu/sonic-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
