---
trigger: always_on
description: **ALWAYS RESPOND IN CHINESE-SIMPLIFIED**
---

# AGENTS.md

**ALWAYS RESPOND IN CHINESE-SIMPLIFIED**

## 1. 项目概述

HiMarket 是一个 AI 开放平台，提供 API 产品管理、开发者门户、AI 对话、云 IDE（HiCoding）、MCP Server 托管等功能。

本仓库是前后端一体的 monorepo：`himarket-server/`（Spring Boot 3.2.11 / Java 17）+ `himarket-web/`（React 18 / TypeScript / Vite）。

## 2. 快速命令

```bash
make compile                    # 快速编译（跳过测试和格式检查）
make build                      # 完整构建（编译 + 格式检查 + 测试）
make test                       # 运行单元测试
make lint                       # 代码格式检查（Spotless）
make lint-fix                   # 自动修复代码格式
make run                        # 编译并启动后端服务
```

Demo 凭据：管理员 `admin` / `admin` | 开发者 `user` / `123456` | 后端端口：`8080`

## 3. 后端架构（himarket-server）

### 模块依赖

```
himarket-dal (数据层)  ←  himarket-server (业务层)  ←  himarket-bootstrap (启动配置层)
```

严格单向依赖：上层可以依赖下层，下层不能依赖上层。

### 包结构

```
com.alibaba.himarket
├── config/                 # Spring 配置（ACP、SLS、可观测性）
├── controller/             # 26 REST Controllers，按领域组织
│   ├── AdministratorController   /admins        @AdminAuth
│   ├── DeveloperController       /developers    @DeveloperAuth
│   ├── ProductController         /products      @AdminAuth
│   ├── ConsumerController        /consumers     @DeveloperAuth
│   ├── PortalController          /portals       @AdminAuth
│   ├── GatewayController         /gateways      @AdminAuth
│   ├── NacosController           /nacos         @AdminAuth
│   ├── ChatController            /chats         SSE 流式
│   ├── CodingSessionController                  云 IDE
│   ├── McpServerController       /mcp-servers   MCP 管理
│   ├── SkillController           /skills        技能管理
│   ├── WorkerController          /workers       Worker 管理
│   ├── SandboxController         /sandboxes     沙箱管理
│   └── ...
├── core/                   # 横切关注点
│   ├── advice/             # ResponseAdvice（统一响应包装）、ExceptionAdvice
│   ├── annotation/         # @AdminAuth, @DeveloperAuth, @AdminOrDeveloperAuth, @PublicAccess
│   ├── security/           # JWT 过滤器、SecurityContext
│   ├── exception/          # BusinessException + ErrorCode
│   ├── event/              # Spring Events（ProductDeletingEvent 等）
│   └── utils/              # 工具类
├── dto/                    # 请求/响应 DTO
│   ├── params/             # 请求参数
│   └── result/             # 响应结果
├── service/                # 业务逻辑
│   ├── impl/               # Service 实现
│   ├── hichat/             # AI 对话模块（SSE + 多 LLM 策略）
│   ├── hicoding/           # 云 IDE 模块（WebSocket JSON-RPC 2.0）
│   ├── gateway/            # 网关集成（APIG/Apsara/Higress/MSE）
│   ├── mcp/                # MCP Server 管理
│   ├── sandbox/            # 沙箱管理
│   ├── vendor/             # 供应商集成
│   └── task/               # 异步任务
└── (himarket-dal)
    ├── entity/             # JPA 实体
    ├── repository/         # Spring Data JPA Repository
    ├── converter/          # 类型转换器
    └── support/            # 枚举与支持类
```

### 关键约定

