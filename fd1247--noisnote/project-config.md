---
trigger: always_on
description: This file provides guidance to Codex when working in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working in this repository.

## 项目概述

音频转录与总结工具是一个面向 Windows 的 PySide6 / Qt Widgets 桌面应用。用于录制系统音频、录制麦克风或导入本地音视频文件，将输入保存为历史记录，再通过本地 ASR 模型转录文字，并可调用 OpenAI 兼容的 LLM API 生成总结。

```text
创建录音或导入音视频 -> 保存历史记录 -> ASR 转录（含逐句时间轴） -> 可选 LLM 总结 -> 音频回放 / 导出 -> 查看结果
```

## 技术栈

|   层级   |                        技术                        |
| -------- | -------------------------------------------------- |
| GUI      | PySide6 / Qt Widgets                               |
| 系统音频 | SoundCard (WASAPI Loopback)                        |
| 音频回放 | QMediaPlayer / QAudioOutput                        |
| ASR 推理 | Qwen3-ASR GGUF（ONNX + llama.cpp），本地模型推理     |
| 时间戳   | Qwen3-ForceAligner GGUF（ONNX + llama.cpp）         |
| LLM 总结 | OpenAI 兼容 API / Anthropic 兼容 API                |
| 音频处理 | ffmpeg / ffprobe                                   |
| 模型下载 | ModelScope（优先）+ GitHub（备用）                  |
| 日志     | JSON Lines，写入 `~/Documents/NoisNote/logs/` |
| 测试     | pytest，Qt 测试使用 `QT_QPA_PLATFORM=offscreen`     |
| 打包     | PyInstaller（onedir 模式）                          |

## 常用命令

```bash
# 启动应用
python main.py

# 运行常规单元测试
python -m pytest tests/test_qt_history.py tests/test_qt_models_gguf.py tests/test_qt_model_workers.py -q

# 运行扩展单元测试（含回放、时间轴、导出、对话框、预处理、转录子进程）
python -m pytest tests/test_qt_main_window_p0.py tests/test_qt_main_window_ch07.py tests/test_qt_dialogs.py tests/test_qt_history_widgets.py tests/test_timestamp_alignment_app.py tests/test_audio_preprocess.py tests/test_transcription_worker.py -q

# 运行 WASAPI 录音手动测试（需要 Windows 音频设备）
python tests/test_wasapi_record.py

# 安装依赖
pip install -r requirements.txt
```

## 项目结构

```text
main.py                                  # 应用入口
src/
  __init__.py                           # 包版本信息
  app/                                  # 应用核心
    application.py                      # QApplication 初始化、日志、异常钩子
    main_window.py                      # 主窗口骨架、UI 布局、Mixin 组装
    config.py                           # 默认配置、配置读写、模型清单
    version.py                          # 语义化版本号
    update.py                           # GitHub Releases 更新检查
    diagnostics.py                      # ASR 运行时诊断工具
  audio/                                # 音频录制模块
    recorder.py                         # AudioRecorder 录制引擎
    device_manager.py                   # DeviceManager WASAPI 设备管理
    types.py                            # CaptureMode、CaptureSettings 等数据类
    preprocess.py                       # 音视频探测、格式转换
  asr/                                  # ASR 转录引擎
    engine.py                           # TranscriptionEngine 高层封装
    runtime.py                          # Qwen3AsrGgufRuntime vendor 封装
    types.py                            # 数据模型（含 TimelineSegment/Token、进度、设备解析）
    timestamps.py                       # 时间轴生成、HTML/SRT 导出、时间格式化
    worker_process.py                   # ASR 子进程入口，通过 JSON Lines stdout 通信
    utils.py                            # 转录文本清理
  llm/                                  # LLM 总结服务
    summarizer.py                       # Summarizer（system/user 角色分离的 prompt 格式）
  history/                              # 历史记录管理
    service.py                          # HistoryService CRUD、时间轴读写、ASR 元数据
    storage.py                          # 文件存储和元数据管理
    types.py                            # HistoryRecord、HistoryStatus
  model_registry/                       # 模型下载管理
    service.py                          # ModelService 模型目录与校验
    downloader.py                       # ModelDownloadManager 任务生命周期
    worker.py                           # ModelDownloadWorker 下载线程
    download.py                         # 下载源选择、文件下载、解压、校验
    types.py                            # ModelStatus、ModelCatalogEntry 等
  handlers/                             # MainWindow Mixin（每个文件对应一个功能域）
    media_import.py                     # 文件导入和拖拽
    remote_import.py                    # 远程公开视频链接导入（yt-dlp、字幕优先、音频降级）
    recording.py                        # 录音设备和流程
    processing.py                       # 处理状态和结果保存
    transcription.py                    # ASR 转录和重新转录
    summary.py                          # LLM 总结
    settings.py                         # 设置导航和配置保存
    history_view.py                     # 历史列表搜索、筛选、选择、右键菜单、详情加载
    timeline_view.py                    # 逐句时间轴展示、回放位置高亮、复制/格式化
    playback.py                         # 音频回放、进度条、倍速、快捷键
    export.py                           # 导出入口（转录/timeline/总结 -> txt/srt/markdown）
  workers/                              # 后台线程
    transcription.py                    # TranscriptionWorker（子进程 ASR）
    summary.py                          # SummaryWorker
    preprocess.py                       # AudioPreprocessWorker
    remote_import.py                    # RemoteProbeWorker / RemoteImportWorker
  remote_import/                        # 远程链接导入服务
    ytdlp_client.py                     # yt-dlp 适配、cookies、站点错误归类
    service.py                          # 字幕下载/解析，失败后音频降级
    subtitles.py                        # SRT/VTT/常见 JSON 字幕转换
    errors.py                           # 远程导入错误类型与用户提示
    types.py                            # RemoteMediaInfo / RemoteImportOptions 等数据结构
  ui/                                   # Qt 界面组件
    styles.py                           # 全局 Qt 样式表（含回放栏、时间轴、设置、对话框等样式）

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fd1247/NoisNote](https://github.com/fd1247/NoisNote) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
