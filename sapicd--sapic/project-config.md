---
trigger: always_on
description: sapic（picbed）是一个基于 Flask 的自建图床，**无 ORM、纯 Redis 存储**，
---

# sapic — 项目上下文

sapic（picbed）是一个基于 Flask 的自建图床，**无 ORM、纯 Redis 存储**，
**存储后端完全插件化（钩子体系）**，当前版本 v1.16.7。

技术栈：Python 3.10+, Flask, Redis, Jinja2, PyJWT, semver, bleach
前端：layui + jQuery + Jinja2 模板
详细架构见 `ARCH.md`，贡献流程见 `CONTRIBUTING.md`。

> Python 版本下限是 **3.10**（`utils.tool.raise_version()` 在应用启动时会强制校验，低于 3.10 直接
> `RuntimeError`）。因此可自由使用 3.10 语法（`X | Y` 联合类型、match 语句、内置泛型 `dict[str]` 等）。
> 注意：历史代码为兼容旧版本而保留的 `_compat` 兼容写法、无注解风格，仍按"精准修改"原则保持不变。

## 目录结构

所有代码以 `src/` 为 Python 模块根（在此目录下运行），不使用包外相对导入：

```
src/
├── app.py        应用入口：Flask 实例、全局钩子、蓝图注册、错误处理
├── config.py     配置：.cfg 文件 → 环境变量 → 默认值（可直接 python config.py 打印）
├── version.py    版本号 __version__
├── sapicd.py     gunicorn 配置（生产部署）
├── views/        路由层  front.py(页面) / api.py(/api/* JSON)
├── libs/         领域核心  storage.py(Redis 单例) / hook.py(插件系统)
├── utils/        基础设施  tool.py(纯工具) / web.py(依赖 Flask 上下文)
│                          / cli.py / _compat.py / exceptions.py / log.py
├── hooks/        内置钩子，一个 .py 即一个插件
├── templates/    public/ control/ layout/ email/ ref/
├── static/       layui/ mymod/ sdk/ share.js/ upload/(up2local 落盘目录)
├── tests/        unittest 测试（test_*.py）
└── Makefile      dev/test/start/stop/reload/restart/status/clean
```

依赖方向单向：`views → libs/utils → utils.tool`。
`utils.tool` 不依赖 Web 框架，`utils.web` 才依赖 `request` / `g` / `current_app`。

## 常用命令

```bash
cd /path/to/sapic/src
make test                     # python -m unittest discover -p "test_*.py"
python config.py              # 打印当前生效配置
flask sa create               # 创建超级管理员
flask sa clean                # 清理数据
flask sa upgrade 1.16         # 版本数据结构迁移（可选 1.6-1.7 / 1.7-1.8 / 1.16）
cd ../docs && make html       # 构建 Sphinx 文档
```

Python 分支：尽量在 dev 分支编码，PR 合并到 dev。

## Python 编码规范

### 基本风格

- 遵循 **PEP 8**，缩进 4 空格，不使用 Tab，文件末尾保留一个空行
- 行宽与既有代码保持一致（约 79 字符）
- 文件头固定写法：`# -*- coding: utf-8 -*-` + 三引号模块 docstring + 版权块

```python
# -*- coding: utf-8 -*-
"""
    utils.xxx
    ~~~~~~~~~

    模块职责一句话描述（英文）。

    :copyright: (c) 2026 by staugur.
    :license: BSD 3-Clause, see LICENSE for more details.
"""
```

### 导入顺序

按以下顺序分组，组间空一行：

1. 标准库（`os`, `json`, `time`...）
2. 第三方库（`flask`, `redis`, `semver`...）
3. 项目内部（`views.*` / `libs.*` / `utils.*` / `hooks.*` / `config` / `version`）

风格要点：

- 每个模块单独一行：`import os`、`from os.path import join, exists`
- 同组字母序排列；新模块插到合适位置，不要追加在文件末尾
- 存在循环导入风险时（如 `config`、`utils.web`），允许在函数内延迟导入
- 同一包内可用相对导入（如 `from .storage import get_storage`），跨包一律绝对导入

```python
# 正确示例
import json
from random import choice

from flask import Blueprint, g, request
from redis.exceptions import RedisError

from config import GLOBAL
from utils.tool import rsp, sha1, get_today
from utils.exceptions import ApiError
```

### 命名规范

