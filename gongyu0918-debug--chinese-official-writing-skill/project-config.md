---
trigger: always_on
description: 本文件适用于整个仓库。后续 agent 接手本仓库时，优先遵守这里的发布、review 和测试约定；若与用户最新指令冲突，以用户最新指令为准，但不得伪造未运行的测试结果。
---

# AGENTS.md

本文件适用于整个仓库。后续 agent 接手本仓库时，优先遵守这里的发布、review 和测试约定；若与用户最新指令冲突，以用户最新指令为准，但不得伪造未运行的测试结果。

当前接手入口只保留本文件。当前 GitHub / ClawHub 发布状态为 `chinese-official-writing@1.5.1`；SkillHub 目标项目 `https://skillhub.cn/skills/chinese-official-writing` 已提交 `1.5.1`，`tags.latest=1.5.1` 且 `iconUrl` 已切到蓝底 Q 版图标，但公开 `latestVersion.version` 仍显示 `1.5.0`，需等待 SkillHub 审核/安全扫描/内容审核切换。交接时必须重新核验公开版本、审核状态和 SkillHub `iconUrl`。下方 1.4.1 到 1.5.0 内容均为历史接手记录，不代表当前 live 版本。

## 基本工作纪律

1. 所有代码或文档修改必须通过 git commit 留痕。commit message 需说明修改目的、影响范围和验证方式。
2. 每次认为任务可交付前，必须运行 smoke test 或最小验证流程；无法运行时说明原因和人工验证步骤。
3. 每累计 5 次 commit，或修改范围明显扩大时，暂停开发，执行轻量 review、基线对比、轻量消融测试和回归检查。
4. 代码 review、大范围 diff、长文件阅读、第三方实现对比等可能污染主上下文的任务，优先交给 subagent 或独立上下文；subagent 结论必须经源码、diff、测试或日志交叉验证后再采纳。
5. 不允许伪造测试结果；未实际运行的命令不得写成已通过。

## 中文公文 Skill 发布验证

`tools/run_real_prompt_ablation.py` 是发布级确定性消融工具，但它不调用 LLM，只能证明 skill 包、reference、lint 和评估入口具备相应支撑。它不能替代真实写稿实测。

社区技能借鉴必须先设门禁。SkillHub、ClawHub、GitHub 或其他社区实现只作为思路、流程形态、检查维度和 prompt/markdown 写法参考，禁止直接誊抄代码、脚本、正则、模板库、大段 prompt、固定话术或模板正文。每个候选借鉴点都要满足本技能边界：不新增重排版引擎，不扩大默认联网，不默认强制确认，不破坏用户模板和字段式材料，不把社区技能的坏代码、坏 prompt、硬禁词或公众号式风格污染到本技能；落地前先归纳共性问题，落地后必须和上一基线做消融。

涉及 `chinese-official-writing` 的版本 review、发布或回归判断时，必须同时做两类验证：

1. **基线消融**：用 detached worktree 固定上一发行基线，只和上一基线比，不和 no-skill 混比。示例：
   - `git worktree add --detach output\release-baselines\github-1.4.1 74dc5be8e1c9dfa30b0ef3f484c3eb5edc8a7fed`
   - `python .\tools\run_real_prompt_ablation.py --baseline-root output\release-baselines\github-1.4.1 --baseline-label baseline-1.4.1 --current-root . --out output\real-prompt-vs-1.4.1-release-1.4.3`
   - 判断口径：current 失败才是发布阻断；baseline 只允许在新补充用例上失败。
2. **真实写稿实测**：用真实用户式 prompt 让写作 agent 产出短稿或改稿，再让独立 verifier 只看“原 prompt + 成稿”判断，不由主上下文自行判定。稿件可以不长，但必须检验指令遵循、要点置入、文种格式、禁止事项、事实边界和是否把格式改坏。

真实写稿实测至少覆盖：

