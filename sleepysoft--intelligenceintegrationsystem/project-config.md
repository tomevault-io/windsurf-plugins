---
trigger: always_on
description: > 本文档供 AI 编码助手阅读。如果你第一次接触本项目，请先阅读本文件，再阅读 `README.md` 和 `doc/IntelligenceDesign.md`。
---

# IntelligenceIntegrationSystem — Agent Guide

> 本文档供 AI 编码助手阅读。如果你第一次接触本项目，请先阅读本文件，再阅读 `README.md` 和 `doc/IntelligenceDesign.md`。

---

## 1. 项目概述

**IntelligenceIntegrationSystem（IIS）** 是一个开源情报（OSINT）整合系统，核心流程为：

```
抓取新闻/情报 → 提交到情报中心 → AI 分析/评分/清洗 → 归档 → 发布/查询
```

系统从全球主流媒体抓取公开新闻（RSS 或列表页），通过 AI（LLM）进行结构化分析、评分、去重，最终将高价值情报归档到数据库，并通过 Web 提供检索、聚合、态势推演等功能。

当前主分支为 **v2**（2026-02-15 起切换），兼容 v1 数据，无需数据库升级。

---

## 2. 技术栈

| 层级 | 技术 |
|------|------|
| 语言 | Python 3.10+ |
| Web 框架 | Flask（主服务）、FastAPI（部分子服务） |
| WSGI 服务器 | Waitress（Windows 默认）、Gunicorn（Linux 可选）、Flask dev server |
| 数据库 | MongoDB（文档存储）、SQLite（用户认证、AI 客户端状态日志） |
| 向量数据库 | ChromaDB + `sentence-transformers`（默认模型 `BAAI/bge-m3`） |
| 爬虫 | Playwright、requests、beautifulsoup4、crawl4ai、feedparser |
| AI 客户端 | OpenAI 兼容 API、SiliconFlow、Zhipu、Google Gemini、Ollama 等 |
| 数据校验 | Pydantic v2 |
| 任务调度 | APScheduler + 自定义 `AdvancedScheduler` |
| 前端 | 服务端渲染 Jinja2 模板 + 少量原生 JS/CSS（无前后端分离） |
| GUI（工具类） | PyQt5 |

---

## 3. 目录结构与模块划分

### 3.1 根目录核心文件

| 文件 | 职责 |
|------|------|
| `GlobalConfig.py` | 全局路径、代理、超时、默认端口等常量 |
| `IntelligenceHub.py` | **核心引擎**。管理队列、AI 分析线程、后处理线程、向量化线程、定时任务 |
| `IntelligenceHubStartup.py` | **启动组装**。读取配置、初始化 MongoDB/VectorDB/AIClientManager、组装 Hub 和 WebService |
| `IntelligenceHubLauncher.py` | **WSGI 启动器**。自动选择 Waitress/Gunicorn/Flask dev server，带健康检查与自动重启 |
| `CrawlerServiceEngine.py` | **爬虫服务入口**。插件化任务管理、文件系统监控、热重载、爬虫治理后台 |
| `prompts_v2x.py` | AI 分析 Prompt 定义表 |

### 3.2 主要子目录

- **`AIClientCenter/`** — AI 客户端管理中心
  - 支持多厂商、多账号、Token 轮询、余额监控、故障切换
  - 关键类：`AIClientManager`, `BaseAIClient`, `StandardOpenAIClient`, `OuterTokenRotatingOpenAIClient`
  - 配置方式：将 `AIClientConfigExample.py` 复制为 `_config/ai_client_config.py` 并修改

- **`CrawlTasks/`** — 具体抓取任务模块
  - 每个文件对应一个媒体源的抓取逻辑（如 `task_crawl_bbc.py`, `task_crawl_nhk_ic.py`）
  - 由 `CrawlerServiceEngine.py` 动态加载，支持热重载

- **`IntelligenceCrawler/`** — 爬虫框架与治理
  - `CrawlerPlayground.py`：可视化“所见即所得”生成爬虫配置
  - `CrawlerCodeGenerator.py` / `CrawlerCodePrototype.py`：代码生成
  - `CrawlerGovernanceBackend.py` / `CrawlerGovernanceCore.py`：爬虫治理与调度
  - `CrawlPipeline.py` / `Discoverer.py` / `Extractor.py` / `Fetcher.py`：流水线组件

