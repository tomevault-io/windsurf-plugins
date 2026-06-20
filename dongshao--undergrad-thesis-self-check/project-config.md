---
trigger: always_on
description: 给南京大学软件工程专业本科毕业论文送审前的自检 skill。学生提供论文 PDF，先从 PDF 抽题目并请学生确认，自动判定"工程型 / 学术型"并请学生确认，再询问"是否团队子模块"（如是追问总项目+本人模块），按对应类别 32-34 条 checklist（含 3-4 条红线 + 6 条参考文献规范）逐项审查，给出带原文页码定位的反馈、红线高亮；并对软件工程图表规范性（用例图 / 架构图 / ER 图 / 类图 / 流程图等）做 UML 规范检查；最后可选追加 AI 文风+错别字检查。报告写到 <论文同目录>/<论文题目>-review.md（题目来自 PDF 内容而非文件名）。**不做**优/良/中/及格/不及格的总分估计。仅在用户明确说"本科论文自检""本科论文自查""本科送审前自查""我的本科论文有什么问题"等触发词且提供 PDF 路径时启用。
---


# NJU SE 本科论文自检 skill

给本科生送审前自查用。本 skill 是学生视角的 checklist 逐项审查。

## 何时使用

仅在用户明确说出以下触发词之一**且**提供论文 PDF 路径时启用：
- 「本科论文自检」「本科论文自查」「本科送审前自查」
- 「我的本科论文有什么问题」「我的本科论文能过吗」
- 「undergrad self-check」「pre-defense check」（且提供 PDF）

不要只看到 PDF 就触发。

## 调用格式

```
本科论文自检 /path/to/paper.pdf
本科论文自检 /path/to/paper.pdf --type=engineering   # 跳过类型判定
本科论文自检 /path/to/paper.pdf --type=academic
本科论文自检 /path/to/paper.pdf --type=engineering --team   # 标记团队论文（仍会追问总项目+模块）
本科论文自检 /path/to/paper.pdf --auto                # 全自动模式（任何 Gate 不再询问）
```

`--type` 可选值：`engineering`（工程型）/ `academic`（学术型）。
`--team` 标记团队子模块；不带则进入团队 Gate 询问。
`--auto` 触发**全自动模式**：用户消息中含「全自动」「自动模式」「auto」「--auto」「-y」「yes to all」任一关键词即视为开启。

## 全自动模式（AUTO_MODE）

开启后，整条流程从步骤 1 跑到步骤 11 一气呵成，**任何 Gate 都不再向用户索要确认**——本 skill 的 Claude 用自己的判断直接通过。具体差异如下：

| 环节 | 缺省（人工模式） | 全自动模式 |
|---|---|---|
| 步骤 2 抽题确认 | 把抽到的题目贴出来，等 y / 纠正 | 直接用抽到的 `title_clean` 作为最终题目，不询问；如四种策略全部失败回退到文件名，写入报告"附注：题目抽取退化"段 |
| 步骤 1 骨架异常 | 列出问题，问是否继续 | 直接继续；**不在报告里加诊断附注**（最终结论按真实结构给出，读者无需感知中间诊断） |
| 步骤 3 类型判定 | 给判定+证据，等 y / 纠正 | 用判定结果直接进入步骤 4；**不在报告里加"类别判定依据"段**（判定结果通过整篇报告所用 checklist 类型隐含体现）；仅当信号严重对半、按"工程型默认"处理时，在报告头部加一行「类别存疑，建议人工复核」 |
| 步骤 4 团队判定 | 询问是否团队 + 追问总项目/模块 | 默认按 solo 处理（不读 overrides）；如骨架元信息或题目里出现"团队 / SELABS / 模块"等强信号，按 team=yes 处理但 `team_project` / `my_module` 设为"未提供（AUTO_MODE 跳过追问）"，并在报告头部写明 |
| 步骤 8 图表能力探针失败 | 提示用户并跳过本步 | 自动跳过；在报告第七节写明"当前模型不支持图像输入，跳过 UML 图形评估，仅做基于图注的文本检查" |
| 步骤 11 是否做文风检查 | 询问用户 y/n | 默认进行；如未安装 humanizer-zh 则跳过文风部分但仍执行错别字检查，并在第十一节顶部写明"未安装 humanizer-zh，本节仅含错别字检查；如需补充 AI 文风扫描请安装后重跑" |

