---
trigger: always_on
description: MedgeClaw is a biomedical AI research assistant. You are the execution layer —
---

# CLAUDE.md - MedgeClaw Project Instructions

## Project Overview

MedgeClaw is a biomedical AI research assistant. You are the execution layer —
users describe analyses in natural language, and you write + run the code.

## 🔄 项目同步（重要）

**MedgeClaw 与 OpenClaw 的集成通过 `.medgeclaw-sync.yml` 配置文件管理。**

### 初始化/更新同步
```bash
cd <MedgeClaw项目目录>
python3 sync.py
openclaw gateway restart
```

### 同步内容
- 项目文档（MEDGECLAW.md, IDENTITY.md）→ OpenClaw workspace
- 自定义 skills → OpenClaw workspace/skills/
- SOUL.md / AGENTS.md 追加 MedgeClaw 身份段落
- openclaw.json 添加 MedgeClaw skills 路径

### 修改同步配置
编辑 `.medgeclaw-sync.yml`，无需改 `sync.py` 或 `medgeclaw-init.sh`。

## 📁 输出路径约束（重要）

**所有任务输出必须写入以下目录，不得写入项目根目录或其他位置：**

| 输出类型 | 路径 | 说明 |
|---------|------|------|
| 数据分析任务 | `data/<task_name>/output/` | CSV、图表、报告 |
| Dashboard | `data/<task_name>/dashboard/` | state.json, dashboard.html, serve.py |
| 科学写作 | `writing_outputs/<date>_<topic>/` | LaTeX、PDF、参考文献 |
| 临时文件 | `data/<task_name>/temp/` | 中间产物 |

**禁止写入：**
- ❌ 项目根目录
- ❌ `outputs/`（已废弃，仅保留兼容）
- ❌ OpenClaw workspace

**`.gitignore` 已配置忽略所有输出目录，确保不会误提交数据。**

## Execution Environment

**You run on the host, but execute code inside the Docker container.**

The analysis environment (Python, R, RStudio, JupyterLab, all scientific packages)
lives in the `medgeclaw` Docker container. Always execute scripts via:

```bash
# Python
docker exec medgeclaw python3 /workspace/path/to/script.py

# R
docker exec medgeclaw Rscript /workspace/path/to/script.R

# Install packages if needed
docker exec medgeclaw pip install <package>
docker exec medgeclaw Rscript -e 'install.packages("<package>", repos="https://cran.r-project.org")'
```

If `docker exec` fails with a permission error, use `sg docker -c "docker exec ..."` instead:

```bash
sg docker -c "docker exec medgeclaw python3 /workspace/path/to/script.py"
```

**Path mapping:** The host path `./data/` maps to `/workspace/data/` inside the container.
Write scripts using `/workspace/` paths. The host path `./outputs/` maps to `/workspace/outputs/`.

**Never run analysis scripts directly on the host** — it may lack R, specific Python
packages, or have different versions. The container is the canonical environment.

## Directory Conventions

- `data/` — input data files (read-only, user-provided)
- `data/<task_name>/output/` — per-task analysis results (CSV, images, reports)
- `data/<task_name>/dashboard/` — per-task dashboard (state.json, dashboard.html, serve.py)
- `outputs/` — legacy shared output directory (prefer per-task output/ for new tasks)
- `visualization/` — HTML dashboards and interactive visualizations
- `skills/` — OpenClaw skill definitions (dashboard, biomed-dispatch, cjk-viz)
- `scientific-skills/` — K-Dense 140 scientific skills (git submodule, read-only)

Always `mkdir -p` output and dashboard directories before writing files there.

## Code Style

- Python preferred for data analysis. Use pandas, scipy, matplotlib, seaborn.
- R available for bioinformatics packages (DESeq2, Seurat, edgeR, etc.)
- Chinese labels in all visualizations (this is a Chinese user base)
- matplotlib: **绘图前必须执行 CJK 字体检测**，参考 `skills/cjk-viz/SKILL.md`。
  不要硬编码字体名，使用 `skills/cjk-viz/scripts/setup_cjk_font.py` 自动检测。
  backend 使用 `Agg`。
- HTML dashboards: self-contained (base64-embed all images), professional CSS

## Scientific Skills 参考（K-Dense）

**无论是否使用 Claude Code，遇到生物医药或科研场景任务时，必须主动查阅相关的
K-Dense Scientific Skills 作为参考。**

Skills 位于 `scientific-skills/scientific-skills/` 目录，每个子目录包含一个
`SKILL.md`，描述了该工具/领域的最佳实践、代码模板和注意事项。

### 使用流程

1. **识别任务涉及的工具或领域**（如 RNA-seq → `deseq2`、单细胞 → `scanpy`、
   分子对接 → `diffdock`、文献检索 → `biorxiv-database`）
2. **读取对应的 SKILL.md**：`scientific-skills/scientific-skills/<skill-name>/SKILL.md`
3. **参考其中的代码模板、参数建议、注意事项**来编写代码
4. 如果涉及可视化，同时参考 `skills/cjk-viz/SKILL.md` 确保中文正常显示

### 常见任务 → Skill 映射

| 任务类型 | 推荐 Skill |
|----------|-----------|
| 差异表达分析 | `deseq2`, `edger` |
| 单细胞分析 | `scanpy`, `anndata` |
| 通路富集 | `gseapy`, `enrichr` |
| 蛋白结构预测 | `esm`, `alphafold-database` |
| 分子对接 | `diffdock`, `deepchem` |
| 药物数据库查询 | `drugbank-database`, `chembl-database` |
| 临床试验检索 | `clinicaltrials-database` |
| 变异注释 | `clinvar-database`, `cosmic-database` |
| 文献检索 | `biorxiv-database`, `citation-management` |
| 生存分析 | `lifelines`（在 `exploratory-data-analysis` 中） |
| 科研绘图 | `scientific-visualization`, `matplotlib`, `plotly` |
| 数据探索 | `exploratory-data-analysis` |
| SVG 信息面板 | `svg-ui-templates`（列表、清单、流程图、报告） |
| 飞书图文汇报 | `feishu-rich-card`（图片上传 + Card Kit 交互卡片） |

如果不确定用哪个 skill，可以 `ls scientific-skills/scientific-skills/` 浏览完整列表。

## Scientific Writer 参考（K-Dense）

**遇到学术写作、文献综述、基金申请、临床报告等写作任务时，必须查阅 Scientific Writer Skills。**

Skills 位于 `scientific-writer/skills/` 目录，每个子目录包含一个 `SKILL.md`。
项目级指令见 `scientific-writer/CLAUDE.md`。

### 常见写作任务 → Skill 映射

| 任务类型 | 推荐 Skill |
|----------|-----------|
| 科研论文（IMRaD） | `scientific-writing` |
| 文献综述 | `literature-review` |
| 基金申请书 | `research-grants` |
| 临床报告 | `clinical-reports` |
| 临床决策支持 | `clinical-decision-support` |
| 治疗方案 | `treatment-plans` |
| 市场研究报告 | `market-research-reports` |
| 引用管理/BibTeX | `citation-management` |
| 实时文献检索 | `research-lookup` |
| 科学示意图 | `scientific-schematics` |
| AI 图像生成 | `generate-image` |
| 学术幻灯片 | `scientific-slides` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xjtulyc/MedgeClaw](https://github.com/xjtulyc/MedgeClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
