---
trigger: always_on
description: Personal life journaling system with dual-dimension indexing. Use when user says 'record journal', 'log this', 'search logs', 'generate summary', '记日志', '写日记', '搜索记录', '生成摘要'. Features: auto weather, semantic search, attachment handling.
---


# Life Index Agent Skill

> **Authority Chain**: `bootstrap-manifest.json` 是 freshness / authority anchor。涉及安装、升级、repair、环境判断时，必须先刷新 `bootstrap-manifest.json`，再按其 `required_authority_docs` 刷新对应文档，然后才允许 sync checkout 或 route 判断。
> **工具参数与错误码**: 详见 [API.md](docs/API.md)

---

## Triggers & When to Use

| 意图 | 触发词 | 工具 |
|:---:|:---|:---|
| 记录日志 | "记日志"、"记录一下"、"写日记"、"记下来"、"log this"、"record this"、"write journal" | `write_journal` |
| 搜索日志 | "查找日志"、"搜索记录"、"找一下关于...的日记"、"search journal"、"find log" | `search_journals` |
| 智能搜索 | "帮我回忆..."、"我和女儿之间有哪些珍贵的回忆？"、"smart search" | `smart_search`（默认确定性 scaffold；LLM 编排需 `--use-llm`） |
| 历史同日 | "去年今天在做什么"、"历史上的今天"、"on-this-day"、"历史同日" | `on_this_day` |
| 编辑日志 | "修改日志"、"补充日记"、"更新记录"、"edit journal"、"update log" | `edit_journal` |
| 实体图谱 | "列出实体"、"解析人物关系"、"entity graph"、"谁是谁的..." | `entity` |
| 生成摘要 | "生成摘要"、"月度总结"、"年度总结"、"generate summary" | `generate_abstract` |
| 修订历史 | "查看修订历史"、"编辑记录" | `edit_journal` |
| 定时报告 | 日报/周报/月报/年报 | Life Index 无内置 scheduler；由宿主平台定时能力编排 CLI |

---

## Quick CLI Reference

**⚠️ 所有命令须在技能根目录（本文件所在目录）下执行。所有 Python/CLI 命令必须通过虚拟环境调用。**

**跨平台 venv 路径规则**：
- **Linux/macOS/WSL**: `.venv/bin/life-index` 或 `.venv/bin/python`
- **Windows**: `.venv\Scripts\life-index` 或 `.venv\Scripts\python`（首次排障/验证时优先显式使用此路径）

```bash
# 统一 CLI（推荐）
.venv/bin/life-index write --data '{"title":"...","content":"...","date":"2026-03-14","topic":["work"],"abstract":"...","mood":[],"people":[],"project":"","tags":[],"links":[]}'
.venv/bin/life-index search --query "关键词" --topic work --level 3
.venv/bin/life-index search --query "学习"  # 默认 keyword-only；加 --semantic 启用语义搜索
.venv/bin/life-index smart-search --query "我和女儿之间有哪些珍贵的回忆？"  # 确定性检索 scaffold（默认不调用 LLM）
.venv/bin/life-index smart-search --query "..." --use-llm  # 显式启用 LLM 编排搜索
.venv/bin/life-index smart-search --query "..." --explain  # 展示 Agent 决策详情
.venv/bin/life-index smart-search --query "..." --include-evidence  # 含 evidence pack + 检索诊断
.venv/bin/life-index on-this-day --date 2026-05-19 --years-back 3       # 历史同日回顾
.venv/bin/life-index edit --journal "Journals/2026/03/life-index_2026-03-14_001.md" --set-location "Beijing"
.venv/bin/life-index entity --list
.venv/bin/life-index entity --resolve "乐乐的奶奶"
.venv/bin/life-index abstract --month 2026-03
.venv/bin/life-index weather --location "Lagos,Nigeria"
.venv/bin/life-index index           # 增量更新
.venv/bin/life-index index --rebuild # 全量重建
.venv/bin/life-index health          # 安装健康检查

# Windows 用户主动写入时可用文件参数；onboarding 不应创建首写验证日志
.venv\Scripts\life-index write --data @journal-entry.json

# 开发者模式
.venv/bin/python -m tools.write_journal --data '{...}'
.venv/bin/python -m tools.search_journals --query "关键词"
.venv/bin/python -m tools.edit_journal --journal "..."
.venv/bin/python -m tools.generate_abstract --month 2026-03
.venv/bin/python -m tools.query_weather --location "Lagos,Nigeria"
.venv/bin/python -m tools.build_index
```

