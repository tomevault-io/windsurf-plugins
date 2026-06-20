---
trigger: always_on
description: Use when user provides a short video platform URL or local video/audio file and wants subtitles/AI summary, or when user asks to list their own AI Douyin historical tasks. Triggers on v.douyin.com, xhslink.com, bilibili.com, b23.tv, YouTube URLs, local .mp4/.mp3/.wav files, or task history requests.
---


# 视频转字幕与 AI 总结技能

## Overview

将短视频平台（抖音、小红书、B 站、YouTube 等）视频或本地视频/音频文件转换为字幕文本并生成 AI 摘要。

**核心流程：**
- **在线视频：** 获取视频信息 → 下载视频/直接抓字幕 → 选择字幕后端 → 生成字幕 → AI 总结
- **本地文件：** 提取音频（如需） → 选择字幕后端 → 生成字幕 → AI 总结

默认使用本地 `faster-whisper`，也支持通过环境变量切换到火山引擎 VC API。
YouTube 优先使用 `yt-dlp` 直接抓取人工字幕或自动字幕；只有没有可用字幕时，才需要下载音视频并回退到 ASR。

## When to Use

- 用户提供短视频平台链接（抖音、小红书、B 站、YouTube 等），要求提取字幕或生成总结
- 用户提供本地视频/音频文件路径，要求转字幕或生成总结
- 用户要求查看自己的 AI Douyin 历史任务、最近任务、任务列表
- 需要将视频内容转为文字

**不适用于：** 实时语音识别、直播字幕

## 外部依赖

| 依赖 | 用途 | 必需 |
| --- | --- | --- |
| **AI Douyin API Key** | 推荐的视频解析/下载代理；注册后可用免费额度，成功解析下载直链后扣 1 积分 | 仅抖音/小红书/B 站需要 |
| **TikHub API** | 可选高级/自托管方案：使用自己的 TikHub Token 直接解析 | 可选 |
| **Python 3.9+** | 运行 `faster-whisper` helper | 仅 `ASR_BACKEND=faster-whisper` 时需要 |
| **faster-whisper** | 本地语音转文字 | 仅 `ASR_BACKEND=faster-whisper` 时需要 |
| **字节跳动 VC API** | 云端语音转文字 | 仅 `ASR_BACKEND=volcengine` 时需要 |
| **FFmpeg** | 从视频提取音频 | ✅（音频文件可跳过） |
| **yt-dlp** | 下载 B 站视频；抓取 YouTube 字幕 | 仅 B 站或 YouTube 需要 |
| **jq** | 解析 AI Douyin/TikHub JSON 响应 | 在线视频模式需要 |

## 环境变量

通过环境变量读取，支持以下任意方式配置：

**方式一：.env 文件（推荐）** — 在 skill 目录下创建 `.env` 文件：

```bash
ASR_BACKEND="faster-whisper"
VIDEO_INFO_PROVIDER="ai-douyin"
AI_DOUYIN_API_BASE="https://ai-douyin.top9.cc"
AI_DOUYIN_API_KEY="your_ai_douyin_api_key"
TIKHUB_TOKEN=""

FW_MODEL_SIZE="small"
FW_DEVICE="auto"
FW_COMPUTE_TYPE=""
FW_PYTHON=""

BYTEDANCE_VC_TOKEN="your_token"
BYTEDANCE_VC_APPID="your_appid"
```

**方式二：Shell 配置** — 添加到 `~/.zshrc` 或 `~/.bashrc`：

```bash
export ASR_BACKEND="faster-whisper"
export VIDEO_INFO_PROVIDER="ai-douyin"
export AI_DOUYIN_API_BASE="https://ai-douyin.top9.cc"
export AI_DOUYIN_API_KEY="your_ai_douyin_api_key"
export TIKHUB_TOKEN=""

export FW_MODEL_SIZE="small"
export FW_DEVICE="auto"
export FW_COMPUTE_TYPE=""
export FW_PYTHON=""

export BYTEDANCE_VC_TOKEN="your_token"
export BYTEDANCE_VC_APPID="your_appid"
```

说明：
- `ASR_BACKEND`：可选，默认 `faster-whisper`
- `VIDEO_INFO_PROVIDER`：可选，默认 `ai-douyin`；可改为 `tikhub` 使用自有 TikHub Token
- `AI_DOUYIN_API_BASE` / `AI_DOUYIN_API_KEY`：推荐的视频解析代理；抖音/小红书/B 站需要；YouTube 不需要
- `TIKHUB_TOKEN`：可选高级/自托管方案；当 `VIDEO_INFO_PROVIDER=tikhub` 时需要
- `FW_MODEL_SIZE` / `FW_DEVICE` / `FW_COMPUTE_TYPE`：仅 `faster-whisper` 后端使用
- `FW_PYTHON`：可选，指定安装了 `faster-whisper` 的 Python；留空时优先使用安装 helper 创建的默认 venv，再回退系统 `python3`
- `BYTEDANCE_VC_TOKEN` / `BYTEDANCE_VC_APPID`：仅 `volcengine` 后端使用

