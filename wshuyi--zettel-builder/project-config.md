---
trigger: always_on
description: |
---


# zettel-builder

让卡片**自己长出来**:素材进 → 切卡 → 连边 → 巡检 → 拼文。用户在每个分叉点保留决策权,但不必逐张手动建卡。

## 核心理念(凡修改先读一遍)

1. **原子化是为了可重组,不是为了字数小**。判断一张卡是否合格的硬标准:**脱离上下文,半年后的你能不能独立读懂?** 能,就是合格,几百字也行。不能,再短也是垃圾。
2. **"用自己的话重述"不是同义词替换**。必须产生新判断:这个观点和我已知的什么冲突?边界条件是什么?我相不相信?没有这一步,重述只是更慢的复制。
3. **链接的价值在于**写出*为什么连*。盲连等于不连。每条 link 必须带 `reason` 字段(一行话)。
4. **文章不会"自己长出来"——文章是从已有写作方向上的稠密簇中浮现的**。本 skill 帮你更高效地组织既定方向的材料,不替你决定写什么。这是诚实的边界。
5. **触发分两层**:mechanical scan 由 systemd 离线跑(纯 Python,不切卡只列待办);agent scan 由用户或 `/schedule` 触发(真正切卡 + 价值观重述 + 连边)。

参见 `references/philosophy.md`(为什么这样设计 + 与 topic-inspiration 的区别)。

## 路径默认

| 参数 | 默认 | 说明 |
|---|---|---|
| `zettel_root` | `~/zettel/` | 卡片仓根目录,独立 git 仓 |
| `wiki_root` | `~/wiki/published-articles/` | 上游素材来源 |
| `raw_articles` | `${wiki_root}/raw/articles/` | 已发表文章源(241 篇+) |
| `raw_getnote` | `${wiki_root}/raw/getnote/` | GetNote 笔记同步(1974+) |
| `published_source` | `~/Dropbox/cn_articles_published/` | 已发表文章上游(去重对照) |

## 卡片格式硬规则

完整规范见 `references/card-format.md`。一行话总结:

```yaml
---
id: YYYYMMDD-HHMM-<kebab-标题>
status: fleeting | literature | permanent
created: <ISO8601>
source: {type: ..., ref: ...}
tags: [...]
entities: [...]
links: [{id: ..., reason: ...}]
voice_passed: true|false
---

# 标题(直接陈述一个判断,不是名词短语)

正文(150-800 字,可超,以脱离上下文可读为准)。
```

**永远不要**:
- 同时往一张卡塞多个独立判断(切两张)
- 标题写成名词("原子化原则")而不是判断("原子化的目的是可重组")
- 链接只列 id 不写 reason
- 把 raw 原文复制进卡(literature note 只引路径 + 短摘录)

## 四种模式

### Mode 1: ingest — 单次喂料

用法:`/zettel ingest <来源>`。来源可以是:
- 一段贴在对话里的文本
- 一个文件路径(`/zettel ingest /path/to/note.md`)
- 一个 URL(走 markdown-proxy 先抓正文)
- 本次对话上方的若干消息("把上面那段对话切卡")

**步骤**:

1. **来源归一**:把素材转为 Markdown 正文 + 来源元数据(type/ref/timestamp)。
2. **原子切分(v1.13.0 加强:长文必须多卡)**:
   - 识别**所有独立判断**,每个一张卡。判断标志:"X 不是 Y,而是 Z"、"X 的真正原因是…"、"X 和 Y 的边界是…"、"X 表面 ... 实际 ..."
   - **wiki-article 长文必须通读全文找全 candidates**,典型字数对应产卡数:短文 2-3 张 / 中长 3-6 张 / 长文 6-10 张 / 巨长 10-15 张
   - 若 > 1500 字的 article 只产 1 张卡,**必须**在 `processed/<date>.jsonl` 写明理由(否则视为信息压缩失败)
   - 详细判读流程见 `references/voice-alignment.md` Step B.2
   - **列举不算判断**(三条原则、四种类型),除非每条独立成段并含论证
3. **价值观重述**:对每张卡,调 `references/voice-alignment.md` 的协议——读 `references/voice-snapshot.md`(你自己的价值观/文风快照)的核心条款,重写卡的正文。**关键**:不是改文风,是产生新判断。如果原文是引述他人,重述时必须加上"我同意/不同意/部分同意,因为…"。
4. **链接候选**:跑 `scripts/link_candidates.py --card-id <new-id>`,得到 top-10 候选卡。对每个候选,agent 判定是否真连 + 写一行 reason。无连不强求。
5. **去重 + 链接(v1.6.0 agent-judged)**:嵌入只做**召回**,**最终判断必须 agent 读全文**。
   - **召回阶段**(嵌入):算候选卡嵌入,跟 `_index/embeddings.npy` cosine,取:
     - 近邻 top-20(sim 倒序)
     - **远距 top-5**(从 sim<0.5 的卡里随机抽,关键 — 防语义局部扎堆)
   - **判断阶段**(必须 Read 候选卡全文):
     - 对近邻候选:Oven 读全文,判定"这是否真的同一判断"。**不**基于 sim 自动跳过。
       - 真同一判断 → dedup-skip,记 `{action:"dedup-skip", existing:<id>, agent_reason:"..."}`
       - 表面相似但实质不同 → 写新卡,但**必须自动 link** 到那张近似卡,relation + reason **说清差异**
     - 对远距候选:Oven 读全文,判定"有没有非显然的跨域关联"。命中即建立 link,relation 通常 `extend` / `counter`,reason 说明跨域桥梁。
   - **绝不**直接用 sim ≥ 0.85 自动跳过 — 必须 agent 看完判定
   - **绝不**只从近邻候选选 link — 至少考察 5 张远距,防扎堆