全自动模式下需保留的硬停点（**不能跳过**）：
- 论文 PDF 路径不存在 / 不是 PDF → 报错并停止
- `pdfplumber` 未安装 → 报错并提示 `pip install pdfplumber`，停止
- 步骤 1 骨架抽取脚本崩溃（非"骨架不完整"，是真的崩） → 报错并停止
- 步骤 8 PyMuPDF 未安装 → 不停止，跳过步骤 8 并在报告第七节写明"PyMuPDF 未安装，请运行 pip install pymupdf 后重跑此步骤"
- 步骤 11 切块脚本崩溃 → 报错并停止；脚本本身正常但 `split_failed=true` → 不停止，进入"骨架降级模式"（文风仅扫骨架可见的段落），并在副报告头部加警示

全自动模式下需在**对话中实时输出**关键节点，让用户看到进度（一句话一节点即可）：

```
[1/11]  抽骨架完成（{pages} 页 / {chapters} 章）
[1/11]  题目：《...》
[2/11]  题目确认（AUTO_MODE 默认通过）
[3/11]  类别：工程型（依据：第三章"系统架构"+ 第五章"系统测试"）
[4/11]  团队：solo / team={team_project} - {my_module}
[5/11]  已加载 engineering-30.md（34 条）+ overrides（仅 team 模式）
[6/11]  红线扫描：命中 X 条
[7/11]  全量审查完成：✅ A / ⚠️ B / ❌ C
[8/11]  图表规范性：识别 N 张图 / 命中 M 类图 / 警告 K 条
[9/11]  报告写入 {report_path}
[10/11] 摘要已输出到对话
[11/11] 文风+错别字检查：N 块 chunks / 错别字 X 条 / 文风 Y 条
完成。
```

人工模式下保持原行为不变。

## 前置依赖

- `pdfplumber`：如缺失提示用户 `pip install pdfplumber`
- `PyMuPDF`（包名 `pymupdf`，import 名 `fitz`）：步骤 8 渲染整页为 PNG。如缺失提示 `pip install pymupdf`；缺失只导致步骤 8 跳过，不影响其他步骤
- `humanizer-zh` skill（github.com/op7418/Humanizer-zh）：步骤 11 AI 文风检查。如未安装，AUTO_MODE 下跳过文风部分仅做错别字检查；人工模式下询问用户后决定
- 本 skill 自带 `scripts/extract_skeleton.py`（步骤 1 抽骨架）与 `scripts/extract_chunks.py`（步骤 11 切块），无需依赖外部 skill

## 输出位置约定

- **主报告** → `<论文 PDF 所在目录>/<论文题目>-review.md`（学生第一眼能看到）；如已存在则顺延为 `<论文题目>-review-1.md` / `-review-2.md` / ...，**不覆盖前次报告**，便于学生对照修改进度。具体路径由步骤 9 生成的 `$REPORT_PATH` 决定
- **文风副报告** → `<论文题目>-style.md`（与主报告共用同一下标顺延，如 `-review-2.md` 配 `-style-2.md`）；仅当步骤 11 实际执行时生成
- **中间产物** → `<论文 PDF 所在目录>/<TMP_DIR>/`，`<TMP_DIR>` 由步骤 1 顺延决定：默认 `tmp`，若已存在则改用 `tmp1` / `tmp2` / ... 直到找到未占用的目录名。**不覆盖前次产物**，学生可对比多次跑的差异。下文为可读性以 `tmp/` 占位指代当前实际目录。
  - `<TMP_DIR>/skeleton.md` — 步骤 1 抽出的骨架
  - `<TMP_DIR>/figs/page-NNN.png` — 步骤 8 渲染的关键图所在页
  - `<TMP_DIR>/chunks/<range>.txt` + `chunks-index.json` — 仅步骤 11 触发时生成（lazy 切块）

