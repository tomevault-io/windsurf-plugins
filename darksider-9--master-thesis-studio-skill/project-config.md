---
trigger: always_on
description: name: master-thesis-studio
---

---
name: master-thesis-studio
描述: "用于中文硕士论文项目，尤其是东南大学风格 Word 模板或已有草稿的论文工作流。用户需要初始化论文工作区、确认题目和大纲、生成或续写中文章节 Markdown、管理参考文献/图表/表格/公式/代码/数据资产、清洗占位符，或通过内置 Word/XML 脚本安全生成 DOCX 时使用。"
description: Use this skill for Southeast University-style master's thesis projects from a Word template or existing draft, including optional reverse parsing of an existing DOCX, user intake, outline planning, chapter drafting, academic asset management, Markdown visualization, references, figures, tables, formulas, and safe DOCX generation through the bundled Word/XML scripts.
---

## UTF-8 Reading Guard

Most instructions in this skill are Chinese and all Markdown/JSON files are UTF-8. If Chinese text appears garbled, do not edit or reason from the garbled output. Re-read files with an explicit UTF-8 command:

```bash
python -c "from pathlib import Path; import sys; sys.stdout.reconfigure(encoding='utf-8'); print(Path('SKILL.md').read_text(encoding='utf-8'))"
```

For short targeted reads, prefer Python over PowerShell `Get-Content`:

```bash
python -c "from pathlib import Path; p=Path('references/writing_workflow.md'); print(p.read_text(encoding='utf-8'))"
```

Load only the needed reference file. Do not bulk-read every Markdown file unless the task is a full Skill audit.



# Master Thesis Studio

这是一个面向中文硕士论文写作与 Word 自动生成的 Skill。它把论文写作流程拆成两层：

1. **写作组织层**：问诊、确认题目、大纲、章节计划、资产清单、写作边界和中文学术表达。
2. **Word/XML 执行层**：用内置脚本把 Markdown 写入 Flat OPC XML，再生成新的 DOCX。

除非用户明确要求快速演示，否则必须先确认项目事实和写作边界，再写正文或生成 Word。

## 总原则

- 所有新生成的论文标题、章节标题、正文、图题、表题、摘要、说明文字默认使用中文。英文只保留在模型名、算法名、指标名、数据集名、代码路径、文件名和必要术语中。
- 不编造真实文献、实验结果、样本量、指标或数据来源。没有依据时写“待补充”“待确认”或使用占位符。
- 不覆盖原始 Word 模板。原始模板固定保存在 `01_template/original_template.docx`。
- 新 DOCX 只写入 `10_output/`。XML 中间态只写入 `09_state/`。
- 创建新论文项目时，必须先新建一个独立项目文件夹，并把该文件夹作为论文项目根目录；不要直接在 Skill 根目录、工作区根目录或桌面散落运行初始化。
- 初始化和后续生成命令应在该论文项目文件夹内运行，项目参数使用 `.`；调用脚本时使用 Skill 中 `scripts/` 的绝对路径，或明确从 Skill 根目录传入项目绝对路径。
- 用户提供已有 Word 时，先确认是否需要反解析。反解析是可选入口，不是必经流程；如果用户只想把该 Word 作为格式模板，可以只初始化模板，不抽取正文内容。
- 写回 Word 前必须检查章节 Markdown、占位符、参考文献、图表和状态文件是否一致。
- 修改占位符、XML 写回、参考文献或章节 Markdown 规则前，先阅读对应 `references/` 文件。

## 第一次响应

第一次使用本 Skill 时，不要直接写论文。先用简短问题确认：

- 论文题目是否已确定，是否需要辅助拟题。
- 学科方向、研究对象、核心方法、创新点是否已确定。
- 当前阶段：未开始、已有想法、已有部分章节、已有完整初稿、只需格式化输出。
- 用户希望的模式：快速初稿、边确认边写、续写已有内容、根据代码/数据转写、只做 Word 写回。
- 已有资产：Word、Markdown、PDF、笔记、参考文献、代码、数据、图片、表格、公式、实验记录。
- 如果已有 Word：它是“格式模板”“已有论文内容”还是“两者都是”；是否希望反解析章节、图、表、公式到项目目录。
- 如果执行反解析：反解析后想做什么，例如只盘点资产、继续写作、修订润色、补缺失章节、重新导出 Word，或做 round-trip 校验。
- 是否允许模拟数据、示例图表或示例实验结果。不允许时只能基于用户材料写作。
- 使用哪个模板。默认使用 `examples/Template.docx`。

