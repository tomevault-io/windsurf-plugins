---
trigger: always_on
description: 更新日期：2026-07-21。本文用于 compact 后继续工作，优先级高于根据旧对话重新推断架构。不要再次引入已经否定的设计。
---

# Stone Memory 当前状态与后续计划

更新日期：2026-07-21。本文用于 compact 后继续工作，优先级高于根据旧对话重新推断架构。不要再次引入已经否定的设计。

## 用户的核心偏好

- 用户是个人免费用户；不要设计需要用户管理多个摘要版本的系统。
- 不要过度设计字段、状态和关系表。能从原始 feature/feeling 读取的语义，不要拆成十几个变量重复保存。
- SQLite 是正式数据源；JSON/JSONL 只保留必要的运行时 archive/full 兼容用途。
- 用户不要求“什么都记住”。没有进入清洗后 features 的普通内容（例如炸鸡）可以不参与本轮生命周期分析。
- 人工查看和纠错是可选操作，不是自动流水线的审批关卡。用户修改 feature 或规则后重跑即可。
- 保留两种人工锚点：retain 原文锚点用于 rebuild 保留真实对话；event 事件锚点用于标记长期关键事件，作为生命周期保护和主 Agent 巡检信号。两种锚点对应的 feeling 都直接排除出自动压缩候选，无视 relation/work/fact 的阶段和 importance 规则；只有用户从 `retain-config.json` 移除锚点后，下一次完整重算才重新进入压缩范围，不写入永久豁免状态。事件锚点不再绑定旧月摘要功能。

## 硬性架构边界：CLI 是唯一写入口

Stone Memory 的 `stmem` CLI 是所有正式状态变更的唯一公共接口，不只是终端外壳。前端、未来桌面壳、脚本和自动任务不得各自复制或直接调用底层写逻辑。

```text
终端 ────────────────→ stmem 命令 → 正式写逻辑
前端 → 本地 HTTP 适配 → stmem 命令 → 正式写逻辑
自动任务 ────────────→ stmem 命令 → 正式写逻辑
```

执行规则：

- 任何新写功能必须先有一个可独立运行、可 dry-run/校验的 `stmem` 命令，再接前端；不存在对应 CLI 时，不得先在 HTTP route 中实现写入。
- HTTP 层只做身份为本机的参数接收、临时文件、安全转交和命令结果格式化，不直接写配置、SQLite、archive/full、线程 JSONL 或锚点文件。
- 禁止在 HTTP 层直接调用 `createThread`、`ingestRecords`、`applyCoarseWeek`、`permanentlyTrimThread`、`repairThreadIntegrity` 等内部写服务；这些只能位于对应 CLI 命令的正式实现路径中。
- 只读查询与预览可以直接调用 reader/planner/parser，但预览不得悄悄改变任何状态；确认 apply 后必须回到对应 `stmem` 命令。
- API Key 等敏感值不得进入进程参数；通过权限 0600 的临时文件或标准输入交给 CLI，用完立即删除。Stone Memory 是纯本地单用户工具，不预设云端上传：设置页可以从本地 `stmem.json` 读取 Key，以 password 黑点默认遮挡并由小眼睛显隐。
- 前端新增按钮前必须回答：“它实际调用哪个 `stmem` 命令？”答不出来就停止实现，先补 CLI。

当前映射：

```text
创建记忆体       → stmem init --batch-file
更新 init 设置   → stmem init --thread <已有ID> --batch-file
正式导入         → stmem import --apply
挖掘             → stmem mine
自动/手动压缩    → stmem compact / stmem compress
长期隐藏         → stmem hidden
线程重建与裁剪   → stmem rebuild --plan --apply
线程检查与修复   → stmem rebuild --check / --repair
数据库维护       → stmem db
```

## 已确认的数据职责

```text
清洗后的 features → 决定哪些概念值得关注、继承所属 category
feelings            → 保存完整摘要语义、事件和 importance
archive             → 回查用户真实原文，统计次数、日期和时间曲线
```

features 是高价值索引，不是重复证据。feature cleanup 可以继续去重；词的真实频率和时间分布从 archive 计算。生命周期最终处理对象是 feeling，不是词。

Feature miner 必须给全部 category 明确定义，并按“事实本身”而非聊天场景分类。优先选择最具体的唯一类别：饮食及直接反应归 eat，身体健康归 body，睡眠作息归 sleep，明确产出/进度/技术决策归 work，关系身份/称呼/角色/边界/稳定互动归 relation，跨场景行为规律归 habit，地点事实归 location，稳定喜恶/观点/价值判断归 preference，misc 仅作真正无法分类的兜底。和 AI 聊工作或吃饭不自动归 relation；项目期间发生的熬夜、吃饭也不自动归 work。同一事实不能换词后跨库重复投放，只有确实包含两个独立事实时才拆条。API 内置 prompt 与 `operations/memory-miner-operations.md` 必须保持这套边界一致。

新挖掘的 feelings 必须按事件发生时间从早到晚保存：API 内置 prompt 与 `operations/memory-miner-operations.md` 都要明确要求模型按时间排序；落库前仍须使用摘要开头的中文时间做稳定排序，同一时间保持模型原顺序，无法解析时间的条目保持相对顺序并放在最后。该规则只约束新挖掘结果，不自动重排历史 feelings。

