---
trigger: always_on
description: 把一段中文（或任意 Whisper 支持语言）的会议/面试录音用本地 Whisper large-v3 转成文本，再清洗成带说话人标签、修过 ASR 错字、分好章节的 markdown 文档（可选再转成 docx）。跨平台（macOS Apple Silicon 用 mlx-whisper，Windows/Linux/Intel Mac 用 whisper-ctranslate2）。零云端、零订阅、隐私不出本机。适用：替代飞书妙计/通义听悟/Otter.ai 这类付费转录服务、需要在 VS Code 或 Word 里直接拿到可读稿、在 AI agent（Claude Code/Copilot/Codex/Cursor）里端到端跑通。不适用：实时转录、强噪声多人重叠会议、需要严格说话人分离的场景。
---


# Murmur — 本地零成本音频转录与清洗工作流

**仓库**：https://github.com/xiaopengde/murmur
**适用 agent**：GitHub Copilot Agent / Claude Code / OpenAI Codex / Cursor / 任意遵循 [agentskills.io](https://agentskills.io) 的 agent

---

## 0. 触发条件

满足以下**任一**触发该 skill：

- 用户说"帮我转录 / 把这段录音转成文字 / 出逐字稿 / 出会议纪要 / 飞书妙计太贵"
- 工作目录里出现 `.m4a` / `.mp3` / `.wav` / `.mp4` / `.webm` / `.flac` / `.ogg` 文件
- 用户明确说"用 Murmur"

---

## 1. 端到端 5 步流程

### 步骤 A — 环境检查（必跑，不要跳）

**假设用户是新机器**，刚装完 Claude Code / Codex / Copilot，没装其他任何东西。所以先检查环境：

```bash
# macOS / Linux
bash scripts/doctor.sh

# Windows (PowerShell)
powershell -ExecutionPolicy Bypass -File scripts/doctor.ps1
```

doctor 脚本会分两块输出：

1. **核心依赖状态**：`ffmpeg` / `uvx` / `pandoc` / `python3` / 平台和芯片 / 模型缓存（ModelScope / HuggingFace）
2. **Murmur onboarding 状态**：默认输出格式、默认离线模型是否已经由用户明确选择

**如果核心依赖有 ❌**，跑对应的 install 脚本：

```bash
# macOS
bash scripts/install-mac.sh

# Windows (需要管理员 PowerShell)
powershell -ExecutionPolicy Bypass -File scripts/install-windows.ps1
```

安装完再跑一次 `doctor`。如果要在自动化里强校验，使用 strict 模式：

```bash
bash scripts/doctor.sh --strict
powershell -ExecutionPolicy Bypass -File scripts/doctor.ps1 -Strict
```

`--strict` / `-Strict` 只有在核心依赖齐全且 onboarding 已完成时才返回 0；依赖缺失或 onboarding 未完成都会返回非 0。**doctor 如果提示 onboarding 未完成，不要说“可以开始转录”，下一步必须跑 `python scripts/transcribe.py --onboarding`。**

**端到端验证**（首次安装强烈推荐，或排查问题时；会使用临时配置，不污染用户默认值）：

```bash
bash scripts/doctor.sh --smoke                                    # macOS / Linux
powershell -ExecutionPolicy Bypass -File scripts/doctor.ps1 -Smoke  # Windows
```

会自动生成 2 秒测试音频跑完整 pipeline。大陆 Apple Silicon 会优先用 ModelScope large-v3-turbo 4bit（首次约 464MB），其他环境用 tiny 模型（首次约 75MB）；通过后说明 ffmpeg → uvx → mlx/whisper → 文件输出全链路工作。失败时会保留临时目录方便排查。

### 步骤 B — 新用户 / 首次 onboarding（硬门禁）

**无论用户是否已经给了音频文件，都必须先跑：**

```bash
python scripts/transcribe.py --onboarding
```

读取 JSON：

- 如果 `needs_onboarding=false`：可以进入步骤 C。
- 如果 `needs_onboarding=true`：agent **必须**使用 AskQuestion / 候选框让用户选择，不能自己决定，不能直接执行 JSON 里的 example。JSON 会明确包含：
  - `must_ask_user: true`
  - `do_not_choose_for_user: true`
  - `do_not_run_example_without_user_choice: true`

必须向用户询问两个候选项：

1. 默认输出格式：`md` 或 `docx`
2. 默认离线模型：`large-v3-turbo` / `large-v3` / `medium` / `small`

用户选完后，运行：

```bash
python scripts/transcribe.py --init-defaults --format <md|docx> --set-default-model <model>
```

只有这个命令成功后，才允许继续转录。`--format` / `--model` 是单次覆盖参数，**不能绕过首次 onboarding**；未完成 onboarding 时，转录主流程会直接退出并要求先完成 onboarding。

如果用户后来想改默认：

```bash
python scripts/transcribe.py --set-default md       # 或 docx
python scripts/transcribe.py --set-default-model medium
```

### 步骤 C — 跑转录并转述进度

```bash
python scripts/transcribe.py <音频文件> [--lang zh] [--output-dir .] [--model medium] [--cn]
```

脚本内部会：

1. 用 `ffmpeg` 把任意输入转成 16kHz 单声道 WAV（**关键**——直接喂 m4a 会触发 Whisper 幻觉循环）
2. 检测平台：
   - **Apple Silicon Mac** → `uvx --from mlx-whisper mlx_whisper`（GPU 加速，最快）
   - **Windows / Linux / Intel Mac** → `uvx whisper-ctranslate2`（CTranslate2 后端，CPU 也很快）
3. 输出 `转录原稿.txt` + `字幕.srt` 到目标目录
4. 清理临时 WAV

> ### ⛔️ 进度转述硬约束（agent 必读，最常翻车的一步）
>
> **反模式（你大概率会犯，越自信的模型越容易犯）**：把这条转录命令当成「一个会返回结果的调用」，用一次性、长时间的前台阻塞等待（例如 `Waiting 12m for shell` / 一个 timeout=30min 的同步调用）等它跑完，期间不回头看终端。你可能觉得「下载和推理中间没我什么事，等结束再汇报最高效」——**这是错的**。脚本每 30 秒就在终端打一行心跳（`📥 模型下载/准备中... 已用时 x:xx` / `⏳ 转录推理中... 已用时 x:xx`），但只要你一次性阻塞到结束，这些心跳全被你埋掉，用户面对的就是 **十几分钟纯静默的终端**，完全不知道是在下载、在推理、还是已经卡死。说一句「我会盯着进度」**不等于**你真的在盯。
>
> **必须这样做（不可省略、不可优化掉）**：
> 1. **禁止**用单次「阻塞等到命令结束」的方式跑转录。要么把它放后台跑，要么用**短** timeout（如每次最多等 30–45 秒）然后**反复**回来读终端。
> 2. **每约 30 秒**必须主动 poll 一次终端输出。
> 3. 每次 poll 后，只要看到**新的**心跳或阶段变化（`📥 下载/准备` → `⏳ 转录推理` → `✅ 完成`），**立刻**用一句前台消息转述给用户（例如「模型还在下载，约 40%」「已进入转录推理，已用时 2:00」「转录完成，正在整理输出」）。
> 4. 从命令开始到结束，中间**不允许出现一次超过 ~45 秒的静默**。如果你发现自己正打算「等它跑完再说」，停下——那就是上面的反模式。
>
> 首次没有缓存时，**下载/准备完成后**才算进入推理阶段；这两个阶段都要分别转述，不要把「还在下载」说成「在推理」。

**🇨🇳 大陆网络**：transcribe.py 会按时区/语言自动判断是否在大陆。命中后默认使用 `--model-source auto`：
- 对已验证映射，优先从 ModelScope 下载模型到 `~/.cache/murmur/models/`，然后把本地目录交给现有 `mlx-whisper` / `whisper-ctranslate2` 推理。
- 对没有 ModelScope 映射的模型，继续给 whisper 子进程注入 `HF_ENDPOINT=https://hf-mirror.com`（HuggingFace 镜像）和 `UV_INDEX_URL=https://pypi.tuna.tsinghua.edu.cn/simple`（uv 拉依赖走清华）。

> ⚠️ **走 ModelScope 时注意 VPN**：ModelScope 是国内 CDN，大陆**直连最快**。如果用户开了**全局 VPN / 代理**，流量会绕到海外再回国内，反而拖慢下载。看到走 ModelScope 时，agent 应提醒用户：「正在从国内源 ModelScope 下载，如开着全局 VPN 建议临时关掉直连更快」。（这点和走 HuggingFace 时相反——HF 路线开 VPN 才快。）

可手动指定模型源：

```bash
python scripts/transcribe.py 录音.m4a --model-source modelscope  # 强制优先 ModelScope
python scripts/transcribe.py 录音.m4a --model-source hf          # 强制原 HuggingFace/引擎默认源
```

当前已验证映射：
- Apple Silicon / `mlx-whisper`：`large-v3-turbo` → `mlx-community/whisper-large-v3-turbo-4bit`（ModelScope，约 464MB；下载后自动适配 `model.safetensors` → `weights.safetensors`）
- Windows / Linux / Intel Mac / `whisper-ctranslate2`：`large-v3-turbo` → `mobiuslabsgmbh/faster-whisper-large-v3-turbo`（ModelScope，约 1.62GB，CTranslate2 格式；需要按目标平台 smoke test）

用户已经手动设过的同名环境变量**不会被覆盖**。显式 `--cn` / `--no-cn` 强制单次开关。

**持久化偏好**（避免每次都加 `--cn`）：

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xiaopengde/murmur](https://github.com/xiaopengde/murmur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
