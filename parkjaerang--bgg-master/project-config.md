---
trigger: always_on
description: > 本文档供代码 Agent、开发人员和运维人员共同使用。它定义工作方式、架构边界、业务不变量、验证门槛和发布流程；项目方向和未完成任务以 [`PROJECT_PLAN.md`](PROJECT_PLAN.md) 为准。
---

# BGG Agent、开发与运维指南

> 适用范围：本仓库及其子目录。
>
> 最后整理：2026-07-10。
>
> 本文档供代码 Agent、开发人员和运维人员共同使用。它定义工作方式、架构边界、业务不变量、验证门槛和发布流程；项目方向和未完成任务以 [`PROJECT_PLAN.md`](PROJECT_PLAN.md) 为准。

## 1. 信息来源与优先级

遇到描述不一致时，按以下顺序判断：

1. 当前代码、Alembic migration、实际配置校验和运行结果。
2. 本文档中的仓库规则与业务不变量。
3. `PROJECT_PLAN.md` 中的当前风险、优先级和任务。
4. `README.md`、`README.zh-CN.md` 和 `deploy/DEPLOY_LAN.md` 中的使用说明。
5. Git 历史仅用于追溯，不作为当前行为的直接依据。

上级目录中的 `report_board`、`Post` 和 `chat_bot` 文档只提供工程经验，不是 BGG 的配置来源。不要直接复制它们的端口、密钥、数据模型、命令或权限规则。

`BGG_frontend/` 只作为前端开发时的页面结构、视觉样式、交互效果、文案和图片素材参考，不是当前应用的正式代码或配置来源。可以选择性地把其中合适的前端表现迁入正式 `frontend/`，但不得复制或依赖其中基于 `localStorage` 的业务数据、模拟 API、认证、路由、数据结构或运行配置。所有关键设置、后端行为、数据库模型、Alembic migration、权限规则、API 契约、部署方式和业务事实均以本 Git 仓库中的当前正式代码、配置及实际运行结果为准。

## 2. 接手任务时的固定流程

### 2.1 开始前

- 确认当前目录是 `/home/merrill/report-board/BGG`。
- 执行 `git status --short --branch`，区分已有修改和本次修改。
- 阅读 `PROJECT_PLAN.md`，确认任务优先级及已知阻塞。
- 只打开与任务有关的代码、migration 和文档；行为以真实代码为准。
- 检查 `.env`、数据库和上传目录是否属于生产或真实业务数据。
- 若任务会改变 schema、删除数据、发邮件、开放端口或重启服务，先明确备份和影响范围。

### 2.2 实施中

- 保留用户已有的未提交修改，不覆盖、不回滚、不顺手格式化无关文件。
- 先修根因，再补兼容；临时兼容必须写明删除条件。
- 路由负责 API 边界，Pydantic model 负责输入输出，service 负责可复用业务事务。
- 写操作的权限和业务校验必须由后端执行，前端隐藏按钮不能代替鉴权。
- 修改数据库模型时同步创建 Alembic migration，不在启动时隐式修改 schema。
- 修改前端功能时同步检查韩文、中文和英文文案。
- 修改高风险行为时先定义失败、取消、并发和重试语义。

### 2.3 结束前

- 运行与风险相匹配的检查，记录实际命令和结果。
- 执行 `git diff --check`，检查意外文件和敏感信息。
- 更新应受影响的文档，但不要创建重复计划或完成报告。
- 交付说明包含：改了什么、验证了什么、还剩什么、是否需要迁移或运维动作。

## 3. 项目地图

```text
BGG/
├── AGENTS.md                 # Agent、开发和运维共同规则
├── PROJECT_PLAN.md           # 方向、风险、阶段和未完成任务
├── README.md                 # 韩文使用入口、API 和配置概览
├── README.zh-CN.md           # 中文使用入口、API 和配置概览
├── .env.example              # 可提交的配置模板，不含真实密钥
├── run.sh                    # 建环境、执行迁移、启动 8100
├── deploy/
│   ├── bgg.service           # systemd 服务定义
│   └── DEPLOY_LAN.md         # 局域网运行手册
├── backend/
│   ├── main.py               # FastAPI 入口、路由和静态前端挂载
│   ├── app/
│   │   ├── config.py         # 环境变量和生产配置保护
│   │   ├── database.py       # async engine、session、SQLite WAL
│   │   ├── dependencies.py   # 管理端鉴权依赖
│   │   ├── orm_models.py     # SQLAlchemy ORM
│   │   ├── models/           # Pydantic schema
│   │   ├── routers/          # API 路由
│   │   └── services/         # 预约、SSE、邮件等业务逻辑
│   └── alembic/versions/     # schema 变更的唯一迁移历史
├── frontend/
│   ├── *.html                # 原生页面
│   ├── js/                   # 页面逻辑、API 调用、i18n
│   ├── css/                  # 公共和页面样式
│   └── img/                  # 版本化静态图片
└── uploads/                  # 运行期上传文件，不提交内容
```

## 4. 运行架构

BGG 是 FastAPI 直接服务原生 HTML/CSS/JS 的同源应用：