5.5. **filename + id 生成(v1.9.0,关键)**:写盘前确定 filename 和 frontmatter:
   - `id`(frontmatter)= 13 字符 `YYYYMMDD-HHMM` 纯时间戳。**不**含标题文字
   - `aliases`(frontmatter)= 完整 H1 标题作为第一条(Obsidian 双链友好,改标题不破链)
   - `filename` = `<short-id>-<主语-主结论>.md`,slug ≤ 20 中文字符,首词主语,英文小写连字符
   - 完整规则见 `references/card-naming.md`
   - 链接(frontmatter `links:` + 正文 `[[]]`)永远引用短 id
6. **图片搬运(v1.4.0)**:若 source 是含 `![](...)` 的 raw markdown,先跑 `python3 scripts/migrate_images.py --card-id <id> --source <raw-path>` 把图复制到 `assets/<id>/`,记录路径 mapping。后续正文用新路径引用关键图(支撑本卡判断的;不必全搬)。
7. **写盘(v1.6.0 self-contained + 实质化引用 + Obsidian 双链)**:`cards/<id>.md`,正文结构:
   - `## 原文要点` 节(blockquote 完整搬运 200-500 字原文,含图)
   - `## 我的判断` 节(选边表态 + 边界 + **必须自然引述 ≥1 张相关卡**)
   - `## 相关卡片` 节(Obsidian 双链索引)

   **关键(v1.6.0):实质化引用**。`## 我的判断` 不能只是孤立陈述,必须在正文里**自然引述**至少一张相关卡:
   - 错(只有 ## 相关卡片 节列条目):`## 相关卡片\n- [[X]] — support: ...`
   - 对(## 我的判断 正文里融入):`...如 [[X]] 所论 A 边界在 B,但本卡聚焦的是 C。[[X]] 强调测量,本卡关心如何处置——这是同一光谱上的两端。`
   
   引述必须言之有物(说出"为什么相关"+"差异点"),不是机械加 `[[]]`。validator 检查 `## 我的判断` 节里至少有 1 个 `[[]]`,否则 voice_passed=false。

   `## 相关卡片` 节继续做 index(给 cluster_inspect 等脚本用),validator 校验两边数量一致。
8. **git push 硬条款**(v1.2.0):写完所有卡片后**必须**执行 `cd ~/zettel && git add -A && git commit -m "feat(ingest): <一句话摘要 N 张卡>" && git pull --rebase origin main && git push origin main`。不依赖 systemd timer 兜底。push 失败必须显式报错(常见原因:无网、远程有冲突)。
9. **回报**:列出新建卡片清单 + 每张的链接情况 + dedup-skip 数量 + push 是否成功。

### Mode 2: scan — 消化 mechanical queue

用法:`/zettel scan`(或由 OpenClaw cron / event-driven trigger)。

**前置**:`scan_mechanical.py` 由 systemd 每小时跑,把 `raw/` 新增文件写进 `_queue/*.json`。Agent scan 消化这些 queue 项。

**步骤**:

1. **状态检查**:读 `_queue/` 当前积压;读 `_state/last_scan.json`;**读 `_state/mode.json` 判定当前模式**(`bootstrap` 或 `steady`)。
2. **批量限额**(v1.3.0 mode-aware):
   - `bootstrap` 模式:N=20(库存高速消化期,挑战长 session token budget)
   - `steady` 模式:N=5(日常消化,event-driven 触发)
   - 读 `_state/mode.json` 中 `thresholds.bootstrap_batch_size` / `steady_batch_size`,出错时 fallback 默认值。
3. **对每个 queue 项**:
   - 读 source 原文(literature note 只引路径,不复制原文)
   - 走 ingest 的 Step 2-5(切卡 + 重述 + 链接 + 写盘)
   - 处理完把 queue 项移到 `_state/processed/<date>.jsonl`
4. **顺手 inspect**:跑一次 Mode 3 inspect 的轻量版,若发现文章就绪簇,发 Telegram 提醒。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wshuyi/zettel-builder](https://github.com/wshuyi/zettel-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
