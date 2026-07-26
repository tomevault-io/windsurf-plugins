---
trigger: always_on
description: > 本文件作为 GitHub Copilot 及其他 AI 辅助工具的上下文指令文件，同时也是本项目的开发规范文档。
---

# PyMKUI 项目开发规范与 AI 辅助指令

> 本文件作为 GitHub Copilot 及其他 AI 辅助工具的上下文指令文件，同时也是本项目的开发规范文档。  
> 最后更新：2026-03-18（补充 ZLM API 访问规范：get_zlm_base_url / get_forward_headers）

---

## 一、项目概述

**PyMKUI** 是 ZLMediaKit 的业务管理前端工程，采用前后端分离架构：

| 层次 | 目录 | 技术栈 | 说明 |
|------|------|--------|------|
| 前端 | `frontend/` | 原生 HTML + Tailwind CSS + 原生 JS | SPA 风格，iframe 子页面组织 |
| 后端 | `backend/` | Python 3.12 + FastAPI + SQLite | ZLMediaKit Python 插件模式运行 |
| 媒体服务 | 外部进程 | ZLMediaKit (C++) | 通过 HTTP API 交互 |
| 数据库 | `data/` | SQLite (`pymkui.db`) | 业务数据持久化 |

### 运行架构

```
浏览器
  └─ ZLMediaKit 内置 HTTP 服务 (port 8080)
        ├─ 静态文件服务 → frontend/ 目录
        ├─ /index/api/* → ZLMediaKit 原生 C++ HTTP API
        └─ /index/pyapi/* → Python FastAPI (通过 mk_plugin 注入到 ZLMediaKit)
                                └─ SQLite 数据库
```

### 端口规划

| 端口 | 协议 | 服务 |
|------|------|------|
| 8080 | TCP | ZLMediaKit HTTP / WS / HLS / FLV / FMP4 |
| 8443 | TCP | ZLMediaKit HTTPS / WSS |
| 1935 | TCP | RTMP 推拉流 |
| 8554 | TCP | RTSP |
| 10000 | TCP/UDP | RTP/RTCP (GB28181) |
| 8000 | UDP | WebRTC ICE/STUN |

---

## 二、目录结构规范

```
pymkui/
├── .github/
│   └── copilot-instructions.md   # 本文件（AI 指令 + 开发规范）
├── frontend/
│   ├── index.html                # 主框架页（导航 + iframe 容器）
│   ├── login.html                # 登录页
│   ├── css/
│   │   ├── common.css            # 全局公共样式
│   │   ├── index.css             # 主框架样式
│   │   └── login.css             # 登录页样式
│   ├── js/
│   │   ├── api.js                # ★ 全局 API 封装对象（Api.*）
│   │   ├── dashboard.js          # 仪表板逻辑
│   │   ├── streams.js            # 流管理逻辑
│   │   ├── network.js            # 网络连接管理
│   │   ├── pull-proxy.js         # 拉流代理管理
│   │   ├── settings.js           # 服务器配置
│   │   ├── protocol-options.js   # 转协议预设管理
│   │   ├── whip.js               # WHIP 推流
│   │   ├── config_comments.js    # 配置项注释说明
│   │   └── lib/                  # 第三方库（jessibuca 等）
│   └── pages/                    # iframe 子页面
│       ├── dashboard.html
│       ├── streams.html
│       ├── network.html
│       ├── pull-proxy.html
│       ├── push-proxy.html
│       ├── settings.html
│       ├── protocol-options.html
│       ├── whip.html
│       └── auth.html
├── backend/
│   ├── config.py                 # 路径配置
│   ├── database.py               # SQLite 数据库操作类
│   ├── mk_logger.py              # 日志模块（适配 mk_loader/print 两种模式）
│   ├── mk_plugin.py              # ★ ZLMediaKit Python 插件入口
│   ├── py_http_api.py            # ★ FastAPI HTTP API 定义
│   ├── shared_loop.py            # 全局 asyncio 事件循环
│   └── requirements.txt
├── data/
│   └── pymkui.db                 # SQLite 数据库文件
└── README.md
```

