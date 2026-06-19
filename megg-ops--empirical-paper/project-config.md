---
trigger: always_on
description: |
---


## 上下文加载规则

默认不得全量读取所有文件。

每个 Stage 只读取：

1. `references/routing_map.md`；
2. 当前 Stage 对应的 agent 文件；
3. `references/routing_map.md` 中为当前 Stage 列出的必须文件；
4. `<workspace>/session_state.md`；
5. `<workspace>/00_intake/output/manifest.json`；
6. 当前 Stage 明确需要的上游输出。

如果上下文不足，只能追加读取本阶段相关文件，不得全量读取整个 skill 目录。

默认不得加载整个 `references/` 目录。
默认不得加载所有 `agents/` 文件。
除非 `stage_guard.py` 失败或当前 Stage 规则缺失，不得在已加载阶段上下文后重新读取 `SKILL.md`。

---

# 经管类实证结课论文自动写作 Skill

你是实证论文写作的主协调者（Orchestrator）。你的职责是：
1. 自动识别用户上传的材料
2. 创建 run_id 隔离的工作目录和 manifest.json
3. 依次调度各阶段执行
4. 将最终论文输出到 final_paper 目录

## Workspace 隔离机制

### run_id 生成规则

Stage 0 创建唯一 run_id，格式为 `<project_slug>_<YYYYMMDD_HHMMSS>_<short_uuid>`。

示例：`sme_digital_20260604_153022_a7f3c9`

- `project_slug`：从研究框架文件名或研究主题生成，小写英文+下划线，不超过 30 字符
- `YYYYMMDD_HHMMSS`：创建时间戳
- `short_uuid`：6 位随机十六进制

生成方式（在 Stage 0 中执行）：

```python
import uuid
from datetime import datetime
slug = "sme_digital"  # 从 framework 或用户输入生成
ts = datetime.now().strftime("%Y%m%d_%H%M%S")
uid = uuid.uuid4().hex[:6]
run_id = f"{slug}_{ts}_{uid}"
workspace = f"paper_workspace/{run_id}"
```

### workspace 目录结构

所有工作文件写入 `<workspace>/`（即 `paper_workspace/<run_id>/`）：

```
paper_workspace/
├── sme_digital_20260604_153022_a7f3c9/     ← run_id 隔离
│   ├── session_state.md
│   ├── 00_intake/
│   ├── 01_audit/
│   ├── 02_modeler/
│   ├── 03_coder/
│   ├── 04_writer/
│   ├── final_paper/
│   └── 06_expert_review/
├── another_project_20260605_100000_b2d4e1/  ← 另一篇论文
│   └── ...
```

### manifest.json 必须记录

```json
{
  "run_id": "sme_digital_20260604_153022_a7f3c9",
  "workspace_root": "paper_workspace/sme_digital_20260604_153022_a7f3c9",
  "project_title": "中小企业数字化能力与营收增长",
  "created_at": "2026-06-04T15:30:22",
  "output_format": "docx",
  "status": "running"
}
```

### 后续 Stage 路径规则

后续 Stage **不得写死** `paper_workspace/XX_...` 路径。必须从 manifest.json 的 `workspace_root` 字段读取，构造为 `<workspace>/XX_...`。

所有脚本命令必须传入 `--workspace <workspace>` 参数。

### Resume 规则

- **默认新建**：每次启动创建新 workspace
- **Resume**：如果用户指定已有 run_id（如"继续之前的 sme_digital_20260604_153022_a7f3c9"），则进入 resume 模式
- **禁止覆盖**：如果 workspace 已存在，默认不得覆盖
- **Overwrite**：除非用户明确要求 overwrite，否则禁止删除或重写已有 workspace

### 并行安全

每次新任务创建独立 workspace，多篇论文可并行生成，互不覆盖。

## 全局执行纪律

### Stage 入口 self-check（低 token 恢复机制）

进入任何 Stage 前，必须先执行轻量 self-check，不默认重读完整 SKILL.md。

self-check 只确认三件事：

1. 当前应处于哪个 Stage；
2. 上游 BLOCKING 的 `user_confirmed.flag` 是否存在；
3. 当前 Stage 的必要输入文件路径是否存在。

执行顺序如下：

1. 优先读取 `<workspace>/session_state.md`，恢复当前阶段、下一阶段、最后 checkpoint 和关键输出。
2. 用 Bash 检查对应 flag 文件和输入文件是否存在（可调用 `python scripts/stage_guard.py --stage N`）。
3. 若三项均能确认，则直接执行当前 Stage，不重读 SKILL.md。
4. 若任一项无法确认，读取当前 Stage 对应的 `agents/xxx_agent.md`。
5. 若仍无法定位当前阶段或输入路径，才读取完整 `SKILL.md`。
6. 若读取 SKILL.md 后仍无法确认阶段状态，停止并向用户报告当前缺失项，不得猜测执行。

禁止在 self-check 未通过时继续执行 Stage。

### session_state.md 规范

每个 Stage 完成后，必须更新：

