---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目定位

livecaption 是 macOS（Apple Silicon）本地实时英文转录 + 中文翻译的命令行工具，无 UI。ASR 用 mlx-audio 跑 NVIDIA `nemotron-3.5-asr-streaming-0.6b`（cache-aware 真流式 transducer），端点检测用 Silero VAD（mlx 版）；翻译用 mlx-lm 跑腾讯混元 `Hy-MT2`。两者都走 Apple GPU/MLX、共享统一内存——VAD 把静音段挡在 encoder 外省 GPU。**全进程 mx 求值都过 `runtime.MLX_LOCK` 串行化**（MLX 多线程并发求值无官方保证）：ASR 侧整段计算持锁（毫秒级），翻译侧用 `stream_generate` 逐 decode step 取锁——既保证线程安全，又不让一次翻译把 partial 冻住数秒。

## 常用命令

```bash
uv sync                                      # 装依赖
uv run livecaption --source mic|system|both  # 运行；首次自动下载模型（ASR ~1.2GB + VAD + 说话人分离 ~225MB + 翻译 ~2GB）
uv run livecaption --source file --file x.wav --out y.md  # 转录音频文件，跑完即退出（全流程端到端测试入口）
uv run livecaption --list-devices            # 列出麦克风设备
bash scripts/build_audiotee.sh               # 编译系统音频捕获二进制（仅 system/both 需要）

# 验证：本项目无正式测试框架，用这些手动冒烟脚本
uv run python scripts/smoke_asr.py           # ASR 端到端：加载模型 + 解码自带 test wav，对照离线结果
uv run python scripts/smoke_translate.py     # 翻译端到端
uv run python scripts/smoke_diff.py          # _inline_diff 纯逻辑测试（无模型，秒级）
uv run python scripts/smoke_mel.py           # 增量 mel 与整句一次性计算的一致性（改 _mel_grow 后必跑）
uv run python scripts/diag_system_audio.py   # 系统音频 / 权限自检（播放声音看能否捕获到非零数据）
uvx ruff check .                             # lint（规则集见 pyproject）
```

## 架构：三阶段线程流水线

音频从采集到上屏经过三个用队列解耦的线程：

```
AudioSource(后台)  ──queue──▶  AsrWorker(线程)  ──final 句──▶  Translator(线程)  ──▶  Renderer / FileWriter
 mic: sounddevice 回调           mlx-audio nemotron-3.5         mlx-lm + Hy-MT2
 system: audiotee 子进程         + Silero VAD 端点切句
                                 partial / final 双事件
```

- **`audio.py`** — `MicSource`(sounddevice 回调)、`SystemAudioSource`(读 audiotee 子进程 stdout)、`FileSource`(解码音频文件)，统一把 16k float32 mono chunk 放进 `self.queue`，流结束放 `SENTINEL`。实时源绝不阻塞（队列满了丢帧）；**FileSource 例外：阻塞 put 不丢帧**（无实时性约束，端点检测数 VAD 帧不依赖墙钟）。`SystemAudioSource` 有 supervisor 线程做断流看门狗：健康的 tap 静音时也持续输出零字节流，**完全无数据 ≥5s（`SYSTEM_AUDIO_STALL_SEC`）= tap 已死**（实测诱因：切换默认输出设备，音频在新设备继续播、tap 还挂在旧设备上 IO 停转），此时杀掉重启 audiotee 重新 tap 当前设备，字幕自动恢复——曾因纯阻塞 read 静默饿死整条流水线且无任何提示。
- **`asr.py`** — `Recognizer`(共享权重) + `OnlineStream`(单流状态机：Silero VAD 判开口/静音，push 式增量 encoder + RNNT 解码) + `AsrWorker`(消费队列、把事件转回调)。发两种事件：`on_partial`(中间结果，随说话变) 和 `on_final`(端点后定稿)。diarize（默认开）时一句可能切成多个 final（每说话人一段），label 带 `#S1` 后缀。
- **`translate.py`** — `Translator` 后台加载 Hy-MT2，**只翻译 final 句**。翻译堵塞绝不反压音频（积压 ≥10 句时向 stderr 告警一次）。`context_size>0` 时用最近 N 句原文做翻译背景。生成用 `stream_generate` 手动驱动、每个 decode step 取一次 `runtime.MLX_LOCK`。模型加载失败会打印 "translation disabled" 并退出线程，转录照常。输出侧有三道防护（都不改官方 prompt 模版）：(1) 源句 < `MT_CONTEXT_MIN_WORDS` 词时不带上下文（碎片输入易让模型转去翻背景块）；(2) `_strip_boilerplate` 剥掉 "根据背景信息，以下是译文：" 这类元文本前缀；(3) 上下文回声检测（译文与上一句译文的字符 bigram 重合 ≥45% = 模型翻了背景而非原文，实测阈值能区分劫持与同话题相邻句），命中则去掉上下文重翻一次。
- **`runtime.py`** — 进程级 `MLX_LOCK`（中立归属，asr 和 translate 都 import 它，谁也不依赖谁）。
- **`render.py`** — `Renderer`(rich.Live 终端，底部活动区刷 partial、上方滚动 final) + `FileWriter`(追加写文件)。配色按 `--theme`(auto/light/dark) 分三套：单套固定色总会在某种背景下隐身，所以**只有译文真正随背景变**(default=默认前景+加粗"不赌颜色"，dark=亮青，light=深青蓝)，其余元素用对两种背景都中性可见的固定样式(grey50 中灰、说话人调色板)——**一律不用 `dim`**(dim 相对背景降亮度，两种背景都偏淡，正是用户反馈"看不清"的根因)。说话人 S1–S4 各一色(`_SPEAKER_PALETTE`，按主题分套且刻意避开青系以防与译文撞色、避开绿系以防与 diff 新词撞色；S1 品红/S2 蓝/S3 橙/S4 红跨主题保持同色相)。diff 配色：纠掉的词用中灰删除线（被取代的内容不该比正文显眼；红色曾与 S4 标签混淆），新词绿色。auto 从 `COLORFGBG` 末段探测背景，探测不到回退 default。
- **`cli.py`** — 装配层：解析参数 → 建音频源 → 每个源一个 recognizer + AsrWorker → 回调接到 renderer/writer/translator → 等 Ctrl-C。
- **`config.py`** — 所有可调参数集中在此（端点 rule、采样参数、上下文句数、prompt 模版）。
- **`models.py`** — 模型下载/文件解析、audiotee 二进制定位。

