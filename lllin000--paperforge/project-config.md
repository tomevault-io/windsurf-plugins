---
trigger: always_on
description: > 本文档面向 **安装完成后的新用户** 和 **AI Agent**。如果还没有安装 PaperForge，请通过 Obsidian 插件市场安装，或查看 [README.md](README.md) 中的安装说明。Docs 版本与 v1.4.18 对应。
---

# PaperForge - Agent Guide

> 本文档面向 **安装完成后的新用户** 和 **AI Agent**。如果还没有安装 PaperForge，请通过 Obsidian 插件市场安装，或查看 [README.md](README.md) 中的安装说明。Docs 版本与 v1.4.18 对应。

---

## 0. 安装后检查清单（第一次使用前必做）

```
[ ] Zotero 已安装 + Better BibTeX 插件已启用
[ ] Obsidian 已打开当前 Vault
[ ] PaperForge 已安装 (pip install paperforge)
[ ] PaddleOCR API Key 已配置（在 .env 中）
[ ] 目录结构已创建（安装向导会自动完成）
[ ] Zotero 数据目录已链接到 <system_dir>/Zotero
[ ] Better BibTeX 已按下方步骤导出 JSON 到 <system_dir>/PaperForge/exports/
```

> 安装向导是增量式的：如果你选择的 Vault 或目录里已经有文件，PaperForge 只会补充缺失的目录和文件，不会删除已有内容。

### Better BibTeX 自动导出配置

这一步应在安装向导完成之后再做，因为 `exports/` 目录需要先由安装流程创建。

1. 打开 Zotero
2. 对你要同步的文献库或分类右键 → `导出...`
3. 选择格式：**Better BibTeX JSON**
4. 勾选 **"Keep updated"**（自动导出）
5. 保存到：`{你的Vault路径}/<system_dir>/PaperForge/exports/`
6. JSON 文件名会作为 Base 名称，例如 `library.json`、`骨科.json`

---

## 1. 核心架构（v2.1 契约驱动）

PaperForge 在 v2.1（1.4.17rc4）重构为 **契约驱动架构**，分 5 层：

```
CLI/Plugin 调用
    ↓ PFResult 契约 {ok, command, version, data, error}
┌─────────────────────────────────────────────┐
│  commands/    CLI 分发层                      │
├─────────────────────────────────────────────┤
│  adapter/bbt, zotero_paths, frontmatter    │  ← 独立可测
│  services/sync_service       ← 编排适配器   │
├─────────────────────────────────────────────┤
│  core/result, errors, state   ← 共享数据契约 │
├─────────────────────────────────────────────┤
│  worker/sync, ocr, status     ← 机械劳动     │
│  setup/6个类                  ← 安装流程     │
├─────────────────────────────────────────────┤
│  schema/field_registry.yaml   ← 字段注册表   │
│  doctor/field_validator.py    ← 字段校验     │
└─────────────────────────────────────────────┘
```

| 层级 | 组件 | 触发方式 | 作用 |
|------|------|----------|------|
| **契约层** | `core/`（PFResult, ErrorCode, 状态机） | 被所有模块引用 | 定义 CLI/Plugin/Worker 之间的数据交换格式 |
| **适配器层** | `adapters/`（bbt, zotero_paths, frontmatter） | 被服务层调用 | 封装外部数据格式与 I/O 操作 |
| **服务层** | `services/sync_service` | 被 worker 调度 | 编排适配器，实现业务逻辑 |
| **Worker 层** | `worker/`（sync, ocr, status, repair 等） | Python CLI | 后台自动化（机械劳动） |
| **Agent 层** | `/pf-deep`, `/pf-paper` | 用户手动触发 | 交互式精读（深度思考） |

**操作速查**：
| 你要做什么 | 在终端输入 | 在 OpenCode 输入 |
|-----------|-----------|-----------------|
| 同步 Zotero 并生成笔记 | `paperforge sync` | `/pf-sync` |
| 运行 OCR | `paperforge ocr` | `/pf-ocr` |
| 查看精读队列 | `paperforge deep-reading` | `/pf-deep`（精读具体文献） |
| 查看系统状态 | `paperforge status` | `/pf-status` |
| 修复状态分歧 | `paperforge repair` | （终端操作） |
| 验证安装配置 | `paperforge doctor` | （终端操作） |
| 查看帮助 | `paperforge --help` | （终端操作） |

