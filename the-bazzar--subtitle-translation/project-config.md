---
trigger: always_on
description: 本文件是项目的唯一权威文档。代码路径、行为、配置以当前仓库为准；不要引用历史脚本或本地视频项目路径。
---

# AGENTS.md

本文件是项目的唯一权威文档。代码路径、行为、配置以当前仓库为准；不要引用历史脚本或本地视频项目路径。

## Overview

`download -> whisper(json) -> beautify(json words) -> glossary -> translate -> split -> proofread -> ass -> burn`

Windows 主机必须使用 PowerShell 7，旧版 Windows PowerShell 5.x 会导致 `.ps1` 脚本报错。升级命令：

```powershell
winget install Microsoft.PowerShell
```

项目使用本地工具完成下载、语音识别、时间轴处理和硬压，使用远程 LLM API 完成 glossary、翻译、分割和校对。主字幕入口是 WhisperX `.json`，SRT 不再作为输入缓存。

## Repository Layout

所有运行脚本位于仓库根目录：

```text
├── pipeline.ps1              # Windows: download -> whisper -> translate_srt.py -> ffmpeg-burn
├── pipeline.sh               # Linux/WSL: 同流程
├── download.ps1              # Windows: yt-dlp 下载视频和元数据
├── download.sh               # Linux/WSL: yt-dlp 下载视频和元数据
├── whisper.ps1               # Windows: WhisperX 生成词级 JSON
├── whisper.sh                # Linux/WSL: WhisperX 生成词级 JSON
├── translate_srt.py          # JSON 美化 + glossary + 翻译/分割/校对 + SRT/ASS 导出
├── ffmpeg-burn.ps1           # Windows: ffmpeg ASS 硬压
├── ffmpeg-burn.sh            # Linux/WSL: ffmpeg ASS 硬压
├── mpv-burn.ps1              # Windows: mpv 硬压备选
├── mpv-burn.sh               # Linux/WSL: mpv 硬压备选
├── batch.ps1                 # Windows: 多 URL 批处理
├── batch.py                  # Linux/WSL: 多 URL 批处理
├── setup.ps1                 # Windows: 安装依赖
├── setup.sh                  # Linux/WSL: 安装依赖
├── .env.ps1                  # PowerShell 读取 .env 的共享模块
├── template.ass              # ASS 模板；保留历史 Style: zh / bi-en / bi-zh
├── .env.example              # 环境变量模板
├── providers.example.json    # LLM provider 配置模板
├── glossary_prompt.example.md
├── translate_prompt.example.md
├── proofread_prompt.example.md
├── split_prompt.example.md
├── AGENTS.md
├── README.md
└── .agents/skills/
    ├── beautify/SKILL.md
    ├── download/SKILL.md
    ├── knowledge/SKILL.md
    ├── translate/SKILL.md
    └── whisper/SKILL.md
```

本地文件 `.env`、`providers.json`、`cookies.txt`、`glossary_prompt.md`、`translate_prompt.md`、`proofread_prompt.md`、`split_prompt.md` 和生成产物均不应提交。

## Pipeline Flow

### Windows `pipeline.ps1`

1. `download.ps1` 下载视频、封面、`.info.json`、`.description`、`.tags.txt`
2. `whisper.ps1` 调用 WhisperX，只输出 `<base>.json`
3. `translate_srt.py --only-beautify` 美化 JSON 中的 word 时间轴，输出 `<base>.beautified.json`
4. `translate_srt.py --only-glossary` 在翻译前生成或复用 `glossary.md`
5. `translate_srt.py` 整句翻译、AI 分割、词级对轴、split event 校对，输出最终字幕
6. `ffmpeg-burn.ps1` 可选硬压双语 ASS 到 `burned.mkv`

### Linux/WSL `pipeline.sh`

流程与 Windows 对齐，使用 `download.sh`、`whisper.sh`、`translate_srt.py`、`ffmpeg-burn.sh`。两个 pipeline 都实时透传各步骤输出。

### Output Chain

```text
video -> json -> beautified.json -> glossary.md
      -> split.<source>.srt / split.<target>.srt
      -> <source>.proofread.ass / <target>.ass / <source>-<target>.ass
      -> burned.mkv
```

默认 `.env.example` 设置 `PIPELINE_SKIP_BURN=1`，推荐先人工校对字幕，再决定是否硬压。

## Step Behavior

### download

