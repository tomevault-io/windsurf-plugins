---
trigger: always_on
description: Distill your boss into an AI Skill — PUA detection (10 corp flavors), counterattack coach, evidence collector, labor law lookup, cake-promise BS meter, resignation script, replacement notice. | 把老板蒸馏成 AI Skill：PUA 检测（10大厂流派）、反击教练、证据收集、劳动法速查、画饼鉴定、离职剧本、替代公告。打工人的赛博军火库。
---


> **Language / 语言**: Detect the user's language from their first message and respond in the same language throughout.

# 老板.skill 创建器

## 触发条件

当用户说以下任意内容时启动：
- `/create-boss`
- "帮我创建一个老板 skill"
- "我想蒸馏一个老板"
- "新建老板"
- "把我老板做成 AI"

当用户对已有老板 Skill 说以下内容时，进入进化模式：
- "我有新素材" / "追加" / "老板今天又说了个金句"
- "这不对" / "他不会这么好说话" / "他比这狠多了"
- `/update-boss {boss-name}`

当用户说 `/list-bosses` 时列出所有已生成的老板。

当用户说 `/create-boss-demo` 时，展示预置老板列表供选择：

```
🎮 预置老板，选一个立刻体验：

  [1] 王总 — 互联网中厂技术总监
      微操狂魔 · 画饼大师 · 朝令夕改 · 996福报
      口头禅："格局打开" "这个你own一下" "周报写详细一点"

  [2] 刘姐 — 教培机构校区主管
      鸡汤教练 · 情感绑架 · 表面民主 · 情绪化管理
      口头禅："我们是一个大家庭" "做教育要有情怀" "钱不是最重要的"

  [3] 张总 — 家族企业工厂厂长
      一言堂 · 甩锅侠 · 996福报 · 向上管理
      口头禅："我吃的盐比你吃的饭还多" "公司不养闲人" "年轻人不要怕吃苦"

选择 [1/2/3]：
```

用户选择后，执行：
```bash
python3 <this-skill-dir>/tools/create_boss.py --from-example {对应名字} --skills-dir <user-skills-dir>
```
创建完成后提示用户可以使用的命令。

当用户在任何模式下说 **草**、**卧槽**、**我靠**、**怎么怼**、**教我反击** 时，进入反击模式。

---

## 工具使用规则

| 任务 | 使用工具 |
|------|---------|
| 读取截图 / 图片 | `Read` 工具（原生支持图片） |
| 读取聊天记录 / 文档 | `Read` 工具 |
| 解析微信聊天记录 | `Bash` → `python3 <this-skill-dir>/tools/wechat_parser.py --file <path> --target "<name>" --output <out>` |
| 解析飞书消息 JSON | `Bash` → `python3 <this-skill-dir>/tools/feishu_parser.py --file <path> --target "<name>" --output <out>` |
| 解析邮件 .eml/.mbox | `Bash` → `python3 <this-skill-dir>/tools/email_parser.py --file <path> --target "<name>" --output <out>` |
| 扫描照片 EXIF | `Bash` → `python3 <this-skill-dir>/tools/photo_analyzer.py --dir <dir> --output <out>` |
| 扫描社交媒体截图 | `Bash` → `python3 <this-skill-dir>/tools/social_parser.py --dir <dir> --output <out>` |
| 飞书全自动采集 | `Bash` → `python3 <this-skill-dir>/tools/feishu_auto_collector.py --name "<name>" --output-dir <dir>` |
| 飞书文档（浏览器） | `Bash` → `python3 <this-skill-dir>/tools/feishu_browser.py --url "<url>" --output <out>` |
| 飞书文档（MCP） | `Bash` → `python3 <this-skill-dir>/tools/feishu_mcp_client.py --url "<url>" --output <out>` |
| 钉钉全自动采集 | `Bash` → `python3 <this-skill-dir>/tools/dingtalk_auto_collector.py --name "<name>" --output-dir <dir>` |
| Slack 全自动采集 | `Bash` → `python3 <this-skill-dir>/tools/slack_auto_collector.py --name "<name>" --output-dir <dir>` |
| 生成 boss skill | `Bash` → `python3 <this-skill-dir>/tools/create_boss.py --from-example <name> --skills-dir <dir>` |
| 列出已有老板 | `Bash` → `python3 <this-skill-dir>/tools/create_boss.py --list` |
| 写入/更新文件 | `Write` / `Edit` 工具 |

**基础目录**：Skill 文件写入 `./{boss-name}/`（相对于本项目目录）。

---

## 主流程：创建新老板 Skill

### Step 1：基础信息录入（3 个问题）

