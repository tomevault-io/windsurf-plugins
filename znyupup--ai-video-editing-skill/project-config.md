---
trigger: always_on
description: AI Agent自动剪辑旅行Vlog的完整工作流。从原始素材到成品视频，系统级只需ffmpeg，其余在Python venv内完成。by nyx研究所 (GitHub @znyupup · B站/小红书 @nyx研究所)
---


# AI Agent 自动剪辑旅行 Vlog

**Author:** nyx研究所 · [GitHub](https://github.com/znyupup) · [B站 @nyx研究所](https://space.bilibili.com/4330525) · 小红书 @nyx研究所 · [X @znyupup_music](https://x.com/znyupup_music)

> 把一堆手机拍的旅行素材，用AI自动剪成一个完整vlog。
> 最小依赖：ffmpeg + Python(whisper或FunASR+Pillow) + 视觉API。系统级只装ffmpeg，其余pip装。

## 触发条件

- 用户有一批旅行/日常视频素材，想自动剪成vlog
- 用户说"帮我剪个视频"、"自动剪辑"、"vlog剪辑"

## 前置要求

### 1. 系统工具

| 工具 | 用途 | 安装 |
|------|------|------|
| ffmpeg | 视频裁剪/编码/拼接/抽帧/音量检测 | `brew install ffmpeg` (macOS) / `apt install ffmpeg` (Linux) |
| Python 3.9+ | 脚本胶水 | macOS/Linux 系统自带 |

### 2. Python依赖（先检测，已有则跳过）

**先检测系统是否已安装所需包，不要无脑创建 venv：**

```bash
# 检测 whisper
python3 -c "import whisper; print('✅ whisper已安装:', whisper.__file__)"

# 检测 Pillow
python3 -c "from PIL import Image; print('✅ Pillow已安装')"
```

**⚠ 重要：不要用 `2>/dev/null` 吃掉错误！要看到实际报错才能判断是真没装还是PATH问题。**

**⚠ macOS 注意：** 检查系统已安装的包再决定是否需要 venv。如果系统 python3 已经能 `import whisper` 和 `from PIL import Image`，直接用就行，不要盲目创建 venv 导致找不到已有的包。

**仅在检测不通过时才安装：**

```bash
# 方案A: 直接装到用户环境（推荐）
pip install openai-whisper Pillow

# 方案B: 如果用户环境有冲突，再用 venv
python3 -m venv .venv && source .venv/bin/activate
pip install openai-whisper Pillow
```

**⚠ 不要每个项目都新建 venv 重装一遍！whisper模型文件1.4GB，pip install也要几分钟。**

**⚠ ASR引擎二选一：Whisper 或 FunASR**

本工作流支持两个ASR引擎，优先用 whisper，whisper 装不上就用 FunASR：

| | Whisper (OpenAI) | FunASR (阿里达摩院) |
|---|---|---|
| 安装 | `pip install openai-whisper` | `pip install funasr modelscope torchaudio` |
| 模型 | medium (1.4GB, 从GitHub下载) | paraformer-zh (1.05GB, 从ModelScope下载) |
| 附加模型 | 无 | VAD模型 + 标点模型 (首次运行自动下载, 共~50MB) |
| 中文效果 | 好 | 好（与whisper medium相当，细节略多） |
| 速度 | 14s音频 ≈ 5-10s (CPU) | 14s音频 ≈ 1.9s (CPU, RTF=0.132) ⚡ |
| 时间戳粒度 | 句级别 (每句话一个start/end) | 字级别 (每个字一个时间戳) |
| 模型下载源 | GitHub (国内不稳定，容易失败) | ModelScope (国内极快，~15MB/s) |
| 隐性依赖 | torch | torch + torchaudio (⚠ 必须额外装) |

**检测已安装的ASR引擎：**
```bash
python3 -c "import whisper; print('✅ whisper可用')" 2>/dev/null \
  || python3 -c "from funasr import AutoModel; print('✅ funasr可用')" 2>/dev/null \
  || echo "❌ 没有可用的ASR引擎，需要安装一个"
```

**⚠ FunASR 安装注意事项：**
- 必须装 torchaudio！`pip install funasr modelscope` 不会自动装 torchaudio，import 时会报 `No module named 'torchaudio'`
- 正确安装命令：`pip install funasr modelscope torchaudio`
- 首次运行会自动从 ModelScope 下载3个模型（ASR + VAD + 标点），总共约1.1GB，国内速度快
- 模型缓存在 `~/.cache/modelscope/hub/models/iic/` 目录下
- import 时会有 urllib3 的 SSL 警告（NotOpenSSLWarning），可忽略，不影响功能

**标题卡方案自动选择：**
```bash
ffmpeg -filters 2>&1 | grep drawtext
# 有drawtext → 直接用ffmpeg，不需要Pillow
# 没有 → 用Pillow生成透明PNG再overlay（macOS brew ffmpeg通常没编freetype）
```

**可选（参考分析阶段用）：**
```bash
which yt-dlp && echo "✅ yt-dlp已安装" || pip install yt-dlp
python3 -c "import scenedetect" 2>/dev/null && echo "✅ scenedetect已安装" || pip install scenedetect
```

### 3. 视觉理解模型（API）

需要一个能理解图像内容的视觉模型API，用于分析素材画面。

**要求：**
- 支持 OpenAI Chat Completions 格式（messages + image_url）
- 支持 base64 图片输入（本地素材抽帧后编码上传）
- 中文理解能力（需要描述画面内容、标注镜头类型等）

**推荐模型：**

| 模型 | 费用 | 说明 |
|------|------|------|
| 智谱 GLM-4.6V-Flash | 免费 | 注册 https://open.bigmodel.cn 即用，中文理解好 |
| GPT-4o | 付费 | 效果最好 |
| Qwen-VL | 付费 | 阿里云，中文好 |

**调用示例：**

```python
import base64, json, urllib.request

API_URL = 'YOUR_VISION_API_ENDPOINT'   # 替换为你的视觉模型端点
API_KEY='***'               # 替换为你的API Key
MODEL = 'YOUR_MODEL_NAME'             # 替换为你的模型名

with open('frame.jpg', 'rb') as f:
    img_b64 = base64.b64encode(f.read()).decode()

payload = {
    'model': MODEL,
    'messages': [{'role': 'user', 'content': [
        {'type': 'image_url', 'image_url': {'url': f'data:image/jpeg;base64,{img_b64}'}},
        {'type': 'text', 'text': '简洁描述画面内容，标注：镜头类型(远/全/中/近/特写)、拍摄手法(固定/手持/移动)、画面氛围。格式：内容|类型|手法|氛围'}
    ]}],
    'max_tokens': 200
}

req = urllib.request.Request(API_URL, json.dumps(payload).encode(),
    {'Content-Type': 'application/json', 'Authorization': f'Bearer {API_KEY}'})
with urllib.request.urlopen(req, timeout=30) as resp:
    result = json.loads(resp.read())
    print(result['choices'][0]['message']['content'])
```

**⚠ 视觉API调用注意事项：**
- 输入格式：ffmpeg抽帧为JPEG → base64编码 → `data:image/jpeg;base64,{b64}` 放入 image_url
- 抽帧尺寸：不需要原始分辨率，缩到 720p 即可（节省上传带宽和token）
- 端点区分：有些平台视觉模型和文本模型端点不同，确认用对
- 限流：高峰时段容易被限流，加 retry + sleep

### 不需要的（踩过的坑）

- ❌ 剪映/CapCut — v10.4+ 项目文件加密，无法程序化操作
- ❌ moviepy — ffmpeg命令行已够用，多一层抽象反而不灵活
- ❌ capcut-cli (GitHub) — 基于明文JSON，对加密后的新版无效
- ❌ ImageMagick — Pillow已够用，不需要再装一个图像工具

## 工作流（7个阶段）

### 阶段1: 素材盘点

目标：了解你有什么素材。

```bash
# 批量获取素材信息
for f in footage/*.{MOV,mp4,MP4}; do
  echo "=== $f ==="
  ffprobe -v quiet -print_format json -show_format -show_streams "$f" \
    | python3 -c "import json,sys; d=json.load(sys.stdin); \
      s=d['streams'][0]; f=d['format']; \
      print(f\"  时长: {float(f['duration']):.1f}s\"); \
      print(f\"  分辨率: {s['width']}x{s['height']}\"); \
      print(f\"  编码: {s['codec_name']}\")"
done
```

输出一份素材清单：数量、总时长、分辨率分布、拍摄时间范围。

### 阶段2: 参考研究（可选但强烈建议）

目标：建立"好vlog长什么样"的认知。不做这步直接剪，效果会很差。

**步骤：**
1. 找2-3个同类型优质vlog（B站/YouTube）
2. `yt-dlp` 下载720p视频 + 音频
3. `whisper` 转录旁白（提取叙事结构）
4. `scenedetect` 检测镜头切换（统计节奏）
5. `ffmpeg` 抽关键帧 + 视觉API分析（理解画面构成）

**已验证的规律：**

- 镜头节奏: 平均3-4秒/镜头是黄金节奏
- 内容配比: 美食40-45% + 风景30% + 人物15% + 日常15%
- 镜头类型: 近景/中景为主(60-65%)，全景/远景穿插(35-40%)
- 叙事结构: 精彩片头 → 出发/到达 → 按地点串联 → 情感升华收尾

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [znyupup/ai-video-editing-skill](https://github.com/znyupup/ai-video-editing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
