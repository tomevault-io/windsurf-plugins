---
trigger: always_on
description: AsterDrive 是面向小团队的 Rust 自托管文件基础设施项目。它关注文件存储控制、可靠大文件上传、个人/团队工作空间、分享、回收站、版本历史、WebDAV/WOPI、远端存储节点和运维可观测性。
---

# AsterDrive

AsterDrive 是面向小团队的 Rust 自托管文件基础设施项目。它关注文件存储控制、可靠大文件上传、个人/团队工作空间、分享、回收站、版本历史、WebDAV/WOPI、远端存储节点和运维可观测性。

当前代码来自通用 Rust + React 服务模板长期演进，但已经是 AsterDrive 自己的产品。修改时要围绕云盘/文件基础设施语义组织代码，不要把其他项目的领域概念带进来。

## 工作前必须先看

- 先读现有代码模式，再动手。看不清模式就停下问 1547，别凭感觉硬写。
- 修改前优先从现有入口追链路：`src/api/routes/*` -> `src/services/*` -> `src/db/repository/*` / `src/storage/*` / `src/webdav/*`。
- 前端改动先看 `frontend-panel/AGENTS.md`，那里有更细的 TypeScript、i18n、UI/UX、Base UI 组件坑位约束。
- 这个仓库可能有大量未提交改动。不要回滚用户改动；只改任务相关文件，遇到同文件交叉改动先读清楚再动手。
- code review comments 进来时，先分辨真问题还是误报；真实问题分批修，修完每批都编译/测试。别被机器人牵着鼻子走。

## 项目结构

```text
src/                         Rust 后端
src/api/                     primary/follower 路由、DTO、OpenAPI、中间件、响应封装
src/api/routes/              REST API、公开分享、内部存储、远端隧道路由
src/cache/                   cache trait 以及 memory/Redis 实现
src/cli/                     doctor、config、database-migrate、node enroll 等运维 CLI
src/config/                  静态配置、运行时配置定义、配置规范化、模板
src/db/                      数据库连接、reader/writer 句柄、repository
src/entities/                SeaORM Entity
src/runtime/                 AppState、primary/follower 启动、关闭、周期任务
src/services/                Auth、file、folder、upload、share、team、policy、task、audit、WebDAV/WOPI 等业务层
src/storage/                 存储驱动、连接器、远端协议、multipart/stream 能力抽象
src/types/                   共享枚举、DTO 辅助类型和 DB wrapper 类型
src/utils/                   crypto、ID、path、number、email、RAII 等工具
src/webdav/                  WebDAV/DeltaV 协议接入、文件系统、锁、属性和传输
migration/                   SeaORM migration crate
api-docs-macros/             OpenAPI 辅助宏
frontend-panel/              React + Vite 前端，构建产物嵌入后端
developer-docs/              开发说明和架构文档
docs/                        用户/部署文档站
tests/                       集成测试、迁移测试、OpenAPI 导出测试
```

## 技术栈

- 后端: Rust 2024, actix-web 4, SeaORM 2.0-rc, tokio, jsonwebtoken, argon2
- 数据库: SQLite 默认，兼容 MySQL/PostgreSQL
- 缓存: memory/Redis 后端；不再支持 noop cache，也不要新增 `cache.enabled` 这类禁用缓存分支
- 存储: local filesystem、S3-compatible object storage、Azure Blob、OneDrive、remote AsterDrive follower node
- 协议: REST API、WebDAV/DeltaV、WOPI、remote internal storage protocol
- 前端: React 19, Vite, TypeScript native-preview/tsgo, Tailwind CSS 4, shadcn/ui(Base UI), Biome, Vitest, Playwright
- OpenAPI: `utoipa` + `api-docs-macros` + `openapi-typescript`
- 嵌入: `rust-embed` 将 `frontend-panel/dist/` 编译进二进制

## 开发命令

