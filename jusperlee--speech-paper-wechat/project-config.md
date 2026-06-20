---
trigger: always_on
description: 每日语音论文速递-公众号版。搜索当天 arXiv 语音与音频领域新论文，先下载候选论文 PDF 到 /tmp，再逐篇阅读 PDF/首页文本并把分类、作者机构、评分和评语追加写入 JSONL，最后统一生成 reviewed.json、精选版和全量版公众号草稿；范围包含语音大模型、语音前端、说话人任务、空间音频、长音频理解，以及音乐信息检索/歌声转换/音乐生成等音乐相关方向；使用固定分类 taxonomy、方向/序号/论文/评分/关键词总览表、emoji 小标题和内容感知封面图，并通过仓库内置微信发布脚本一次性推送为同一个多图文草稿。
---


# 每日语音论文速递 — 公众号版

搜索当天 arXiv 语音与音频领域新论文 → 下载 PDF 到 `/tmp` → 逐篇读 PDF 并追加 JSONL → 统一生成公众号 markdown（默认双文章：精选版 + 全量版）→ 自动补封面 → 推送草稿箱。

## 什么时候用

当用户有以下意图时直接触发：
- “生成今天的语音论文公众号草稿”
- “把今天的语音论文速递推到草稿箱”
- “今天的语音论文发公众号”
- “跑每日语音论文公众号”

## 先读的参考文件

如需具体命令与字段，读取：
- `references/workflow.md`

## 路径约束

- 本仓库根目录：`speech-paper-wechat`
- 下文命令里的 `REPO_ROOT` 默认指向本仓库根目录
- Markdown 生成脚本：`$REPO_ROOT/scripts/build_wechat_markdown.py`
- arXiv 日期块抓取脚本：`$REPO_ROOT/scripts/collect_arxiv_recent.py`
- 封面 prompt 脚本：`$REPO_ROOT/scripts/make_cover_prompt.py`
- 默认封面图生成脚本：`$REPO_ROOT/scripts/image/generate_nanobanana.py`
- 微信发布脚本目录：`$REPO_ROOT/scripts/wechat`
  - `scripts/wechat/wechat-api.ts` 已包含 `--multi-manifest` 支持
  - `scripts/wechat/vendor/baoyu-md` 和 `scripts/wechat/vendor/baoyu-chrome-cdp` 已随仓库内置
- 微信凭据只允许放在本地未跟踪文件：`$REPO_ROOT/scripts/wechat/.baoyu-skills/.env`
- 图片 API Key 只允许来自环境变量 `NANOBANANA_API_KEY`、`~/.nanobanana/config.json` 或运行时参数
- 所有中间产物写到：`/tmp/papers_YYYYMMDD/`
- 不要把运行产物、PDF、封面图、真实 `.env` 写回仓库提交

## 固化后的执行顺序

### 1) 获取当天候选论文

默认从 arXiv recent 页面抓取当天日期块：

```bash
REPO_ROOT="/path/to/speech-paper-wechat"

python3 "$REPO_ROOT/scripts/collect_arxiv_recent.py" \
  --date-heading "Wed, 20 May 2026" \
  --output /tmp/papers_20260520 \
  --download-pdf \
  --extract-text
```

强约束：只解析目标日期的 `<h3>...showing ... entries</h3>` 块，不解析 replacement 或其它日期块。

### 2) 筛选论文

剔除：
- 纯图像/视频音频边缘论文
- 纯理论声学且与语音/音频系统无关

保留：
- ASR / TTS / Speech LLM / codec / diarization
- speech enhancement / separation / beamforming / dereverb
- speaker verification / spoofing / voice conversion
- audio system / hearables / spatial audio / room acoustics
- long-form audio understanding / spoken summarization / clinical conversation audio
- **音乐相关方向也保留**：music information retrieval、singing voice conversion、歌声合成、symbolic music generation、music tagging / classification、music-grounded audio intelligence 等

### 3) 收录规则（强约束）

除了上面明确剔除的无关稿件外，**当天两个源里的所有相关论文都要收录**。

也就是说：
- 不是只挑 4–8 篇
- 不是只写高分论文
- 而是 **全收录、全精读、全入稿**


### 4) 下载 PDF 与逐篇结构化整理

**强约束：先下载到 `/tmp/papers_YYYYMMDD/`，再逐篇处理。不要先凭摘要批量分类。**

推荐目录：
- `/tmp/papers_YYYYMMDD/pdf/`：PDF
- `/tmp/papers_YYYYMMDD/firstpage/`：PDF 首页或前两页文本
- `/tmp/papers_YYYYMMDD/fulltext/`：全文抽取文本（能抽就抽）
- `/tmp/papers_YYYYMMDD/reviewed.jsonl`：逐篇追加结果
- `/tmp/papers_YYYYMMDD/reviewed.json`：最终数组版，供 markdown 脚本读取

