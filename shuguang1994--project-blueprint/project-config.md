---
trigger: always_on
description: > AI 编程助手的强制开发规范。v1.9.0 | 更新: 2026-09-14（建立 2026-08-01）
---

# Project Blueprint 项目开发规范 (AGENTS.md)

> AI 编程助手的强制开发规范。v1.9.0 | 更新: 2026-09-14（建立 2026-08-01）
> AI 工具: Trae | 加载: always_applied

***

## 一、项目身份

- **项目**: Project Blueprint — 为新项目一键建立完整 AI 编程规范体系（AGENTS.md + 文档骨架 + CI/CD + 测试制度 + Git 规范）的开源 AI Agent 技能包。
- **形态**: 纯 Markdown 项目，无代码、无构建、无测试、无运行依赖。核心逻辑为 `SKILL.md`（113 行索引层 + `references/step-*.md` 7 个 Step 细节），配套 `references/` 知识库与 `scripts/` 门禁层。
- **技术栈**: Markdown (SKILL.md 格式) + 95 组件条目知识库（18 个二级章节 / 16 个技术栈维度） + MCP 工具知识库 + 门禁配方库 + 宪法层生长机制 + WebSearch 联网回退
- **版本**: v1.9.0（语义化版本，tag 发布）
- **仓库**: GitHub `origin` = https://github.com/shuguang1994/project-blueprint / Gitee `gitee` = https://gitee.com/shuguang1994/project-blueprint
- **作者**: 曙光 (shuguang1994) | License: MIT
- **安装**: `npx skills add shuguang1994/project-blueprint`（国际）/ `npx skills add https://gitee.com/shuguang1994/project-blueprint.git`（国内）/ `dsh plugin --profile web add 'github:shuguang1994/project-blueprint'`（DeepSeek Harness）

## 二、常用命令

```bash
# 本仓库无编译/测试命令，常用操作为 Git 双远程推送：
git add <files>                                # 暂存（按文件逐一添加，勿 git add -A）
git commit -m "<type>(<scope>): <description>" # 提交
git push origin main                           # 推送 GitHub
git push gitee main --tags                     # 推送 Gitee 镜像 + 版本标签
git fetch origin && git fetch gitee            # 同步双远程
git revert <commit>                            # 线上问题回滚

# 本仓库门禁层（scripts/）实跑：
npm run verify                                 # 统一入口（= node scripts/verify.mjs）
npm run verify:ci                              # 全量（= --stage=ci，CI 用同一入口）
node scripts/verify.mjs --stage=pre-push       # 按阶段过滤
node scripts/check-constitution.mjs            # 宪法自校验（AGENTS.md 红线 ↔ 门禁清单）

# 门禁装配点（见 4.6）：
#   CI      → .github/workflows/verify.yml（push / PR 自动跑 --stage=ci）✅ 已装配
#   pre-push → 未装配（单行启用：git config core.hooksPath .githooks，见 4.6）⚠️

# 技能安装/更新（验证对外文档描述一致性时参考）：
npx skills update project-blueprint

# DSH 插件包同步（修改根 SKILL.md / references/ 后，发版前运行）：
node dsh-plugin/scripts/sync-skill.mjs
```

## 三、Boundaries

**Allowed**: `SKILL.md`、`references/`、`README.md`、`README_CN.md`、`CHANGELOG.md`、`PROJECT_STATUS.md`、`AGENTS.md`、`docs/`（仅对外内容）、`internal-docs/`（本地内部文档，受 `.gitignore` 约束不发布）、`scripts/`、`.github/workflows/`（门禁装配点）、`.trae/specs/`（spec 驱动开发三件套）、`.gitignore`、`dsh-plugin/`（不含 `dsh-plugin/skills/`，由同步脚本生成）

**Ask First**:
- 版本号升级（vX.Y.Z）或破坏性变更（如 Step 流程重构、文件重命名）
- 修改 `SKILL.md` 中的流程步骤、触发条件、输出格式约定
- 新增/删除 `references/` 下的参考文件
- 双远程仓库的 git 操作（push/force/tag）

**Never Touch**: `.git/`、`LICENSE`（协议条款）、证书/密钥、任何 `.env` 文件、`node_modules/`

## 四、强制规范

### 4.1 文档规范

```
✅ README.md 与 README_CN.md 内容保持同步（中英对照，同一特性两处都要更新）
✅ 新增特性同时更新：README / README_CN / CHANGELOG / PROJECT_STATUS
✅ 初始化项目时生成 CHANGELOG.md（[Unreleased] 占位，首次发版后转版本号记录）
✅ 文件名中英双语标注，按 A/B/C/D/E 五级分类存放
✅ 代码块必须闭合（开闭围栏语言标记一致），防止后续章节被误渲染
✅ 对外数字口径唯一源：同一指标全仓一致，以知识库实际条目数为准（如组件条目 95 / 技术栈维度 16）
✅ 公开边界：`docs/` 只放面向社区的内容（发布说明）；自审 / 竞品对标 / 评估复核类放 `internal-docs/`（受 `.gitignore` 约束，不随公开仓发布），并在 `docs/README.md` 的「编号预留」表登记编号（否则 `docs-consistency` 报缺号 error）
✅ 新增知识库条目后同步更新 README 技术栈覆盖表
```

