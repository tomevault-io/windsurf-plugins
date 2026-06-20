---
trigger: always_on
description: |
---


# Voice Memo Sync 🎙️

Intelligent voice/video transcription and organization system.  
智能语音/视频转录与整理系统。

---

## Quick Start / 快速开始

```bash
# Run installation script / 运行安装脚本
cd ~/.openclaw/workspace/skills/voice-memo-sync
./scripts/install.sh
```

**What it does / 安装内容:**
1. Creates data directory `memory/voice-memos/` / 创建数据目录
2. Creates config file `config/voice-memo-sync.yaml` / 创建配置文件
3. Creates Apple Notes folder "Voice Memos" / 创建 Apple Notes 文件夹
4. Checks dependencies and prompts installation / 检查依赖并提示安装

---

## When to Use / 何时使用

✅ **USE this skill when user:**
- Sends voice/audio/video files / 发送语音/音频/视频文件
- Sends YouTube/Bilibili URLs / 发送 YouTube/B站 链接
- Sends transcript text files / 发送转录文本文件
- Says "sync voice memos", "process recording", "organize this video"
- 说「同步语音备忘录」「处理录音」「整理这个视频」

❌ **DO NOT use when:**
- User just wants to play audio/video / 用户只想播放音视频
- User asks about music/podcasts without transcription needs / 询问音乐/播客但不需要转录

---

## Supported Formats / 支持格式

### ⚡ Metal GPU Acceleration (NEW)

On Apple Silicon, `whisper-cpp` provides 15-20x faster transcription:

| Audio | CPU (openai-whisper) | Metal GPU (whisper-cpp) |
|-------|---------------------|------------------------|
| 5 min | ~5 min | ~20 sec |
| 30 min | ~30 min | ~2 min |
| 60 min | ~60 min | ~4 min |

```bash
# Install for Metal acceleration (recommended)
brew install whisper-cpp
```

The skill auto-detects and uses Metal when available.

| Type / 类型 | Formats / 格式 | Processing / 处理方式 |
|-------------|----------------|----------------------|
| Voice Memos | .qta, .m4a | Apple native (QTA metadata) → Whisper fallback |
| Audio | .mp3, .wav, .aac, .flac | Whisper local transcription |
| Video | .mp4, .mov, .mkv, .webm | ffmpeg extract → Whisper |
| YouTube | URL | summarize CLI → yt-dlp fallback |
| Bilibili | URL | yt-dlp download → Whisper |
| Text | .txt, .md | Direct read, skip transcription |
| Documents | .doc, .docx | textutil convert → process |
| Structured | .json, .csv | Parse and extract text |
| iCloud | Configured paths | Scheduled sync |

---

## Processing Pipeline / 处理流程

```
Input (File/URL/Text)
        │
        ▼
┌─────────────────────────────────────┐
│     1. Source Detection            │
│     来源识别                        │
│  Voice Memo / URL / File / Text    │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│     2. Save Source Metadata        │
│     保存源信息                      │
│  → memory/voice-memos/sources/     │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│     3. Transcription               │
│     转录提取                        │
│  Priority: Apple > Text > summarize│
│           > Whisper-local > API    │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│     4. Save Raw Transcript         │
│     保存原始转录                    │
│  → memory/voice-memos/transcripts/ │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│     5. LLM Deep Processing         │
│     LLM深度整理                     │
│  • Read USER.md & MEMORY.md        │
│  • Clean up spoken language        │
│  • Extract key points & insights   │
│  • Identify TODOs & connections    │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│     6. Save Processed Result       │
│     保存处理结果                    │
│  → memory/voice-memos/processed/   │
└─────────────────┬───────────────────┘
                  │
          ┌───────┴───────┐
          ▼               ▼
┌─────────────────┐ ┌─────────────────┐
│ 7a. Apple Notes │ │ 7b. Reminders  │
│ Structured note │ │ Create TODOs   │
│ with #hashtags  │ │ 创建提醒       │
└────────┬────────┘ └────────┬───────┘
         │                   │
         └─────────┬─────────┘
                   ▼
┌─────────────────────────────────────┐
│     8. Update Index                │
│     更新索引                        │
│  → memory/voice-memos/INDEX.md     │
└─────────────────────────────────────┘
```

---

## Data Structure / 数据结构

```
memory/voice-memos/           # All data, searchable via memory_search
├── INDEX.md                  # Processing records index / 处理记录索引
├── sources/                  # Original file metadata / 原始文件元数据
│   └── YYYY-MM-DD_xxx.json
├── transcripts/              # Raw transcripts / 原始转录文本
│   └── YYYY-MM-DD_source_title.md
├── processed/                # LLM processed content / LLM处理后内容
│   └── YYYY-MM-DD_source_title.md
└── synced/                   # Sync records / 同步记录
    └── YYYY-MM-DD_source_title.json
```

---

## Apple Notes Output Format / 输出格式

The skill reads `USER.md`, `SOUL.md`, and `MEMORY.md` to provide **personalized analysis**:
- Deep insights tailored to user's research/work focus
- Connections to active projects and ongoing interests  
- Actionable recommendations based on user's decision style
- Critical thinking that challenges assumptions

处理时会读取 `USER.md`、`SOUL.md` 和 `MEMORY.md` 提供**个性化分析**：
- 结合用户研究/工作重点的深度洞察
- 与活跃项目和持续关注领域的关联
- 基于用户决策风格的行动建议
- 挑战假设的批判性思考

```
🎙️ [Auto-generated Title / 智能生成的标题]

📅 Date | ⏱️ Duration | 👤 Source
🏷️ #tag1 #tag2 #tag3


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ying-wen/voice-memo-sync](https://github.com/ying-wen/voice-memo-sync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