确认结果必须同步写入：

- `00_project/project_manifest.md`
- `00_project/thesis_master_index.md`
- `00_project/decisions_log.md`
- `09_state/project_state.json`

## 阶段执行表

| 阶段 | 先阅读什么 | 更新或生成什么 | 执行什么脚本 |
| --- | --- | --- | --- |
| 0. 判断项目状态 | 项目根目录、`00_project/`、`09_state/project_state.json`、`03_chapters/` | 判断是新项目、续写项目还是只需导出；如果是新项目，先创建独立项目文件夹并进入该文件夹 | 无 |
| 1. 初始化工作区 | 用户给定模板路径；默认模板 `examples/Template.docx` | 在当前论文项目文件夹内生成 `00_project/`、`01_template/`、`03_chapters/`、`04_figures/`、`05_tables/`、`06_code/`、`07_data/`、`08_refs/`、`09_state/`、`10_output/` | 在项目文件夹内运行：`python <skill_dir>/scripts/init_thesis_workspace.py . --template <docx_path>` |
| 1A. 可选反解析已有 Word | 用户确认该 Word 是已有论文内容且希望抽取内容 | `03_chapters/` 章节草稿、`04_figures/` 真实图片、`05_tables/` 表格 `.md/.csv`、`09_state/reverse_parse_assets.json`、反解析报告 | `python <skill_dir>/scripts/reverse_parse_docx.py <project_dir> --docx <user.docx>` |
| 2. 转换模板 | `01_template/original_template.docx` | `01_template/template.flat.xml` | `python scripts/flat_opc_converter.py toxml <input.docx> <output.xml>` |
| 3. 解析模板结构 | `references/xml_mapping_spec.md`、`01_template/template.flat.xml` | `09_state/parsed_structure.json` | `python scripts/parse_template_xml.py <template.flat.xml> <parsed_structure.json>` |
| 4. 生成项目控制文件 | `references/writing_workflow.md`、`assets/project_state.schema.json` | `00_project/*.md`、初始 `03_chapters/chXX_plan.md`、`03_chapters/chXX_draft.md`、`09_state/project_state.json` | `python scripts/generate_planning_files.py <project_dir>` |
| 5. 启动问诊和资产盘点 | `references/writing_workflow.md`、已有 `00_project/*.md`、各资产 manifest | 更新 project manifest、master index、decisions log、figures/tables/code/data manifest | 通常无脚本，必要时手动编辑 Markdown |
| 6. 章节计划 | `references/writing_workflow.md` 的章节计划规则、`00_project/thesis_master_index.md`、相关资产 | `03_chapters/chXX_plan.md` | 通常无脚本 |
| 7. 章节正文 Markdown | `03_chapters/chXX_plan.md`、`references/writing_workflow.md`、`references/placeholders.md`、相关 `08_refs/`、`06_code/`、`07_data/`、`04_figures/`、`05_tables/` | `03_chapters/chXX_draft.md` 或兼容命名的 `ch*.md` | 通常无脚本 |
| 8. 参考文献处理 | `references/reference_rules.md`、`08_refs/` | 规范化引用占位符和参考文献对象 | `python scripts/reference_tools.py format <refs.json> --style "GB/T 7714"` |
| 9. Markdown 写入 XML | `references/placeholders.md`、`references/xml_mapping_spec.md`、`03_chapters/`、`09_state/project_state.json` | `09_state/current_working.xml`、XML 快照、`09_state/current_content_manifest.json` | `python scripts/apply_markdown_to_xml.py <project_dir> --out 09_state/current_working.xml` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [darksider-9/master-thesis-studio-skill](https://github.com/darksider-9/master-thesis-studio-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
