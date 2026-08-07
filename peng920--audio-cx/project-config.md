---
trigger: always_on
description: 本目录(`/data/github/audio-cx`,WSL Ubuntu)是一个**视频角色语音提取流水线**,目标是从影视视频中提取特定角色的语音,产出 **TTS 训练数据**(分段 wav + JSONL 清单)。
---

# Agent 环境说明 — 音频角色提取流水线

本目录(`/data/github/audio-cx`,WSL Ubuntu)是一个**视频角色语音提取流水线**,目标是从影视视频中提取特定角色的语音,产出 **TTS 训练数据**(分段 wav + JSONL 清单)。

## 项目目标

采集并整理某个特定角色的语音素材,用于 **Qwen3-TTS** 模型微调训练。最终输出是符合 Qwen3-TTS 训练格式的音文对数据。

## 执行环境(重要)

### 运行在 WSL2 而非 Windows

所有 Python 脚本在 **WSL2 Ubuntu 的 conda 环境 `audio_extract`** 中运行,**不在 Windows 的 .venv 里**。原因:Windows 下 torch CUDA 下载受阻、torchcodec DLL 装不上;WSL2 有现成的 torch/torchaudio 2.11+cu130 且 GPU 直通正常。

**调用方式(WSL 内):**
```bash
cd /data/github/audio-cx
/root/miniconda3/envs/audio_extract/bin/python scripts/cli.py process -v 01
```
- Python 解释器:`/root/miniconda3/envs/audio_extract/bin/python`(Python 3.13)
- conda 环境 `audio_extract`(torch/torchaudio 2.11 cu130,RTX 4070 12GB)
- 从 Windows 侧可通过 `wsl -d Ubuntu -- bash -lc 'cd /data/github/audio-cx && ...'` 调用

### 中文路径的坑(必读)

**命令行传中文路径会编码错乱。** 所有中文路径**必须在 Python 脚本内部用变量/`__file__` 拼接**,绝不能通过 `wsl bash -c "中文"` 或命令行参数传递。脚本已用 `__file__` 自动定位 config,无需传路径参数。

## 流水线架构(11 阶段)

```
s1 抽音轨(ffmpeg) → s1b 去伴奏(Demucs,GPU) → s1c 去噪(DeepFilterNet3,GPU)
→ s2 转写+情感检测(SenseVoiceSmall,GPU,120s块级仅参考) → s3 说话人分离(pyannote community-1,GPU)
→ s4 声纹比对锁定目标(Resemblyzer) + diarization边界选段 + merge + 逐段短段转写 → s4b 导出审核文件
→ [人工审核] → s5 切片输出 → s6 响度匹配 → s7 导出训练集 → s8 质量校验
```