```text
Browser
  ├── /, /*.html, /css, /js, /img
  ├── /api/* --------------------> FastAPI routers
  ├── /api/sse ------------------> SSE client registry
  └── /uploads/* ----------------> uploaded files

FastAPI routers
  ├── Pydantic validation
  ├── require_auth for admin writes
  ├── SQLAlchemy AsyncSession
  └── services for transaction/mail/SSE

Database
  ├── SQLite: development and single-host verification
  └── PostgreSQL: intended production database
```

API 路由必须在通配页面路由之前挂载。新增页面路由时不得吞掉 `/api/*`、`/uploads/*` 或静态资源请求。

## 5. 核心业务不变量

### 5.1 医院与内容

- 医院 ID 是数据库主键，不使用数组位置或页面顺序作为身份。
- 详情页、预约配置、封锁时段、促销和评价通过医院 ID 关联。
- 前端业务数据以 API 为准，不重新引入 localStorage 作为事实来源。
- localStorage 只允许保存语言、活动标签等纯 UI 偏好。
- 删除医院前必须明确关联预约、内容和上传文件的保留策略。

### 5.2 预约与时段

- 同一医院、日期和时间只能有一个 `blocked_slots` 记录，数据库唯一约束是最终并发防线。
- 创建预约和写入 `reason=reservation` 的封锁时段必须在同一事务中完成。
- 冲突必须返回 `409`，不能静默覆盖或创建双重预约。
- 取消预约时只释放该预约产生的封锁时段，不删除管理员手动封锁。
- 改动预约或时段后应通知对应医院的 SSE 客户端刷新可用性。
- 用户查询接口只返回完成查询所需的最少字段，不暴露后台备注或其他用户资料。

### 5.3 CSV 和批量操作

- “预览”必须是无副作用操作；用户确认前不得修改数据库。
- “替换”和“合并”必须有明确、可测试的服务端语义。
- 取消、解析失败或保存失败时，原数据必须保持不变。
- 批量导入需要限制大小、编码和字段，并报告跳过或失败的记录。

当前 CSV 预览仍违反以上规则，属于 `PROJECT_PLAN.md` 的 `P0-02`，不要基于现状继续扩展。

### 5.4 认证与权限

- 目标状态是 Cookie-only 管理认证：JWT 存 HttpOnly Cookie，不返回给 JavaScript。
- 所有管理端写接口使用统一后端鉴权依赖。
- 公开接口只开放预约提交、最小化预约查询、公开内容读取和 SSE 等必要能力。
- 登录失败不能泄漏密码、token、内部路径或配置。
- `ENV=production` 下弱默认密钥和默认管理员密码必须阻止启动。

当前 Bearer/sessionStorage 兼容路径属于 `PROJECT_PLAN.md` 的 `P0-04`，新代码不得继续依赖它。

### 5.5 邮件与外部服务

- API 返回成功不等于邮件已经投递；界面文案必须区分保存、排队、成功和失败。
- SMTP 密码、Google Maps key 等第三方凭据只放环境变量或服务端安全配置。
- 外部服务失败不能破坏已提交的核心业务事务。
- 用户姓名、电话、邮箱、咨询和预约内容属于敏感数据，不写入普通调试日志。

## 6. 后端开发规则

- Python 使用 4 空格、snake_case、类型标注和 SQLAlchemy 2.x 风格。
- 路由只做参数、权限和响应边界；跨路由复用逻辑放入 `backend/app/services/`。
- API 输入输出放入 `backend/app/models/`，避免直接把任意 dict 当作长期契约。
- 状态字段使用明确允许值，不接受任意字符串。
- 事务中任何异常都应 rollback，再返回稳定的 HTTP 错误。
- 不在异常响应中返回数据库 SQL、绝对路径、密钥或 traceback。
- SQLite 和 PostgreSQL 都要支持的字段与查询，不使用仅单一数据库可用的隐式行为。
- 时间字段新增或重构前先定义时区策略；不要继续混用不明确的 naive UTC、本地时间和字符串日期。

## 7. 数据库与迁移规则

- `backend/app/orm_models.py` 是当前 ORM，`backend/alembic/versions/` 是 schema 历史。
- 任何新增表、字段、索引、唯一约束或类型变化都必须有 migration。
- migration 应支持空库升级和已有库升级，并尽量可重复验证。
- 不删除或改写已发布 migration；用新的 revision 修正。
- 应用启动不自动 `create_all()` 或修改生产 schema。
- 发布前备份数据库和上传目录，迁移后检查当前 revision 和关键数据量。
- SQLite 文件复制备份时应确保一致性，不能忽略活动中的 WAL；优先在维护窗口停止写入或使用 SQLite backup 方法。
- PostgreSQL 使用数据库原生备份工具，并实际验证恢复。

已知问题：当前环境中 `alembic current` 和 `alembic upgrade head` 可能不退出，见 `P0-01`。在该问题解决前，不把迁移超时误报为成功，也不绕过迁移直接发布。

## 8. 前端开发规则

- 继续使用原生 HTML/CSS/JS，不为单个需求引入前端框架或构建链。
- 同源 API 基址由 `frontend/js/common.js` 的 `API_BASE` 约定管理。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [parkjaerang/BGG_master](https://github.com/parkjaerang/BGG_master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