- **`ServiceComponent/`** — 业务组件层
  - `IntelligenceHubDefines_v2.py`：Pydantic 数据模型（`CollectedData`, `ArchivedData`, `ProcessedData` 等）
  - `IntelligenceAnalyzerProxy.py`：AI 分析代理，调用 LLM 并解析结果
  - `IntelligenceQueryEngine.py` / `IntelligenceStatisticsEngine.py`：MongoDB 查询与统计
  - `IntelligenceVectorDBEngine.py`：向量检索封装
  - `IntelligenceAggregationEngine.py` / `DynamicGraphEngine.py`：情报聚合与态势图谱推演
  - `IntelligenceScoringEngine.py`：评分引擎
  - `UserManager.py`：用户认证管理（SQLite）
  - `PostManager.py` / `RSSPublisher.py`：文章发布与 RSS 生成

- **`VectorDB/`** — 向量数据库服务
  - `VectorDBBService.py`：独立 Flask 服务，提供向量存储、检索、聚类 API
  - `VectorStorageEngine.py`：底层存储引擎（基于 ChromaDB）
  - `ClusterAnalysisPipeline.py` / `aggregation/`：聚类分析与离线/在线聚合

- **`Scraper/`** — 抓取抽象层
  - `ScraperBase.py`, `RequestsScraper.py`, `PlaywrightRawScraper.py`, `PlaywrightRenderedScraper.py`, `Crawl4AI.py`

- **`Tools/`** — 通用工具
  - `MongoDBAccess.py`：MongoDB 封装（带导出功能）
  - `DateTimeUtility.py`, `CommonPost.py`, `RSSFetcher.py`, `CyberSecurity.py` 等

- **`MyPythonUtility/`** — 可复用基础设施
  - `AdvancedScheduler.py`, `ArbitraryRPC.py`, `DictTools.py`, `FileSqliteHyridDB.py`, `ObserverNotifier.py`, `ChatLLM.py`, `WsReverseRPC.py` 等

- **`PyLoggingBackend/`** — 自定义日志后端
  - 支持日志文件监控、Web 查看器、历史归档、TLS 线程隔离日志

- **`Workflow/`** — 抓取流程编排
  - `CommonFlowUtility.py`：通用提交流程（提交到 `/collect`）
  - `RssFeedsBasedCrawlFlow.py`：基于 RSS 的简易抓取流程
  - `IntelligenceCrawlFlow.py`：适配 IIS 的抓取流程

- **`Test/`** — 演示/测试脚本
  - 以 `Test` 开头，但多为手动运行的演示代码，非自动化单元测试

- **`Scripts/`** — 运维脚本
  - `UserManagerConsole.py`：用户增删改查
  - `rebuild_vector_index.py`：重建向量索引
  - `MongoDBShiftDatetime.py`, `mongodb_exporter.py`：数据迁移/导出

### 3.3 数据/配置目录（程序与数据分离）

| 目录 | 用途 |
|------|------|
| `_config/` | 用户手动编辑的配置文件 |
| `_data/` | 程序运行时数据（向量库、SQLite、爬虫治理记录）**需重点备份** |
| `_export/` | 程序导出的数据（JSON、MongoDB 导出） |
| `_log/` | 日志文件及历史归档 |
| `_products/` | 产物目录（预留） |

---

## 4. 构建与运行

### 4.1 环境准备

1. 安装 Python 3.10+
2. 安装并启动 MongoDB（默认 `localhost:27017`）
3. 下载向量模型 `BAAI/bge-m3`（供 VectorDB 使用）
4. 克隆仓库并拉取子模块：
   ```bash
   git submodule update --init --recursive
   ```

### 4.2 安装依赖

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
pip install -r AIClientCenter/requirements.txt
pip install -r MyPythonUtility/requirements.txt
pip install -r IntelligenceCrawler/requirements.txt
pip install -r PyLoggingBackend/requirements.txt

# 安装 Playwright 浏览器
playwright install chromium
```

若依赖冲突，可尝试：
```bash
pip install -r requirements_freeze.txt
```

### 4.3 配置

1. **主配置**：复制 `_config/config_example.json` → `_config/config.json`
   - 修改 MongoDB 地址、Token、向量库路径等
   - 公开搜索限制：`intelligence_hub_web_service.public_search` 控制未登录用户的 `per_page`、最大页码、时间窗口、向量 `top_n`、速率限制、并发上限等。未登录用户仅允许按归档时间（Archive Time）搜索，避免 Publish Time 与 Archive Time 双重过滤。
2. **AI 服务配置**：复制 `AIClientCenter/AIClientConfigExample.py` → `_config/ai_client_config.py`
   - 填入实际 API Key、模型地址、优先级、分组限制等
3. **用户数据库**：运行 `python Scripts/UserManagerConsole.py` 创建管理员账号

### 4.4 启动服务

完整功能需要启动 **3 个进程**（可分布在不同机器）：

**A. 核心 Web 服务**
```bash
python IntelligenceHubLauncher.py
```
- 默认监听 `0.0.0.0:5000`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SleepySoft/IntelligenceIntegrationSystem](https://github.com/SleepySoft/IntelligenceIntegrationSystem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
