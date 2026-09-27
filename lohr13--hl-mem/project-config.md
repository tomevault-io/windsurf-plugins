---
trigger: always_on
description: HL-Mem 是面向 AI Agent 的本地优先记忆系统。核心设计：事件溯源双通道 + 双时间模型 + 证据链 + slot+tags 分类体系 + importance 联动 TTL + 多因子召回 + 完整生命周期管理。
---

# HL-Mem 项目 AGENTS.md

## 项目概述

HL-Mem 是面向 AI Agent 的本地优先记忆系统。核心设计：事件溯源双通道 + 双时间模型 + 证据链 + slot+tags 分类体系 + importance 联动 TTL + 多因子召回 + 完整生命周期管理。

**当前版本：v1.1.7（2026-09-08）**

## 技术栈

- **运行时**：Python 3.12+，FastAPI + uvicorn
- **存储**：SQLite WAL + FTS5（全文检索）+ 向量 BLOB（默认 `sqlite_scan` 两阶段精确扫描；可选 `sqlite_vec` 后端）
- **LLM 提取**：API 密钥通过 .env 配置，provider/model 通过 TOML 配置，使用结构化 JSON 输出
- **Embedding**：API 密钥通过 .env 配置，provider/model/维度通过 TOML 配置
- **Reranker**：API 密钥通过 .env 配置，provider/model 通过 TOML 配置
- **Provider 插件**：`hl_mem.providers` Entry Point + 显式 allowlist；LLM/Embedding/Reranker 稳定，Image Experimental；宿主统一治理 HTTP 与用量
- **分类体系**：SLOT_REGISTRY（15 operational slot + 40 topic tags；Phase 18 已接入检索，soft boost 默认开启，独立 tag channel 默认关闭待评测）
- **TTL**：retention 纯函数（scope × importance 三档）
- **近重复治理**：摄入层确定性 near-copy 复用 + 维护层 `dedup_pairs` 轮转审查 + 召回层有界动态折叠；旧 DedupJudge 保持 audit-only
- **包管理**：uv（lockfile: uv.lock）
- **测试**：pytest + pytest-asyncio（asyncio_mode=auto），全量 unittest 由 GitHub Actions 验证

## 代码结构

```
src/hl_mem/
├── api/                    # FastAPI 适配层
│   ├── server.py              # REST API (24 route operations)
│   └── schemas.py             # Pydantic DTO
├── application/            # 共享应用服务
│   ├── ingest.py              # IngestService
│   ├── recall.py              # RecallService
│   ├── deletion.py            # DeletionService 物理删除闭包
│   ├── forget.py              # ForgetService 入口适配
│   └── restore.py             # tombstone restore replay
├── domain/                 # 纯领域逻辑（不依赖基础设施）
│   ├── claims/                # claim 写入/冲突/去重/retention/query_tags
│   ├── temporal.py            # 双时间可见性
│   ├── relations.py           # 记忆关系
│   ├── entity.py              # 实体归一化
│   ├── recall.py              # 召回领域逻辑
│   └── content.py             # 多模态内容协议
├── core/                   # 纯数学
│   └── vector.py              # cosine similarity
├── ingest/                 # 数据摄入
│   ├── admission.py          # 纯函数 Claim 准入策略
│   ├── llm_extractor.py       # LLM 提取器
│   ├── extractors.py          # FakeExtractor / LLMExtractor
│   ├── chunking.py            # 结构感知分块
│   ├── embedder.py            # Embedding 向量化
│   └── event_filter.py        # 事件预过滤
├── llm/                    # LLM 客户端（Provider 解耦）
│   ├── client.py              # LLMClient
│   ├── providers.py           # 百炼/智谱/OpenAI-compatible
│   └── types.py               # LLMRequest/LLMResponse
├── plugins/                # 公共 Provider 契约、发现、Registry、宿主传输与用量代理
├── recall/                 # 召回层
│   ├── staged_pipeline.py     # FTS + Dense RRF、soft boost 与可选 Reranker
│   ├── trace.py               # SearchTrace 可观测性
│   ├── ranking.py             # 多因子排序
│   ├── reranker.py            # Reranker 重排
│   ├── relation_expansion.py  # 一跳关系扩展
│   └── observation.py         # 派生记忆构建
├── storage/                # 存储层（按职责拆分）
│   ├── database.py            # SQLite WAL + migration runner
│   ├── claims.py              # ClaimRepository
│   ├── events.py              # EventRepository
│   ├── evidence.py            # EvidenceRepository
│   ├── experience.py          # ExperienceRepository
│   ├── jobs.py                # JobRepository
│   ├── relation_proposals.py  # 关系候选审计
│   ├── usefulness.py          # 反馈效用聚合
│   ├── backup.py              # 在线备份
│   ├── tombstones.py          # 独立删除账本 sidecar
│   └── migrations/            # 57 SQL migrations (001-057) + Python data migrations
├── workers/                # 后台任务
│   ├── worker.py              # Job 租约/进度/维护循环
│   ├── job_handlers.py        # Job handler 与分派边界
│   ├── integrity.py           # dangling 引用巡检
│   ├── ttl.py                 # TTL 过期
│   ├── decay.py               # 置信度衰减
│   ├── consolidate.py         # LLM 语义归并
│   ├── deduplicate.py         # 跨 subject 语义去重
│   ├── backfill_expires_at.py # TTL 回填工具
│   ├── discover_relations.py  # 关系候选发现
│   ├── mental_models.py       # Mental Model 维护
│   ├── rebuild_usefulness.py  # usefulness 重建
│   └── induce_policies.py     # 策略归纳
├── experience/             # Experience 通道
│   └── service.py             # Episode/Trace/Policy
├── evaluation/             # Benchmark / LongMemEval
├── observability/          # 审计日志与 LLM spans
├── security/               # 图片输入边界与 retention 策略
├── adapters/hermes/        # Hermes 集成
│   ├── provider.py            # HermesMemoryProvider
│   └── plugin/                # 薄委托层
├── mcp/
│   └── server.py              # MCP 工具契约
├── components.py           # 统一组件工厂
├── settings.py             # Settings dataclass + 校验
├── protocols.py            # 接口协议
├── errors.py               # 异常族
├── http_utils.py           # 统一重试工具
├── lifecycle.py            # 状态机守卫
└── cli.py                  # CLI 入口
```

## 测试

开发过程中只运行与改动直接相关的测试，不启用并行：

```powershell
.venv\Scripts\python.exe -m pytest tests/unit/test_relevant_file.py -q --tb=short
```

最终候选通常只运行一次核心全套；只有该次运行暴露问题并导致代码修改时，才允许第二次：

```powershell
.venv\Scripts\python.exe -W error::ResourceWarning -m pytest tests/ `
  -m "not release_only" -n 4 -q --tb=short --durations=25
```

发布前另行串行运行一次发布型验证：

```powershell
.venv\Scripts\python.exe -m pytest tests/ -m release_only -q --tb=short
```

相同 commit 的 fast-forward 合并复用已有验证结果，不重复全量测试。Python 3.13 是唯一的 CI 权威测试环境；
包的 Python 安装范围保持不变，其他版本可安装不代表获得 CI 兼容性承诺。

## 关键设计决策

### 写入管线

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lohr13/hl_mem](https://github.com/lohr13/hl_mem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