---

## 2. 完整数据流

```
Zotero 添加文献
    ↓ Better BibTeX 自动导出 JSON
<system_dir>/PaperForge/exports/library.json
    ↓ 运行 sync
<resources_dir>/<literature_dir>/<domain>/<key> - <Title>.md（正式笔记，含 frontmatter）
    ↓ 用户在正式笔记 frontmatter 中设置 do_ocr: true
运行 ocr → <system_dir>/PaperForge/ocr/<key>/
    ↓ 用户在正式笔记 frontmatter 中设置 analyze: true
运行 deep-reading（查看队列，确认就绪）
    ↓ 用户执行 Agent 命令
/pf-deep <zotero_key>
    ↓ Agent 生成
正式笔记中新增 ## 🔍 精读 区域
```

---

## 3. 目录结构（Lite 版，5 个核心目录）

```
{你的Vault根目录}/
├── <resources_dir>/
│   ├── <literature_dir>/                    ← 正式文献笔记（sync 生成，含 frontmatter 状态跟踪）
│   │   ├── 骨科/
│   │   ├── 运动医学/
│   │   └── ...（你的分类）
│
├── <system_dir>/
│   ├── PaperForge/
│   │       ├── exports/                   ← Better BibTeX 自动导出的 JSON
│   │       │   └── library.json
│   │       ├── ocr/                       ← OCR 结果（每个文献一个子目录）
│   │       │   └── ABCDEFG/               ← Zotero key 作为目录名
│   │       │       ├── fulltext.md        ← OCR 提取的全文
│   │       │       ├── images/            ← 图表切割图片
│   │       │       ├── meta.json          ← OCR 元数据（含 ocr_status）
│   │       │       └── figure-map.json    ← 图表索引（自动创建）
│   │       ├── indexes/                   ← 索引缓存（formal-library.json 等）
│   │       └── config/                    ← 领域-收藏夹映射等配置
│   └── Zotero/                        ← Junction/Symlink 到 Zotero 数据目录
│
├── <agent_config_dir>/                         ← OpenCode Agent 配置（自动创建）
│   └── skills/
│       └── literature-qa/             ← 深度阅读 Skill
│           ├── scripts/
│           │   └── ld_deep.py         ← /pf-deep 核心脚本
│           ├── prompt_deep_subagent.md ← Agent 精读提示词
│           └── chart-reading/         ← 14 种图表阅读指南
│
├── .env                               ← API Key 等敏感配置
└── AGENTS.md                          ← 本文件
```

### 各目录作用速查

| 目录 | 内容 | 谁生成/修改 |
|------|------|------------|
| `<resources_dir>/<literature_dir>/` | 正式文献笔记（含 frontmatter + 精读内容） | sync 生成，Agent 写入精读 |
| `<system_dir>/PaperForge/exports/` | Better BibTeX JSON 导出 | Zotero 自动导出 |
| `<system_dir>/PaperForge/ocr/` | OCR 全文 + 图表切割 | ocr worker 生成 |
| `<system_dir>/PaperForge/indexes/` | 索引缓存（formal-library.json） | sync 生成 |
| `<system_dir>/PaperForge/config/` | 领域-收藏夹映射等 | 用户/安装向导配置 |
| `<system_dir>/Zotero/` | Zotero 数据目录的链接 | 安装时手动创建 junction |

---

## 4. 核心 Workers（Lite 版，4 个）

### sync
- **作用**：检测 Zotero 中的新条目并生成正式文献笔记
- **运行时机**：添加新文献到 Zotero 后，或需要更新笔记格式时
- **输出**：
  - `<resources_dir>/<literature_dir>/<domain>/<key> - <Title>.md`
- **示例**：
  ```bash
  paperforge sync
  # Legacy (备用):
  # python -m paperforge sync --vault "{vault路径}"
  ```

### ocr
- **作用**：将 PDF 上传到 PaddleOCR API，提取全文文本和图表
- **触发条件**：正式笔记 frontmatter 中 `do_ocr: true`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LLLin000/PaperForge](https://github.com/LLLin000/PaperForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