- 响应包装：Controller 直接返回业务对象，`ResponseAdvice` 自动包装为 `{"code":"SUCCESS","data":{...}}`，禁止手动包装
- 业务异常：`throw new BusinessException(ErrorCode.XXX, detail)`，`ExceptionAdvice` 统一处理
- 认证注解：`@AdminAuth`(管理员)、`@DeveloperAuth`(开发者)、`@AdminOrDeveloperAuth`(两者皆可)、`@PublicAccess`(无需认证)
- 安全：无状态 JWT，Token 有效期 7 天
- 数据库：Flyway 迁移（`himarket-bootstrap/src/main/resources/db/migration/`），MariaDB/MySQL
- 代码风格：Spotless + Google Java Format（AOSP），编译阶段自动检查
- 事件驱动：Spring Events 实现模块间松耦合（`ProductDeletingEvent`、`PortalDeletingEvent` 等）

> 后端代码规范详见：`himarket-server/BACKEND_CODING_STANDARDS.md`

## 4. 前端架构（himarket-web）

两个独立前端应用，均基于 React 18 + TypeScript + Vite + Ant Design + Tailwind CSS：

| 应用 | 路径 | 说明 |
|------|------|------|
| 开发者门户 | `himarket-web/himarket-frontend/` | 面向开发者的产品浏览、订阅、AI 对话、HiCoding |
| 管理后台 | `himarket-web/himarket-admin/` | 面向管理员的产品管理、网关配置、用户管理 |

技术栈：React Router、Axios、i18next 国际化、Monaco Editor、react-markdown。

> 前端各自有独立的 `package.json`，在对应目录下 `npm install && npm run dev` 启动。
>
> 前端代码规范详见：
> - 开发者门户：`himarket-web/himarket-frontend/FRONTEND_CODING_STANDARDS.md`
> - 管理后台：`himarket-web/himarket-admin/ADMIN_CODING_STANDARDS.md`

## 5. 本地开发及验证流程

### 5.1 数据库访问

数据库连接信息通过以下方式提供（优先级从高到低）：
- shell 环境变量（直接 export 或写入 `~/.zshrc` / `~/.bashrc`）
- `~/.env` 文件（`scripts/run.sh` 启动时会自动 source）

需要包含以下变量：`DB_HOST`, `DB_PORT`(3306), `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`

```bash
mysql -h "$DB_HOST" -P "$DB_PORT" -u "$DB_USERNAME" -p"$DB_PASSWORD" "$DB_NAME" -e "YOUR_SQL_HERE"
```

注意事项：
- 只执行 SELECT 查询，除非用户明确要求修改数据
- 不要在回复中展示完整的密码、密钥等敏感字段

### 5.2 启动后端服务

```bash
./scripts/run.sh
```

脚本自动完成：加载 `~/.env` → 优雅关闭旧进程 → 编译打包 → 后台启动 jar → 轮询等待就绪。
退出码为 0 表示启动成功，非 0 表示失败。

### 5.3 接口验证

后端运行在 `http://localhost:8080`，接口路径不带 `/portal` 前缀。使用 JWT Bearer Token 认证。

**⚠️ curl 验证规范（必须遵守）**：

1. **每个 curl 调用独立执行**：禁止在一个 shell 命令中串联多个 curl 调用。一个命令只做一件事，分步执行、逐步验证。
2. **用临时文件传递数据**：curl 输出写入 `/tmp/hm_*.json`，后续用 `python3 -c` 单独解析。避免在 shell 命令中内联 Python 解析（zsh glob 会把 `['key']` 当文件匹配导致报错）。
3. **Token 获取模板**：
   ```bash
   # Step 1: 登录，结果写文件
   curl -s -X POST http://localhost:8080/admins/login \
     -H 'Content-Type: application/json' \
     -d '{"username":"admin","password":"admin"}' > /tmp/hm_login.json

   # Step 2: 提取 token（独立命令）
   python3 -c "import json; print(json.load(open('/tmp/hm_login.json'))['data']['access_token'])" > /tmp/hm_token.txt
   ```
4. **业务接口调用模板**：
   ```bash
   TOKEN=$(cat /tmp/hm_token.txt)
   curl -s -H "Authorization: Bearer $TOKEN" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [higress-group/himarket](https://github.com/higress-group/himarket) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