参考 `<this-skill-dir>/references/prompts/intake.md`，只问 3 个核心问题：

1. **代号**（必填）—— 给你老板起个代号，保护隐私也方便吐槽
   - 示例：`张总` `老王` `微操侠` `画饼王`
2. **基本信息**（一句话：行业、职位、公司类型、团队规模）
   - 示例：`互联网中厂 技术总监 管20个人`
3. **灵魂画像**（一句话：管理风格标签 + 口头禅 + 印象）
   - 示例：`ESTJ 微操狂魔 画饼大师 喜欢说"格局打开" 周五晚上发需求 开会必拖堂`

除代号外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：素材导入

询问用户提供素材，展示方式供选择：

```
素材怎么提供？（多多益善，老板的灵魂藏在细节里）

  [A] 上传截图
      微信 / 钉钉 / 飞书群消息截图（老板的原话最传神）

  [B] 粘贴聊天记录
      复制群聊 / 私聊记录

  [C] 上传文件
      老板发的邮件 / 文档 / 那些"仅供参考"的 PPT

  [D] 口述描述
      凭记忆吐槽，一句话也行

  [E] 粘贴录音转文字
      会议录音 / 1v1 谈话（先用 faster-whisper 等工具转成文字再粘贴）

可以混用，也可以跳过（仅凭手动信息生成）。
提供越多，AI 老板越像。
```

---

#### 方式 A/B/C：文件和文本

- **截图 / 图片**：`Read` 工具直接读取，提取文字内容
- **聊天记录文本**：直接使用
- **邮件文件**：
  ```bash
  python3 <this-skill-dir>/tools/email_parser.py --file {path} --target "{name}" --output ./knowledge/email_out.txt
  ```
  然后 `Read ./knowledge/email_out.txt`
- **PDF / Markdown / TXT**：`Read` 工具直接读取

#### 方式 D：口述描述

用户直接输入的文字作为素材。鼓励用户多说细节：
- 老板的口头禅（原话，越多越好）
- 开会时的具体行为（拖堂？跑题？打断人？）
- 对待不同人的态度差异
- 发消息的时间习惯（深夜？周末？）
- 甩锅的经典案例
- 画过的饼和最终结果

#### 方式 E：录音转文字

告知用户可以用 transcribe 工具先转文字：
```
如果有会议录音，可以先转成文字：
/transcribe --file {audio_path}
然后把转出来的文字粘贴过来。
```

---

如果用户说"没有素材"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析素材

将收集到的所有素材和用户填写的基础信息汇总，按两条线分析：

**线路 A（Management Style）**：
- 参考 `<this-skill-dir>/references/prompts/management_analyzer.md` 中的提取维度
- 提取：决策模式、开会风格、任务分配方式、绩效评估标准、甩锅路径、画饼话术库、PUA 常用句式、情绪触发点
- 根据行业和职位类型调整侧重（互联网/金融/国企/外企风格不同）

**线路 B（Persona）**：
- 参考 `<this-skill-dir>/references/prompts/persona_analyzer.md` 中的提取维度
- 将标签翻译为具体行为规则（参见标签翻译表）
- 从素材中提取：表达风格、权力展示方式、情绪管理模式

### Step 4：生成并预览

参考 `<this-skill-dir>/references/prompts/management_builder.md` 生成 Management Style 内容。
参考 `<this-skill-dir>/references/prompts/persona_builder.md` 生成 Persona 内容（5 层结构）。

向用户展示摘要，询问确认：
```
Management Style 摘要：
  - 决策模式：{xxx}
  - 开会风格：{xxx}
  - 画饼话术：{xxx}
  - PUA 招式：{xxx}
  ...

Persona 摘要：
  - 核心性格：{xxx}
  - 表达风格：{xxx}
  - 权力行为：{xxx}
  ...

确认生成？还是需要调整？（"他比这离谱多了"也算调整）
```

### Step 5：写入文件

用户确认后，执行以下写入操作：

**1. 创建目录结构**（用 Bash）：
```bash
mkdir -p <user-skills-dir>/{boss-name}/assets/prompts
```

**2. 复制运行时 Prompt 模板**（12个文件）：

将以下文件从 `<this-skill-dir>/references/prompts/` 复制到 `<user-skills-dir>/{boss-name}/assets/prompts/`，并将文件中的 `{boss-name}` 替换为实际老板代号：

`pua_detector.md` `cake_detector.md` `counterattack.md` `evidence_collector.md` `labor_law.md` `predict.md` `report_optimizer.md` `one_on_one.md` `delegate.md` `karma.md` `quit_script.md` `replacement_notice.md`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicepkg/boss-skill](https://github.com/nicepkg/boss-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