不要再往 `/tmp/` 系统临时目录写——重启会丢，也不便于学生回看；也不要把中间产物直接散落在论文目录下。`<paper_dir>/<TMP_DIR>/` 是唯一中转区。

## 步骤执行原则（**强制**）

1. **不得为节省 token / 时间跳过任何步骤或子步骤**——只要前置依赖满足且未触发 SKILL.md 明文允许的退化分支，本 skill 就必须把每个步骤完整跑完。学生付费跑一次自检的预期是「完整审查」，跳过等于交付残缺产品。
2. **仅以下情形允许跳过**：
   - PDF 文件不存在 / 不是 PDF → 整体停止
   - 必需依赖缺失（pdfplumber / PyMuPDF）→ 按 SKILL.md 各步骤明文规定的退化路径处理（如步骤 8 缺 PyMuPDF 时跳过本步骤；不是绕过，必须在最终报告里写明跳过原因 + 安装命令）
   - 模型图像能力探针失败（VISION_CAPABLE=false）→ 步骤 8 进退化路径，仍要做基于图注的文本检查
   - 步骤 11 学生明确选「n 跳过」（人工模式下询问后），或 humanizer-zh 未安装（AUTO_MODE 下降级为仅错别字）
3. **不得编造"未执行"措辞**：报告里如果写了某步骤"未执行"或"跳过"，必须给出具体原因（依赖缺失 / 探针失败 / 用户拒绝），不能是"为节省时间"。
4. **token 不是借口**：本 skill 设计上预期一次完整跑可能消耗较多 token——这是学生选择跑自检时已知的成本。跳步骤换"看起来跑完了"是对学生的欺骗。

## 主流程（11 步）

| 步骤 | 内容 |
|---|---|
| 1 | 抽骨架 + 抽取论文题目（含 tmp 目录顺延） |
| 2 | 题目确认 Gate（AUTO_MODE 跳过） |
| 3 | 类型判定 Gate（工程型 / 学术型；AUTO_MODE 跳过） |
| 4 | 团队子模块 Gate（AUTO_MODE 跳过） |
| 5 | 加载对应 reference + 应用 overrides |
| 6 | 红线扫描（工程 4 条 / 学术 3 条） |
| 7 | 全量审查（剩余 ~30 条） |
| 8 | **软件工程图表规范性检查（新增）** |
| 9 | 生成主报告 |
| 10 | 对话输出摘要 |
| 11 | **（可选）文风与错别字检查（新增）** |

### 步骤 1：抽骨架 + 抽取论文题目

```bash
PAPER_PDF="/path/to/paper.pdf"
PAPER_DIR=$(dirname "$PAPER_PDF")

# tmp 目录顺延：默认 tmp；已存在则 tmp1 / tmp2 / ... 找首个未占用的
# 不要复用或覆盖前次 tmp 目录——学生可能还要回看上一次的 chunks / figs / skeleton
TMP_DIR="$PAPER_DIR/tmp"
i=1
while [ -e "$TMP_DIR" ]; do
  TMP_DIR="$PAPER_DIR/tmp$i"
  i=$((i+1))
done
mkdir -p "$TMP_DIR"
echo "TMP_DIR=$TMP_DIR"   # 后续步骤的 Python 代码块从这里读取实际目录

python3 ~/.claude/skills/undergrad-thesis-self-check/scripts/extract_skeleton.py \
  "$PAPER_PDF" \
  --out "$TMP_DIR/skeleton.md"
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dongshao/undergrad-thesis-self-check](https://github.com/dongshao/undergrad-thesis-self-check) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