---

## 三、后端开发规范（Python / FastAPI）

### 3.1 Python 版本与依赖

- **Python 版本**：3.12+
- **核心依赖**（`requirements.txt`）：
  ```
  psutil>=5.9.0
  httpx>=0.24.0
  fastapi>=0.100.0
  starlette>=0.30.0
  uvicorn[standard]>=0.22.0
  uvloop>=0.17.0
  ```

### 3.2 模块职责

| 模块 | 职责 | 注意事项 |
|------|------|---------|
| `config.py` | 项目路径常量 | 只放路径配置，不放业务逻辑 |
| `database.py` | SQLite CRUD | 所有数据库操作集中此处，禁止在其他模块直接操作 SQLite |
| `mk_logger.py` | 统一日志 | 使用 `log_info/log_warn/log_error`，**禁止直接 `print()`** |
| `mk_plugin.py` | ZLM 插件入口 | 实现 `on_start/on_exit/on_publish/on_play` 等回调 |
| `py_http_api.py` | FastAPI 路由 | 所有 Python 侧 HTTP API 定义在此 |
| `shared_loop.py` | 异步事件循环 | 全局唯一 asyncio loop，供跨线程协程使用 |

### 3.3 API 路由命名规范

- **Python API 路径前缀**：`/index/pyapi/`（区别于 ZLM 原生的 `/index/api/`）
- **命名风格**：小写 + 下划线（`snake_case`），例如：
  - `GET /index/pyapi/get_protocol_options_list`
  - `POST /index/pyapi/add_protocol_options`
  - `POST /index/pyapi/update_protocol_options`
  - `POST /index/pyapi/delete_protocol_options`

### 3.4 API 响应格式规范

所有 Python API 必须与 ZLMediaKit 的响应格式保持一致：

```python
# 成功
{"code": 0, "msg": "操作成功", "data": {...}}

# 失败  
{"code": -1, "msg": "失败原因描述"}
```

**code 取值约定**（与 ZLMediaKit 对齐）：

| code | 含义 |
|------|------|
| `0` | 成功 |
| `-1` | 业务失败（OtherFailed） |
| `-100` | 鉴权失败（AuthFailed） |
| `-200` | SQL 执行失败（SqlFailed） |
| `-300` | 参数非法（InvalidArgs） |
| `-400` | 代码抛异常（Exception） |

### 3.5 请求参数解析规范

FastAPI 路由**必须同时支持** JSON body、form-urlencoded、GET query 三种传参方式。  
使用已有的 `get_param_from_request()` 工具函数，或按以下顺序解析：
1. Query 参数
2. Body（JSON / form）
3. Header

```python
async def my_api(request: Request):
    body_bytes = await request.body()
    content_type = request.headers.get("content-type", "")
    
    if "application/json" in content_type or not content_type:
        data = json.loads(body_bytes.decode("utf-8"))
    elif "application/x-www-form-urlencoded" in content_type:
        parsed = urllib.parse.parse_qs(body_bytes.decode("utf-8"), keep_blank_values=True)
        data = {k: v[0] if len(v) == 1 else v for k, v in parsed.items()}
    # ...
```

### 3.6 调用 ZLMediaKit HTTP API 规范

Python 后端转发请求给 ZLMediaKit 时，**必须**使用以下两个统一工具函数，禁止手动拼接 URL 或手动透传 cookie：

#### `get_zlm_base_url()` — 获取 ZLM 内部访问地址

```python
def get_zlm_base_url() -> str:
    """
    获取 ZLMediaKit 内部访问的 base URL。
    - http.port != 0  → http://127.0.0.1:{http.port}
    - http.port == 0  → https://127.0.0.1:{http.ssl_port}
    """
    http_port = mk_loader.get_config("http.port")
    try:
        http_port = int(http_port)
    except (TypeError, ValueError):
        http_port = 0

    if http_port != 0:
        return f"http://127.0.0.1:{http_port}"
    else:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZLMediaKit/pymkui](https://github.com/ZLMediaKit/pymkui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
