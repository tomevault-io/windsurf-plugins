---
trigger: always_on
description: Distill a colleague into an AI Skill. Auto-collect Feishu/DingTalk data, generate Work Skill + Persona, with continuous evolution. | 把同事蒸馏成 AI Skill，自动采集飞书/钉钉数据，生成 Work + Persona，支持持续进化。
---


> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

# 同事.skill 创建器（Claude Code 版）

## 触发条件

当用户说以下任意内容时启动：
- `/create-colleague`
- "帮我创建一个同事 skill"
- "我想蒸馏一个同事"
- "新建同事"
- "给我做一个 XX 的 skill"

当用户对已有同事 Skill 说以下内容时，进入进化模式：
- "我有新文件" / "追加"
- "这不对" / "他不会这样" / "他应该是"
- `/update-colleague {slug}`

当用户说 `/list-colleagues` 时列出所有已生成的同事。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------|
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析飞书消息 JSON 导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py` |
| 飞书全自动采集（推荐） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py` |
| 飞书文档（浏览器登录态） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py` |
| 飞书文档（MCP App Token） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py` |
| 钉钉全自动采集 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py` |
| 解析邮件 .eml/.mbox | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./colleagues/{slug}/`（相对于本项目目录）。
如需改为全局路径，用 `--base-dir ~/.openclaw/workspace/skills/colleagues`。

---

## 主流程：创建新同事 Skill

### Step 1：基础信息录入（3 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，只问 3 个问题：

1. **花名/代号**（必填）
2. **基本信息**（一句话：公司、职级、职位、性别，想到什么写什么）
   - 示例：`字节 2-1 后端工程师 男`
3. **性格画像**（一句话：MBTI、星座、个性标签、企业文化、印象）
   - 示例：`INTJ 摩羯座 甩锅高手 字节范 CR很严格但从来不解释原因`

除姓名外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：原材料导入

询问用户提供原材料，展示四种方式供选择：

```
原材料怎么提供？

  [A] 飞书自动采集（推荐）
      输入姓名，自动拉取消息记录 + 文档 + 多维表格

  [B] 钉钉自动采集
      输入姓名，自动拉取文档 + 多维表格
      消息记录通过浏览器采集（钉钉 API 不支持历史消息）

  [C] 飞书链接
      直接给文档/Wiki 链接（浏览器登录态 或 MCP）

  [D] 上传文件
      PDF / 图片 / 导出 JSON / 邮件 .eml

  [E] 直接粘贴内容
      把文字复制进来

可以混用，也可以跳过（仅凭手动信息生成）。
```

---

#### 方式 A：飞书自动采集（推荐）

首次使用需配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --setup
```

配置完成后，只需输入姓名，自动完成所有采集：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --name "{name}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 1000 \
  --doc-limit 20
```

自动采集内容：
- 所有与他共同群聊中他发出的消息（过滤系统消息、表情包）
- 他创建/编辑的飞书文档和 Wiki
- 相关多维表格（如有权限）

采集完成后用 `Read` 读取输出目录下的文件：
- `knowledge/{slug}/messages.txt` → 消息记录
- `knowledge/{slug}/docs.txt` → 文档内容
- `knowledge/{slug}/collection_summary.json` → 采集摘要

如果采集失败（权限不足 / bot 未加群），告知用户需要：
1. 将飞书 App bot 添加到相关群聊
2. 或改用方式 B/C

---

#### 方式 B：钉钉自动采集

首次使用需配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py --setup
```

然后输入姓名，一键采集：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py \
  --name "{name}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 500 \
  --doc-limit 20 \
  --show-browser   # 首次使用加此参数，完成钉钉登录
```

采集内容：
- 他创建/编辑的钉钉文档和知识库
- 多维表格
- 消息记录（⚠️ 钉钉 API 不支持历史消息拉取，自动切换浏览器采集）

采集完成后 `Read` 读取：
- `knowledge/{slug}/docs.txt`
- `knowledge/{slug}/bitables.txt`
- `knowledge/{slug}/messages.txt`

如消息采集失败，提示用户截图聊天记录后上传。

---

#### 方式 C：上传文件

- **PDF / 图片**：`Read` 工具直接读取
- **飞书消息 JSON 导出**：
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py --file {path} --target "{name}" --output /tmp/feishu_out.txt
  ```
  然后 `Read /tmp/feishu_out.txt`
- **邮件文件 .eml / .mbox**：
  ```bash
  python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py --file {path} --target "{name}" --output /tmp/email_out.txt
  ```
  然后 `Read /tmp/email_out.txt`
- **Markdown / TXT**：`Read` 工具直接读取

---

#### 方式 B：飞书链接

用户提供飞书文档/Wiki 链接时，询问读取方式：

```
检测到飞书链接，选择读取方式：

  [1] 浏览器方案（推荐）
      复用你本机 Chrome 的登录状态
      ✅ 内部文档、需要权限的文档都能读
      ✅ 无需配置 token
      ⚠️  需要本机安装 Chrome + playwright

  [2] MCP 方案
      通过飞书 App Token 调用官方 API
      ✅ 稳定，不依赖浏览器
      ✅ 可以读消息记录（需要群聊 ID）
      ⚠️  需要先配置 App ID / App Secret
      ⚠️  内部文档需要管理员给应用授权

选择 [1/2]：
```

**选 1（浏览器方案）**：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py \
  --url "{feishu_url}" \
  --target "{name}" \
  --output /tmp/feishu_doc_out.txt
```
首次使用若未登录，会弹出浏览器窗口要求登录（一次性）。

**选 2（MCP 方案）**：

首次使用需初始化配置：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py --setup
```

之后直接读取：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --url "{feishu_url}" \
  --output /tmp/feishu_doc_out.txt
```

读取消息记录（需要群聊 ID，格式 `oc_xxx`）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py \
  --chat-id "oc_xxx" \
  --target "{name}" \
  --limit 500 \
  --output /tmp/feishu_msg_out.txt
```

两种方式输出后均用 `Read` 读取结果文件，进入分析流程。

---

#### 方式 C：直接粘贴

用户粘贴的内容直接作为文本原材料，无需调用任何工具。

---

如果用户说"没有文件"或"跳过"，仅凭 Step 1 的手动信息生成 Skill。

### Step 3：分析原材料

将收集到的所有原材料和用户填写的基础信息汇总，按以下两条线分析：

**线路 A（Work Skill）**：
- 参考 `${CLAUDE_SKILL_DIR}/prompts/work_analyzer.md` 中的提取维度
- 提取：负责系统、技术规范、工作流程、输出偏好、经验知识
- 根据职位类型重点提取（后端/前端/算法/产品/设计不同侧重）


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChenZhongxi-sjtu/WanHunFan-Cyber-company](https://github.com/ChenZhongxi-sjtu/WanHunFan-Cyber-company) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
