---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## 项目概述

这是一个代理订阅链接采集工具，从多个公开网站自动采集免费代理订阅链接，并生成 Clash 和 V2Ray 格式的配置文件。项目使用 Python 3.13+ 开发，通过 GitHub Actions 定时自动运行。

## 开发环境设置

### 依赖管理

项目使用 `uv` 作为包管理器：

```bash
# 安装依赖
cd src
uv sync --locked

# 运行主程序
uv run main.py

# 使用代理模式运行
uv run main.py --proxy

# 运行测试
cd src && uv run pytest

# 运行测试（带覆盖率）
cd src && uv run pytest --cov=. --cov-report=term-missing
```

### 主要依赖

- `lxml`: HTML 解析
- `requests[socks]`: HTTP 请求和 SOCKS 代理支持
- `pycryptodome`: Paste.to AES-GCM 解密
- `pydantic` / `pydantic-settings`: 配置管理和数据验证
- `tenacity`: 重试机制
- `tqdm`: 进度条显示
- `pyyaml`: YAML 文件处理
- `tabulate`: 表格输出辅助
- `pytest`: 测试框架

## 核心架构

项目采用**分层架构**，职责清晰分离：

```
┌─────────────────────────────────────────────────────────┐
│                      main.py                            │
│                   (入口 & 编排)                          │
└─────────────────────────────────────────────────────────┘
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│  collectors/  │  │   services/   │  │    config/    │
│   (采集器)     │  │    (服务)     │  │    (配置)     │
└───────────────┘  └───────────────┘  └───────────────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  ┌───────────────┐
                  │     core/     │
                  │ (核心模型/接口) │
                  └───────────────┘
```

### 1. 核心层 (`core/`)

定义基础数据结构和接口，不依赖其他模块：

- **`interfaces.py`**: Protocol 接口定义（`HttpClient`）
  - `HttpClient.get()` 支持 `headers` 和 `check_html` 内容检查回调
- **`models.py`**: 数据模型
  - `DownloadTask`: 下载任务（含可选的内容处理器）
  - `ProxyInfo`: 代理信息（含健康度评分算法）
  - `CollectorResult`: 采集结果
    - `from_cache`: 标识是否复用已成功采集的缓存结果
  - `FileManifest` / `SiteManifest`: 文件和站点清单
  - `ProxySourceConfig`: 代理源配置（URL、权重、代理类型）
  - `ProxyCache`: 代理缓存
- **`exceptions.py`**: 自定义异常类
  - `CollectorError` → `NetworkError` / `ProxyError` / `ParseError` / `DownloadError` / `ValidationError`

### 2. 配置层 (`config/`)

使用 Pydantic 进行配置管理，支持环境变量和 `.env` 文件：

- **`settings.py`**: 配置类
  - `AppConfig`: 应用配置（输出目录、manifest 文件等）
  - `ProxyConfig`: 代理配置（代理源、缓存、健康度阈值等）
  - `CollectorConfig`: 采集器配置（超时、并发数等）
  - `Config`: 全局配置聚合

### 3. 服务层 (`services/`)

提供可复用的业务服务：

- **`http_service.py`**:
  - `HttpService`: 基础 HTTP 请求（带重试）
  - `ProxyPool`: 代理池管理（健康度排序）
  - `ProxyHttpService`: 支持代理池的并发请求
  - `get()` / `fetch_with_proxies()` 均支持 `check_html` 校验响应内容
- **`proxy_service.py`**:
  - `ProxyValidator`: 代理验证器
  - `ProxyService`: 代理获取和验证服务
- **`proxy_cache_service.py`**: 代理缓存服务
- **`manifest_service.py`**: Manifest 管理服务
- **`file_processor.py`**: 文件后处理（注入订阅信息节点，重复处理会先移除旧节点避免重复注入）
- **`paste_to_service.py`**: Paste.to 分享获取、解析、解密服务

### 4. 采集器层 (`collectors/`)

插件式架构，通过装饰器自动注册：

- **`base.py`**: 基类和注册表
  - `BaseCollector`: 采集器基类
  - `CachedCollectorResult`: 内部控制流，用于命中 `today_page` 缓存后跳过下载
  - `register_collector`: 注册装饰器
  - `COLLECTOR_REGISTRY`: 采集器注册表
- **`mixins.py`**: 通用 Mixin 和辅助函数
  - `TwoStepCollectorMixin`: 两步采集（首页 → 今日页面 → 下载）
  - `DateBasedUrlMixin`: 基于日期的 URL 构建
  - `safe_xpath()` / `safe_xpath_all()`: 安全的 XPath 查询函数
- **`sites/`**: 具体站点采集器实现

### 5. 工具层 (`utils/`)

- **`logging_config.py`**: 日志配置
- **`extractors.py`**: 内容提取器
  - `extract_by_regex()`: 正则提取
  - `unescape_backslashes()`: 常见反斜杠转义还原
  - `create_regex_extractor()`: 创建正则提取器（用于 DownloadTask.processor）
  - `create_download_tasks_from_regex_rules()`: 按文件名和正则规则生成下载任务
- **`check.py`**: HTML 内容检查
  - `default_check_html()`: 默认非空检查
  - `check_html_contains()`: 生成关键字包含检查器
- **`youtube.py`**: YouTube 播放列表和视频跳转链接解析
- **`paste_to.py`**: Paste.to URL 解析、payload 预处理、AES-GCM 解密和密码策略

## 采集器模式

### 基础采集器

```python
from collectors.base import BaseCollector, register_collector
from core.models import DownloadTask

@register_collector
class SimpleCollector(BaseCollector):
    name = "simple"
    home_page = "https://example.com"

    def get_download_tasks(self) -> list[DownloadTask]:
        # 直接返回下载任务
        return [
            DownloadTask(filename="clash.yaml", url="https://example.com/clash.yaml"),
            DownloadTask(filename="v2ray.txt", url="https://example.com/v2ray.txt"),
        ]
```

### 两步采集器（推荐）

大多数站点需要先访问首页获取今日链接，再解析下载地址：

```python
from typing import Optional
from collectors.base import BaseCollector, register_collector
from collectors.mixins import TwoStepCollectorMixin, safe_xpath, safe_xpath_all
from core.models import DownloadTask

@register_collector
class TwoStepCollector(TwoStepCollectorMixin, BaseCollector):
    name = "twosite"
    home_page = "https://example.com"

    def get_today_url(self, home_html: str) -> Optional[str]:
        """从首页获取今日链接"""
        links = safe_xpath_all(home_html, '//a[contains(text(), "今日")]/@href', self.name)
        return links[0] if links else None

    def parse_download_tasks(self, today_html: str) -> list[DownloadTask]:
        """从今日页面解析下载任务"""
        tasks = []
        clash_url = safe_xpath(today_html, '//a[contains(@href, "clash")]/@href', self.name)
        if clash_url:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cook369/proxy-collect](https://github.com/cook369/proxy-collect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
