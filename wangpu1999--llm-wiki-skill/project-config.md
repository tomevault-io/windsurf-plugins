---
trigger: always_on
description: 用 Codex 直接维护客制化 LLM Wiki / Markdown 知识库。Use when the user wants Codex to ingest sources, query with citations, lint, analyze knowledge graph, manage review/deep-research queues, or emulate nashsu/llm_wiki without an app, API key, or local model server.
---


# LLM Wiki Skill

这个 skill 用来让 Codex 直接维护一个可长期积累的 Markdown wiki：原始资料保持不可变，Codex 增量生成和更新 wiki，Obsidian 只是可选的浏览器。

## 核心约定

- 保持三层结构：`raw/` 原始资料、生成的 `wiki/`、控制文档 `purpose.md` 和 `schema.md`。
- Codex 负责生成和维护 wiki 页面；人类负责选择资料、设定优先级、做关键判断。
- 优先把有长期价值的回答写回 wiki，而不是只留在聊天里。重要回答进入 `wiki/queries/` 或 `wiki/synthesis/`。
- 使用 `[[wikilinks]]`、YAML frontmatter、来源引用、`wiki/index.md`、`wiki/overview.md` 和只追加的 `wiki/log.md`。
- 除非用户明确要求导入、移动或删除 source 文件，否则不要修改 `raw/sources/` 下的原始资料。
- 不把整个 Obsidian vault 自动纳入 wiki。`2-Learnings/EmbodiedWorld/`、`2-Learnings/Courses/`、`0-Daily/`、`1-Meetings/` 是宽输入池；`3-Projects/` 是围绕特定项目组织学习、方案和工作推进的项目层；`4-Research-Wiki/raw/` 是已经决定进入该专题 wiki 的精选证据库；`4-Research-Wiki/wiki/` 是 Codex 编译后的理解层。
- 判断资料是否进入 `raw/`，看它是否值得成为该专题的长期证据，而不是看它有没有 property/frontmatter。

## 原子笔记原则

Research-Wiki 的长期理解层遵循 Zettelkasten / 原子笔记理念：它不是资料堆放处，也不是把 source 改写成另一篇长文，而是把可复用的想法编译成彼此连接的小卡片。

- 一条稳定笔记只承载一个核心想法：一个 concept、claim、question、method、research-practice 或清晰的关系判断。
- `source` 页面可以稍长，用来保存来源上下文；`concept`、`claim`、`question`、`research-practice` 应尽量原子化，不写成无边界综述。
- 写笔记时先问：未来的我会在什么情境下重新需要这个想法？它应该和哪些旧笔记互相发现？
- 不做 source 原文搬运。用自己的话重写理解，同时保留来源引用、PDF/arXiv/网页链接和必要的原句定位。
- 优先创建或更新多条互相链接的小笔记，而不是把多个可独立复用的想法塞进一个大 synthesis。
- 每条 durable note 至少说明：单一想法是什么、来自哪里、连接到哪些旧笔记、还存在哪些待验证问题。
- 每条 durable note 必须能通过 `wiki/index.md`、`wiki/overview.md` 或一条清晰的 wikilink 路径重新找到；孤立笔记不算完成。
- 如果一条内容没有未来复用途径、没有可连接对象，也不能形成概念/主张/问题/方法论，就暂时留在 Daily、Projects 或 raw，不进入 wiki 理解层。

## 输入层边界

推荐关系：

```text
2-Learnings/EmbodiedWorld / 2-Learnings/Courses / 0-Daily / 1-Meetings
= 未筛选或半筛选输入池

3-Projects
= 项目级学习、方案、推进记录和待验证判断

4-Research-Wiki/raw
= 已决定进入世界模型研究语境的证据库

4-Research-Wiki/wiki
= Codex 编译后的概念、主张、问题、综合与方法论
```

操作原则：

- 外层资料池可以宽，`4-Research-Wiki/raw` 必须精选，`4-Research-Wiki/wiki` 必须编译。
- `2-Learnings/EmbodiedWorld/` 不放进 `4-Research-Wiki/` 内部；它是世界模型专题学习层、paper cards 和候选池，不再包一层 `Papers/`。
- `2-Learnings/Courses/` 单独保存课程资料、课件和课程笔记；课程中的稳定概念或方法论应先蒸馏为 study note，再进入 Research-Wiki。
- `0-Daily/` 和 `1-Meetings/` 不全量进入 `raw/`；只把蒸馏后的高价值片段放入 `raw/sources/daily-distillations/` 或 `raw/sources/meeting-extracts/`。
- `3-Projects/` 不保存实验代码、训练日志和论文写作工程；它保存项目目标、方案推理、定向学习、任务拆解、风险判断和阶段总结。
- Projects 中稳定、可复用、跨项目成立的概念、claim、open question 或 research-practice，才进一步蒸馏进入 `4-Research-Wiki/`。
- README 只保留在 vault 根目录下的一级模块中：`0-Daily/README.md`、`1-Meetings/README.md`、`2-Learnings/README.md`、`3-Projects/README.md`、`4-Research-Wiki/README.md`。不要在二级或更深层目录继续散落 README；需要说明时合并进对应一级 README。
- 一份材料进入 `raw/` 前，先问：它是否值得成为 Research-Wiki 的证据？
- 如果不能生成或更新 concept、claim、question、synthesis 或 research-practice，就暂时不要进入 `4-Research-Wiki/wiki/`。
- 进入 `4-Research-Wiki/wiki/` 时，先判断是否应拆成原子笔记：source summary 负责记录来源，长期理解优先拆成可链接的 concept、claim、question 或 research-practice。
- 原始输入文件可以没有 property/frontmatter；生成的 wiki 页面必须有 frontmatter。
- 多机器同步时，skill 源文件保存在 vault 内的 `.tools/skills/llm-wiki-skill/`；每台 Mac 通过 `~/.codex/skills/llm-wiki` 软链接到该目录。新机器按 `Vault Setup.md` 运行 `.tools/setup-vault-mac.sh`，不要手动复制整份 `~/.codex/`。