### 4.2 SKILL.md 编写规范

```
✅ 执行原则：探测优先 / 最小侵入 / 不确定就问 / 一步一验证
✅ 步骤编号固定：Step 1 自主发现引擎 → Step 7 持续自适应机制
✅ 规模上限：SKILL.md 正文 ≤ 200 行、各 references/step-*.md ≤ 500 行；超限按 Step 拆到 references/
✅ 引用 knowledge-base.md 时按 ### [组件名] 定位，不读全文
✅ 未知组件触发联网回退，且 {currentYear} 用系统真实年份，禁止硬编码
❌ 不在 SKILL.md 中硬编码固定文件列表 / 固定映射表（保持"零固定表"设计）
❌ 不写本项目特定信息（IP、人名、公司名）— 用占位符
```

### 4.3 knowledge-base.md 条目格式

条目基础三段，高频组件附加第 4 段 `Gate`（新增条目一律四段齐全）：

```
### [组件名]
**Commands**: 精确可执行命令
**Conventions**: ❌/✅ 规范要点
**CI job**: GitHub Actions yaml 片段
**Gate**: 可机检红线 → 检查方式（命令 / 脚本要点 / 适用条件）
```

### 4.4 版本与发布规范

```
✅ 语义化版本：major.minor.patch，破坏性变更升 major（如 v1.4.0 自主发现引擎重构）
✅ CHANGELOG.md 按版本号倒序记录，标注 Breaking Change / Added / Changed / Fixed
✅ 每次发版：CHANGELOG 更新 → tag 打版本 → push origin main → push gitee main --tags
✅ PROJECT_STATUS.md 同步更新版本演进表
```

### 4.5 架构原则

```
✅ 高内聚低耦合 — 各 Step 职责单一，Step 间通过探测结果流转
✅ 复用已有代码 — 优先复用 references/ 已有条目，避免重复定义
✅ 增量友好 — 已有项目只补缺失，不覆盖已有配置
✅ 三层递进 — 知识库精确匹配 → 命名模式启发 → 联网搜索
✅ 组合优于继承 / 避免全局状态 / 纯函数优先
```

### 4.6 门禁即规则（本仓库同样适用）

```
✅ 本仓库虽为纯 Markdown 项目，仍适用元规则：新增/修改阻断级规范时必须同步可执行检查
   （本仓库门禁层为 scripts/gates.json + verify.mjs + check-constitution.mjs；
     skill 侧校验参考实现保留在 references/docs-check.mjs 与 references/drift-check.mjs，不复制到 scripts/）
✅ 门禁装配点（本仓库实况——声明必须与装配一致，不得只写不接）：
   CI = .github/workflows/verify.yml（push / PR 自动跑 --stage=ci）已装配；
   pre-push = 未装配（无生效钩子配置），本地按需：git config core.hooksPath .githooks
✅ gates.json 的 stage 是分类字段（供 --stage= 过滤），不等同于「已接线」；未接线的阶段须显式标注
✅ 无法机检的规则须标注 [无门禁] 并写明原因
✅ 三条元规则（无门禁不立规 / 缺陷必闭环 / 契约唯一源）完整原文见 references/ai-work-protocol.md 第八章
✅ 文档契约：docs/ 下文档须有 > 版本: … | 更新: … | 状态: … 状态头
✅ 规模阈值：AGENTS.md ≤ 300 行 / 单篇文档 ≤ 600 行，超限按职责域拆分到 docs/
```

## 五、模块速查表

| 文件 | 职责 |
|------|------|
| `SKILL.md` | 核心逻辑索引层（113 行）：触发条件 / 执行原则 / Step 索引与按需加载表 / 输出验收清单 / 参考文件索引 |
| `references/step-1-discovery.md` ~ `step-7-adaptive.md` | 7 个 Step 实现细节（按需加载；Step 5.5 门禁装配并入 step-5；均 ≤ 500 行） |
| `README.md` / `README_CN.md` | 中英文项目文档：安装、能力、工作流程、贡献指南 |
| `CHANGELOG.md` | 版本记录（v1.0 ~ v1.9.0 + [Unreleased]） |
| `PROJECT_STATUS.md` | 项目状态、版本演进、独立抽离指南、已知局限、下一步计划 |
| `docs/` | **公开**文档：`README.md` 索引（A~E 分类 + 公开边界 + 编号预留登记）+ D 级发布说明 |
| `internal-docs/` | **不发布**的内部文档（自审 / 竞品对标 / 评估复核类），受 `.gitignore` 约束 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shuguang1994/project-blueprint](https://github.com/shuguang1994/project-blueprint) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
