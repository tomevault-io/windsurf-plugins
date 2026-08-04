---
trigger: always_on
description: 本文件为 Claude Code 在 Paper-Tutor-Skills 仓库的工作指导。
---

# CLAUDE.md

本文件为 Claude Code 在 Paper-Tutor-Skills 仓库的工作指导。

## 项目概述

Paper-Tutor-Skills 是一套装进编程智能体的**学术辅导** agent skills，简体中文优先，面向中文研究生与科研工作者。v0.1.0 起提供 25 个命令入口 / 23 个 skill 目录，覆盖学术研究 5 阶段全生命周期。

核心分工：**AI 负责效率（检索、整理、核对、结构化），人负责研究决策（想法、判断、数据、结论）**。

产品定位是**辅导，不是代笔**：不代生成研究想法与原创结论、不代跑实验、不编造数据与引用、不端到端无人确认出稿、不做规避 AI 检测、不出具伦理豁免判断、不接收他人在审稿件。每条边界的审查判据、依据与出口指引在 [边界拒绝清单](skills/_shared/references/边界拒绝清单.md)（A 类产品边界 8 条 + B 类拒绝的机制 4 条 + 四问审查判据）——**新功能提案先过这把尺子**。

两条不与其他目标权衡的底线：**不编造**（文献核验以真实 API 响应为准，不凭模型记忆）、**留痕如实**（使用记录如实反映人机分工，不美化）。

## 常用命令

```bash
# 全量测试（CI 同命令，秒级跑完）
python3 -m pytest tests/ -q --ignore=tests/datasources/smoke_real_api.py

# 单个 skill 的测试
python3 -m pytest tests/paper-verify -q

# 真实 API 冒烟（打 Crossref / OpenAlex 等真实端点，有网络与配额依赖，按需手工跑）
python3 -m pytest tests/datasources/smoke_real_api.py -q

# 把 skills/ 镜像同步安装到本地测试项目（幂等；本地开发脚本，不入库）
python3 sync_skills.py --list                                   # 干跑，只打印计划
python3 sync_skills.py                                          # 同步全部
python3 sync_skills.py --skill paper-init --target <skills 目录>  # 选择性同步
```

本机无 `python` 别名，一律用 `python3`；若环境装了 pytest-asyncio 须补 `-p no:asyncio`。CI 配置见 [`.github/workflows/tests.yml`](.github/workflows/tests.yml)（push / PR 触发，只跑 pytest，无覆盖率门与 lint 门）。

## 架构

**`skills/` 是分发单元**——整目录成套安装。skill 之间以相对路径（`../_shared/...`）互引，单个 skill 拆出去会断链。

```text
skills/
├── paper-<word>/          # 一个 skill 一个目录，目录名 = 命令名去斜杠
│   ├── SKILL.md           # 必需：frontmatter（name、description）+ 行为指令
│   ├── references/        # 按需：分层加载的详细规则（核验协议、话术库、格式规则）
│   ├── scripts/           # 按需：本 skill 专用的确定性脚本
│   └── assets/            # 按需：模板等静态资源
└── _shared/               # 唯一的非 skill 目录（下划线前缀 = 永不放 SKILL.md）
    ├── commands.yaml      # 命令主清单（单一事实来源）
    ├── paper_shared/      # Python 包根
    │   ├── datasources/   # 文献 API 客户端 + 缓存 / 分批退避 / DOI 注册机构路由
    │   └── toolchain.py   # pandoc / xelatex / 中文字体探测（doctor 与 typeset 共用）
    └── references/        # 跨 skill 权威定义（见下「共享约定」）
```

`_shared/` 的准入规则：**至少两个 skill 使用**的资源才进来，单 skill 专用的留在自己的 `scripts/` / `references/`。

数据源层已接入 arXiv、OpenAlex、Semantic Scholar、Crossref、PubMed、ERIC，DOI 按注册机构分流（Crossref / DataCite / ISTIC；ISTIC 即中文 DOI，径直落「待人工核对」）。

其余目录：`tests/` 与 skills 平行（每个 skill 一个子目录 + `fixtures/` 共享语料）；`evals/` 放裸模型对比记录；`scripts/` 放仓库级工具（`extract_changelog_notes.py`）；入库的文档只有 `docs/prd/`（产品定位与设计理念，权威）与 `docs/examples/`。

### 插件分发（Claude Code 与 Codex）

仓库根的清单文件让**仓库自身成为一个插件市场**，两个宿主都能一键装：

| 宿主 | 清单文件 | 用户侧命令 |
| --- | --- | --- |
| Claude Code | `.claude-plugin/plugin.json` + `.claude-plugin/marketplace.json` | `/plugin marketplace add cabbage2000-lab/paper-tutor-skills` |
| Codex | `.codex-plugin/plugin.json` + `.agents/plugins/marketplace.json` | `codex plugin marketplace add cabbage2000-lab/paper-tutor-skills` |