> 安装与运行时说明见 [AI Douyin 配置指南](./docs/ai-douyin-setup.md)、[TikHub 申请指南](./docs/tikhub-setup.md)、[faster-whisper 安装指南](./docs/faster-whisper-setup.md) 和 [火山引擎开通指南](./docs/bytedance-vc-setup.md)

## 执行步骤

### 步骤 0：判断输入类型

根据用户输入判断处理模式：

- **在线视频模式**：输入为 URL
  - **抖音/TikTok**：`douyin.com`、`v.douyin.com`、`tiktok.com`
  - **小红书**：`xiaohongshu.com`、`xhslink.com`
  - **B 站**：`bilibili.com`、`b23.tv`
  - **YouTube**：`youtube.com`、`youtu.be`
- **本地文件模式**：输入为本地文件路径
  - 本地**视频**文件（`.mp4`、`.mov`、`.avi`、`.mkv` 等）→ 从步骤 3（提取音频）开始
  - 本地**音频**文件（`.mp3`、`.wav`、`.m4a`、`.flac` 等）→ 跳过步骤 3，直接从步骤 4（转写）开始

### 步骤 0.5：读取后端配置

先读取 `ASR_BACKEND`，未配置时默认使用 `faster-whisper`：

```bash
SKILL_DIR="${SKILL_DIR:-$HOME/.codex/skills/video-to-subtitle-summary}"
[ -d "$SKILL_DIR" ] || SKILL_DIR="$HOME/.claude/skills/video-to-subtitle-summary"
ENV_FILE="$SKILL_DIR/.env"

read_env() {
  local key="$1"
  if [ -f "$ENV_FILE" ]; then
    grep "^${key}=" "$ENV_FILE" | head -1 | cut -d'=' -f2- | tr -d '"' | tr -d "'"
  else
    printenv "$key"
  fi
}

ASR_BACKEND="$(read_env ASR_BACKEND)"
[ -z "$ASR_BACKEND" ] && ASR_BACKEND="faster-whisper"

echo "ASR_BACKEND=$ASR_BACKEND"
```

支持值：
- `faster-whisper`
- `volcengine`

### 步骤 0.6：环境检查（必须首先执行）

在开始任何处理之前，先检查当前模式和当前字幕后端需要的依赖。

```bash
SKILL_DIR="${SKILL_DIR:-$HOME/.codex/skills/video-to-subtitle-summary}"
[ -d "$SKILL_DIR" ] || SKILL_DIR="$HOME/.claude/skills/video-to-subtitle-summary"
ENV_FILE="$SKILL_DIR/.env"

read_env() {
  local key="$1"
  if [ -f "$ENV_FILE" ]; then
    grep "^${key}=" "$ENV_FILE" | head -1 | cut -d'=' -f2- | tr -d '"' | tr -d "'"
  else
    printenv "$key"
  fi
}

ASR_BACKEND="$(read_env ASR_BACKEND)"
[ -z "$ASR_BACKEND" ] && ASR_BACKEND="faster-whisper"
VIDEO_INFO_PROVIDER="$(read_env VIDEO_INFO_PROVIDER)"
[ -z "$VIDEO_INFO_PROVIDER" ] && VIDEO_INFO_PROVIDER="ai-douyin"
AI_DOUYIN_API_BASE="$(read_env AI_DOUYIN_API_BASE)"
[ -z "$AI_DOUYIN_API_BASE" ] && AI_DOUYIN_API_BASE="https://ai-douyin.top9.cc"
AI_DOUYIN_API_KEY="$(read_env AI_DOUYIN_API_KEY)"
TIKHUB_TOKEN="$(read_env TIKHUB_TOKEN)"
BYTEDANCE_VC_TOKEN="$(read_env BYTEDANCE_VC_TOKEN)"
BYTEDANCE_VC_APPID="$(read_env BYTEDANCE_VC_APPID)"
FW_PYTHON="$(read_env FW_PYTHON)"
[ -z "$FW_PYTHON" ] && [ -x "$HOME/.cache/video-to-subtitle-summary/faster-whisper-venv/bin/python" ] && FW_PYTHON="$HOME/.cache/video-to-subtitle-summary/faster-whisper-venv/bin/python"
[ -z "$FW_PYTHON" ] && FW_PYTHON="python3"

MISSING=""

if [ "{INPUT_MODE}" = "url" ]; then
  if [ "{PLATFORM}" = "douyin" ] || [ "{PLATFORM}" = "xiaohongshu" ] || [ "{PLATFORM}" = "bilibili" ]; then
    if [ "$VIDEO_INFO_PROVIDER" = "ai-douyin" ]; then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imlewc/video-to-subtitle-summary-skill](https://github.com/imlewc/video-to-subtitle-summary-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
