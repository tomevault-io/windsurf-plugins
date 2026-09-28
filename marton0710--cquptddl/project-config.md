---
trigger: always_on
description: 给 AI/新人的跨会话速查手册。先读这里，别再从零探索。约 4k 行 Python，全部代码在 `src/cquptddl/`。
---

# AGENTS.md — CQUPTDDL（重邮聚合截止线）后端

给 AI/新人的跨会话速查手册。先读这里，别再从零探索。约 4k 行 Python，全部代码在 `src/cquptddl/`。

## 1. 项目是什么

聚合 **学在重邮 / 学习通 / 雨课堂** 三平台的作业，提供 REST API；可选把作业同步为 **Meet课程表** 事件，并通过 QQ 机器人（qqchan）推送临期/新作业提醒。

- Python **>=3.14**（用到 `uuid7`、`itertools.batched`），包管理 **uv**。
- 入口：`cquptddl:main` → `uvicorn cquptddl:app`（见 `src/cquptddl/__init__.py`）。
- 运行：`uv run cquptddl`；依赖装到 `.venv/`（含 `uvicorn`、`dotenv` 等可执行文件）。
- **无项目级 lint/类型检查配置**（`pyproject.toml` 里没有 ruff/ty 段，也没有 `ruff.toml`/`ty.toml`），但开发者**全局安装了 `ty` 与 `ruff`**，改动后必须用它们检查（见 §7）。`tests/` 有 `test_ics_feed.py`（ICS 渲染的纯函数单测）和 `test_ics_subscription.py`（用 `aiosqlite` 内存库跑订阅的增删改查与拉取统计），都用 pytest；dev 依赖 `aiosqlite`、`respx`、`pytest`，其中 `respx` 目前未被任何代码使用。
- `README.md` 是空文件。分支：当前 `v2`（开发分支），`master` 落后于 `v2`；远端 `git@github.com:marton0710/CQUPTDDL.git`。
- 部署：`Dockerfile` 用 python:3.14-alpine，从自建源 `pypi.bail.asia` 装包，前端目录挂到 `/fe`（`FRONTEND_DIR`），暴露 8000。

## 2. 架构骨架（务必先理解这 4 个机制）

### 2.1 依赖注入：`core.factory`
- `core.depends_session` / `depends_client`：FastAPI 依赖，交出 session/client。
- `core.get_session()` / `get_client()`：把上面两个包成 `asynccontextmanager`，**给非路由代码（事件回调、后台任务）用**。
- `depends_session` 在 `yield` **之后** `commit()`。所以路由里不 commit 也会落库；但后台任务用 `get_session()` 时同样会自动 commit —— 不要让同一 session 并发。
- `depends_client` 自动加 `User-Agent: CQUPTDDL/<版本>`，传 `headers=` 时是合并而非覆盖。

### 2.2 符号表 `core.symbol`：服务层解耦层
`core.export("名字", 函数)` / `core.call("名字", *args)`。**跨 service 调用一律走符号表**，不直接 import 别的 service。
全部导出名（grep `core.symbol.export` / `core.export` 可复核）：
- `auth.*`：`password_login`、`get_login_qrcode`、`qrcode_login`、`relogin`、`get_user_from_token`、`refresh_token`、`logout`、`delete_account`
- `crypto.aes_encrypt` / `crypto.aes_decrypt`
- `platform.get_auth_method` / `bind` / `unbind` / `valid_cookie` / `fetch_homework`
- `homework.refresh_homework` / `complete` / `get_cached_homework` / `get_cached_homework_count` / `get_last_refresh_time` / `get_user_dying_homeworks` / `get_user_homeworks_with_deadline`
- `qqpush.configure` / `get_user_config_from_qqchan_id` / `push_dying_homeworks`
- `meetschedule.bind` / `unbind`
- `ics.list_subscriptions` / `create_subscription` / `delete_subscription` / `render_feed`
⚠️ 名字重复 `export` 会抛 `NameError`；改函数名要同步改 `call()` 里的字符串（无类型检查兜底）。

