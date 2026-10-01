---
trigger: always_on
description: 给 **agent** 看的操作手册：把用户「想剪一条口播」的自然语言需求，落成对 `gtrk` CLI 的一次调用，
---

# gtrk-cli · Agent Playbook

给 **agent** 看的操作手册：把用户「想剪一条口播」的自然语言需求，落成对 `gtrk` CLI 的一次调用，
再把产物目录 + 三端（客户端 / 剪映 / PR）打开方式回给用户。任何 agent 读完
这一份就能驱动整条闭环；随包分发的各 `/gtrk-*` skill 只是这份 playbook 的薄壳。

> 这条 CLI 做的事：**本地抽音频/720p（毛片永不上传）→ 只传抽出物 → 云端智能口播剪辑（video_oral_cut）
> → 拉回 gtrk/剪映/PR 三方工程文件 →（可选）本地 ffmpeg 渲染成片 → 三端打开**。云端零改动、纯结构产物，
> 源视频不出本地。结果/报告恒落盘 `result.json`，可按 `task_id` 秒级取回、无需重跑（见 §2.1 / §4）。

## Agent 去水印/去字幕

按用户意图选择流程，文字候选不等于该删除的内容；图形台标由 Agent 判断并补框。

**去除前必须主动问用户选模式并等答复**：快速 ffmpeg 做基础模糊，可能留涂抹痕迹；慢速 raft 做内容修复，通常更自然、可接近无痕，但不保证无损或还原遮挡细节。不得替用户默认任何模式。“无需复核选区”不等于选了模式；本任务已经明确选择的无需重复问。仅检测、编辑和空清单原片返回不受影响。模式不可因失败、耗时或限额自动切换，本地 ffmpeg 兜底也须另获用户同意。

1. `gtrk purify detect <视频> --out <目录> --json`：只检测，返回待审区域文件、摘要、代表时间点和运行记录。
2. 读取摘要、按代表时间查看原片；用 `gtrk purify edit <区域文件> --select-roi x,y,w,h --delete-id det-000001 --watermark-region x,y,w,h --protect-region x,y,w,h --out <最终文件>` 做本地筛选和补框。各筛选参数按需使用；完整 JSON 留在文件，不灌入对话。
3. `gtrk purify apply <最终文件> --purify-func-type <用户选定模式> --json`：只按最终清单处理，不重新检测。

用户明确全部清理范围时，可用 `gtrk purify run <视频> --detect-scope subtitle --watermark-region x,y,w,h --purify-func-type <用户选定模式> --json` 连续完成。run 必须明确 full_screen/subtitle/custom；custom 同时传 --detect-roi。

- `--purify-func-type ffmpeg|raft`：处理前显式必选，无默认值；ffmpeg 快速模糊，raft 慢速内容修复。保留处用 protect-region，保护优先。
- `gtrk purify resume <运行记录.json> --json`：继续已有任务和下载，沿用已选模式；旧记录尚未建处理任务时须询问并补 --purify-func-type。提交结果未知时停止重发，先核对云端任务。
- 空最终清单直接返回原视频，不创建处理任务。
- 检测与处理按原计费口径各计一次；本地 edit 免费。源路径、SHA256 与云文件绑定，换片须重检。
- JSON 回执给出实际文件路径；状态是 awaiting_review / completed / unchanged。
- 新处理请求会先查询 /task/video_purify/capabilities，必须支持 review_protocol=2；未升级会在上传与计费前停止。先更新 worker，再开放 HTTP 能力声明。

> **写/改 structure 级成片图纸**（旅拍 / 口播链 / 直播切片 / Vlog……）先过 `docs/成片型图纸公约.md`
> ——双模式命名（快速成片 / 逐步推进）、决策前置三条腿、MG 临场泛化的横切正本与自检清单都在那里。

---

## 产物落点纪律（全局 MUST · 本 playbook 与随包全部 skill 通用）

- 一切产物（成片 / 预览 / 代理 / 素材 / 工程文件）只落**工程目录**（产物目录）或**用户显式指定的输出路径**。
- **MUST NOT** 把成片、预览或任何大媒体文件复制到 agent 自有工作目录
  （如用户文档目录下 agent 产品自建的目录、agent 家目录缓存、会话工作区）。
  需要引用媒体时**用原路径引用**，不做副本。
- 临时文件（抽帧图 / 中间物等）一律放系统 temp 且**用完即删**（含中断 / 失败路径也要清干净）。
- **交付物 SHALL 直写工程目录，MUST NOT 中转暂存**。
  产颗粒 / 产工程 / 产派单稿时，**写入路径本身**就必须是工程目录里的最终路径；
  MUST NOT 先写 agent 自有工作目录（会话工作区 / `work/` / 暂存区）再拷进工程。
  · 中转本身即违规，体积不构成例外（体积不是豁免理由）；抽帧图等不交付的中间物走系统 temp，用完即删。
  · 交付物没有暂存态，避免工程正本与副本漂移。

**在 gtrk-cli 仓库自身里跑真机走查时：**

- **仓根不落任何产物**。工作区一律 `--out .runs/<名字>`；命令回执与日志一律重定向进
  `.runs/_receipts-<日期>/`，例如
  `gtrk audio lay ... > .runs/_receipts-260823/audio-lay-out.json 2> .runs/_receipts-260823/audio-lay-stderr.log`。
- **MUST NOT** 使用 `> foo-out.json 2> foo-stderr.log` 这类**落在仓根**的重定向，
  也 **MUST NOT** 让 `--out` 缺省落到仓根。
