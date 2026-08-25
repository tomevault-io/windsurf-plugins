---
trigger: always_on
description: 本文件定义 AI 协作智能体在 GitCandy 仓库工作的规范与约束。所有 AI 辅助生成的代码、文档、迁移脚本和测试都必须遵守本文件。
---

# GitCandy 智能体协作约束

本文件定义 AI 协作智能体在 GitCandy 仓库工作的规范与约束。所有 AI 辅助生成的代码、文档、迁移脚本和测试都必须遵守本文件。

本约束结合了 GitCandy 当前 ASP.NET MVC 5 到 ASP.NET Core 10 MVC + EF Core 的迁移目标，以及 SonnetDB 的智能体约束组织方式、代码习惯、PR 习惯和 `.editorconfig` 风格。后续智能体开始工作前必须先阅读根目录的 `ROADMAP.md`。

参考来源：

- SonnetDB AGENTS.md: https://github.com/IoTSharp/SonnetDB/blob/main/AGENTS.md
- SonnetDB .editorconfig: https://github.com/IoTSharp/SonnetDB/blob/main/.editorconfig

---

## 项目目标

GitCandy 是一个基于 ASP.NET MVC 的 Git 代码托管服务，支持公有/私有仓库、用户、团队、HTTP Git Smart Transport 和 SSH transport。

当前目标是按 `ROADMAP.md` 将项目从 ASP.NET MVC 5 / .NET Framework 4.5 迁移到 ASP.NET Core 10 MVC + EF Core。用户认证采用 ASP.NET Core Identity 标准体系，不兼容旧 `Users` / `AuthorizationLog` / 密码版本表；旧账户代码只作为业务语义参考。需要保留的是 GitCandy 的公开 URL、Git HTTP/SSH 协议行为、权限模型语义和部署可迁移性。

目标运行形态默认是一个 GitCandy 应用进程承载 Web UI、HTTP API、Git Smart HTTP、内置 SSH、scheduler 和后台索引入口。外部 OpenSSH、独立 Git daemon、独立 worker、shell 脚本编排只作为迁移稳定后的可选适配，不作为第一阶段默认架构。

当前派单焦点：

- 优先做可验证的垂直切片，不做大爆炸迁移。
- 优先打通 ASP.NET Core 10 MVC 外壳、EF Core + Identity 新数据库、登录/仓库列表、Git Smart HTTP clone/fetch/push。
- UI 现代化、前端构建链、全面架构重写都属于迁移稳定后的独立任务。

---

## 强制约束

以下约束不得违反。如需例外，必须在变更说明中明确理由、影响范围、回滚方式和验证结果。

### 1. 迁移边界

- 任何迁移型任务必须对应 `ROADMAP.md` 中的一个 Milestone 或一个明确垂直切片。
- 一个变更只做一件事：不要把框架迁移、UI redesign、密码升级、依赖升级、schema 演进混在一起。
- 先保护行为，再改善结构。Git HTTP、SSH、权限语义和新数据库正确性优先于代码形式。
- 不回滚用户已有改动。动手前查看 `git status --short`，只改与当前任务相关的文件。
- 默认在 `master` 主分支上继续迁移工作；不得擅自切换、创建、删除或重命名分支。如确需临时分支，必须先得到用户明确确认。

### 2. 目标技术栈

- 新代码目标为 `net10.0`。
- Web 框架目标为 ASP.NET Core MVC，使用 `Program.cs`、middleware pipeline、endpoint routing、built-in DI。
- 数据库目标为 EF Core + ASP.NET Core Identity，默认保留 SQLite 支持，同时保持 SQL Server 路径可行。
- 新项目使用 SDK-style csproj、central package management、`global.json`、`.slnx`。
- 新 ASP.NET Core 项目不得新增 `System.Web`、`System.Web.Mvc`、`System.Web.Optimization`、`System.Data.Entity` 依赖。
- 旧 MVC5 项目已在迁移完成后退役；历史实现通过 Git 历史和 `docs/migration` 行为基线查阅，不得重新引入旧项目或旧框架依赖。

### 3. 编译器与 analyzer

- 新 SDK-style 项目必须启用 `ImplicitUsings`。
- 新核心类库和测试项目优先启用 `Nullable`；旧代码迁移时可分模块渐进启用，不允许为消除警告随意加 `!`。
- 新项目最终目标是 `TreatWarningsAsErrors=true`；迁移早期若因旧代码噪声暂缓，必须在项目文件或 PR 说明中标注原因。
- 不得用无说明的 `#pragma warning disable` 或 broad suppressions 压制与业务逻辑、空值、安全、EF 查询相关的警告。
- 热路径代码可逐步采用 SonnetDB 风格的性能 analyzer 习惯：避免不必要 LINQ、重复分配、重复枚举、反复创建 `JsonSerializerOptions`。

### 4. 依赖约束

- 新运行时依赖必须有明确收益、维护成本评估和替代方案说明。
- 优先使用 .NET BCL、ASP.NET Core、EF Core 官方包和项目已有能力。
- 数据访问层允许使用 EF Core Provider；不要混入 Dapper 或第二套 ORM。
- 新 JSON 代码优先使用 `System.Text.Json`。除非为兼容旧行为，不新增 `Newtonsoft.Json`。
- `LibGit2Sharp` 升级必须单独成任务，附带 native binary、API 差异和 Git 操作回归验证。
- 不提交 `bin/`、`obj/`、`packages/`、`.nupkg`、数据库实例、日志、缓存、密钥等生成物或敏感物。

