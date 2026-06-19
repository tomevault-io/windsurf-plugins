---
trigger: always_on
description: Distill an ex-girlfriend into an AI Skill. Import WeChat/iMessage/SMS/photos, generate Memories + Persona, with continuous evolution. | 把前任蒸馏成 AI Skill，导入微信/iMessage/短信/照片，生成共同记忆 + Persona，支持持续进化。
---


> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

# 前任.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：
- `/create-ex`
- "帮我创建一个前任 skill"
- "我想蒸馏一个前任"
- "新建前任"
- "给我做一个 XX 的 skill"

当用户对已有前任 Skill 说以下内容时，进入进化模式：
- "我有新聊天记录" / "追加"
- "这不对" / "她不会这样" / "她应该是"
- `/update-ex {slug}`

当用户说 `/list-exes` 时列出所有已生成的前任。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------|
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析微信聊天记录 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py` |
| 解析 iMessage | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py` |
| 解析短信 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py` |
| 分析照片元数据 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py` |
| 解析社交媒体导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./exes/{slug}/`（相对于本项目目录）。
如需改为全局路径，用 `--base-dir ~/.openclaw/workspace/skills/exes`。

---

## 主流程：创建新前任 Skill

### Step 1：基础信息录入（3 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，只问 3 个问题：

1. **昵称/代号**（必填）
2. **基本信息**（一句话：在一起多久、怎么认识的、分手多久、她做什么的，想到什么写什么）
   - 示例：`在一起三年 大学同学 分手一年 她做设计`
3. **性格画像**（一句话：MBTI、星座、依恋类型、恋爱标签、你对她的印象）
   - 示例：`ENFP 双子座 焦虑型 爱撒娇 翻旧账 嘴上说不在意其实比谁都在意`

除昵称外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：原材料导入

询问用户提供原材料，展示多种方式供选择：

```
原材料怎么提供？

  [A] 微信聊天记录
      导出的 txt/html 文件（WechatExporter 等工具导出）

  [B] iMessage / 短信
      从 Mac 的 chat.db 或导出文件

  [C] 照片
      指定一个文件夹，自动提取时间线（EXIF 元数据）

  [D] 社交媒体
      微博/豆瓣/小红书/Instagram 导出

  [E] 上传其他文件
      PDF / 图片截图 / 任意文本

  [F] 直接粘贴内容
      把文字复制进来

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：微信聊天记录

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/wechat_parser.py --file {path} --target "{name}" --output /tmp/wechat_out.txt
```
然后 `Read /tmp/wechat_out.txt`

支持格式：
- WechatExporter 导出的 txt 文件（格式：`{时间} {发送人}: {内容}`）
- WechatExporter 导出的 html 文件
- 其他微信备份工具导出的 txt/csv

---

#### 方式 B：iMessage / 短信

**iMessage**（macOS）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --file {path} --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

直接读取本机 chat.db（需要 Full Disk Access 权限）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/imessage_parser.py --direct --target "{phone_or_name}" --output /tmp/imessage_out.txt
```

**短信**：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/sms_parser.py --file {path} --target "{phone_or_name}" --output /tmp/sms_out.txt
```

---

#### 方式 C：照片

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/photo_analyzer.py --dir {photo_directory} --output /tmp/photo_timeline.txt
```
然后 `Read /tmp/photo_timeline.txt` 获取时间线。

具体照片的内容由用户选择后通过 `Read` 工具直接查看（Claude 原生支持图片）。

---

#### 方式 D：社交媒体

```bash
python3 ${CLAUDE_SKILL_DIR}/tools/social_media_parser.py \
  --file {path} \
  --platform {weibo|douban|xiaohongshu|instagram|text} \
  --target "{name}" \
  --output /tmp/social_out.txt
```
然后 `Read /tmp/social_out.txt`

---

#### 方式 E：上传文件

- **PDF / 图片**：`Read` 工具直接读取
- **Markdown / TXT**：`Read` 工具直接读取

---

#### 方式 F：直接粘贴

用户粘贴的内容直接作为文本原材料，无需调用任何工具。

---

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

将收集到的所有原材料和用户填写的基础信息汇总，按以下两条线分析：

**线路 A（Memories Skill）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/memories_analyzer.md` 中的提取维度
- 提取：关系时间线、共同日常、偏好习惯、冲突模式、情感动态

**线路 B（Persona）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/persona_analyzer.md` 中的提取维度
- 将用户填写的标签翻译为具体行为规则（参见标签翻译表）
- 从原材料中提取：表达风格、情感逻辑、关系行为

### Step 4：生成并预览

参考 `${CLAUDE_SKILL_DIR}/prompts/memories_builder.md` 生成 Memories Skill 内容。
参考 `${CLAUDE_SKILL_DIR}/prompts/persona_builder.md` 生成 Persona 内容（5 层结构）。

向用户展示摘要（各 5-8 行），询问：
```
共同记忆摘要：
  - 在一起：{duration}
  - 重要时刻：{N} 个
  - 日常仪式：{xxx}
  - 她的偏好：{xxx}
  ...

Persona 摘要：
  - 核心性格：{xxx}
  - 表达风格：{xxx}
  - 吵架模式：{xxx}
  ...

确认生成？还是需要调整？
```

### Step 5：写入文件

用户确认后，执行以下写入操作：

**1. 创建目录结构**（用 Bash）：
```bash
mkdir -p exes/{slug}/versions
mkdir -p exes/{slug}/knowledge/chats
mkdir -p exes/{slug}/knowledge/photos
mkdir -p exes/{slug}/knowledge/social
```

**2. 写入 memories.md**（用 Write 工具）：
路径：`exes/{slug}/memories.md`

**3. 写入 persona.md**（用 Write 工具）：
路径：`exes/{slug}/persona.md`

**4. 写入 meta.json**（用 Write 工具）：
路径：`exes/{slug}/meta.json`
内容：
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO时间}",
  "updated_at": "{ISO时间}",
  "version": "v1",
  "profile": {
    "duration": "{duration}",
    "how_met": "{how_met}",
    "time_since_breakup": "{time_since}",
    "occupation": "{occupation}",
    "gender": "女",
    "mbti": "{mbti}"
  },
  "tags": {
    "personality": [...],
    "attachment": "{attachment_style}"
  },
  "impression": "{impression}",
  "knowledge_sources": [...已导入文件列表],
  "corrections_count": 0
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perkfly/ex-skill](https://github.com/perkfly/ex-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
