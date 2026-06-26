---
trigger: always_on
description: > **CLAUDE.md 是行为规范的唯一真相源；AGENTS.md 是其镜像，二者须逐字一致**（仅 agent 专属命名差异除外：Claude Code↔Codex、`.claude/skills/`↔`.agents/skills/`、skills 镜像中入口文件指称 CLAUDE.md↔AGENTS.md 互换；守护测试 `scripts/tests/test_release_guards.py::TestMirrorSync`）。改任一文件时必须同步另一份。
---

# GroundMap — Schema（行为规范）

> **CLAUDE.md 是行为规范的唯一真相源；AGENTS.md 是其镜像，二者须逐字一致**（仅 agent 专属命名差异除外：Claude Code↔Codex、`.claude/skills/`↔`.agents/skills/`、skills 镜像中入口文件指称 CLAUDE.md↔AGENTS.md 互换；守护测试 `scripts/tests/test_release_guards.py::TestMirrorSync`）。改任一文件时必须同步另一份。

> 本文件是**所有外部 agent 操作知识库时**的行为规范。Claude Code、Cursor 或任何接入知识库的 agent 在执行操作前必须读取本文件。

---

## 项目定位

**本项目准备在 GitHub 上开源，提供给所有人使用**。这意味着 agent 在执行任何操作（写代码、写文档、写 markdown、改 schema、加注释、命名变量等）时必须以"公开项目、面向全球开发者"为前提：

- 代码、注释、commit message、文档**不得包含**任何个人隐私信息（真实姓名、邮箱、私钥、token、API key、内网地址、私人路径硬编码等）
- 不得包含仅作者本人能理解的语境（"我昨天和老王说的那个" / "公司内部那套" / 具体客户名）
- 设计与文档应假设**陌生贡献者也能读懂并复现**——路径用相对路径或环境变量，依赖与运行步骤显式声明，避免"只在我机器上能跑"的假设
- 示例数据应可公开分发，不引用未授权的第三方版权内容
- `my_thoughts/`、`raw/` 中的私人笔记与原始资料**默认不进入开源仓库**（已由 `.gitignore` 处理），但 agent 仍应避免把这些内容里的敏感片段复制到 `wiki/` / `scripts/` / `web/` 等会公开的目录

---

## 核心设计原则（不可违反）

1. **知识库不调用 LLM**。所有 LLM 推理由外部 agent 完成；知识库本身只暴露 MCP 工具与 REST API，不内嵌任何 agent runtime、LLM SDK 或对话能力。
   - **本条原则的范围**：`scripts/`、`web/`（KB 核心）严禁内嵌 LLM SDK。
   - **例外**：`tools/debug-console/` 是独立子项目，**作为 KB 的外部客户端存在**，可以引入 LLM SDK。它只通过 HTTP 调主 `web/` 的 REST API（`/api/agent-tool` 等），不直接读 markdown / `.cache/`。删掉 `tools/` 整个目录不影响 KB 任何功能。
2. **markdown + Git 是唯一真相源**。SQLite 索引（`.cache/index.db`）是派生层，可随时从 markdown 全量重建。删 `.cache/` 系统仍能跑。
3. **完整页面优先**。所有读取工具返回**完整页面或完整 H2/H3 段**，绝不返回 chunk。
4. **严禁 embedding 召回**。embedding 模型 / 向量存储 / 文档切片不出现在系统的任何"找相关内容"逻辑中。检索靠 BM25 全文 + 元数据过滤 + agent 阅读完整页面。
5. **写权限硬约束**：写 `raw/**`、`my_thoughts/**`、含 `#human-only` 标签或 `locked: true` frontmatter 的文件 → 工具直接拒绝（PermissionError），不是 ask、不是 warn、是 deny。
6. **删除即标记**：所有"删除"操作只能改 `status: deprecated`，绝不真删文件；历史信息有内在价值。

---

## 实际操作入口（无 MCP）

本项目**没有 MCP server**（v0.5 已废弃，见文末「未来演进路线」）。所有 KB 操作走两条路：

| 层级 | 入口 | 用途 |
|---|---|---|
| **工作流** | Skill：`kb-ingest` / `kb-query` / `kb-lint` / `kb-conflict-resolve` / `kb-export` | 端到端的摄入 / 查询 / 周检 / 冲突 / 导出 |
| **原子操作** | `python scripts/k.py <subcommand>` | outline / search / read-section / read-block / backlinks / outlinks / annotate-section / list-source-issues / list-broken-refs / list-to-update / list-orphans / list-conflicts / health 等 |
| **直接读写** | `Read` / `Edit` / `Write` | `wiki/**` markdown 文件 |

