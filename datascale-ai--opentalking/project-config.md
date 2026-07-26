---
trigger: always_on
description: 本文件给进入 OpenTalking 仓库的团队 agent 使用。开始任何任务前先读本文件，再按任务类型读取对应 README、docs、配置和测试。这里记录的是当前代码形态下的协作规则，不替代源码。
---

# AGENT.md

本文件给进入 OpenTalking 仓库的团队 agent 使用。开始任何任务前先读本文件，再按任务类型读取对应 README、docs、配置和测试。这里记录的是当前代码形态下的协作规则，不替代源码。

## 任务入口

先确认工作目录和状态：

```bash
cd <opentalking-repo>
git status --short
```

优先使用 `rg` / `rg --files` 查找文件和符号。不要用旧记忆里的 `src/opentalking/...` 路径；当前 OpenTalking 是 flat layout，库代码在仓库根目录的 `opentalking/`，应用入口在 `apps/`。

做代码修改前先读：

- `README.md`：当前用户路线、启动入口、模型部署叙事。
- `docs/zh/docs/architecture.md`：系统架构、组件职责、OpenTalking 与 backend 边界。
- `docs/zh/docs/model-adapter.md`：`mock` / `local` / `direct_ws` / `omnirt` backend 规则。
- `docs/zh/model-deployment/talking-head.md`：模型路线和状态说明。
- `configs/default.yaml`：当前默认配置。
- `pyproject.toml`、`Makefile`、`apps/web/package.json`：开发、测试、构建命令。

英文文档存在于 `docs/en/`。用户可见的能力或命令变更通常需要同步 `docs/zh/` 和 `docs/en/`，或在 PR 说明中明确尚未同步。

## 项目边界

OpenTalking 是实时数字人对话编排层，负责：

- WebUI、API、会话状态、事件流、WebRTC。
- LLM、STT、TTS provider 的调用和串联。
- Avatar / voice 资产管理。
- `LLM -> TTS -> talking-head backend -> WebRTC` 的运行时流水线。
- 按模型选择 `mock`、`local`、`direct_ws` 或 `omnirt` backend。

OpenTalking 不负责：

- 重模型权重的完整生命周期和多卡调度。
- OmniRT 内部 worker、队列、CUDA / Ascend runtime。
- LLM、TTS、STT 服务本身的托管。
- TURN、认证、账号、生产级权限系统。

OmniRT 是可选的外部推理服务。只有模型配置为 `backend: omnirt` 时，OpenTalking 才通过 `OMNIRT_ENDPOINT` 派生 `/v1/audio2video/{model}` WebSocket 路由。Sibling repo 常见路径是：

```bash
cd <omnirt-repo>
```

OmniRT 当前是 `src/omnirt/` layout，主要入口和验证命令以 `../omnirt/README.md`、`../omnirt/pyproject.toml`、`../omnirt/docs/` 为准。

## 关键目录

```text
opentalking/
├── opentalking/
│   ├── core/          # Settings、model_config、bus、registry、types、session store
│   ├── providers/     # llm / stt / tts / rtc / synthesis provider
│   ├── models/        # 本地模型 adapter，例如 quicktalk、wav2lip
│   ├── pipeline/      # session、speak、recording 流水线
│   ├── runtime/       # worker、task consumer、timing、runtime server
│   ├── avatar/        # Avatar bundle 加载、校验、预处理
│   ├── voice/         # 音色资产和复刻存储
│   └── events/ media/ # 事件 schema、媒体工具
├── apps/
│   ├── api/           # FastAPI 路由、schema、service
│   ├── unified/       # 开发友好的单进程入口
│   ├── web/           # React + Vite + TypeScript
│   └── cli/           # doctor、download、bench 等命令
├── configs/           # default.yaml、profiles、inference、synthesis 配置
├── scripts/           # start_unified.sh、quickstart、部署辅助脚本
├── tests/             # pytest 单元和集成测试
└── docs/              # MkDocs 文档站，zh/en 双语
```

不要把 `models/` 下的权重、缓存、生成媒体、私有 avatar 资产提交进 git。

## 启动与运行

推荐入口是：

```bash
bash scripts/start_unified.sh --mock
```

常见路线：

