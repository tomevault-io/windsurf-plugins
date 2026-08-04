---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

MixCut Windows 版 —— macOS 原生应用 [MixCut](https://github.com/RoshanGH/mixed_cut) 的 Windows 移植版，
功能完整对齐。面向广告投放团队的 AI 视频混剪工具：导入广告素材，AI 按语义切分镜头并标注类型，
再智能排列组合生成多条差异化混剪广告。

技术栈：**C# + WPF + .NET 8**（`net8.0-windows`），目标系统 Windows 10 及以上（x64）。

---

## 🥇 最高原则：商业化 ToC 软件标准（一切环节的总闸 ⚠️⚠️⚠️）

**MixCut Windows 不是工程 demo、不是内部工具、不是「能跑就行」的 MVP。它是一款最终要卖给真实付费用户的 ToC 桌面软件，每一个环节都必须按商业化产品的标准来要求。**

> 「无论是哪一个功能，无论写哪一个文档，最终目标是一定要以一个商业化 ToC 软件的要求来要求每一个环节。」—— 项目用户原话（2026-05-29）

### 适用范围（每一个环节都要套用这把尺）

| 环节 | 商业化 ToC 标准的具体含义 |
|---|---|
| **新功能** | 必须达到本文件 §「商用 C 端软件丝滑标准」全部 10 条；否则不算完成 |
| **Bug 修复** | 修一个不能改坏三个；必须自验证再让用户测；故障描述要翻译成人话 |
| **UI / 交互** | 像剪映 / Final Cut Pro 一样丝滑，没有静默成功 / 静默失败 / 卡顿 / 神秘消失的窗口 |
| **错误提示** | 不许把 stack trace / JsonException / ExitCode=-XXX 直接丢用户看；必须翻译为人能看懂的描述 + 重试入口 |
| **进度反馈** | 任何 >1s 的操作必须有可视进度；多阶段任务显示「N/M 当前阶段」；细到单个视频/单个文件 |
| **依赖下载** | 国内国外都能用 —— 没国内可访问源就不做依赖下载化（用打包代替）|
| **安装包** | 装上即跑，不挑机器；不能依赖目标机器装过 VS / VC++ Redist / .NET SDK |
| **文档（README / Release notes / Issues）** | 用户读得懂；不堆英文报错；Gitee / GitHub 各自当独立发布渠道，不互相引导 |
| **发版 notes** | 用户视角讲「新增了什么 / 修了什么 / 怎么下载」；不是 commit log 复制粘贴 |
| **代码注释 / 错误码** | 给未来维护者读 —— 解释为什么这么做，而不是这是什么 |
| **测试与回归** | 改完自验证再让用户测（详见 §「自我验证铁律」）；发版前在干净环境跑完整 e2e |
| **issue / PR 描述** | 让 Windows 端 / Mac 端 / 国内代理跑步的工程师都能照着复现 + 实施 |

### 判定原则（每次完成任务前先自问）

1. **这个完成度，敢卖钱吗？** 不敢 → 没完成。
2. **真实用户首次打开就能用吗？** 需要他改注册表 / 装 VC Redist / 加白名单 → 没完成。
3. **国内国外用户都能正常下载与更新吗？** 一边断了 → 没完成。
4. **如果是我自己付了 199 元下载的，我会满意这一段体验吗？** 不会 → 没完成。
5. **如果出错了，用户看屏幕能不能知道发生了什么 + 下一步怎么办？** 不能 → 没完成。

### 红线（任何一条踩中即视为不达标）

- ❌ 发版前没在干净 Win 10/11 环境跑完整 e2e（导入 → ASR → 切分 → 生成方案 → 导出 → 播放）
- ❌ 任何用户面板上能看到的英文 stack trace / 原生报错码
- ❌ 「能跑通」就汇报完成（必须达到 §「商用 C 端软件丝滑标准」基线）
- ❌ 国内 Gitee 渠道引导用户去 GitHub 下载
- ❌ Release notes / issue 描述写得像 commit log，不是给最终用户看的
- ❌ 任何「让用户当第一测试者」的偷懒做法

**本节是整个 CLAUDE.md 的总闸 —— 当下游章节的规则与本节冲突时，本节优先。当下游规则需要解释「为什么这么严」，回到本节查就行。**

---

## 🪟 目标平台铁律：Windows 10 + Windows 11 双系统兼容（必须 ⚠️⚠️⚠️）

**本项目目标用户机器是 Windows 10 和 Windows 11（x64）。任何改动、任何依赖、任何发版都必须保证：用户下载安装包 → 双击 setup.exe → 装完即用，Win 10 / Win 11 双平台都不允许出现「装完不能跑」「需要装别的东西」「需要改注册表」「需要装防火墙白名单」等情况。**

> 「这个项目要兼容 Win 10 系统和 Win 11 系统」—— 项目用户原话（2026-05-29）

### 支持范围

| 系统 | 最低版本 | 架构 | 状态 |
|---|---|---|---|
| **Windows 10** | 1809（17763）及以上 | x64 | ✅ 必须支持 |
| **Windows 11** | 全部版本 | x64 | ✅ 必须支持 |
| Windows 10 1809 之前 | n/a | n/a | ❌ 不支持（.NET 8 硬性下限） |
| Windows 7 / 8 / 8.1 | n/a | n/a | ❌ 不支持 |
| Windows ARM64 | n/a | n/a | ❌ 暂不支持 |
| Windows 10/11 N 版（不含 Media Pack） | n/a | n/a | ✅ 导出/缩略图/**预览**均走自带 ffmpeg（v0.7.x 预览已统一到 `FfmpegFramePlayer` 进程外裸帧管道，不再依赖系统编解码器），N 版 / 缺 HEVC 扩展机器同样可用；有 N 版机时仍建议真机 e2e 兜底 |

> .NET 8 官方最低支持 Win 10 1809。低于此版本系统市占率 < 1%，不在支持范围。

### 装上即跑（铁律）

用户首次安装后，**必须**不依赖以下任何外部条件就能跑全部核心功能：

| 项 | 必须自带 | 不依赖用户机器有 |
|---|---|---|
| .NET 8 Runtime | ✅ 已 self-contained 打包 | ❌ 不要求用户装 .NET Desktop Runtime |
| VC++ Redistributable (2015-2022) | ✅ publish/bin/ 已含 6 个 VC Runtime DLL | ❌ 不要求用户装 VC++ Redist |
| OpenMP runtime (vcomp140) | ✅ publish/bin/ 已含 | ❌ 不要求用户装 Office |
| FFmpeg / ffprobe / whisper-cli | ✅ publish/bin/ 已含 | ❌ 不要求用户装 FFmpeg |
| 视频解码（HEVC/VP9/AV1 等） | ✅ 导出/缩略图/预览**全部**走自带 ffmpeg（v0.7.x 起预览也已统一，见 §兼容性总纲） | ❌ 不要求用户装系统编解码器 / HEVC 扩展 / VLC |
| Whisper 语音模型（按需下载） | ⚠️ 首次用 ASR 时下载 | ✅ 应用内有下载进度 UI + 国内镜像源 |

> 注：v0.6.1 已移除 LibVLC（NuGet 包 + VlcBootstrap.cs + 365 plugins，见 commit `af0ddd7`）；v0.7.x 预览统一到自带 ffmpeg 进程外裸帧管道（`FfmpegFramePlayer`，与导出同源、不碰系统编解码器）。§兼容性总纲的「预览掉队」待修项**已闭环**。

发版前自检 grep 关键字：`[VcRuntimeDiag] all 6 VC Runtime DLLs present`、`[EnvDiag] pass=True`。

### 发版前 Win 10/11 双平台兼容性 checklist

**每次改动引入新 native 依赖（LibVLC、ffmpeg 升级、whisper 升级等）必须跑完**：

- [ ] **静态分析**：对每个**新引入的 native dll** 跑 PE import 表扫描（`Get-PeImports`），列所有依赖 dll → 对比 publish/bin/ 和 publish/libvlc/ 是否完备 → 缺啥补啥（参考 v0.4.1 vcomp140 沉淀方法）
- [ ] **构建机自验**：远端 publish + 启动 + grep 启动期诊断日志 `[VcRuntimeDiag]` `[EnvDiag]` 全绿
- [ ] **干净 Win 10 e2e**（**有干净机时必跑**）：装安装包 → 启动 → 完整跑导入 → ASR → 切分 → 方案生成 → 导出 → 用系统播放器播一遍
- [ ] **干净 Win 11 e2e**（**有干净机时必跑**）：同上
- [ ] **N 版 Win 10/11 e2e**（**有 N 版机时必跑**）：N 版不含 Windows Media Foundation，确认导出/缩略图/**预览**（v0.7.x 起均走自带 ffmpeg）都不撞 mfplat.dll、正常出画
- [ ] **无干净机时**：dumpbin 静态分析 + 构建机用「新用户账号 + 不继承 VC++ Redist」模拟干净环境跑

### 已知容易撞双平台兼容性的雷区（动相关代码时必查）

| 雷区 | 表现 | 防范 |
|---|---|---|
| **VC Runtime 缺失** | 干净机启动崩，`ExitCode=-1073741515` (`STATUS_DLL_NOT_FOUND`) | v0.3.0 沉淀：必带 6 个 VC Runtime DLL |
| **vcomp140 缺失** | ggml-cpu / 部分 OpenMP 加速代码崩 | v0.4.0 沉淀：必带 vcomp140 + concrt140 |
| **ffmpeg codec-private 选项** | N 卡 / I 卡 / A 卡 不同 GPU 路径行为不一致 | v0.4.0 沉淀：构建机只能测一种 GPU，其它 GPU 路径需用户实测 |
| ~~预览依赖系统编解码器~~ ✅ 已解决 | （历史）HEVC / iPhone「高效」格式 hover 预览曾报 `0xC00D109B 该格式可能需要系统媒体编解码支持` | v0.7.x 起 `InlineVideoPlayer` 已改走自带 ffmpeg（`FfmpegFramePlayer` 进程外裸帧管道），与导出同源、不碰系统编解码器 —— 遗留架构不一致已消除 |
| ~~WPF MediaElement seek 闪帧~~ ✅ 已解决 | （历史）hover 播放分镜先闪视频第 0 秒 | v0.3.0 行为；v0.6.0 曾用 LibVLC 根治但引入卡死→v0.6.1 退回 MediaElement 接受闪帧；v0.7.x 预览统一到自带 ffmpeg 裸帧管道后彻底消除（首帧就绪前不显示播放层，见 §H） |
| **Windows SmartScreen 拦截** | 首次启动「Windows 已保护你的电脑」拦 | 长期：买 EV 代码签名证书；短期：用户点「仍要运行」 |
| **antivirus 误报** | whisper-cli / ffmpeg 被识为可疑 | 短期容忍；发版 notes 提示用户加白名单 |
| **Win 10 1809 之前** | .NET 8 安装失败 | 不在支持范围，安装包不主动检查（用户极少） |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RoshanGH/mixcut-windows](https://github.com/RoshanGH/mixcut-windows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