```bash
# 后端
cargo run
cargo check
cargo test
cargo test --lib <test_filter>
cargo test --test <test_name> <test_filter>
cargo test --features openapi --test generate_openapi
cargo test --features metrics

# 指定集成测试数据库后端
ASTER_TEST_DATABASE_BACKEND=sqlite cargo test --test test_database_backends
ASTER_TEST_DATABASE_BACKEND=postgres cargo test --test test_database_backends
ASTER_TEST_DATABASE_BACKEND=mysql cargo test --test test_database_backends

# 前端
cd frontend-panel
bun install
bun run dev
bun run build
bun run check
bun run test
bun run test:e2e
```

跑 Rust 单元/集成测试时必须优先缩小范围，使用 `cargo test --lib <filter>` 或 `cargo test --test <test_name> <filter>`，不要直接跑无目标的大范围 `cargo test <filter>` 把全包编译时间炸开。批量修复后再跑 `cargo check` 和相关测试。改动 OpenAPI schema 后先跑 `cargo test --features openapi --test generate_openapi`，再到 `frontend-panel/` 跑 `bun run generate-api`。

## 当前核心能力

- 本地认证: setup/register/login/refresh/logout/me/sessions、用户偏好、头像、SSE 事件
- 外部认证/MFA: provider 配置、外部登录流程、WebAuthn/Passkey 基础能力
- 文件工作流: folder/file CRUD、上传、下载、移动、复制、删除、恢复、永久删除、版本历史
- 工作空间: personal workspace 和 team workspace 共用核心链路，通过 scope 切换作用域
- 分享: 公开分享页、密码、过期、下载次数、直链、预览直链
- 上传: direct、chunked resumable、presigned、multipart、remote relay/presigned 等策略
- 存储策略: local、S3-compatible、Azure Blob、OneDrive、remote follower node、policy group 路由
- WebDAV/WOPI: 独立 WebDAV 账号、锁、DeltaV 子集、Office 预览/编辑启动会话
- 远端节点: primary/follower 模式、internal storage API、direct/reverse tunnel/auto 传输
- 运维: runtime config、audit logs、background tasks、health、metrics、doctor、migration CLI

## 产品域边界

新增功能要围绕 AsterDrive 的文件基础设施组织，命名直接表达业务含义：

- 文件与目录: `file`, `folder`, `file_blob`, `file_version`, `workspace`, `trash`
- 工作空间与权限: `personal`, `team`, `member`, `role`, `quota`, `scope`
- 分享与公开访问: `share`, `public_link`, `download_token`, `preview`
- 上传链路: `upload_session`, `chunk`, `multipart`, `presigned`, `complete`, `cancel`
- 存储与路由: `storage_policy`, `policy_group`, `connector`, `driver`, `object_key`, `blob`
- 远端节点: `remote_node`, `follower`, `internal_storage`, `reverse_tunnel`, `managed_ingress`
- 协议能力: `webdav`, `wopi`, `lock`, `delta_v`
- 运维能力: `task`, `audit`, `runtime_config`, `doctor`, `health`

不要引入和当前产品无关的领域词来伪装功能。需要处理外部对象存储时，从 AsterDrive 的存储策略、连接器、驱动和上传策略建模，不要另起一套平行抽象。

## API 约定

项目 REST API 使用统一响应体：

```json
{ "code": "success", "msg": "", "data": { } }
```

失败使用稳定字符串错误码，定义在 `src/api/api_error_code.rs` 的 `AsterErrorCode`。内部错误类型是 `src/errors.rs` 的 `AsterError`，通过 `ResponseError` 统一转 HTTP 响应和日志。

新增项目 API 应继续使用这套 envelope 和错误码体系。例外包括：

- 文件下载、缩略图、预览、公开直链等直接返回流或二进制响应的接口
- SSE，例如 storage events
- Prometheus metrics text exposition
- WebDAV/DeltaV 协议响应
- WOPI 协议端点需要满足 WOPI host 的字段、状态码和错误行为
- follower internal storage protocol 和 reverse tunnel 内部传输需要满足内部协议签名/预签名约定

协议端点不能为了项目内部 envelope 破坏客户端兼容性。如果协议错误格式与 `AsterError` 不一致，单独建协议错误映射层，不要污染全局错误系统。

## 后端代码约定


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AsterCommunity/AsterDrive](https://github.com/AsterCommunity/AsterDrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