### 关键设计决策（动代码前必读）

- **只翻 final 不翻 partial**：partial 反复变，翻它浪费算力且闪烁；句子级翻译质量更好。改这个会破坏整个延迟/质量平衡。
- **two-pass 纠偏**：partial 用流式小 look-ahead（`ASR_ATT_CONTEXT=[56,6]`，560ms 延迟）求快；定稿时用 `ASR_FINAL_ATT_CONTEXT=[56,13]`（官方最高精度档）整句重解，final 和翻译输入用重解版（`asr._second_pass`）。token 一旦吐出流式端不回改——纠偏只发生在 final 这一步。有纠正时 final 事件带 `_inline_diff` 的 spans，终端在 final 行内渲染（纠掉的词灰色删除线、新词绿色，`--no-diff` 关）；diff 用 `_diff_key` 归一化比较（忽略大小写和词缘标点）——two-pass 的纠正大多只是标点/大小写微调，逐个渲染成删改对会让满屏红绿噪声，归一化后只有真正的换词才显示；文件和翻译永远用干净的纠正后文本。
- **时间戳 = 句子开始时刻**：`started_at` 在一句话第一个非空 partial 出现时记录（`asr.py`），partial 与 final 共用同一个值，所以两个回调都带它。diarize 把一句切成多段 final 时，第二段起按段首 token 的 80ms 时间戳从锚点推算各自的开始时刻（事件 5 元组的 offset 字段，None=锚点段）。renderer 里**不要**用 `datetime.now()`。
- **回调签名是贯穿三层的契约**：`on_partial(label, text, started_at)` / `on_final(label, text, started_at, diff=None)`（diff 是 two-pass 纠正的 inline spans，只有 renderer 消费；writer/translator 永远拿干净文本）；翻译侧是 `Translator.submit(label, text, started_at)` → `on_translation(label, src, zh, started_at)`（started_at 透传，输出端用它把译文挂回原文——翻译落后时 ZH 行会插在后续 EN 行之后）。改签名必须同步改 asr/translate → cli(`handle_*`) → render，否则运行时 TypeError（import 检查抓不到）。
- **both 模式 = 一份共享 Recognizer 权重 + 每源一个 OnlineStream/AsrWorker，共享一个 translator/renderer/writer**。每源用 `label`("me"/"them") 区分，label 用作 `_partials` 字典 key；终端从不显示它，文件也**只在多源（both）时**写 `[me]`/`[them]` 前缀（`FileWriter(show_label=len(sources)>1)`）——单源时 label 每行都一样，纯噪声，不写。
- **push 式 stepper 是手工移植的**：mlx-audio 只有 pull 式接口（整段音频进 generator 出），`asr._StreamingEncoder` 把它的 `stream_encode` 簿记逐行改写成 push 式。改这块前先读 mlx-audio 的 `stt/models/nemotron_asr/streaming.py` 原文对照。

## 非显而易见的约束（都是踩过的坑）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [six-ddc/livecaption](https://github.com/six-ddc/livecaption) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