```bash
# 首次跑通 API / WebUI / LLM / TTS / WebRTC，不需要权重
bash scripts/start_unified.sh --mock

# 消费级 GPU，本地 QuickTalk
bash scripts/start_unified.sh --backend local --model quicktalk

# 消费级 GPU，本地 Wav2Lip
bash scripts/start_unified.sh --backend local --model wav2lip

# 远端高质量模型，通过 OmniRT
bash scripts/start_unified.sh --backend omnirt --model flashtalk --omnirt http://<gpu-server>:9000
```

`scripts/quickstart/*` 仍然保留，适合更底层的服务调试、端点配置和停止服务。面向新用户或 README 流程时优先使用 `scripts/start_unified.sh`。

前端默认地址是 `http://localhost:5173`。后端端口默认 `8000`，可通过 `--api-port` 和 `--web-port` 覆盖。停止 quickstart 启动的进程：

```bash
bash scripts/quickstart/stop_all.sh
```

## 配置规则

OpenTalking 配置来源有优先级，不要只看一处就下结论：

1. 进程环境变量和 `.env`，前缀通常是 `OPENTALKING_`。
2. legacy 环境变量，例如 `OMNIRT_ENDPOINT`、`DASHSCOPE_API_KEY`、`FLASHTALK_WS_URL`。
3. `configs/default.yaml` 或 `OPENTALKING_CONFIG_FILE` / `CONFIG_FILE` 指向的 YAML。
4. 代码默认值。

模型 backend 解析位于 `opentalking/core/model_config.py`。优先级是内置默认、YAML 中 `models.<name>.backend`、再到 `OPENTALKING_<MODEL>_BACKEND` 环境变量。`scripts/start_unified.sh --backend local --model quicktalk` 会导出 `OPENTALKING_QUICKTALK_BACKEND=local` 并覆盖 YAML。

当前默认配置需要以 `configs/default.yaml` 为准。写文档或 review 时不要把 README 推荐路线和 YAML 默认值混为一谈。README 当前推荐：

| 场景 | 推荐模型 | 推荐 backend |
| --- | --- | --- |
| 首次验证 | `mock` | `mock` |
| 消费级 GPU 本地路线 | `quicktalk` | `local` |
| 轻量口型同步 | `wav2lip` | `local` / `omnirt` |
| 高质量远端推理 | `flashtalk` | `omnirt` |

backend 含义：

| backend | 含义 | 典型代码位置 |
| --- | --- | --- |
| `mock` | 内置占位合成，CI 和首次验证使用。 | `opentalking/providers/synthesis/mock.py` |
| `local` | 进程内加载 `opentalking/models/<name>/` adapter。 | `opentalking/models/` |
| `direct_ws` | 直接连接单模型 WebSocket 服务。 | `opentalking/providers/synthesis/backends.py` |
| `omnirt` | 从 `OMNIRT_ENDPOINT` 派生 OmniRT audio2video 路由。 | `opentalking/providers/synthesis/omnirt.py` |

OmniRT 相关变量写清楚：

- `OMNIRT_ENDPOINT`：OmniRT base URL，例如 `http://127.0.0.1:9000`。
- `OMNIRT_AUDIO2VIDEO_PATH_TEMPLATE`：默认 `/v1/audio2video/{model}`。
- `OPENTALKING_OMNIRT_AUDIO2VIDEO_PATH_TEMPLATE`：OpenTalking Settings 前缀形式。
- `OMNIRT_API_KEY`：可选 Bearer Token。
- `OPENTALKING_<MODEL>_BACKEND`：模型级 backend 覆盖，例如 `OPENTALKING_QUICKTALK_BACKEND=local`。

LLM / TTS / STT 相关变量修改时，明确区分 `.env`、`Settings`、legacy env 和请求级覆盖。TTS 前端可能传入 provider 覆盖，不要只根据 `.env` 默认值判断用户实际选择。

## 开发命令

安装 Python 依赖：

```bash
uv sync --extra dev --python 3.11
source .venv/bin/activate
```

需要本地模型 runtime 时：

```bash
uv sync --extra dev --extra models --python 3.11
```

兼容 fallback：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install --index-url https://pypi.tuna.tsinghua.edu.cn/simple -e ".[dev]"
```

前端：

```bash
cd apps/web
npm ci
npm run dev
npm run typecheck
npm run build
```

常用后端命令：

```bash
make test
make lint
pytest
pytest tests -v

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datascale-ai/opentalking](https://github.com/datascale-ai/opentalking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