下文「四大操作流程」中出现的 `read_page` / `archive_analysis` / `mark_conflict` / `append_log` / `git_commit` 等是**抽象动作名（接口契约）**，描述 agent 应做什么，**不是可调用的 MCP tool**。落地实现 = skill 文档 + `k.py` 子命令 + `web/lib/operations.ts` 的 server action。

> **不要尝试 `ToolSearch "mcp__kb__*"` 或调用任何 `mcp__kb__*` 工具** —— 它们不存在，会返回 `No matching deferred tools found`。

---

## 目录结构

```
groundmap/                   # 引擎根（通用代码 + 规范）
├── CLAUDE.md                # 行为规范（唯一真相源）
├── AGENTS.md                # CLAUDE.md 的镜像（Codex 入口）
├── GroundMap-设计文档.md     # 系统设计文档
├── scripts/                 # 自动化脚本（k.py、convert.py）——通用
├── web/                     # Web 管理台（Next.js）——通用
├── .claude/skills/          # Claude Code 技能定义——通用
├── workspaces/              # 多主题工作区（可切换）；本仓自带 3 个示例库
│   ├── smb-ecommerce/       # ← 示例：跨境电商
│   │   ├── wiki/            # agent 维护的 Wiki（可读写，随仓分发）
│   │   ├── raw/             # 原始资料（agent 不可改；版权原因不随仓分发，仅留 .gitkeep）
│   │   ├── exports/         # 输出物归档
│   │   ├── my_thoughts/     # 人类专属区（agent 只读；不随仓分发，仅留 .gitkeep）
│   │   ├── .cache/          # SQLite 索引（gitignored，可重建）
│   │   └── log.md           # 操作日志
│   ├── rag-evolution/       # ← 示例：RAG 演化史 (2023-2025)
│   │   └── ...
│   └── ai-ml-demo/          # ← 示例：AI/ML（v0 归档库，多数页 status: deprecated）
│       └── ...
├── wiki/                    # 引擎级 wiki（仅 _templates）
├── tools/                   # 独立子工具，不属于 KB 核心
│   └── debug-console/       # 调试界面（v0.3）
└── .cache/                  # 引擎级缓存
```

---

## 双仓库同步约定（dev ↔ release）

本仓库（`AI知识库/`）与 `groundmap-release/` 是**两个独立 Git 仓库**承担不同角色：

| 仓库 | 角色 | 数据 | 分支 |
|---|---|---|---|
| `AI知识库/`（本仓） | **开发版** | 含实际 wiki / raw / exports 数据 | `rag-evolution-ip-standard` 等 |
| `groundmap-release/` | **发布版** | 引擎 + 3 个精选示例 demo 库（仅 `wiki/` 随仓；`raw/`、`my_thoughts/` 不分发）+ 已审的发布准备改动 | `main` |

**哪些修改必须双仓同步**（任一改完都需在另一仓做对应改动，否则下次 sync 漂移）：

1. **`scripts/k.py`、`scripts/convert.py`、`scripts/section_parser.py`、其他通用引擎代码**：
   同步整个文件；release 的 workspace fallback 逻辑（k.py 第 2644-2671 行）保留不动。
2. **`scripts/tests/`**（含 `TestMirrorSync` 守护）：**完全镜像**——dev 改了测试，release 必须改相同处。
3. **`.claude/skills/kb-*/SKILL.md` 与 `.agents/skills/kb-*/SKILL.md`**：SKILL.md 内容**逐字相同**（`.claude ↔ .agents` 由 `TestMirrorSync.test_skills_mirror` 守），dev 与 release 之间靠人肉 / rsync 同步。
4. **`web/`（Next.js 管理台）**：dev 与 release 同步主要 UI 改动；release 可能含更多发布准备（i18n key 整理、未发布特性等），合并时以 release 为基线。
5. **`docs/`**（用户文档）：dev 写新内容 → 同步到 release；release 的发布准备改动（demo 视频、新手教程）一般不回 dev。

**哪些修改不要镜像到 release**：

- dev 里**实际在用的**工作数据（`workspaces/<name>/raw/**`、私人 `my_thoughts/**`、`exports/**`）——不镜像到 release。release 自带的是另一套**精选 demo 库**：仅 `wiki/` 随仓分发，`raw/`、`my_thoughts/` 不分发。
- dev 专属的实验性 lint / 临时脚本。

**不变量清单**（任一变动都视作"破坏不变量"、必须同时同步）：

- `RELATION_TYPES` 白名单 7 类（k.py ↔ web/lib/markdown.ts）
- `WIKILINK_RE` 正则（k.py ↔ web/lib/markdown.ts）
- `TestMirrorSync` 的归一化规则（CLAUDE.md ↔ AGENTS.md ↔ `.claude/skills ↔ .agents/skills`）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Qinbf/groundmap](https://github.com/Qinbf/groundmap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