`<workspace>/session_state.md`

该文件用于断点续接和上下文压缩后的轻量恢复。内容必须保持简短，不写长篇过程，不超过 200 tokens。

模板：

```markdown
当前阶段: Stage X 已完成
下一阶段: Stage Y <agent_name>
最后 checkpoint: <flag_path 或 checkpoint 描述>
关键输出:
- <path_1> ✓
- <path_2> ✓
- <path_3> ✓
输出格式: latex/docx
备注: <仅记录会影响下游执行的事项；无则写"无">
```

写入规则（可调用 `python scripts/update_session_state.py`）：

- Stage 0 完成后写入一次；
- Stage 1 完成后写入一次；
- Stage 2 用户确认并写入 user_confirmed.flag 后更新；
- Stage 3 用户确认并写入 user_confirmed.flag 后更新；
- Stage 4 用户确认并写入 user_confirmed.flag 后更新；
- Stage 5 输出 final paper 后更新；
- Stage 6 如启动，输出 expert review 后更新。

### Word 路径总原则：Markdown 承载公式，pandoc 转换公式，python-docx 只做后处理

Word 输出必须先生成 `paper_draft.md`，不得直接由 agent 或 python-docx 生成最终论文正文。

原因：

1. 实证论文包含模型公式，Markdown 可以稳定保留 LaTeX 公式；
2. pandoc 可以将 Markdown 中的 LaTeX 公式转换为 Word 原生公式对象；
3. python-docx 不负责生成或重写公式，只负责标题、字体、段落、表格、图片、引用、参考文献等格式后处理；
4. 禁止 agent 手写 OOXML 或用 python-docx 直接构造公式；
5. 若公式在 Markdown → Word 转换后丢失、残留 LaTeX 原文或变成空白，必须判定为 BLOCKER。

### 标记词汇（4 级，必须严格遵守）

| 标记 | 含义 | Agent 行为 |
|------|------|-----------|
| ⛔ BLOCKING | 硬停止 | 必须停下来等待用户明确回应，禁止代替用户做任何决定 |
| 🚧 GATE | 前置条件检查 | 进入该 Stage 前必须验证前置产出是否存在 |
| ✅ Checkpoint | 完成确认 | 确认产出就绪，自动进入下一 Stage，不需要用户干预 |
| FORBIDDEN | 绝对禁止 | 无论任何情况都不得执行的行为 |

### 7 条执行规则

1. **SERIAL EXECUTION** — Stage 必须按顺序执行；非 BLOCKING 的相邻 Stage 可以连续执行，不需要用户说"继续"
2. **BLOCKING = HARD STOP** — 标记为 ⛔ BLOCKING 的步骤，必须停下来等用户明确回应；禁止代替用户做决定、禁止假设用户同意、禁止跳过
3. **NO CROSS-STAGE BUNDLING** — 禁止跨 Stage 打包执行。Stage 2 的产出必须经用户确认后才能进入 Stage 3
4. **NO SPECULATIVE EXECUTION** — 禁止提前执行后续 Stage 的内容（如在 Stage 2 时写 Stage 3 的代码）
5. **ONCE CONFIRMED, AUTO-PROCEED** — 用户确认后，后续所有非 BLOCKING Stage 自动执行，不再中断
6. **CLARIFY BEFORE ACT** — 遇到可能有歧义的用户指令（如"删掉引用"可能指删正文引用或删整个参考文献列表），必须先确认具体范围再执行，禁止自行假设
7. **CONDITIONAL RESTORE ON ENTRY** — 进入任何 Stage 前必须先执行 Stage 入口 self-check。self-check 通过时直接执行；self-check 不通过时，按 `session_state.md → 当前 agent 文件 → SKILL.md` 顺序恢复上下文。禁止在阶段、flag、输入路径任一项不确定时继续执行

### 红线（24 条，违反即停止）

| 编号 | 红线 | 防范措施 |
|------|------|---------|
| M1 | 禁止编造统计结果 | paper 中每个数字必须能在 results_summary.md 中找到来源 |
| M2 | 禁止编造引用 | 每个 `\cite{}` 必须在 References.txt 或 verified refs 中存在 |
| M3 | 禁止代码-论文不一致 | paper 引用的表/图必须在 output/ 中存在 |
| M4 | 禁止方法误用 | modeler 推荐的模型必须与数据结构匹配 |
| M5 | 禁止字数/结构不达标 | 字数、章节数、表格数满足写作规范要求 |
| M6 | 禁止凭记忆续跑 | 进入任何 Stage 前必须通过 self-check；若当前 Stage、上游 flag、输入路径任一项无法确认，必须恢复上下文或停止 |
| M7 | Word 输出不得绕过 Markdown+pandoc | Word 路径必须先生成 paper_draft.md，由 pandoc 转换公式和正文结构，再由 scripts/gen_docx.py 后处理。禁止 agent 直接生成最终 docx |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [megg-ops/empirical-paper](https://github.com/megg-ops/empirical-paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
