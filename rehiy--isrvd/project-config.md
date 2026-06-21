---
trigger: always_on
description: > 本文件是 `isrvd` 仓库根规范入口。子目录可有专项 `AGENTS.md`，目标：可执行、可验证、低歧义。
---

# AGENTS.md — isrvd Agent 操作指南

> 本文件是 `isrvd` 仓库根规范入口。子目录可有专项 `AGENTS.md`，目标：可执行、可验证、低歧义。

---

## 1) 指令优先级

冲突时：用户明确需求 → 安全稳定性 → 本规范 → 现有代码风格
无法消解时：不泄露敏感信息、不引入破坏性变更、不修改需求范围外逻辑

---

## 2) 工作流

- **先理解再改动**：定位模块、调用链、类型与边界，不基于猜测修改
- **小步提交**：最小可行改动，每步可解释"为什么改、改了什么、如何验证"
- **变更后验证**：执行相关静态检查；无法完整验证时说明风险
- **同步更新文档**：修改代码时，必须同步更新所有相关的说明文档，确保文档与代码始终一致

---

## 3) 项目架构

### 分层依赖方向

`config → internal/registry → pkgs → internal/service → internal/server`

- `pkgs/`：原生客户端层，返回 SDK/底层类型
- `internal/service/`：业务组合、类型转换、参数校验
- `internal/server/`：HTTP 入口，解析请求 → 调用 service → 返回响应
- `internal/registry/`：外部服务初始化、生命周期管理与可用性检查

### 禁止

- `pkgs/` 依赖 `internal/`
- `handler` 中堆叠业务逻辑
- `service/handler` 直接从配置创建外部客户端

### 内聚与耦合

- 同领域功能聚合同包（`pkgs/docker/`、`pkgs/swarm/`、`pkgs/caddy/`），类型就近定义，不集中 `types.go`
- 层间通过接口或注入解耦；前端全局状态通过 Pinia `usePortal()` 聚合访问
- 判定：改一个功能只需改一处；若需同时改多个包同名函数，说明内聚不足

---

## 4) 代码变更同步文档规范

修改代码时，必须同步更新所有相关的说明文档，确保文档与代码始终一致。

### 适用范围

所有涉及以下变更的场景：

- 新增/修改/删除 API 路由或参数
- 新增/修改/删除 数据结构（Request/Response 字段）
- 新增/修改/删除 业务逻辑或工作流
- 新增/修改/删除 配置项或环境变量
- 新增/修改/删除 Shell 脚本中的命令或参数

### Docs 文件结构

```
docs/
├── SKILL.md                      ← 索引 + 决策树 + 常见工作流
├── scripts/
│   └── api.sh                    ← 认证持久化 + API 调用封装
└── references/
    ├── docker/{containers,images,networks,volumes,registries}.md
    ├── swarm/{info,services,tasks}.md
    ├── apisix/{routes,upstreams,consumers,ssl}.md
    ├── caddy/{routes,certs,config}.md
    ├── system/{overview,config,account,filer,cron}.md
    ├── overview/
    ├── compose.md
    ├── shell.md
    └── ssh/{hosts,sftp}.md
```

### 需要同步更新的文件

| 代码变更位置 | 需同步更新的文档 |
|---|---|
| `internal/server/ctrl_docker.go` | `docs/references/docker/` 下对应资源文件 |
| `internal/server/ctrl_swarm.go` | `docs/references/swarm/` 下对应资源文件 |
| `internal/server/ctrl_apisix.go` | `docs/references/apisix/` 下对应资源文件 |
| `internal/server/ctrl_caddy.go` | `docs/references/caddy/` 下对应资源文件 |
| `internal/server/ctrl_compose.go` | `docs/references/compose.md` |
| `internal/server/ctrl_cron.go` | `docs/references/system/cron.md` |
| `internal/server/ctrl_system.go` / `ctrl_account.go` | `docs/references/system/` 下对应文件 |
| `pkgs/*/`（数据结构变更） | 对应 docs 文件中的字段表 |
| 新增路由/模块 | `docs/SKILL.md` 索引表 + 决策树 |
| 脚本相关变更 | `docs/scripts/api.sh` |

### 执行步骤

1. **识别影响范围**：修改代码后，定位对应的 docs 文件（按模块+资源查找）
2. **同步更新文档**：
   - API 变更 → 更新对应 docs 文件的 bash 用法、请求体、响应字段
   - 数据结构变更 → 更新字段表，确保与 Go struct json tag 一致
   - 新增路由 → 在 `SKILL.md` 索引表和决策树中添加条目
3. **验证格式**：所有 API 必须以 `isrvd_get/post/put/patch/delete/upload` bash 用法为主要表达方式，不得只写 HTTP 格式

### 注意事项

