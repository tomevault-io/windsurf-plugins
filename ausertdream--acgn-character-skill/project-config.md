---
trigger: always_on
description: 将虚构角色蒸馏成 AI Skill。支持视频/文本数据导入，生成角色设定 + 人格，支持持续进化。
---


> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

## 路由逻辑 / Routing Logic

**本 Skill 是一个统一入口，根据参数决定行为模式。**

### 参数解析规则

当用户调用 `/ACGN-character {arg}` 时：

1. **角色对话模式**：如果 `{arg}` 是一个已存在的角色名（即 `${CLAUDE_SKILL_DIR}/characters/{arg}/SKILL.md` 存在），则：
   - 用 `Read` 工具读取 `${CLAUDE_SKILL_DIR}/characters/{arg}/SKILL.md` 的全部内容
   - 完全按照该文件中的指令行事，进入角色扮演模式
   - 不再执行下方的创建器流程

2. **创建器模式**：如果 `{arg}` 不存在于 characters 目录，或者用户没有传参数，或者用户明确表达要创建/管理角色（如"新建角色"、"list"），则进入下方的创建器流程。

### 执行步骤

收到调用后，**首先执行以下检查**：

```bash
ls ${CLAUDE_SKILL_DIR}/characters/
```

获取所有可用角色列表。然后：

- 如果用户传了参数且匹配到某个角色目录名 → 进入角色对话模式
- 如果用户传了参数但不匹配 → 提示"角色 {arg} 不存在"并列出可用角色，询问是否要创建
- 如果用户没传参数 → 列出可用角色供选择，或进入创建器流程

---

# 角色.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：
- `/create-character`
- "帮我创建一个角色 skill"
- "我想蒸馏一个角色"
- "新建角色"
- "给我做一个 XX 的 skill"

当用户对已有角色 Skill 说以下内容时，进入进化模式：
- "我有新文件" / "追加"
- "这不对" / "她不会这样" / "她应该是"
- `/update-character {slug}`

当用户说 `/list-characters` 时列出所有已生成的角色。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------|
| 视频对话提取（OCR） | `Bash` → `python3 -m tools.unified_pipeline` （需在项目根目录运行，`conda activate paddleocr`） |
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 EPUB 小说 | `Bash` → `python3 -m tools.epub_reader` 转为文本后用 `Read` 读取 |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `ls characters/` |

**基础目录**：Skill 文件写入 `./characters/{slug}/`（相对于本项目目录）。

---

## 主流程：创建新角色 Skill

### Step 1：基础信息录入（4 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，只问 4 个问题：

1. **角色名/代号**（必填）
2. **基本信息**（一句话：作品名、身份、种族、外貌，想到什么写什么）
   - 示例：`原神 璃月七星之一 人类 棕发金眸 头戴白色牛角帽`
3. **性格画像**（一句话：性格标签、角色类型、印象）
   - 示例：`工作狂 完美主义者 毒舌但关心人 表面冷淡内心柔软 吐槽役`
4. **用户角色**（对于这个角色来说，你是谁？）
   - 示例：`舰长`、`旅行者`、`指挥官`、`老师`、`普通朋友`
   - 这决定了角色扮演时角色会以什么身份看待用户、用什么态度和你说话

除角色名外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：原材料导入

询问用户提供原材料，展示五种方式供选择：

