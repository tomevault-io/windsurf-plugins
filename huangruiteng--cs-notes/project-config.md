---
trigger: always_on
description: 本文件适用于 Codex CLI/App 在本仓库内工作，补充而不替代 `.trae/rules/project_rules.md` 与 `.openclaw-memory/AGENTS.md`。
---

# AGENTS.md

本文件适用于 Codex CLI/App 在本仓库内工作，补充而不替代 `.trae/rules/project_rules.md` 与 `.openclaw-memory/AGENTS.md`。

如果本机存在 `.local/AGENTS.md`，必须先读取并作为本地私有补充指令执行；`.local/` 内容不提交、不同步、不外泄。

## 1. 仓库定位

`CS-Notes` 是一个复合工作系统，不只是笔记仓库：

- `Notes/`：长期知识管理
- `创作/`：写作与观点表达
- `公司项目/`：公司相关 WIP，默认视为私密内容
- `.trae/`：todo、规则、web 管理、执行控制面
- `.openclaw-memory/`、`.trae/openclaw-skills/`、`Notes/snippets/`、`.codex/`：agent / workflow / skill 实验区

工作时优先优化整个系统，而不是把文件当成彼此孤立的文档。

## 2. 默认原则

1. 先回答问题，再把结果落到最合适的位置。
2. 能自主推进就推进；只有在用户决策或手动操作不可替代时才停下。
3. 优先做真实改动，少做临时文件、演示性产物或伪完成。
4. 尊重现有结构与文风，尤其是 `Notes/`、`创作/`、`公司项目/`。
5. 重要结论尽量附上来源链接、文件路径或具体产物，保证可追溯。
6. 如果在“快速回复”和“可持续落盘”之间犹豫，优先后者。
7. 笔记整理类任务默认直接执行；高复杂度的代码/端到端任务可以先给简短 plan 供 review，或只问一两个关键问题。

## 3. Shell 与上下文

### Shell 策略

默认使用独立命令，保留并发能力：

```bash
zsh -lc 'source ~/.zshrc; <command>'
```

仓库已有 `.codex/environments/environment.toml` 配置 `script = "source ~/.zshrc"`，手动执行时也保持一致。

只有一串命令明确依赖共享 shell 状态、且反复加载 `~/.zshrc` 成本明显时，才使用：

```bash
/Users/bytedance/CS-Notes/Notes/snippets/codex-persistent-shell.sh
```

使用规则：

- 该脚本会在单个 TTY session 内只加载一次 `~/.zshrc`
- 适用于共享 cwd、env、alias、shell function、virtualenv / conda / nvm 状态
- 不要把 persistent shell 当全局默认，否则会削弱并发并增加状态污染风险

### 最小上下文

不要先通读整个仓库。按需加载：

- 常规必读：`README.md`、`.trae/rules/project_rules.md`、`.trae/documents/PROJECT_CONTEXT.md`
- workflow / memory：`.openclaw-memory/MEMORY.md`、`.openclaw-memory/AGENTS.md`、相关 memory 文件
- todo / 执行：`.trae/todos/todos.json`、相关 skill、`.trae/web-manager/WORKFLOW.md`
- 写作：先读 `创作/` 下 2-3 篇代表文章
- 笔记整合：先在 `Notes/` 广搜，再检查候选文件结构

### Markdown 文件打开

当需要在本机为用户打开 Markdown 文件时，默认使用 Typora：

```bash
open -a Typora <file.md>
```

## 4. 工作流

### A. 笔记整合

1. 先在 `Notes/` 中广搜最佳落点。
2. 修改 Markdown 前先看结构，优先用 `Notes/snippets/markdown_toc.py`。
3. 优先插入现有 section；确实没有合适位置再新增小节。
4. 语言尽量压缩，不为“更整洁”而删除用户原内容。
5. 外部材料必须附来源链接。
   - 如果结论来自开源仓库、官方文档、prompt 或代码文件，不要只写仓库名或文件名；尽量附到具体文件 / section 的可点击链接。
   - 对 GitHub 源码或 prompt 做机制沉淀时，优先使用 commit-pinned permalink，并在 `.local` archive 中保留读取 commit，方便以后复核。
