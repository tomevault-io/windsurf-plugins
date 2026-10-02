---
trigger: always_on
description: Decis 是"一个 API 跑所有轻量决策模型"的推理服务框架。它把 jev / TypeSafe System One 的线格式实现一次，把各家开源决策模型（kev、Laya 等）作为可插拔引擎接进来。
---

# AGENTS.md — Decis 工程约束

Decis 是"一个 API 跑所有轻量决策模型"的推理服务框架。它把 jev / TypeSafe System One 的线格式实现一次，把各家开源决策模型（kev、Laya 等）作为可插拔引擎接进来。

本文件是**在这个仓库里工作的契约**。它写给 AI agent，也写给人类。规则不是建议，是约束；违反约束的改动即使"能跑"也不接受。

> **当前状态**：契约层（`schema.py` / `render.py` / `answers.py` / `errors.py` / `auth.py`）、引擎抽象、
> `paths.py` 权重解析、`scheduler.py`、`config.py`、`cli.py`（含 `decis download`）、`docker/Dockerfile`
> 与 CI 都已实现，三个真实模型家族跑在同一个契约后面：
> `uv sync --extra laya && uv run decis serve --engine laya-multilingual`、
> `uv sync --extra kev && uv run decis serve --engine kev-0.8b`、
> `uv sync --extra jeff && uv run decis serve --engine jeff-qwen3.5-0.8b`。
> **未实现**：kev 的 prefix 缓存路径、`/metrics`；`docs/design.md` 里那批只在设计稿里出现的旋钮
> （`DECIS_BATCH_MODE`、`DECIS_BATCH_MAX_WAIT_MS`、`DECIS_BATCH_MAX_SIZE`、`DECIS_MAX_QUEUE`、
> `DECIS_MAX_QUESTIONS`、`DECIS_MAX_STATE_CHARS`、`DECIS_CORS_ORIGINS`、`DECIS_HTTP_WORKERS`、
> `DECIS_ENGINE_WORKERS`、`DECIS_RATE_LIMIT_RPM`、`DECIS_RATE_LIMIT_TOKENS_PER_S`）没有任何代码读，别照它们写文档。`docs/design.md §11` 的目录树是
> 目标结构，其中未出现的文件即尚未实现的部分。
>
> **引擎注册表**：出厂六个——`laya`、`laya-multilingual`、`laya-typed-decisions`、`kev-0.8b`、
> `jeff-qwen3.5-0.8b`、`jeff-gemma4-e2b`。其中两个需要第二个仓库：`kev-0.8b` 是适配器 + Qwen3.5-0.8B
> 基座（见 `paths.BaseModel`），而两个 Jeff 是**全权重微调**，一个目录里就是全部，`bases` 为空。
> **注册表里没有测试替身**：无权重的确定性测试替身住在 `tests/fixture_engine.py`，由 `tests/conftest.py`
> 以 id `stub` 只在测试进程里注册，出厂镜像永远不会用它作答。
>
> **镜像**：`.github/workflows/docker-build.yml` 按引擎构建、推送到 Docker Hub 的**一个**仓库
> `chaitin/decis`，**引擎就是 tag**（`chaitin/decis:laya-multilingual`）。命名空间来自工作流的
> `IMAGE_NAMESPACE`（默认 `chaitin`，仓库变量 `DOCKERHUB_NAMESPACE` 可覆盖），**不从
> `DOCKERHUB_USERNAME` 推导**——那是登录身份，不是发布目标。tag 规则：默认分支只给引擎名，其中
> `laya-multilingual` 的内置权重变体另外拿裸 `latest`；**只有 release tag 追加版本**（`<engine>-v1.2.0`）；
> 不带权重的变体带后缀 `-runtime`，只在 release 或手动 dispatch 时发布，且**没有第二个名字**。
> `playground` 是唯一的非引擎镜像（release 为 `playground-<version>`）。多架构（amd64 + arm64 原生
> runner）、带 SBOM 与 provenance；只有 Docker Hub 一个 registry，**GHCR 不再推送**。
> tag 全表在 `docs/deployment.md`，规则的理由在工作流的注释里。
>
> **本地编排**：`docker-compose.yml` 是部署文件，用 profile 选引擎，**profile 名 == 引擎 id ==
> image tag == `--engine` == `DECIS_DEFAULT_ENGINE`**。权重在构建期写入镜像的 `DECIS_MODEL_DIR=/models`，
> **不许往那里挂卷**（挂上去会盖掉镜像里已有的权重，容器转去联网下载而不报错）。compose 层的探针打
> `/readyz`（该层不会因探针失败重启容器，所以 `--wait` 是真的就绪等待），镜像自带的 `HEALTHCHECK`
> 仍是 `/healthz`，给编排器用。`docker-compose.override.yml` 靠**文件名**被 Compose 自动叠上，
> 于是源码目录里的裸 `docker compose up` 用本仓库源码构建 `decis-local:*`，部署只拷基文件。
>
> **容器内已实测的边界**：内置权重的引擎镜像与 playground 镜像都从 `chaitin/decis` 拉下来跑过——
> 前者在 `HTTP_PROXY`/`HTTPS_PROXY` 指向一个没有服务监听的端口时从 `/models/...` 加载、零下载，并作出完整契约响应
> （无凭证 403、错 key 401）。**未验证**：`-runtime` 变体只验到能构建、能合并、体积对（没拉下来跑过）、
> kev 的容器内冷启动、kev 的 compose 路径。两个 Jeff 镜像在 2026-09-30 第一次构建并推送成功，
> 体积已从 registry manifest 读出并记进 `docs/deployment.md`，但**同样没拉下来跑过**——它们的
> 容器内冷启动不在上面那条"已实测"里。报镜像相关的结论不要超出这个范围。