**安装 / 首次验证 / 故障恢复指针**：
- 首次安装、upgrade、repair、fresh install 判断 → 读 `AGENT_ONBOARDING.md`
- `ModuleNotFoundError`、venv 损坏、`health` 异常、Windows 首次写入转义问题 → 先读 `AGENT_ONBOARDING.md`
- 写入成功后的状态字段解释（`needs_confirmation` / `index_status` / `side_effects_status` / 附件处理计数）→ 读 `docs/API.md` 中 `write_journal` 返回语义

## Project Structure

```
life-index/                         # 技能根目录
├── SKILL.md                       # [本文件] 技能定义
├── tools/                         # 可执行工具目录
│   ├── write_journal/             # 写入日志（天气查询、附件处理、索引更新）
│   ├── search_journals/           # 搜索日志（L1/L2/L3 + 语义搜索）
│   ├── smart_search/              # 默认确定性智能检索 scaffold；--use-llm 才启用 LLM 编排
│   ├── edit_journal/              # 编辑日志（修改元数据、追加内容）
│   ├── entity/                    # 实体图谱（list/add/resolve/update）
│   ├── generate_index/            # 生成索引树（monthly/yearly/root）
│   ├── build_index/               # 构建索引（FTS5 + 向量索引）
│   ├── query_weather/             # 查询天气
│   ├── backup/                    # 备份日志数据
│   ├── verify/                    # 数据完整性校验
│   ├── timeline/                  # 时序摘要流
│   ├── on_this_day/               # 历史同日回顾
│   ├── migrate/                   # Schema 链式迁移
│   ├── eval/                      # 搜索质量评估
│   ├── dev/                       # 开发/验收辅助工具
│   └── lib/                       # 共享库（SSOT）
├── docs/                          # API.md, ARCHITECTURE.md
└── references/                    # WEATHER_FLOW.md, SCHEDULE.md
```

**关键约定**：
- **虚拟环境**: 所有命令通过 `.venv/bin/`（Windows: `.venv\Scripts\`）前缀调用
- **用户数据目录**: `~/Documents/Life-Index/`（日志、附件、索引，与代码物理隔离）
- **跨平台路径**: 自动处理（Agent 传原始路径即可，工具自动转换 Windows↔WSL）
- **健康检查**: 遇到异常时先运行 `.venv/bin/life-index health` 诊断

---

## Core Constraints

### Content Preservation (MUST)

**100% 保留用户原始输入**：
- 不修改段落结构
- 不改变标题层级
- 不转换列表格式
- 不添加序号标记
- **⚠️ 不修改文件名（不在中英文间添加空格）**

```markdown
# ❌ 错误
用户输入："1、完成A 2、完成B"
Agent 改成："1. 完成A\n2. 完成B"

# ❌ 错误（文件名被修改）
用户附件："C:\Users\test\Opus审计报告.txt"
Agent 改成："C:\Users\test\Opus 审计报告.txt"  ← 添加了空格

# ✅ 正确
用户输入什么，content 和附件路径就原封不动传递什么
```

### Guardrails

- **永不删除文件**：编辑只修改内容
- **数据隔离**：数据在 `~/Documents/Life-Index/`，与代码分离
- **天气处理**：详见 [WEATHER_FLOW.md](references/WEATHER_FLOW.md)

```markdown

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrDexter6000/life-index](https://github.com/DrDexter6000/life-index) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