6. 一份材料跨多个主题时，拆分落到多个位置，不强塞进一个文件。
7. `Notes/` 内的数学公式默认面向 Typora 阅读：真正的公式用块级 `$$...$$`，不要用 ```text 代码块伪装公式；普通 schema / 字段列表才用代码块。
8. 如果用户提供或原文包含对理解机制确有必要的关键图，优先保存到目标 Markdown 同名资源目录（如 `Notes/AI-Applied-Algorithms/`），再用相对路径链接（如 `![...](./AI-Applied-Algorithms/xxx.png)`）；不要只依赖聊天截图或外链。
9. 对内部飞书 / ByteTech / 公司文档做笔记时，写入 git 关联笔记库前必须脱敏：不要写内部 URL、临时 token、内部人员/团队细节或非公开实现；优先引用公开论文、开源仓库、官方文档、release note。若材料来自内部文档但也有公开实现，公开笔记只能写“基于公开/开源材料分析”，完整内部来源和私有判断只放 `.local/`。
10. `S21`、`A89` 这类材料候选库编号只用于 `.local/LEARNING_MATERIAL_CANDIDATES.md` / `.local/LEARNING_MATERIAL_ARCHIVE.md` 的追踪；写入 `Notes/` 的长期笔记标题时不要带这些编号，标题应面向主题本身。
11. 笔记整理和 `读完` 闭环完成前，必须做一次“上层 high-level 同步检查”：判断新材料是否改变目录页、领域框架、综述表、概念地图、路线图或跨材料排序；若改变，就适当更新上层笔记，而不是只把内容塞进局部 section。
12. `读完` 闭环中，公开主笔记可以压缩成框架化表达，但用户读后感里的具体抓手不能被抽象掉；必须在主笔记或 `.local/LEARNING_MATERIAL_ARCHIVE.md` 中保留每个关键机制、benchmark、schema、figure/table、failure case、术语判断或质疑点，无法保留时要说明跳过原因。

### B. 写作与公司项目

- 文风要求：平实、凝练、有立场、结构清晰、重分析与比较，避免 AI 套话。
- 做较大写作前，先读 `创作/` 下 2-3 篇文章对齐语气。
- 涉及 `公司项目/` 时，先读 `公司项目/01-公司项目创作pipeline.md`，并按该 pipeline 执行。

### C. Todo 驱动执行

- 单一数据源：`.trae/todos/todos.json`
- 用户说 `推进项目` / `推进TODO` / `推进todo` 时，默认优先关注 **本仓库** 的 TODO、文档、脚本和笔记工作流；不要主动把重心滑到 `agent-harness` 仓库。
- 只有当本仓库 TODO 明确指向 `agent-harness`、用户明确要求查看 `agent-harness`、或需要生成给 `agent-harness` 主控的转发稿时，才读取/引用 `agent-harness` 状态。
- 新增 todo：优先使用 `todo-adder`
- 执行 todo：优先走 `priority-task-reader`
- 真正开工前，先把任务改成 `in-progress` 并写入 `started_at`
- 推进任务必须带来真实产物，不要只改状态文本
- 明确区分：AI 可独立完成的任务 vs 必须等待用户动作的任务
- 用户说 `推进TODO` / `推进todo` 时，默认不是只做一个小修，而是做一次 **批推进**：
  - 目标批量：3-5 个低风险小切口，或 1 个主任务 + 2-4 个配套的文档/脚本/状态/规则更新。
  - 先列出本轮 batch，确认它们共享同一目标、写入范围不冲突、不会触碰公司私密内容或高风险 git 操作。
  - 对每个子切口都要有可验证产物；如果只够推进 1 个，必须说明为什么不能批推进。
  - 批推进完成后，统一回写 `.trae/todos/todos.json`、刷新 `.local/CODEX_TODO_TRIAGE_INDEX.md`，并汇总实际产物。
  - 这条规则服务长程 agent 目标：把 TODO 推进变成 plan -> batch execution -> checkpoint -> replayable record，而不是零散单步响应。
- `推进TODO` 的默认目标是让本仓库工作系统更好，而不是消费 `.local/LEARNING_MATERIAL_CANDIDATES.md` 队列中的具体材料。优先从流程优化、机制优化、效率优化、素材探索能力、笔记重构、脚本/skill 改进、索引治理、用户动作队列压缩中找切口；只有用户明确说 `素材：`、`调研`、`请你读`、`精读`、`读完` 或点名某个材料时，才处理具体 material。
- 推进 TODO 时，如果发现值得同步给 `agent-harness` 主控 agent 的设计判断、实验建议、数据字段、benchmark 变体或风险提醒，必须生成一段可直接转发的 `Agent Harness 主控转发稿` 给用户；不要假设 Codex 可以直接指挥另一个仓库的主控 agent。
- `Agent Harness 主控转发稿` 应该短、明确、可执行，包含：背景、建议动作、验收标准、相关文件/链接；如果只是想法而非行动，不要打扰主控 agent。

### D. Tooling / agent / web-manager

重点目录：`.trae/openclaw-skills/`、`.trae/web-manager/`、`.trae/web-manager/templates/`、`.openclaw-memory/`、`Notes/snippets/`、`.codex/`

处理这些目录时：

1. 保持 Codex、Trae、OpenClaw 的互通性。
2. 优先改善默认工作路径，而不是只做 demo。
3. 涉及模板、迁移、打包时，检查模板和脚本是否需要同步更新。
4. 注意区分“项目定制改动”与“通用模板改动”，避免写错层级。

### E. 调研 / 找素材

当用户说“素材：”“调研”“找素材”“材料雷达”“学习材料”时，默认按素材管线处理：

**指令约定**

- `素材：<链接/文本>`：用户投喂材料。读取、保留原始链接、分档、摘要并写入候选库；读不到就标记 Unread 并说明需要用户补正文/截图/导出。
- `调研：<问题/方向>`：主动调研指令。围绕问题做多源召回、追一手来源、去噪分档，并给出下一步学习/产出建议。
- `请你读：<材料 id / 链接 / 标题>`：Codex 先读原文 / repo / 数据入口，再给精要内容、核心设计、用户本人是否需要继续读、对当前 artifact 的改造点；输出时必须区分“材料值得 Codex 读透 / 落成设计语言”和“用户本人需要亲自精读原文”，并尽量细分到 section / figure / table / code / doc page 粒度，不要把二者混成一个判断；若判断“读 Codex 摘要即可 / 用户不用亲读”，摘要必须达到替代用户首读的密度，覆盖核心机制、关键字段 / schema、实验或证据、局限和 artifact 映射，而不是只给结论；若材料有关键数学机制或指标定义，在原有讲解基础上补核心公式 / 数学直觉，但不要为了公式而公式；`精读` 保留为兼容别名。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangruiteng/CS-Notes](https://github.com/huangruiteng/CS-Notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