| 类型 | 风格 | 示例 |
|------|------|------|
| 模块/文件 | snake_case | `up2local.py`, `web.py` |
| 类 | PascalCase | `HookManager`, `JsonResponse` |
| 函数/方法 | snake_case | `get_site_config()`, `upimg_save()` |
| 变量 | snake_case | `upload_path`, `img_url` |
| 常量 | UPPER_SNAKE_CASE | `ALLOWED_EXTS`, `MAX_UPLOAD` |
| 私有成员 | 双下划线 / 单下划线 | `__hooks`, `_compat` |

### 类型注解

- 项目历史代码基本不带注解，**不要大范围补注解**
- 新增函数若需标注，按 3.10+ 写法：`str | None`、`dict[str, Any]`；也可用 `typing` 的
  `Optional`, `Dict`, `List`（`utils/tool.py` 有既有先例）
- 修改已有函数时保持其原有风格（带注解的保持带注解）

```python
def create_redis_engine(redis_url: str | None = None):
    ...
```

### 注释与文档

- **模块级**：文件头三引号，含 `:copyright:` / `:license:` 字段
- **类和函数**：docstring 采用 **Sphinx RST** 格式，用 `:param:`、`:returns:`、`:raises:`
  `` .. versionadded:: `` 等指令
- 模块/函数描述用英文；面向用户的中文说明在 `dfr()` 翻译字典中维护（见下）
- 行内注释用英文，项目惯用 `#:` 开头作为段落级说明注释
- 复杂逻辑注释解释"为什么"，不要复述代码本身

```python
def get_current_timestamp(is_float=False):
    """获取当前时间戳

    :param bool is_float: True则获取10位秒级时间戳，否则原样返回
    """
    return time() if is_float else int(time())
```

### 字符串与格式化

- 项目混用 `%` 与 `.format()`：**改动哪个文件就沿用该文件的既有风格**，不在同一文件内混用两种方式
- 拼接路径用 `os.path.join` / `posixpath.join`，不要手拼 `/`

### 异常处理

- 自定义异常：`ApiError`（返回 JSON）、`PageError`（返回 `public/error.html`），均继承 `PicbedError`
- 签名：`ApiError(message, code=-1, status_code=200)`
- 视图函数可直接 `res = dict(code=1, msg=...)` 返回，也可 `raise ApiError(...)`
- 禁止裸 `except:`，至少 `except Exception:` 或明确异常元组
- Redis / HTTP / 第三方 SDK 调用需捕获特定异常并转换为 `ApiError` 或带 `msg` 的失败响应
- 不用 `print()` 打日志，统一用 `utils.tool` 的 `logger` / `err_logger`

### API 错误规范（含国际化）★

面向用户的消息统一 **英文原文 + `dfr()` 中文翻译**：

- `raise ApiError("No valid backend storage service")` 中的 message 必须是英文
- 新增英文文案必须同步在 `utils/web.py` 的 `dfr()` → `trans["zh-CN"]` 字典中登记中文，
  否则中文用户只能看到英文（代码会 `logger.debug("Miss translation: %s")`)
- 不要在视图里返回中文 `msg`，这会绕过翻译体系
- 保持第三方客户端兼容：`/api/upload` 的 `format` / `status_name` / `ok_code` / `msg_name`
  参数行为不可变更（PicGo / uPic 依赖）

### Redis 数据操作

两类存储边界要分清：

- **系统级键值**：走 `libs/storage.py` 的 `RedisStorage`（`get_storage()`），数据 JSON 序列化后
  统一放 `picbed:dat` hash（`siteconfig` / `hookloadtime` / `hookthirds` / `hookstate`）
- **业务数据**：走 `utils.web` 的模块级 `rc`，key 统一由 `utils.tool.rsp()` 生成
  （自带 `picbed:` 前缀，如 `rsp("image", sha)` → `picbed:image:{sha}`）
- 新增 key 必须遵循上述前缀规则，并在 `ARCH.md` 的 key 表中补充说明
- 批量查询用 `pipeline` + `hmget`，禁止在循环里单条 `hgetall`
- 兼容 Redis 3.x：`hmset` 已统一替换为 `hset`，新增代码同样使用 `hset`
- 数据结构变更无法通过 ORM 迁移，需要在 `utils/cli.py` 的 `upgrade` 命令中手写 Redis 迁移脚本

## 钩子（插件）开发规范

`hooks/*.py` 一个文件即一个钩子，新增钩子必须满足：

```python
# -*- coding: utf-8 -*-
"""

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sapicd/sapic](https://github.com/sapicd/sapic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
