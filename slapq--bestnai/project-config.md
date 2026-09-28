---
trigger: always_on
description: 本文件为在 `BestNAI` 项目中工作的 AI coding agent 提供项目级指引。除非用户明确指定其他目录，本文件适用于 `BestNAI/` 下的全部代码与文档。
---

# AGENTS.md

本文件为在 `BestNAI` 项目中工作的 AI coding agent 提供项目级指引。除非用户明确指定其他目录，本文件适用于 `BestNAI/` 下的全部代码与文档。

## 项目概览

BestNAI 是一个基于 FastAPI 的 NovelAI 图片生成网关，包含：

- NovelAI 图片生成 API 封装
- 多 Key FIFO 调度与单 Key 单并发租约机制
- 高层/低层生成接口
- SSE 流式生成接口
- BYOK（用户自带 NovelAI Key）接口
- OpenAI / NewAPI 兼容接口
- 图片上传、内存缓存、`upload_id:<id>` 复用
- Anlas 预估与模型别名点数限制
- 简单前端与管理页面

这是一个功能堆叠较多但结构已经开始失控的项目。后续维护应优先减少复杂度、隔离 API 交互核心、修正 Anlas 估算与文档/实现不一致的问题。

## 技术栈

- Python 3.12+
- FastAPI
- Uvicorn
- Pydantic v2
- Pillow
- python-dotenv
- python-multipart
- NovelAI SDK，通过 `app/sdk_loader.py` 动态加入 `sys.path`

依赖文件：

```bash
pip install -r requirements.txt
```

启动：

```bash
python run.py
```

默认地址：

```text
http://127.0.0.1:8010/
```

## 重要目录与文件