| 阶段 | 脚本 | 输入 | 输出 | 备注 |
|------|------|------|------|------|
| s1 | pipeline/stages/s1_extract.py | 视频 | work/\<video\>/audio.wav | ffmpeg,16kHz单声道 |
| s1b | pipeline/stages/s1b_separate.py | audio.wav | work/\<video\>/vocals.wav | Demucs htdemucs,config 开关 |
| s1c | pipeline/stages/s1c_denoise.py | vocals.wav | vocals.wav(原地覆盖)+denoise_done.flag | DeepFilterNet 去噪去混响,config 开关 |
| s2 | pipeline/stages/s2_sensevoice.py | vocals.wav(优先) | transcript.json | SenseVoiceSmall,带情感+音频事件检测 |
| s3 | pipeline/stages/s3_diarize.py | vocals.wav(优先) | diarization.json | pyannote community-1,用 .speaker_diarization 取结果 |
| s4 | pipeline/stages/s4_match.py | diarization.json + reference_voice.wav + vocals.wav | target_segments.json | Resemblyzer 锁定说话人 + **diarization 边界选段 + merge + 逐段短段转写**(见决策 5.6) |
| s4b | pipeline/stages/s4b_review.py | target_segments.json + **vocals.wav** | review.wav+review.csv **或** review_full.wav+review_audacity.txt | 审核导出,`--format wav_csv/audacity`,**之后是人工卡点** |
| — | (人工) | review.csv | review.done 标记 | 听review.wav,编辑keep/text |
| s5 | pipeline/stages/s5_slice.py | review.csv(优先)或 target_segments.json + vocals.wav | output/\<video\>/clips/ + manifest.jsonl | 读review.csv时按keep过滤,manifest 含 emotion/events 字段 |
| s6 | pipeline/stages/s6_loudness.py | clips/*.wav | 原地覆盖 + loudness_done.flag | pyloudnorm,ITU-R BS.1770-4,先loudness再限peak |
| s7 | pipeline/stages/s7_export.py | clips + manifest | datasets/{qwen,fish}/ | 插件式导出器(exporters/),自动重采样 |
| s8 | pipeline/stages/s8_validate.py | clips + manifest | quality_report.json + quality.csv | 静音/削波/文本异常/重复检测 |
| 调度 | cli.py / pipeline.py | config.yaml | 串联全流程 + 断点续传 | Typer CLI(推荐) / 旧 argparse(兼容) |

### ASR 引擎切换

默认使用 SenseVoiceSmall。如需切回 whisper:
```yaml
# config.yaml
asr:
  engine: "whisper"   # 改回 whisper
```
两种引擎的输出格式兼容,下游阶段无需改动。

## 已验证的关键技术决策(不要偏离)

这些是实测踩坑后确定的写法,改动前务必理解原因:

### 1. pyannote 用 community-1 模型(不是 3.1)
- `speaker-diarization-3.1` 要拼凑 3 个独立模型且缺 PLDA 文件,**不要用**。
- `speaker-diarization-community-1` 自包含(embedding/segmentation/plda 全在一个 repo),从 ModelScope 下载。
- 路径:`models/pyannote/speaker-diarization-community-1/config.yaml`

### 2. 音频必须用 soundfile 预加载成 waveform dict
pyannote 4.x 默认用 torchcodec 读音频,但统一用 soundfile 预加载更稳(见 `utils.load_audio_as_dict`):
```python
audio_dict = load_audio_as_dict(audio_path)  # {'waveform': (1,time) tensor, 'sample_rate': int}
out = pipeline(audio_dict)
```

### 3. 分离结果用 `.speaker_diarization` 属性
pyannote 4.x 返回 `DiarizeOutput`,**不是** `Annotation`:
```python
out = pipeline(audio_dict)
annotation = out.speaker_diarization  # ← 这个才有 itertracks
```
**不要用** `pipeline.to_annotation(out)`(会报错)。

### 4. 强制离线(全程无 HuggingFace)
HuggingFace 在国内被墙。所有 pyannote/whisper 模型从 **ModelScope** 下载,`utils.py` 顶部已设:
```python
os.environ.setdefault("HF_HUB_OFFLINE", "1")
os.environ.setdefault("TRANSFORMERS_OFFLINE", "1")
```
whisper large-v3 的 model.bin(2.87GB)需手动下载(SSL 大文件不稳),放进 `models/faster-whisper-large-v3/`。

### 5. 情绪声保留策略
TTS 训练需要非语言情绪声(嗯/啊/笑/叹气等)。s5 不过滤无文本片段,而是标 `[non-verbal]` 占位,加 `type: "nonverbal"` 字段,供后续人工标注成 Qwen3-TTS 音频标签(如 `(laughter)` `(sigh)`)。

### 5.5 人工审核卡点(s4b)
声纹比对不 100% 准(相似度接近时可能误判)。s4b 在 s4 后导出 `review.wav`(拼接试听,段间0.5s静音)+ `review.csv`(清单,可编辑 keep/text)。**pipeline 在 s4b 后阻塞**,检测 `review.done` 标记文件存在才继续 s5。
- s5 优先读 review.csv(按 keep 过滤、用修正后的 text),不存在才回退 target_segments.json
- **跳过审核**:两种方式 —— ① `config review.required: false`(pipeline 不阻塞,直通 s5-s8,批量处理用);② 或直接创建 `review.done` 标记(按原始 target_segments 全切)
- review.wav 拼接注意:所有切片必须统一 `-ar 16000 -ac 1` 规格才能用 concat demuxer,否则只拼进第一段
- **review.wav 用 vocals.wav**(去伴奏人声),与最终训练数据同源,无 BGM 干扰
- **两种审核格式**(config `review.format` 或 `--format`):
  - `wav_csv`: 拼接试听(review.wav,段间0.5s静音)+ review.csv(keep/text 可编辑)
  - `audacity`: 完整音频(review_full.wav,原时间轴)+ Audacity 标签(review_audacity.txt,每行 start\tend\ttext)
- s5 读审核文件的优先级: review.csv > review_audacity.txt > target_segments.json
- Audacity 标签解析:每行 `start\tend\ttext`,删除段=删行,所有保留行都切(无 keep 字段)

### 5.6 s4 选段 + 短段逐段转写(s4 核心逻辑,重要)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peng920/audio-cx](https://github.com/peng920/audio-cx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