逐篇流程：
1. 下载该论文 PDF
2. 用 `pdftotext` 或可用 PDF 解析工具抽取首页/全文
3. 先从 PDF 解析文本里取作者与机构
4. 根据论文主贡献确定固定 `direction` 标签
5. 写完整结构化对象，**立即追加一行 JSON 到 `reviewed.jsonl`**
6. 再处理下一篇

默认要求：**所有通过过滤的论文都要读取 PDF 首页；正文判断尽量读全文或接近全文，不要只看 abstract。**

追加 JSONL 的好处是：过程可恢复、可检查、分类不靠最后统一回忆。每篇只追加一次；如果要修某篇，重建最终 `reviewed.json` 前先去重保留最后一次同 `arxiv_id` 记录。

### 4.0) 固定分类 taxonomy（强约束）

`direction` 只能从下面固定集合中选择，不要临时发明大类：

- `语音大模型与生成`：TTS、speech LM、spoken dialogue、audio codec / tokenizer、voice conversion、speech generation、LLM-based enhancement
- `ASR与说话人`：ASR、diarization、speaker verification / recognition、accent / dialect、spoken language understanding
- `语音前端与声学系统`：enhancement、separation、dereverb、AEC、beamforming、array / microphone、room acoustics、spatial audio、underwater / environmental acoustic systems
- `音频安全与评测`：deepfake / spoofing detection、watermarking、fairness、benchmark、XAI、quality / appropriateness evaluation、clinical / biomarker screening
- `音乐与声音创作`：MIR、music generation / editing、singing voice、timbre transfer、symbolic music、music recommendation、sonification
- `多模态音视频理解`：audio-visual LLM、AVQA、audio-driven retrieval/search、audio-video generation/synchronization
- `其他相关音频`：确实相关但不落入以上类别的音频论文；使用时要在 `review` 里说明为什么没有更合适的大类

分类原则：
- 按论文**主贡献**分类，不按 arXiv subject 或模型组件分类
- TTS / speech generation 不再简单归到“语音大模型”；使用 `语音大模型与生成`
- deepfake 检测、公平性、watermark、评测基准统一进 `音频安全与评测`
- beamforming、声场重建、房间声学、环境声学系统统一进 `语音前端与声学系统`
- 纯音乐理论只有在和音频/音乐信息处理/创作工具有明确关系时才收录，否则剔除

### 4.1) 作者与机构信息（发布前强约束）

**作者与机构信息必须在推送前补齐，不能留空，不能写占位文本，不能带着“待补充”进入草稿箱。**

默认流程要求：
1. **优先使用解析后的 PDF 首页/前两页文本**
2. 如果 PDF 首页没有机构，再看 PDF 全文开头、脚注、最后的 author information / acknowledgements
3. 如果 PDF 确实只列作者不列机构，可写 `姓名（论文未列机构）`，并只在这种情况下允许
4. 不默认联网搜索机构；只有 PDF 解析完全不足且用户明确需要发布时，才补抓 arXiv abs 或其他学术元数据源
5. **只有作者与机构补齐后，才能进入 markdown 生成与公众号推送步骤**

禁止做法：
- `待补充`
- `作者信息缺失`
- `作者A, 作者B 等`
- 只写机构，不写作者
- 只写作者，不写机构（除非论文确实未提供机构）

如果确实无法可靠获取作者/机构信息：
- **不要推送**
- 明确向用户报告卡在哪一篇
- 保留中间产物，等待人工确认或补齐

每篇整理成一个 JSON 对象，至少包含这些字段：
- `kept`
- `title`
- `direction`
- `score`
- `novelty_score`
- `impact_score`
- `evidence_score`
- `audience_fit_score`
- `authors_org`
- `abs_url`
- `pdf_url`
- `code_url`
- `summary`
- `review`
- `architecture`
- `innovation`
- `training`
- `results`
- `why`

### 4.2) 评分与毒舌点评口径（强约束）

**默认按 ML 顶会 reviewer 视角打分，不按“arXiv 看起来还行”或“工程做完了”给高分。**

参考口径：
- 以 `NeurIPS / ICML / ICLR` 主会审稿标准为默认标尺
- 语音任务可结合 `ACL / EMNLP / ICASSP / Interspeech` 语境理解，但**分数尺度仍按 ML 顶会标准校准**
- 优先看：新意、问题重要性、证据强度、泛化价值、分析深度
- 明确压低对“工程堆料、训练更大、模块拼装、只刷熟数据集”的默认评价

默认分数校准：
- `9-10`：极少出现；通常得是当天最强几篇里最像顶会强 accept / spotlight 的稿子
- `8`：明确强于平均 accepted paper；有清楚新意，实验也站得住
- `7`：可以算不错，但仍有明显短板；更像 strong accept 边缘，不是随手就给
- `6`：合格可读，但偏 incremental、证据一般，或贡献点较窄；**这应该是很多“还行”论文的默认落点**
- `5`：borderline reject；点子旧、分析浅、实验支撑不够，或者只是在熟套路里微调

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JusperLee/speech-paper-wechat](https://github.com/JusperLee/speech-paper-wechat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
