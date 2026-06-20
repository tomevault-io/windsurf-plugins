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

**群聊采集**（使用 tenant_access_token，需 bot 在群内）：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --name "{name}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 1000 \
  --doc-limit 20
```

**私聊采集**（需要 user_access_token + 私聊 chat_id）：

私聊消息只能通过用户身份（user_access_token）获取，应用身份无权访问私聊。

**前置条件**：

用户需要提供以下信息：
1. **飞书应用凭证**：`app_id` 和 `app_secret`（在飞书开放平台创建自建应用获取）
2. **用户权限**：应用需开通以下用户权限（scope）：
   - `im:message` — 以用户身份读取/发送消息
   - `im:chat` — 以用户身份读取会话列表
3. **OAuth 授权码（code）**：用户在浏览器中完成 OAuth 授权后，从回调 URL 中获取

如果用户缺少以上任何信息，引导他们完成配置。不要假设用户已经配好了。

**获取 user_access_token 的完整流程**：

当用户提供了 app_id、app_secret，并确认已开通用户权限后：

1. 帮用户生成 OAuth 授权链接：
   ```
   https://open.feishu.cn/open-apis/authen/v1/authorize?app_id={APP_ID}&redirect_uri=http://www.example.com&scope=im:message%20im:chat
   ```
   > ⚠️ 注意：`redirect_uri` 需要在飞书应用的「安全设置 → 重定向 URL」中添加 `http://www.example.com`
   
2. 用户在浏览器打开链接，登录并授权
3. 页面会跳转到 `http://www.example.com?code=xxx`，用户复制 code 给你
4. 用 code 换取 token：
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py --exchange-code {CODE}
   ```
   或者你自己写 Python 脚本调飞书 API 换取：
   ```python
   # 1. 获取 app_access_token
   POST https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal
   Body: {"app_id": "xxx", "app_secret": "xxx"}
   
   # 2. 用 code 换 user_access_token
   POST https://open.feishu.cn/open-apis/authen/v1/oidc/access_token
   Header: Authorization: Bearer {app_access_token}
   Body: {"grant_type": "authorization_code", "code": "xxx"}
   ```

**获取私聊 chat_id**：

用户通常不知道 chat_id。当用户有了 user_access_token 但没有 chat_id 时，你应该**自己写 Python 脚本**来获取：

- **方法**：用 user_access_token 向对方的 open_id 发一条消息，返回值中会包含 chat_id
  ```python
  POST https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id
  Header: Authorization: Bearer {user_access_token}
  Body: {"receive_id": "{对方open_id}", "msg_type": "text", "content": "{\"text\":\"你好\"}"}
  # 返回值中的 chat_id 就是私聊会话 ID
  ```
- **注意**：`GET /im/v1/chats` 不会返回私聊会话，这是飞书 API 的限制，不是权限问题，不要尝试用这个接口找私聊
- 如果用户不知道对方的 open_id，可以用 tenant_access_token 调通讯录 API 搜索：
  ```python
  GET https://open.feishu.cn/open-apis/contact/v3/scopes
  # 返回应用可见范围内所有用户的 open_id
  ```

**执行采集**：

拿到 user_access_token 和 chat_id 后：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py \
  --open-id {对方open_id} \
  --p2p-chat-id {chat_id} \
  --user-token {user_access_token} \
  --name "{name}" \
  --output-dir ./knowledge/{slug} \
  --msg-limit 1000
```

**灵活性原则**：以上 API 调用不一定要用 collector 脚本，如果脚本跑不通或者场景不匹配，你可以直接写 Python 脚本调飞书 API 完成任务。核心 API 参考：
- 获取 token：`POST /auth/v3/app_access_token/internal`、`POST /authen/v1/oidc/access_token`
- 发消息（获取 chat_id）：`POST /im/v1/messages?receive_id_type=open_id`
- 拉消息：`GET /im/v1/messages?container_id_type=chat&container_id={chat_id}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kaitzz/ququ-skill](https://github.com/Kaitzz/ququ-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