Daily 输入层包括：

- 文本形式的日常记录：想法、问题、反思、灵感、即时理解。默认写入 `0-Daily/Diary/YYYY-MM-DD.md`；历史研究日记和已整理文本记录保存在 `0-Daily/Diary/Research Diary.md`。注意目录名是大写 `Diary`，不要使用 `0-Daily/diary/` 或旧路径 `0-Daily/Research Diary.md`。
- AI 聊天记录：GPT、Gemini、NotebookLM 等对话。它们是思考痕迹和候选观点，不是默认可靠来源；不需要每天处理，应周期性批量提取核心知识。
- 手机截图：主要来自微信/公众号和小红书。它们默认上下文不完整、来源质量不稳定，需要更强筛选。截图原图放入 `0-Daily/Screenshots/assets/`；整理稿（`0-Daily/Screenshots/YYYY-MM-DD-简短主题.md`）**不放内嵌原图**，结构固定为：**上半「总结」**（主题、要点、与 vault 的衔接、待核验），**下半「全部原文（OCR）」**——将截图中**可见文字尽数转写**（含标题、副标题、段落、列表、小标题、按钮文案、评论等版面文字），并标 `ocr.status`；不是「摘录」或大幅删节。**写作前须校验原图**：若宽高异常（例如宽度仅数十像素的长条），说明导出损坏或误选文件，须请用户重导原图后再 OCR，**禁止**仅凭多模态「读图描述」编造正文。论文线索与判断写在「总结」中即可。B 站内容通常不走截图主流程，更适合以链接或剪藏进入 `0-Daily/Clippings/`。
- Clippings / 剪藏：网页文章、公众号网页、访谈、博客、平台内容等网页剪藏。它们属于 Daily 日常摄取，不等同于已进入 Research-Wiki 的证据；不要把截图整理稿默认放进 `0-Daily/Clippings/`。

处理 Daily 时：

- 不直接把 daily 全量摄入 wiki。
- 每日原始日记主入口是 `0-Daily/Diary/YYYY-MM-DD.md`。如果用户说“写进今天的日记”或“记录到 Daily”，优先写入当天日期文件，而不是写入 `0-Daily/Diary/Research Diary.md`；只有历史整理、跨日合并或用户明确要求时，才维护 `0-Daily/Diary/Research Diary.md`。
- 先生成 `raw/sources/daily-distillations/YYYY-MM-DD.md` 这类蒸馏稿。
- 只有能转化为 concept、claim、question、synthesis 或 research-practice 的部分，才进入 wiki。
- AI 聊天应低频、批量、少手动地处理。不要要求用户每天导出或整理；当用户提供一批 ChatGPT/Gemini/NotebookLM 记录时，只提取核心知识。
- AI 聊天中的事实主张需要追溯到论文、原文、实验或可靠资料；如果暂时无法验证，放入 `reviews.md`。
- 从 AI 聊天中只保留：用户反复追问的问题、被用户认可的判断、概念澄清、研究路线、阅读路线、实验想法、待验证 claim。忽略寒暄、工具性问答、泛泛建议和重复解释。
- 推荐批处理路径：`0-Daily/ai-chats/batches/` 存放定期导出的原始聊天记录，`raw/sources/daily-distillations/YYYY-MM-DD-ai-chats.md` 存放蒸馏后的核心知识。
- 截图内容可以保留启发价值，但事实可靠性需要二次验证。**整理稿**须含可复制检索的 **全文 OCR**（版面文字尽量不漏），且**不嵌原图**，避免体积与检索问题。
- Clippings 先提取正文、元数据、论文线索、核心观点、启发和待验证事实；只有能转化为 concept、claim、question、synthesis 或 research-practice 的内容，才进入 wiki。
- 从 Clippings 提炼长期知识时，不按原文段落机械摘要；先识别可复用的单一想法，再写成原子笔记或更新已有笔记。

截图处理分流：

- 如果截图中识别到论文线索，例如标题、作者、arXiv ID、DOI、会议名、论文页面、论文 PDF 或论文总结，应自动进入论文追踪流程：联网搜索对应论文资源，优先找到 arXiv、DOI、PDF、OpenReview、会议页面、项目页和代码仓库；总结论文内容，并判断它与世界模型研究的相关性。截图整理稿默认写入 `0-Daily/Screenshots/YYYY-MM-DD-简短主题.md`：**不得**用 `![[...]]` 嵌入原图；须按 **总结 / 全部原文（OCR）** 两截书写，原文部分**尽录可见文字**。原图只保留在 `0-Daily/Screenshots/assets/`（整理稿中可用 YAML `source` 指路径，不嵌图）。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WangPu1999/llm-wiki-skill](https://github.com/WangPu1999/llm-wiki-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