---

## 1. 必读文档

| 文档 | 作用 | 什么时候必须读 |
|---|---|---|
| [`docs/api-compatibility.md`](docs/api-compatibility.md) | 对外线格式的**唯一事实来源**，含证据等级（L > S > A > B > C > D） | 任何涉及请求/响应字段的改动 |
| [`docs/api.md`](docs/api.md) | 面向调用方的 API 参考：端点、原语、错误码、容量上限（双语） | 改端点、错误码、容量校验或 `decis` 命名空间时 |
| [`docs/schema/`](docs/schema/) | 由 `src/decis/schema.py` **生成**的 JSON Schema + OpenAPI（`export.py --check` 进 CI） | 改任何线格式模型时；不得手改生成的 JSON |
| [`docs/design.md`](docs/design.md) | 架构、抽象、并发、打包方案 | 任何新增模块或引擎的改动 |
| [`docs/design-review.md`](docs/design-review.md) | 对本设计的**自我审查**：已修正的缺陷、方法论局限、尚未验证的假设 | 动手实现前；以及任何"这个设计是不是已经想清楚了"的疑问 |
| [`docs/feasibility.md`](docs/feasibility.md) | 调查证据、实测数字、风险登记 | 讨论性能预期或选型时 |
| [`docs/contract/`](docs/contract/) | 官方 OpenAPI 快照（L0 测试基准）+ 线上观测原始记录（L 级证据） | 任何契约相关改动；**改前必须跑一次线上差分** |

面向用户的双语指南（`docs/getting-started.md`、`configuration.md`、`deployment.md`、`api.md`、
`engines.md`、`playground.md`、`performance.md`，各有 `.zh-CN.md` 孪生）是**产品的一部分**：
改一种语言就必须改另一种，`tests/test_docs.py` 盯着文件对是否存在、互链、以及相对链接 /
`DECIS_*` 变量 / 代码块语言 / 生成标记在两侧是否一致。

冲突时优先级：`api-compatibility.md` > `design.md` > 其余。

**`design-review.md` 不是历史文档，是活文档。** 它列的"尚未验证"清单在对应验证完成前一直有效。

**M5 已经有数据了（§4-M5），结论是否定**：跨请求批处理不提升吞吐。所以原来那条"不得承诺 QPS"的理由消失了，
但**新理由接上**：现在可以报的是一条**负结论**加一个**实测的吞吐上界**（单进程 24 线程，`laya-multilingual`，
CPU，约 1.2 项/秒，只有 16 个生成项、合成批的串行路径），**不得**把它包装成"批处理带来的高 QPS"，
也**不得**把它外推到 GPU、kev 或真实并发负载——那三样都没有数据。

---

## 2. 唯一事实来源（One canonical home）

每个概念只能有一个实现处。**发现第二处实现就是 bug**，即使两处当前行为相同。

| 概念 | 唯一所在 | 禁止 |
|---|---|---|
| 各层共享的领域类型（`Option`/`PreparedQuestion`/`PreparedRequest`/`ProbDist`） | `src/decis/domain.py` | 在 `schema.py`/`render.py` 里另定义一份；`domain.py` **不得 import 包内任何模块** |
| `state`/`instructions`/`criteria` → 可读文本（任意 JSON 的扁平化） | `src/decis/render.py` | 引擎各自实现 JSON 扁平化；引擎各自做分隔符转义 |
| 「调用方原始 JSON 原样交给需要它的引擎」 | `domain.PreparedQuestion.raw` / `PreparedRequest.raw_state` / `WorkItem.raw_state`，由 `render.py` 填 | 引擎自己去解析请求体；把扁平化后的文本当成 `raw`（Jeff 的提示词是 `json.dumps`，传入错误的文本不会报错，只是答案变差） |
| `--model-path ENGINE=PATH` 的语法与别名规范化 | `src/decis/cli.py: _parse_model_paths`（规范化后写进 `Settings.model_paths`） | 引擎自己解析命令行参数；用把引擎 id 编进**变量名**的方式做覆盖（点号在变量名里没有表示法，见 §9） |
| 第三方 vendored 副本的字节 | `src/decis/engines/_kev_vendor/`、`src/decis/engines/_jeff_vendor/`，各自带 `VENDOR.md` | 就地改 vendored 代码；re-vendor 时不一起改 `VENDOR.md` / `NOTICE` / 守卫里的 sha256 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaitin/Decis](https://github.com/chaitin/Decis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
