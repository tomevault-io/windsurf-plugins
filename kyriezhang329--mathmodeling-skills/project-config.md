---
trigger: always_on
description: - Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
---

# Core Philosophy

- Start from goals, objects, constraints, data, outputs, variables, relationships, and checkable conclusions.
- Do not start from model names or favorite techniques.
- Separate assumptions, observations, derivations, and validated conclusions.

# Workflow Discipline

- Parse the problem before classifying it.
- Classify the task before selecting methods.
- Generate a candidate method pool before committing to one method.
- Validate the method plan before writing code or paper text.
- Run multi-round experiments before locking in the final method.
- Keep every change minimal, traceable, and easy to review.

# Workflow Gates

- Do not select methods before the problem is parsed and classified.
- Do not generate code before a candidate method pool exists.
- Do not write numerical paper claims before results exist.
- Do not hand a subquestion to the paper writer before all three critical rules are satisfied.
- Do not assemble the final paper before QA passes.

# Per-Question Delivery Chain (MANDATORY)

For each subquestion Qx, it is FORBIDDEN to treat "code ran successfully" as "the subquestion is done."

Every subquestion must complete the following chain:

```
Stage 1:  建模手提出多个候选方法
          → methods/Qx/qx_method_candidates.md

Stage 2:  编程手实现多候选方法, 运行实验
          → code/Qx/*.py (or *.m)
          → results/Qx/experiments/roundN/

Stage 3:  编程手生成实验报告, 对比各方法
          → results/Qx/experiments/roundN/qx_experiment_report_roundN.md

Stage 4:  建模手根据实验报告修改、淘汰或合并方法
          → methods/Qx/qx_method_iteration_log.md

Stage 5:  多轮迭代后 (重复 Stage 2-4), 建模手确定最终方法
          → State: Final Method Selected (confirmed by modeler)

Stage 6:  AI/建模手写最终方法详解
          → methods/Qx/qx_final_method_explanation.md

Stage 7:  编程手/result-report-generator 生成最终结果分析
          → results/Qx/reports/qx_final_result_analysis.md

Stage 8:  robustness-checker 执行稳健性分析
          → robustness/Qx/qx_robustness_report.md

Stage 9:  solution-package-builder 生成论文手材料包
          → results/Qx/reports/qx_solution_package_for_writer.md

Stage 10: 论文手撰写 Qx 论文段落
          → paper/sections/qx.tex (or .md)

Stage 11: 建模手审核论文模型部分, 编程手审核论文结果和图表

Stage 12: 审核通过后 Qx 状态改为 Finalized
```

If `qx_final_method_explanation.md`, `qx_final_result_analysis.md`, or `qx_solution_package_for_writer.md` is missing, that subquestion MUST NOT enter the paper writing stage.

# Minimum Files Per Subquestion

Each subquestion Qx must have at least these 6 files before being considered "Ready for Writer":

| # | File | Primary Author | Purpose |
|---|------|---------------|---------|
| 1 | `methods/Qx/qx_method_candidates.md` | 建模手 + method-selector | 候选方法池 |
| 2 | `methods/Qx/qx_method_iteration_log.md` | 建模手 + 编程手 | 方法迭代记录 |
| 3 | `methods/Qx/qx_final_method_explanation.md` | 建模手 + final-method-explainer | 最终方法详解 |
| 4 | `results/Qx/reports/qx_experiment_report.md` | 编程手 + result-report-generator | 多方法实验对比 |
| 5 | `results/Qx/reports/qx_final_result_analysis.md` | 编程手 + result-report-generator | 最终结果分析 |
| 6 | `results/Qx/reports/qx_solution_package_for_writer.md` | solution-package-builder | 论文手材料包 |

# Three Critical Rules (ENFORCED AT GATES)

These three rules are enforced by `workflow-orchestrator` and `paper-section-writer`. Violation of any rule means the subquestion is NOT Ready for Writer.

## Rule 1: 没有最终方法详解，不准写最终论文

`paper-section-writer` MUST NOT write final paper sections for Qx based on `qx_method_candidates.md` or early method plans. The file `methods/Qx/qx_final_method_explanation.md` MUST exist first.

## Rule 2: 没有最终结果分析，不准交给论文手

Program execution outputs alone are NOT a deliverable. The programmer (or `result-report-generator`) MUST produce `results/Qx/reports/qx_final_result_analysis.md` before the writer can proceed.

## Rule 3: 论文手只看材料包，不从零散 results 里猜

The paper writer's primary source for Qx is `results/Qx/reports/qx_solution_package_for_writer.md`. If this file is missing, Qx is NOT Ready for Writer, and the writer should not hunt through scattered results.

# Figure Type Classification

All figures must be classified into one of four types:

| Type | Name | Audience | Paper Use |
|------|------|----------|-----------|
| Type 1 | 诊断图 Diagnostic | 建模手 (modeler) | NOT for paper — internal model debugging only |
| Type 2 | 对比图 Comparison | 建模手 + 论文手 | MAY appear in paper — supports method selection narrative |
| Type 3 | 论文图 Paper | 论文读者 (judges) | MUST appear in paper — supports main claims |
| Type 4 | 附录图 Appendix | 论文读者 (supplementary) | Referenced from main text, shown in appendix |

Type 1 diagnostic figures must NOT appear in the final paper. Type 3 paper figures must be publication-quality (≥300 dpi, clear labels, informative captions).

# Experiments Output Structure

Every round of experiments MUST output to:

```
results/Qx/experiments/roundN/
├── figures/          # All generated figures
├── tables/           # All result tables (.csv)
├── metrics/          # All evaluation metrics (.csv, .json)
├── logs/             # Execution logs
└── run_summary.json  # Machine-readable execution record
```

`run_summary.json` must record: question ID, round number, methods executed, their statuses, input/output files, metrics summaries, random seeds, and environment info.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [KyrieZhang329/MathModeling-skills](https://github.com/KyrieZhang329/MathModeling-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