- 输出目录名和视频基名相同，视频路径形如 `<video_dir>/<video_dir>.<ext>`
- 同步保存 `.png` 封面、`.info.json` 元数据、`.description` 简介、`.tags.txt` 标签
- SponsorBlock 移除 `sponsor,selfpromo`
- `cookies.txt` 通过相对路径引用，必须在仓库根目录运行脚本
- Windows 文件夹名会做 Unicode 标点和非法字符清理，避免引号、破折号等导致跨 Windows/WSL 路径乱码

### whisper

- 已存在 `<base>.json` 时跳过
- 视频先转为 mono 16kHz WAV，再调用 WhisperX
- WhisperX 参数固定为 `--output_format json`
- `.info.json` 中的 `language` 会用于 WhisperX `--language`；缺省回退 `en`
- 输出 JSON 的 `segments[].words[]` 是后续分割对轴的唯一词源

### beautify

- 已合并到 `translate_srt.py`
- 输入 `.json`，输出 `.beautified.json`，不覆盖原始 JSON
- 同步导出 `<base>.scenes.json` 和 `<base>.scenechange.txt`；txt 每行一个秒级场景切换点
- 对每个 word 做场景吸附和边界修复，再用首尾有效 word 回写 segment 起止时间
- 入点吸附到前一个场景切换，出点吸附到下一个场景切换前 `end_offset_frames`
- 只补足最短时长，不再用最大时长截断整句；长句交给 split 阶段

### glossary

- 已合并到 `translate_srt.py`
- 位于 beautify 之后、translate 之前
- 如果 `glossary.md` 已存在且非空，直接复用，不重新总结
- 读取 transcript、`.description`、`.tags.txt`、`.info.json`
- 本地脚本会把 YouTube 原视频元信息前置写入 `glossary.md`，包括标题、作者、上传时间、原简介和标签；这部分不交给远端 LLM 合成
- 配置 `TAVILY_API_KEY` 时联网搜索，未配置时离线总结
- 需要 `TRANSLATE_PROVIDER` 和对应 API key
- `glossary_prompt.md` 仅允许微调 glossary 内容策略，输出格式规则由 `translate_srt.py` 内置 `_GLOSSARY_FORMAT` 强制追加

### translate / split / proofread

- 输入 `.json` 或 `.beautified.json`
- `.beautified.json` 是主缓存，会保存 `translation`、`proofread_text`、`split_events`
- 顺序固定为：整句翻译 -> AI 分割 -> 词级对轴 -> split event 校对
- 翻译使用整句 segment，避免先分割导致上下文破碎
- 分割使用未校对源语言文本匹配 WhisperX words，校对发生在 split event 上
- 分割请求默认附带前后各 1 条 `context_before` / `context_after`，只供远端理解语义和节奏；远端必须只返回 pending item 本身
- `split_status` 明确记录分割缓存状态：`ok`=有效分割，`fallback`=AI 分割失败后整句回退且可重试，`unsplit`=低于阈值或合法保留整句；`split_reason` 是枚举原因码，`split_reason_detail` 是具体诊断文本
- 默认 ASS 模板按 1080p 双语观看调校：`bi-zh` / `bg-bi-zh` 字号 68，`bi-en` / `bg-bi-en` 字号 44；默认 AI 分割阈值是源文超过 72 字符或 3.8 秒
- 翻译、分割、校对的 user prompt 都是 JSON object，顶层包含 `items` array；glossary 和 description 的 user prompt 也是 JSON object；远端 LLM 必须只返回 JSON
- 翻译、分割、校对返回严格 JSON object，顶层 `items` array 使用 `id` 和源/目标 ISO 639 语言代码 key，例如 `id`, `en`, `zh`
- 语言代码 key 由 `${SOURCE_LANG_CODE}` / `${TARGET_LANG_CODE}` 注入；本地解析只匹配这些 ISO code，不匹配完整语言名称或 `source` / `target`
- 对轴时只用源语言 split 的首尾 token 匹配 `words[]`；匹配失败则整句回退到 beautified 时间轴，禁止本地强切
- token normalize 会忽略词内 dash/hyphen，例如 `non-existent` 与 `nonexistent` 可匹配；带空格的 dash 仍作为分隔
- `--no-split` 只跳过 AI 分割，仍会输出 SRT/ASS

输出命名：

```text
<base>.split.<source>.srt
<base>.split.<target>.srt
<base>.<source>.proofread.ass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Bazzar/Subtitle-translation](https://github.com/The-Bazzar/Subtitle-translation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
