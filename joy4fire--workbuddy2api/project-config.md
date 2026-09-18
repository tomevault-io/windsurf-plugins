---
trigger: always_on
description: > 本文件面向 AI 编码代理，读完即可安全地修改本项目。
---

# AGENTS.md — Workbuddy2API 项目代理指南

> 本文件面向 AI 编码代理，读完即可安全地修改本项目。
> **当前待修复问题清单：[CODE_REVIEW_TODO.md](./CODE_REVIEW_TODO.md)**（P0–P3 分级，含位置、修复方案、验收标准——优先按它干活）。
> 人类向文档：`README.md` / `README_EN.md`。

---

## 1. 项目是什么

单用户 FastAPI 网关：把腾讯 WorkBuddy/CodeBuddy（`copilot.tencent.com`）的积分额度包装成三种标准 LLM API 协议，供本地工具（Claude Code、Codex CLI、Cherry Studio 等）调用；附带 SQLite 用量统计与 Vue3 管理界面（WebUI）。

**请求主链路（务必先理解）**：

```
客户端（sk-xxx Key）
  │  POST /v1/chat/completions | /v1/messages | /v1/responses
  ▼
app.py 端点
  ├─ _check_api_key()      # apps 表按 sha256 查 Key → 得到应用名（用于记账归因）
  ├─ _pick_account()       # pool.py 加权随机选号（冷却/额度/到期/优先级多因子）
  ├─ 协议适配器            # anthropic.py / responses.py → 统一转成 OpenAI Chat 格式
  ├─ _enhance_body()       # reasoning.sanitize_body（tool_choice 归一化 + effort 降级）
  │                        # + 可选 desensitize（system/developer 零宽空格防审核误伤）
  ├─ build_upstream_body() # upstream.py 白名单透传 + 强制 stream=true
  └─ _open_upstream()      # 限速器 → get_headers()（token 自动刷新）→ 预取首个 SSE 事件
  ▼                        #（预取保证上游失败时能返回正确 HTTP 状态码，而不是 200+空流）
upstream.py stream_upstream()  # 模块级共享 httpx.AsyncClient，直连腾讯 /v2/chat/completions
  ▼
流式：逐行清洗/转换回原协议格式 → StreamingResponse
非流式：collect_upstream() 或 converter 聚合 → JSONResponse
  ▼
_log_usage()  # 完整输入/输出/思考链入库（刻意全量保留，见 §6 不变量）
```

---

## 2. 技术栈

| 层 | 技术 |
|---|---|
| 后端 | Python 3.11+，FastAPI + uvicorn，httpx（全部 `trust_env=False` 绕代理），标准库 sqlite3 |
| 前端 | Vue 3 + TypeScript + Vite，Ant Design Vue，ECharts（按需引入），hash 路由 |
| 数据 | 单个 SQLite（WAL 模式，单连接 + `threading.Lock`），schema 用 `PRAGMA user_version` 版本化迁移 |
| 部署 | Docker（python:3.11-slim）+ compose；宿主机 `data/`、`auths/` 挂载卷持久化 |

---

## 3. 目录地图