精准补挖必须复用正式 CLI：前端将用户选中的同一天消息时间戳写入临时 batch JSON，并调用 `stmem mine --targeted --batch-file <json>`。模型只接收选中对话；当天已有摘要最多取前五条作为叙述语气样本，禁止把样本重复总结。结果使用 `targeted` source 追加进 SQLite，不覆盖当天既有 feelings，也不改写该日常规挖掘状态。前端对话范围选择须支持 Shift 连选。

新 miner 的 importance 只生成 2、3、5：2 是普通但值得保存的事实/片段，3 是有持续价值的默认等级。feature 的 5 按所属领域判断，只用于极少数对理解用户在该领域具有长期价值、重大变化或不可替代性的核心特征，不能用 relation/work 的标准压低 eat/preference 等库；feeling 的 5 仍保持极少，只用于不可替代的转折、承诺、身份确认、重要边界改变或能解释大量后续互动的根事件。情绪强烈、亲密、争吵、性或技术修复本身不构成 5。历史 1、4 暂不改库。

Feeling 压缩使用 `operations/memory-compressor-operations.md`，与 miner 一样支持 API/subagent。历史 importance 1–3 压成客观事实，4–5 保留核心感受但简写；完整 content 永久保留。`coarse_summary` 必须原样保留原 feeling 开头的完整日期和对应时间，这个时间位置是叙事记忆的核心，不能只靠 sourceDate 补日期。

新 compressor 每条还必须返回 1～3 个可在 archive 回查的具体 `coreTerms`，写入 `feelings.coarse_terms` JSON。优先作品名、人物、食物、地点、项目名等事实核心，禁止“事情、感觉、聊天、喜欢、以后”等空泛骨架。历史 coarse 和用户手写 coarse 可以没有 core terms；缺词时不补跑模型，也绝不自动 hidden。

`coarse_terms` 必须由 compressor 直接从原 feeling 选择事件最相关的 1～3 个原词，不能自己造，也不能把顺带出现的背景词当事件核心。“老公叫她点糖醋排骨”应选“糖醋排骨”；只有事件本身讨论称呼、关系身份或边界时才选“老公”。不要在 hidden 阶段按 relation 等 category 事后改写或过滤模型给出的词，词义正确性应由 compressor prompt 和真实样本验收保证。

未来前端必须区分自动压缩与用户主动编辑。用户新增/移除 retain 原文锚点或 event 事件锚点、手动切换 `daily/coarse/hidden`、编辑已有 `coarse_summary` 后，保存应立即生效并由下一次 rebuild 读取，不必等待 compact；但这些写操作必须先补对应 `stmem` 编辑/锚点命令，再由前端调用，禁止 route 直接更新配置或 SQLite。手动 coarse 使用用户填写的文本，不调用 API/subagent；只有 auto compact 需要模型生成 coarse 文本。手动 hidden 只把注入颗粒度设为 0，仍永久保留完整 `content` 和已有 `coarse_summary`，并允许恢复 daily/coarse。前端应显式提示锚点与 hidden 的语义冲突，避免用户以为 retain 原文锚点仍存在时已经完全停止注入对应原文。

## 当前词语管线

目标不是理解“她喜欢寿司”中的 likes 关系，只需提取“寿司”，再找到包含寿司的 feelings。摘要原文本身已经保存喜欢、不喜欢等语义。

正式方向：

```text
所有清洗后 features
→ 通用名词/连续短语提取
→ 简繁归一化和同词证据合并
→ 回查 archive 用户消息
→ 回查 feelings 与 feeling importance
→ 按 feeling 聚合词语时间证据
→ 生命周期 dry-run
```

### 通用提取器原则

- 所有 feature category 使用同一套算法；不要再为 eat/work/relation 分别写正则。
- 不绑定用户性别：“她、他、TA、用户、自己”等均视为通用主语/停用词。
- AI 只在既有 miner 阶段生成 feelings/features；生产环境不会为不同用户临时让 AI 写提词脚本。
- 提取的是可追踪的记忆概念，不等同于词典名词。除引号、括号列表、称呼、专名、英文和连续名词组合外，也保留有内容的行为与状态词，例如外卖、熬夜、喝茶、焦虑；不能因分词器把“外卖”标成动词就漏掉。
- “她、用户”等主体词以及“喜欢、觉得、经常、需要、进行”等叙述骨架不进入概念词。内容词与语法骨架的边界由通用最小停用词控制，不让用户逐词维护分词器例外。
- 完整短语优先，例如“糖醋里脊”“毕业论文”“记忆连续性”。
- 支持称呼后缀修复，例如“可老师、水母老师”；当前支持老师、先生、女士、姐姐、妹妹、哥哥、弟弟、医生、博士、教授。
- OpenCC 用于简繁统一查询；保留 `opencc-js` 依赖。
- 输出仅包含 term、normalizedTerm、category、featureIds、最高 feature importance、sourceDates 等索引证据；不要输出 likes/cannot_drink 等关系模型。

当前 019 线程共有 720 条清洗后 features，通用提取器得到约 1306 个跨库去重词。按 feature 支持数，全局前十为：

```text
习惯 34、关系 25、玩具 22、工作 20、石头 18、
系统 18、计划 18、论文 17、通宵 16、技术 15
```

这证明词库质量已经足够好，不需要大型外部停用词库。保留最小语法停用词即可。“石头、论文、通宵”等高频词仍然重要，不能把高频或跨库直接等同于噪音。

relation 实测能够提取老公、老婆、糯糯、暗石、石头君、小汤包、安全感、关系边界、记忆连续性、数据完整性等个人词。不要为 019 的私有词硬编码。

## ConceptNet 状态


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wanyu445/stone_memory](https://github.com/wanyu445/stone_memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