- 本轮新增或修改的边界，例如网页复制稿、套嵌文件、字段式申请材料、长文压缩、联网搜索边界。
- 旧能力回归，例如请示/报告不混写、只审不改、去口语化保事实、用户明确模板优先。
- 创建文章和修改文章两类场景。

推荐流程：

1. 主上下文读取 canonical `chinese-official-writing/SKILL.md` 和本轮相关 reference，明确需要覆盖的风险面。
2. 派 3-5 个 writer subagents，分别按 unseen 的真实用户 prompt 写短稿或改稿。不要让它们修改仓库。
3. 派独立 verifier subagent，只给 prompt 和输出，要求逐项判定 PASS/WARN/FAIL：是否遵循用户指令，关键要点是否进入正文，标题/主送/落款/字段/附件关系是否保留，是否编造事实，是否残留 Markdown 或过程说明。
4. 把 verifier 的结论作为真实写作实测结论；确定性消融和 smoke test 只能作为工程回归证据。

### ANTI-AI 对抗评测工作流

只有通过 AI 痕迹 judge / reference 复核的样稿，才允许在发布报告或 README 中称为 `ANTI-AI` 写作能力。这里的门槛只用于评测和发布命名，不进入默认起草流程，不得把检测器写成正文生成阶段的硬阻断。

对抗评测必须先检索 ClawHub、SkillHub、GitHub 或相关社区里的 AI 味检测、AIGC 痕迹判断、查重/重复度判断类 skill 或 reference。优先借鉴其判断维度和报告结构，例如句群同质化、连接词链、重复开头、模板化转折、二元包装句、空泛强评价、同义词循环、口号式结尾、查重式重复片段等；禁止复制其代码、API 调用、规避检测流程、口语化注入策略、固定替换表或大段 prompt。

ANTI-AI 真实写稿测试应采用对抗式 A/B：

1. 红队 writer subagent 只加载当前 `chinese-official-writing` skill 写稿或改稿，至少覆盖起草、改稿、只审不改、口语来源正式化、长文或限字场景；红队不得读取蓝队判分标准以外的结论，也不得修改仓库。
2. 蓝队 judge subagent 只加载 AI 痕迹检测、AIGC 判断、查重/重复度 reference 的判断维度，不加载改写器，不调用 API，不给代改稿。蓝队只看“原 prompt + 红队样稿”，按 AI 痕迹、查重式重复、模板化程度、事实边界、文种格式和用户指令遵循给出 PASS/WARN/FAIL。
3. 首轮至少跑 5 个真实用户式 prompt。先只记录命中问题，不在同一轮里修复；只有出现三次以上共性问题，才进入下一轮最小修复和基线消融。
4. 若 AI 味下降但事实边界变差，不能直接判定通过；必须记录为“降 AI 味与事实保真存在张力”，等三次以上共性问题再做最小修复。
5. 若样稿出现明显模板味、查重式重复片段、空泛强评价或机械三段式，且 judge 判为 FAIL，本轮不得对外宣称 `ANTI-AI` 能力通过。
6. 通过口径以独立 judge 结论为准；主上下文不得自行把普通 `lint`、确定性消融或单次自评包装成 AI 检测通过。

## 1.4.1 -> 1.4.3 接手记录

当前 1.4.3 主线 commit 路径：

- `74dc5be8e1c9dfa30b0ef3f484c3eb5edc8a7fed`：1.4.1 基线，补充联网搜索边界，默认不外搜，不因单位名自动搜索单位风格。
- `80b8bab`：发布 1.4.2 最小护栏修复。
- `cf0fba4`：补网页复制稿和套嵌文件边界，区分通知壳、被印发文件正文、附件标题和网页元信息。
- `7e6f1cd`：补字段式申请、证明、采购明细等材料的字段和单元边界，避免把真实内部格式硬改成散文、表格或编号清单。
- `652afa8`：补长文压缩事实锚定，压缩时保留主体、对象、数字、期限、责任、附件、联系人和反馈渠道，避免多主体分工被压成笼统“有关单位”。
- `d89f320`：同步 1.4.3 版本号和镜像入口。
- `77b56726efab8e2dcffbc8c1f3ae999249c28707`：修复复函 eval stub，使确定性 smoke 不因复函缺少来函收悉/函复要素误报。