这是**增量增强、不替换任何东西**：`skills/` 散装主体原样保留，其他宿主照常按 SKILL.md 装载——与硬规则 3 的跨宿主中立不冲突。

- **plugin 根 = 仓库根**，所以 `skills/` 天然在正确位置。两个 `.*-plugin/` 目录里**只放清单文件**，skill 内容绝不能塞进去（官方明确的坑，放错则一个 skill 都加载不到）。
- **两套清单不能合并**：格式实打实不同——Codex 的 plugin.json 多要 `skills: "./skills/"` 指针与一整块 `interface`（`displayName` / `shortDescription` / `longDescription` / `developerName` / `category` / `capabilities` / `defaultPrompt` 全必填），marketplace 条目的 `source` 是对象且必带 `policy`。四份清单的公共字段由 [`tests/test_plugin_manifest.py`](tests/test_plugin_manifest.py) 守住不漂。
- **`skills/_shared` 会让官方预检脚本报错，但不影响真实安装**——Codex 的 `plugin-creator/validate_plugin.py` 只跳过 `.` 开头的目录，遇到无 SKILL.md 的 `_shared` 会判 `skill '_shared' is missing 'SKILL.md'`。运行时摄取（`codex plugin add`）与 skill 加载器都只认含 SKILL.md 的目录，`_shared` 被静默忽略，23 个 skill 全部正常加载（2026-07-28 实测 codex-cli 0.145.0）。**因此不要为了讨好预检脚本去给 `_shared` 改名或补 SKILL.md**——那会同时踩中硬规则 2 与硬规则 6，换来的只是一个预检脚本的绿灯。
- **plugin name 取 `paper-tutor`**，命令因此呈现为 `/paper-tutor:paper-init`——前缀与命令名都带 paper，确实啰嗦。更简洁的 `/paper:init` 需要把 23 个 skill 目录改名为 `init`/`verify` 等，牵动硬规则 2（`name` = 目录名 = 命令名）、主清单、两个 README 与全部内部引用。**权衡结论：不值得，保持 `paper-tutor`。** 要推翻这个结论，得连同上述几处一起改。

宿主的 skills 目录各不相同，**装错位置的表现是命令一个都不出现**（不是报错，所以很难自查）：Claude Code 读 `~/.claude/skills/` 与项目级 `.claude/skills/`；Codex 读 `~/.codex/skills/` 与项目级 `.codex/skills/`、`.agents/skills/`，**不读 `.claude/skills/`**。散装安装用 `sync_skills.py --target <目录>`（本地脚本，不入库）。

### 发版

**开版本骨架**（`chore: 开 x.y.z 版本骨架`）：把 `CHANGELOG.md` 的段落标题定为 `## [x.y.z] — 待定`，段落里放一块含 `RELEASE-BLOCKER` 哨兵的 HTML 注释，并同步下面第 2 步的九处版本号。骨架期一路往段落里累加条目，CI 照常绿。

**发版**：

1. **收口 CHANGELOG 段落**——写摘要句（一句话说清本版让使用者多了什么能力，或哪个旧能力真的能用了）、**删掉 `RELEASE-BLOCKER` 哨兵注释块**、清掉段落里其余 HTML 注释、标题日期改为实际发版日。这四件事漏了哪样，[`extract_changelog_notes.py`](scripts/extract_changelog_notes.py) 都会让发布流程直接失败——**宁可不发，也不发空白或占位的 Release**（0.1.5 那次骨架一路留到打 tag 前，靠人肉复核才拦住，这道检查是那次的产物）。检查有意只在发版时刻生效、不做成 pytest：骨架在开发期是合法状态，长红的 CI 会让人对红灯脱敏。守卫见 [`tests/test_release_notes.py`](tests/test_release_notes.py)；
2. 同步 `.claude-plugin/` 两份清单、`.codex-plugin/plugin.json`、`skills/_shared/VERSION` 的 `version`，以及两个根 README 的三处版本号（badge / 简介段 / 快速开始段；[`tests/test_plugin_manifest.py`](tests/test_plugin_manifest.py) 守着处处一致）。**README 是最容易漏的一处**——它曾停在 v0.1.2 漂过三个版本；
3. 合入 main 后 `git tag -a vx.y.z -m "..." && git push origin vx.y.z`；
4. [`release.yml`](.github/workflows/release.yml) 自动跑测试并创建 Release。测试不过则不发布。

### 五个结构守卫测试（改动前先知道它们守什么）

它们抓的是「改了 A 忘了同步 B」这类无声漂移，失败诊断路径各不相同：

| 测试 | 守什么 |
| --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cabbage2000-lab/paper-tutor-skills](https://github.com/cabbage2000-lab/paper-tutor-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