```text
BestNAI/
├── app/
│   ├── main.py          # FastAPI app、路由、错误处理、OpenAI 兼容层、BYOK、前端挂载
│   ├── service.py       # 业务逻辑：payload 规范化、Anlas 估算、图片编码/ZIP
│   ├── scheduler.py     # 多 NovelAI Key 调度与 Lease
│   ├── image_store.py   # 内存图片缓存与 upload_id 解析
│   ├── model_alias.py   # 模型名 -anlas-N / -points-N 解析
│   ├── auth.py          # Bearer 认证与 admin/client 权限
│   ├── config.py        # 环境变量配置
│   ├── errors.py        # 平台错误码与错误响应
│   ├── schemas.py       # Pydantic 响应模型
│   └── sdk_loader.py    # NovelAI SDK 路径发现
├── frontend/
│   ├── index.html       # 单页 UI
│   └── admin-keys.html  # Key 管理 UI
├── docs/                # 架构、API、用户、部署文档
├── run.py               # 本地启动入口
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

## 环境变量

不要提交真实 `.env` 或泄露 Key。

关键配置来自 `app/config.py`：

- `BESTNAI_API_KEYS` / `NOVELAI_API_KEYS` / `NOVELAI_API_KEY`
- `BESTNAI_CLIENT_SECRETS`
- `BESTNAI_ADMIN_SECRETS`
- `BESTNAI_CORS_ORIGINS`
- `BESTNAI_HOST`
- `BESTNAI_PORT`
- `BESTNAI_QUEUE_TIMEOUT`
- `BESTNAI_DEFAULT_MAX_ANLAS`
- `BESTNAI_DEFAULT_IS_OPUS`
- `BESTNAI_REQUIRE_AUTH`
- `BESTNAI_MAX_REQUEST_BYTES`
- `BESTNAI_MAX_UPLOAD_BYTES`
- `BESTNAI_MAX_IMAGE_STORE_ENTRIES`
- `BESTNAI_SDK_SRC`

`Settings.from_env()` 在没有 NovelAI API Key 时会抛错，测试导入主应用时要注意。

## 核心请求流

### 高层生成

1. `POST /v1/generate/high-level`
2. `main.generate_hl()` 调用 `estimate_high_level()`
3. `service.estimate_high_level()`：
   - 拒绝中文/CJK prompt
   - 规范化 `size`
   - 解析 `upload_id:<id>`
   - 解析模型 alias
   - 使用 NovelAI SDK 的 `GenerateImageStreamParams` / `GenerateImageParams` 校验
   - 调用 SDK Anlas 估算
4. `enforce_anlas_limit()` 检查点数限制
5. `Scheduler.acquire()` 获取 Key
6. `lease.client.image.generate_stream(params)` 调用 NovelAI
7. 收集 final 图像并返回 JSON / binary / ZIP

### 低层生成

- `POST /v1/generate/low-level`
- `POST /ai/generate-image`
- 直接使用 SDK 的 `ImageGenerationRequest` / `StreamImageGenerationRequest`

### 流式生成

- `POST /v1/generate/high-level-stream`
- `POST /v1/generate/low-level-stream`
- `POST /ai/generate-image-stream`
- 返回 `text/event-stream`
- 错误通过 `event: error` 发送

注意：文档里提到 fake-stream 默认行为，但当前 `app/main.py` 的实现并未真正实现 `fake_stream` 参数分支。不要在新文档中继续扩大这个不一致。

### BYOK

- `POST /v1/generate/byok`
- `POST /v1/generate/byok-stream`
- 需要 `X-NAI-Key: pst-...`
- 直接创建 `AsyncNovelAI(api_key=user_key)`，不走全局 Scheduler

### OpenAI / NewAPI 兼容

- `POST /v1/chat/completions`
- `POST /v1/images/generations`
- `GET /v1/models`

`/v1/chat/completions` 的 user message 内容优先解析为 BestNAI 原生 JSON；解析失败时将纯文本作为 prompt。

## 已知问题与维护警告

### 1. Anlas 估算不可盲信

当前 Anlas 估算依赖 SDK 的：

- `calculate_anlas()`
- `calculate_anlas_from_params()`

并且大量地方硬编码 `is_opus=True`。`Settings.default_is_opus` 存在但没有被实际贯穿使用。因此：

- 非 Opus 用户的消耗可能估错
- NovelAI 官方计费规则更新后可能估错
- 特殊功能（Vibe、Character Reference、i2i、inpaint、多图、多角色）容易与真实扣点不一致
- alias 中的 `-anlas-N` 只是本地软限制，不代表真实 NovelAI 余额或官方计费

修改点数逻辑时必须增加可复现用例，并在文档中明确“估算值不是官方扣费凭证”。

### 2. 结构混乱

`app/main.py` 过大，混合了：

- 路由定义
- NovelAI 异常映射
- 响应构造
- BYOK 逻辑
- OpenAI 兼容逻辑
- 前端静态挂载
- 管理接口

新增功能前应优先拆分，而不是继续往 `main.py` 堆代码。推荐拆分方向：

- `routes/generate.py`
- `routes/estimate.py`
- `routes/byok.py`
- `routes/openai_compat.py`
- `routes/admin.py`
- `core/novelai_client.py`
- `core/response.py`
- `core/anlas.py`

### 3. 文档与实现不完全一致

例如：

- 文档提到 fake-stream，但实现中没有相应 query 参数控制
- README 宣称生产导向，但内存图片缓存、简单 token auth、无审计/限流都不够生产级
- 部分文档描述可能来自计划而非已完成实现

维护时以源码为准；修改行为后同步更新 `docs/`。

### 4. 安全性有限

- Bearer token 只是静态 secret
- 默认 CORS 为空时会开放为 `*`
- 无速率限制
- 无请求体大小中间件，仅部分上传限制
- 上传图片保存在内存中
- admin key 管理只存在进程内，重启丢失

面向公网部署时必须额外加反代鉴权、TLS、限流、日志脱敏和请求体限制。

### 5. 状态不可持久化

以下数据均为内存状态：

- 上传图片缓存
- Scheduler 动态添加/禁用/备注的 key 状态
- 队列状态

不要假设重启后仍存在。

### 6. SDK 路径加载脆弱

`app/sdk_loader.py` 会尝试多个本地路径。跨机器、Docker、CI 环境中可能失败。修改时优先支持显式 `BESTNAI_SDK_SRC`，不要硬编码个人机器路径。

## 编码约定

- 使用 Python 类型标注。
- 维持 Pydantic v2 写法，例如 `model_validate()`、`model_dump()`。
- FastAPI endpoint 应尽量薄；复杂逻辑放到 service/core 模块。
- 对外错误应使用 `PlatformError` 与 `ErrorCodes`，避免直接泄露底层异常。
- 不要打印完整 API Key。使用 `_mask_key()` 风格脱敏。
- 不要在日志、异常、响应中返回完整 `X-NAI-Key`、`BESTNAI_API_KEYS`。
- 图片返回支持 JSON base64、binary、ZIP 时，要同步维护 metadata。
- SSE 错误保持 `event: error` + JSON payload。

## 测试与验证

当前项目没有正式测试套件。修改后至少执行：

```bash
python -m compileall app run.py
```

如环境变量和 SDK 可用，再执行：

```bash
python run.py
```

手动检查：

```bash
curl http://127.0.0.1:8010/health
```

涉及 API 行为时，至少验证：

- `/health`
- `/v1/estimate/high-level`
- `/v1/generate/high-level`（可用 Key 时）
- `/v1/generate/high-level-stream`（可用 Key 时）
- `/v1/upload/image`
- `/v1/chat/completions`（若改兼容层）

涉及调度器时，检查：

- 多 Key 轮询
- 忙碌 Key 不被重复分配
- timeout 后 waiter 被清理

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Slapq/BestNAI](https://github.com/Slapq/BestNAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