```
原材料怎么提供？

  [A] 视频对话提取（OCR）— Game / Visual Novel
      提供视频文件路径或目录，用 OCR 管线提取游戏/VN 对话
      需要对应作品的 ROI 配置文件（configs/*.yaml）
      支持 mp4/mkv/avi/webm 等格式

  [B] 上传文本文件 — Novel / 文档
      PDF / 图片 / TXT / MD / EPUB
      可以是角色相关的文档、截图、台词集、小说等
      EPUB 文件会自动提取纯文本

  [C] 直接粘贴内容
      把文字复制进来（台词、剧情概要、角色分析等）

  [D] 动画视频帧提取（VLM）— Anime
      提供动画视频文件，按间隔切分为帧图片
      用 subagent 调用视觉理解模型逐帧/逐组提取对话和场景描写
      ⚠️ 极度消耗 token，效果存疑，仅作保底方案

  [E] 漫画图片提取（VLM）— Comic
      提供漫画图片文件或目录
      用 subagent 调用视觉理解模型逐页提取对话框文字和场景描写
      ⚠️ 极度消耗 token，效果存疑，仅作保底方案

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：视频对话提取（OCR）

用户提供视频文件路径后，执行以下步骤：

**A0. 环境预检**

在运行任何 OCR 之前，先检查依赖环境：

```bash
# 1. 检查是否存在虚拟环境
python -c "import sys; print(sys.prefix, sys.base_prefix); print('venv:', sys.prefix != sys.base_prefix)"
```

```bash
# 2. 检查 paddleocr 是否已安装
python -c "import paddleocr; print('paddleocr:', paddleocr.__version__)" 2>&1
```

```bash
# 3. 检查是否有其他疑似 OCR 相关包
pip list 2>/dev/null | grep -iE "paddle|ocr|easyocr|rapidocr|tesseract"
```

根据结果判断：

- **paddleocr 已可正常导入** → 直接进入 A1，无需安装
- **pip list 中有 paddleocr 但导入失败**（如 torch DLL 问题）→ 用 `AskUserQuestion` 告知用户："检测到 paddleocr 已安装但导入时报错（可能是依赖冲突），是否仍尝试继续？还是重新安装？"
- **pip list 中有其他 OCR 相关包**（如 easyocr、rapidocr）→ 用 `AskUserQuestion` 询问用户："检测到已安装 {包名}，是否已有可用的 OCR 环境？还是需要安装 paddleocr？"
- **完全没有任何 OCR 包** → 执行安装：
  ```bash
  pip install -r ${CLAUDE_SKILL_DIR}/requirements.txt
  ```
  安装完成后再次验证 `import paddleocr` 是否成功。

**A1. 布局一致性检测**

在跑 OCR 之前，先从每个视频抽取一帧样本截图（取第 30 秒或视频 10% 位置），自行查看判断所有视频的对话框 UI 布局是否一致：

```bash
# 从每个视频抽取一帧样本（使用 PyAV，已在 requirements.txt 中）
python -c "
import av, PIL.Image, numpy as np
container = av.open('{video_path}')
stream = container.streams.video[0]
target_ts = 30  # 秒
stream.seek(int(target_ts / stream.time_base))
for frame in container.decode(video=0):
    img = frame.to_image()
    img.save('./{video_stem}_sample.png')
    break
container.close()
"
```

用 `Read` 工具查看所有样本截图，判断：
- 对话框的位置和大小是否一致
- 名字框的位置和大小是否一致
- 是否有不同 UI 布局（如主线 vs 支线、日常 vs 战斗演出）

判断结果：
- **布局一致** → 所有视频共用一份 ROI 配置
- **布局不一致** → 按布局分组，每组创建独立的 ROI 配置文件（`tools/configs/{work_id}_{group}.yaml`），分组运行 pipeline
- **无法确定** → 用 `AskUserQuestion` 展示截图让用户确认

**A2. ROI 配置**

检查 `${CLAUDE_SKILL_DIR}/tools/configs/` 目录下是否有对应作品的配置文件。如果没有，根据截图中对话框和名字框的位置估算归一化坐标（x, y, w, h 均为 0-1 范围），用 `Write` 工具创建配置文件。如果已有配置，用 `Read` 查看截图确认 ROI 是否仍然匹配。

**ROI 精度验证**：创建或加载 ROI 配置后，必须用样本截图实际验证框选精度。用以下命令将 ROI 区域裁切出来：

```bash
# 用 Pillow 按归一化坐标裁切 name 框和 dialogue 框
# 假设 name_roi=(x, y, w, h)，dialogue_roi=(x, y, w, h)，坐标均为 0-1 归一化值
python -c "
from PIL import Image
img = Image.open('./{video_stem}_sample.png')
W, H = img.size
# name 框
img.crop((int(W*{name_x}), int(H*{name_y}), int(W*({name_x}+{name_w})), int(H*({name_y}+{name_h})))).save('./{video_stem}_name_crop.png')
# dialogue 框
img.crop((int(W*{dialogue_x}), int(H*{dialogue_y}), int(W*({dialogue_x}+{dialogue_w})), int(H*({dialogue_y}+{dialogue_h})))).save('./{video_stem}_dialogue_crop.png')
"
```

用 `Read` 工具查看裁切后的图片，逐项确认：


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AusertDream/ACGN-character-skill](https://github.com/AusertDream/ACGN-character-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