本轮修改思路：

- 坚持 prompt/reference 层最小增强，不新增大规模 lint 规则、不新增排版脚本、不扩大联网搜索默认触发。
- 不做一例一修；把多次出现的真实风险归纳成通用边界：网页稿边界、字段单元边界、长文压缩事实锚定。
- 用户明确模板和局部格式要求优先于默认公文格式；只有破坏文种功能或事实边界时才提示风险。
- `run_real_prompt_ablation.py` 新增用例只作为确定性回归网，不宣称真实写作质量；真实质量要靠 subagent 写稿和独立 verifier 复核。

## 1.4.4 接手记录

1.4.4 的修改应从 1.4.3 发行基线 `77b56726efab8e2dcffbc8c1f3ae999249c28707` 出发做最小增强。前置证据包括：

- `tests/evidence/skillhub-1.4.4-research.md`：SkillHub “公文”81 个结果和 8 个代表技能核验；只允许借鉴流程思路、检查维度和 prompt/markdown 组织方式，不复制社区实现。
- `tests/evidence/real-writing-1.4.4-round1.md`：3 个 writer subagent、8 个真实 prompt、独立 verifier 评分；三次以上共性问题为缺项进正文、字段/事实边界变化、限字不稳、抽象套话、要点位置不准。
- `tests/evidence/real-writing-1.4.4-round2.md`：3 个 writer subagent、3 个真实 prompt、独立 verifier 复核；当前 1.4.4 规则未出现三次以上共性失败，未观察到缺项规则变成阻断，后续只需关注 Markdown 残留、正式落款日期完整度和字数精确度。

1.4.4 候选修复只应覆盖这些共性层面：

- 缺政策依据、截止时间、联系人、反馈渠道等事实时，列正文外待确认，不写“指定渠道”“截止时间前”等泛占位。
- 字段式申请、证明、采购明细和清单式材料只改用户指定字段；新增字段无值时留空或待确认，不推断票据、邮箱、日期等内容。
- 限字压缩必须先锁定不可丢要素，再真实压到用户限制；无法兼顾时说明取舍风险。
- 长篇限字稿件不能只追求低于字数上限；要检查篇幅预算、首中尾完整度、措施安排和结尾落点，避免模型为了限字头重脚轻或草草收尾。
- 抽象词和强评价必须有对象、动作、责任、时限、数据或制度载体。
- Word/docx 交付只做内容定稿后的要素核对和版式衔接，不补造正文事实。

## 1.4.5 接手记录

1.4.5 只收紧 1.4.4 中“事实充分性”的表达方式，不新增硬 lint、不新增联网、不新增排版脚本。

- 事实不足不作为默认中断理由。普通起草、顺稿和改稿必须先完成可用正文，再在正文后用“待确认事项”或当前版本的软性补足标题提示关键缺口。
- 缺口提示是软性写作建议，不做调查问卷式问题清单；只列影响文种功能或执行落地的关键事实，例如金额、日期、联系人、反馈渠道、政策依据、申请事项、审批对象。
- 增删论点、扩写细节、调整表达等写稿人可安排事项，不应列成用户义务。
- 事实不足时不能用强判断补空白。材料未给风险结论、整改结论、检查结论或影响范围时，不自行写“未发现重大隐患”“无异常”“已完成整改”“未影响核心业务”等结论；材料只给问题清单时，也不补写“总体较好”“能够正常开展”等概括性正向判断。
- 第二轮或后续修改中，用户没有补齐上一轮待确认事项，也必须继续执行本轮明确修改请求；不得把待确认事项升级为连续追问或修改阻断。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gongyu0918-debug/chinese-official-writing-skill](https://github.com/gongyu0918-debug/chinese-official-writing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