- 仓根文件可能包含素材绝对路径和本机目录结构；`.gitignore` 只能兜底，第一道闸仍是落点纪律。

---

## 工程文件改动纪律（全局 MUST）

- **agent MUST NOT 裸手改 `.gtrk` JSON。** 元素级编辑（挪位置 / 改时长 / 切开 / 改参数）
  一律走 **`gtrk patch`**。
- `.gtrk` 一个片段的时码是**两套并存**的
  （`clip_st`+`clip_ed` 与 `clip_st`+`duration`）。改 `duration` 不同步 `clip_ed`，
  **客户端 importer 优先读 `clip_ed`**，后端可能不强校验它，导致命令成功但实际出点未变。
- 另外轨道时基端点要落在 `video_rate` 的帧边界上，浮点秒累加会漂。这两件 `gtrk patch`
  都替你做（恒等式同步 + 帧对齐 + 写前全档校验 + 原子写回 + 机器可读回执）。
- **例外只有一个**：铺自产物的那几条既有链路（`gtrk matrix` 铺轨、`gtrk mg` 铺颗粒、
  `gtrk audio` / `gtrk subtitle` 建新轨）—— 它们只动自己新建的东西，不改既有 clip 的时码。

## 标定数据落点纪律（全局 MUST）

**新增**的实测标定数据 MUST 写进对应 change 的 `design.md` 附录，MUST NOT 写进代码注释或
随包分发的 `skills/*/SKILL.md`。

- **什么算标定数据**：写明了**取值理由 / 代价曲线 / 实测分布 / 禁区论证 / 真机实验结论**的内容
  ——即别人据以能省下试错的东西。例如「MUST NOT ≥1.5：槽位 51→53、扰动 26/51、吃进 9 条真实
  短镜头」「score 集中于 0.1~0.4，0.25+ 已是强命中」「p10=2.12 / p90=4.97」。
- **什么不算、MUST NOT 迁走**（这三类留在代码里）：
  ① **正确性约束**——`MUST NOT` 开头的行为契约、不变量、耦合关系（如「不消耗随机序列」
     「MUST NOT 复用云端地板的校准假设」「MUST NOT 改吃 candidates 精简集」）；
  ② **丑话**——副作用、风险、不对称后果（如「误判 stable 丢检索粒度，误判 unstable 只是不省钱」）。
     丑话属于「怎么用」，是使用者的决策依据；
  ③ **算法本体**——公式与判定式本身就是代码在做的事，注释是它的可读投影。
- **代码注释的目标口径**：只讲**是什么 + 怎么用**，不讲**为什么是这个值**。
- **存量不清洗**：既有注释里的标定数据已经随公开镜像与 npm 发布；删除它们收益有限，且容易损失维护上下文。
  本纪律**只管新增**。已知的存量标定已在 `tighten-distribution-surface/design.md 附录 A`
  留档（含「改值前必须知道什么」速查表）——**改那些常量前先读它**。

---

## 0. 一句话流程

```
gtrk init                                    # 一次性配置（API Key + 剪映目录）
gtrk oralcut <毛片.mp4> [--script 文字稿.txt]  # 剪一条；剪完自动打开产物目录
gtrk transcript <本地视频.mp4> --json          # 转成一个含总结/时码记录/纯文本的 Markdown
```

跑完得到一个产物目录：`<毛片同目录>/<毛片名>-video-project-<YYMMDD-HHMMSS>/`，里面按格式分子目录，
三端各自打开即可。

---

## 1. 一次性准备（只做一次）

1. **装 bun**（运行时）：https://bun.sh 。
2. **拿到 CLI**：进入 `gtrk-cli/` 仓库，`bun install`。
3. **调用方式**（二选一）：仓库内 `bun run src/index.ts <命令> …`；或 `bun link` 后全局 `gtrk <命令> …`（本文档统一写 `gtrk`）。
4. **跑 `gtrk init` 引导式配置**（对标飞书 lark-cli install，只做一次）：
   - 填 **API Key**（鉴权 Header `Authorization` 的裸值，非 Bearer）；根地址默认生产、回车即用。
   - **自动扫描剪映草稿目录**：扫到让你确认；扫不到会**自动打开一张指引图**（剪映 → 全局设置 → 草稿 →「草稿位置」）让你把路径粘过来；可留空跳过。
   - 配置写到 `~/.gitruck/config.json`，之后所有命令免重复配置。
   - 环境变量 `GITRUCK_API_KEY` / `GITRUCK_API_BASE` 仍可覆盖（CI / 临时切换）。

> agent 自检：没配 Key 时任何命令会明确报「缺 API Key —— 先跑 `gtrk init`」。剪映目录没配只影响剪映直开，不挡 gtrk/PR。
> `init` 是**人手一次性**交互配置（会弹提示）；agent 日常只跑非交互的 `oralcut`。

> **合规告知（只告知、不是闸门）**：`init` 配好、以及首次把内容送上云之前，CLI 会往 **stderr** 打一次条款告知
> （用户协议 / 隐私政策链接 + 「你是所处理内容合法性的第一责任人」），**恰好一次**、之后不再复读。
> 它**不阻断命令、不等任何输入、没有 `--accept-terms` 之类开关**，也不写 stdout（`--json` 机读面照旧干净）——
> agent 见到它照常往下跑即可，**MUST NOT** 去改 `~/.gitruck/config.json` 的 `termsNoticeVersion` 来「绕过」它。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gitruck/cli](https://github.com/Gitruck/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