- 文档中标注"只读"的字段（如 `create_time`、`update_time`、`id`）也必须列出
- 请求体和响应体中的字段必须与 Go struct 的 json tag 完全一致
- 路由路径必须与 `ctrl_*.go` 中的注册路径完全一致
- 每个 API 端点必须有对应的 bash 示例，不能只写 `GET /api/...` HTTP 格式

---

## 5) 后端编码规范（Go）

### HTTP 与响应

- 状态码用 `net/http` 常量；成功 `httpd.RespondSuccess`，失败 `httpd.RespondError`
- 绑定优先 `ShouldBindJSON/ShouldBindQuery/ShouldBindURI`，绑定失败返回 `err.Error()`
- WebSocket 统一用 `wsConfig.Handler()`，不在 handler 中定义私有配置

### 错误处理

- 禁止 `fmt.Errorf(err.Error())`，使用 `fmt.Errorf("...: %w", err)` 包装
- `pkgs` 调外部失败时透传原始错误；仅本层自有逻辑失败时加上下文

### 命名与日志

- 缩写全大写：`CPU`、`ID`、`URL`、`HTTP`、`JWT`、`CORS`；`JWTToken` 冗余，改用 `JWT`、`JWTCheck`、`JWTUsername`
- 日志统一 `logman`，禁止 `log.Println`/`fmt.Println`；使用键值对不拼接字符串

### 方法命名规范（强制）

**Handler（`internal/server/`）** — 格式：`{module}{Resource}{Action}`

| 操作 | 命名模式 | 示例 |
|---|---|---|
| 列表 | `{module}{Resource}List` | `dockerContainerList`、`apisixRouteList` |
| 单条 | `{module}{Resource}Inspect` | `dockerImageInspect`、`swarmNodeInspect` |
| 创建/更新/删除 | `{module}{Resource}Create/Update/Delete` | `apisixRouteCreate`、`dockerImageDelete` |
| 操作/动作 | `{module}{Resource}Action` | `dockerContainerAction`、`swarmNodeAction` |
| 状态切换 | `{module}{Resource}StatusPatch` | `apisixRouteStatusPatch` |
| 日志/统计 | `{module}{Resource}Logs/Stats` | `dockerContainerLogs`、`dockerContainerStats` |

**Service / Pkgs（`internal/service/`、`pkgs/`）** — 格式：`{Resource}{Action}`（去掉类名前缀）

| 操作 | 命名模式 | 示例 |
|---|---|---|
| 列表/单条/查询 | `{Resource}List` / `{Resource}Inspect` / `{Resource}` | `RouteList()`、`ImageInspect(id)`、`Stat(ctx)`、`Probe(ctx)` |
| 获取详情 | `{Resource}Inspect` | `NodeInspect(ctx, id)` |
| 创建/更新/删除 | `{Resource}Create/Update/Delete` | `RouteCreate(req)`、`RouteDelete(id)` |
| 状态切换 | `{Resource}StatusPatch` | `RouteStatusPatch(id, status)` |
| 特殊操作 | `{Resource}{Verb}` | `ContainerAction(id, action)`、`WhitelistUserCreate(...)` |

- **查询接口不加 `Get` 后缀**：方法名本身已表达"获取"语义（`Stat()`、`Probe()`、`Info()`、`JoinToken()`、`ConfigAll()`），禁止改为 `StatGet()`、`probeGet()` 等形式
- **`Get` 仅用于必要场景**：当方法名去掉 `Get` 后会与已有方法冲突或语义不明时，才可保留 `Get` 后缀
- **模块前缀**：`docker`、`swarm`、`apisix`、`caddy`、`account`、`system`、`filer`、`compose`、`cron`
- **资源名**：单数形式，不重复模块语义
- **禁止**：`动词+资源` 旧式命名（`CreateRoute`、`ListContainers`、`apisixCreateRoute`）
- **注意**：类名为 `Docker` 时 `ContainerList` 不缩写为 `List`；类名为 `Apisix` 时 `RouteList` 不缩写为 `List`

### 文件内组织

文件内顺序（自上而下）：

1. **可复用的 const/var 定义** — 包级常量/变量放在 import 之后、类定义之前，禁止移到文件中间或末尾
2. **类定义、初始化函数** — struct 定义及其 `New*` 构造函数紧随其后
3. **类共有函数、私有函数** — 业务方法按逻辑分组排列；函数使用的类型定义紧贴在首个使用它的方法之上（就近定义），禁止集中到文件顶部，禁止定义在首次使用之后
4. **独立的辅助函数** — 非业务入口的内部工具函数统一放在文件末尾，用 `// ─── 辅助函数 ───` 注释标记

其他规则：

- 请求/响应结构体：定义在对应 handler 子包；业务模型：就近定义在对应 `pkgs` 文件
- 避免跨包重复定义语义相同结构体

### 配置结构体与 Provider


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rehiy/isrvd](https://github.com/rehiy/isrvd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-21 -->