### 2.3 事件总线 `core.bus`（abxbus）
模块**导入时**注册 `core.bus.on(Event, handler)`，靠 `service/__init__.py` 里 `from . import ...` 触发注册 —— **新增 service 模块必须在 `service/__init__.py` 导入**，否则事件不生效。
事件定义全在 `model/event/__init__.py`：`UserRegisterEvent`（建 `QQPushConfig`）、`UserReloginRequiredEvent`、`HomeworkRefreshedEvent`（带 `new_homework_ids`）、`HomeworkDoneEvent`、`PlatformBound/Unbound`（增删刷新 job + 删作业）、`AutoRefreshHomeworkFailedEvent`、`AccountDeletedEvent`、`QQPushConfigChangedEvent`、`InvalidQQChanIDEvent`。

### 2.4 跨 service 传 `User` 还是 `user_id`（重要约定）
service 层接口**刻意不统一**：一部分收 ORM 实体 `User`，一部分只收 `user_id: str`。这不是没改完，按下面规则判断，别"顺手统一"：

- **传 `user_id: str`**：只需要按 uid 过滤 / 归属校验 / 建索引的接口。例：`homework.get_cached_homework` / `get_cached_homework_count` / `get_last_refresh_time` / `complete`、`platform.valid_cookie`、`qqpush.configure`、`platform.unbind`、`meetschedule.bind/unbind`、全部 ics 接口。
- **传 `user: User`**：需要读或**回写** `password` / `ids_cookie` / `token_version` / `name` 的接口，或作为**公开扩展点契约**的接口。例：
  - `auth.relogin` / `login` / `logout` —— 读 `password`（AES 解密）、回写 `ids_cookie` 和 `token_version`。
  - `platform.login` → `base/utils.login_with_ddl_account(user, service)` —— 读 `ids_cookie`。
  - `auth.delete_account` 及其钩子（`before_delete_user_hook` 的签名 `Callable[[AsyncSession, User], Awaitable[None]]`，最后 `session.delete(user)`）—— **钩子是给别的 service 用的通用接口，实体就是契约的一部分**，不要退化成 `user_id`。`meetschedule/event_handlers.on_delete_user` 虽然自身只用 `user.id`，也必须跟着签名走。
- ⚠️ **改签名前先看调用链下游**：`platform/fetch.fetch_homework` 自己只用到 `user.id`，但因为它内部要调 `auth.relogin(session, user, ...)`，所以必须继续收 `User`——这是**传递性依赖**，不是漏改。
- 属性使用分布（改动前实测）：`user.id` 40 处（可退化）、`token_version` 8、`password` 4、`ids_cookie` 3、`name` 1（均不可退化）。重构时以这个比例判断边界在哪。

### 2.5 后台任务
- `core/task.background(coro, name)` 立即跑；`register()` + 启动后 `task.start()` 用于事件循环前注册。
- APScheduler 三处：`homework/refresh_task.scheduler`（按平台间隔刷新作业，job id `homework_fetch_schedule_{uid}_{platform}`，间隔 `homework_cache_base_ttl`+jitter）、`qqpush/globals.scheduler`（推送策略）、`meetschedule/refresh_task.scheduler`（同步）。
- 生命周期：`cquptddl/__init__.py:lifespan` → `core.init()`（建表 + 启 task）→ `service.init()`（建刷新 job、qqpush on_boot、meetschedule start_refresh）；关闭顺序相反。

## 3. 目录职责

```
core/       config(pydantic-settings 读 .env) / db(engine+建表) / factory / event_bus / task / symbol
model/db/   SQLModel 表：User, Homework, PlatformInfo, QQPushConfig, MeetscheduleConfig, MeetscheduleEntry, IcsSubscription
model/schema/  pydantic API 模型与枚举（PlatformEnum、AuthMethod）
model/event/   事件定义
router/api/ FastAPI 路由：auth / homework / ics / platform / qqpush / meetscheule(拼写如此)
middleware/ need_login(token Cookie) / verify_api_key(X-API-Key)
service/    auth, crypto, homework, ics, platform, qqpush, meetschedule
exc.py      所有业务异常（继承 HTTPException，带 status/detail）
```
注意：`model/db/__init__.py` 的 `__all__` 里 `LastRefreshTime`、`PlatformCookies` **已不存在**，是历史残留。

## 4. 关键数据模型

- **User**：主键 `id` = 真实统一认证码；`password` 为 AES 密文（扫码登录为 `None`）；`token_version: UUID` 用于退出登录时批量失效 token。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marton0710/CQUPTDDL](https://github.com/marton0710/CQUPTDDL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