```
Workbuddy2API/
├── workbuddy_one/            # 后端包（唯一 Python 包）
│   ├── app.py                # FastAPI 装配：依赖初始化 + 路由注册 + 安全中间件（~150 行，不放业务逻辑）
│   ├── context.py            # GatewayContext：db/pool/models/scheduler/managers/limiters 共享依赖
│   ├── routes/               # 按业务域拆分的路由模块（每个暴露 register(app, ctx)）
│   │   ├── inference.py      # 三协议推理端点（chat/messages/responses）+ count_tokens + /v1/models
│   │   ├── accounts.py       # 账号列表/启停/优先级/删除 + 上传 + 扫码 OAuth
│   │   ├── apps.py           # 应用 API Key CRUD
│   │   ├── usage.py          # 使用记录（分页/搜索/详情/筛选/瘦身）
│   │   ├── models_admin.py   # 模型目录 + AA 评测
│   │   ├── overview.py       # 概览聚合（含积分预警计算）
│   │   ├── settings.py       # 设置读写（预警/别名等）
│   │   └── webui.py          # /health + WebUI 静态托管（catch-all，必须最后注册）
│   ├── gateway/              # 推理链路可复用逻辑（与路由解耦，函数首参 GatewayContext）
│   │   ├── inference.py      # 鉴权/选号/限速/请求体增强(别名+裁剪+思考)/上游连接重试/用量记账
│   │   ├── attachments.py    # DSH 附件归档 + 输入文本提取（记录用，完整入库）
│   │   ├── sse.py            # SSE 增量解析/Chat 行清洗/流式心跳（pump+队列）
│   │   └── errors.py         # 错误响应构造（safe_err/json_error/err_anthropic/conv_usage）
│   ├── upstream.py           # 上游转发：白名单构造 body、SSE 流、非流式聚合、共享 AsyncClient
│   ├── pool.py               # 账号池：加权随机选号（快到期优先 + 额度/成功率/闲置/优先级因子）、冷却
│   ├── db.py                 # SQLite 层：4 张表 CRUD、版本化迁移框架、用量统计聚合
│   ├── scheduler.py          # asyncio 后台循环（每 60s）：签到/保活/模型刷新/AA 刷新/每日清理
│   ├── models.py             # 模型目录：上游动态拉取 + TTL 缓存 + MODALITY_OVERRIDE 权威模态表
│   ├── benchmarks.py         # Artificial Analysis 评测数据（24h 缓存，key 存 DB settings）
│   ├── credentials.py        # auth 文件读取、token 过期判定与自动刷新（原子写回）
│   ├── oauth.py              # 扫码登录（设备授权流）：oauth_begin / oauth_poll
│   ├── billing.py            # 额度查询（新三接口+旧接口降级）、每日签到；浏览器 UA 绕 WAF
│   ├── reasoning.py          # sanitize_body：tool_choice 归一化 + effort 降级 + developer/思维链归一 + 别名解析 + token 估算
│   ├── desensitize.py        # 敏感词零宽空格注入（仅 system/developer 角色，默认开）
│   ├── ratelimit.py          # 账号级最小间隔限速（默认 1.5s ± 抖动）
│   ├── _crypto.py            # 应用 Key 可逆加密（主密钥 data/.secret_key）
│   ├── config.py             # 环境变量/.env 配置（dataclass）
│   └── __main__.py           # CLI 入口：python -m workbuddy_one [--login]
├── frontend/
│   ├── src/api/              # http.ts（axios 实例 + 拦截器）+ 按域拆分（accounts/usage/models/apps/settings）+ client.ts 组装
│   ├── src/views/            # Overview / Accounts / Models / Usage / Apps / Records 六页
│   ├── src/components/       # AppSidebar / TokenManager / CheckinSettingsModal / QrLoginModal / RecordDetailModal
│   ├── src/styles/           # base.css（布局）+ dark-theme.css（antd 深色覆盖，见 §7 前端要点）
│   └── dist/                 # 构建产物（跟踪进 git；由后端 app.py 直接伺服；改动前端后必须重新 build）
├── tests/                    # unittest 测试（test_core.py、test_models_scheduler.py，56 个用例）
├── data/                     # 运行时数据：workbuddy.db、attachments/、.secret_key   ←机密，见 §8
├── auths/                    # 账号 auth 文件（.info JSON）                          ←机密，见 §8
├── CODE_REVIEW_TODO.md       # 待修复问题清单（P0–P3）
├── Dockerfile / docker-compose.yml / pyproject.toml / .env.example
```

---

## 4. 常用命令（Windows 环境）

```powershell
# 一切命令在 Workbuddy2API/ 目录下执行；Python 一律用 venv 解释器
cd N:\代码\workbuddy2Api\Workbuddy2API

# 跑测试（unittest，不是 pytest；56 个必须全绿）
.\.venv\Scripts\python.exe -m unittest discover -s tests

# 本地起服务（开发调试用）
.\.venv\Scripts\python.exe -m uvicorn workbuddy_one.app:create_app --factory --host 127.0.0.1 --port 8787

# 前端构建（改了任何 .vue/.ts 后必须执行，否则 WebUI 不更新）
cd frontend; pnpm build; cd ..

# Docker 重建（宿主机 data/、auths/ 是挂载卷，数据不丢）
docker compose up -d --build --force-recreate
```

- 默认端口 8787；本机跑用 127.0.0.1，Docker 里 `HOST=0.0.0.0` + 端口映射。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Joy4Fire/Workbuddy2API](https://github.com/Joy4Fire/Workbuddy2API) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