### 5. 安全与数据完整性

- 不记录明文密码、token、authorization header、SSH private key、host key 私钥内容。
- 不削弱私有仓库、团队、owner、管理员权限判断。
- Web 登录使用 ASP.NET Core Identity cookie；Git Basic Auth 必须是独立认证 scheme，不能让浏览器 cookie 逻辑破坏 Git 客户端认证。
- 不兼容旧密码 hash、旧 `_gc_auth` cookie、旧 `AuthorizationLog`。升级后需要重新创建用户，或通过单独导入工具导入非密码类资料。
- Repository/cache/archive/delete 路径必须做路径归一化和边界检查。
- 任何递归删除或移动必须先验证目标绝对路径位于预期根目录下。
- 不把用户输入直接拼进 shell command。调用 `git.exe` 优先使用 `ProcessStartInfo.ArgumentList`。

### 6. 兼容性变更

下列内容一旦改变，必须同步更新文档并说明迁移/回滚方案：

- 公开路由和 Git URL。
- 数据库 schema、索引、collation、默认值、seed 数据。
- Identity cookie、Basic Auth、权限语义。
- 配置键、环境变量、部署方式。
- Git HTTP/SSH 协议行为和响应 header。
- 文件系统布局：repositories、cache、App_Data、host keys、logs。

---

## 代码规范

### 命名

- 类型、方法、属性、事件：`PascalCase`。
- 类型名不要机械添加项目名或仓库名前缀；优先按领域含义和职责命名，例如 `AuthorizationPolicies`、`RoleNames`。只有当类型本身是应用级标识、公开 API 需要消歧，或与框架/第三方类型明显冲突时，才使用 `GitCandy` 前缀，并在变更说明中写明原因。
- 接口：`IXxx`。
- 局部变量、参数：`camelCase`。
- 私有字段：`_camelCase`。
- 常量：`PascalCase`，不要使用全大写下划线风格。
- 异步方法以 `Async` 结尾，除非重写框架方法。

### 格式

- C# 使用 4 空格缩进；XML、props、targets、json、yaml 使用 2 空格缩进。
- 使用 file-scoped namespace 时要与周围文件风格一致；迁移旧文件时不要只为风格改动制造大 diff。
- `using System.*` 放在前面，同组 using 不人为分块。
- 内置类型默认显式写类型；类型显而易见时可以使用 `var`。
- 新增复杂逻辑前写简短注释解释原因，不写“这行代码做什么”的空注释。

### 现代 C# 习惯

- 参数空值校验优先使用 `ArgumentNullException.ThrowIfNull`。
- 字符串比较必须显式指定 `StringComparison`，尤其是用户、团队、仓库名。
- 字典命中并取值时优先 `TryGetValue`。
- 有 `Count`、`Length`、`IsEmpty` 时不要用 LINQ 判空。
- 避免对 `IEnumerable` 多次枚举；必要时物化并说明原因。
- I/O、数据库、Git 进程、网络监听都必须考虑 `CancellationToken` 或 graceful shutdown。
- 新代码优先通过构造函数注入依赖，不新增静态全局状态；保留旧静态入口只作为迁移过渡。

### 异常与日志

- 不吞掉 `IOException`、`UnauthorizedAccessException`、EF Core 异常、Git 进程异常、SSH 协议异常。
- 捕获异常时必须能解释恢复策略；否则记录日志后让上层错误处理中间件处理。
- 用户可预期错误返回明确状态码或模型错误；内部错误不要泄漏敏感路径、token、header。
- 新代码使用 `ILogger<T>`；旧 `Logger` 可通过 adapter 过渡。

### XML 文档注释

- 公共库、跨层接口、权限服务、Git transport、迁移工具的 public API 应写中文 XML 文档注释。
- MVC controller action、DTO、view model 不要求机械补齐 XML 注释，但复杂行为要有清晰命名和测试。

---

## 分层与迁移边界

推荐目标结构以 `ROADMAP.md` 为准：

- `GitCandy`：唯一 ASP.NET Core MVC 主程序和 host，承载 Web UI、Git HTTP、内置 SSH、scheduler、后台任务入口。
- `GitCandy.Core`：领域模型、权限、配置抽象、通用服务。
- `GitCandy.Data`：EF Core DbContext、Identity schema、entity configuration、migrations、数据访问。
- `GitCandy.Git`：LibGit2Sharp、受控 Git transport backend、Git cache、hook pipeline。
- `GitCandy.Ssh`：内置 SSH server 和 SSH Git transport，默认随 ASP.NET Core host 同进程运行。
- `GitCandy.CodeIntelligence`：迁移稳定后的代码索引、符号图谱、Agent Memory 和 MCP/API 能力。
- `tests/*`：单元测试、集成测试、Git protocol smoke tests。

边界规则：

- Controller 保持薄，不直接写复杂 EF 查询和文件系统操作。
- 权限判断沉到服务或 authorization handlers，不散落在视图和 controller。
- EF Core entity configuration 放在 Data 层，不在 Web 层拼模型。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IoTSharp/GitCandy](https://github.com/IoTSharp/GitCandy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-23 -->
