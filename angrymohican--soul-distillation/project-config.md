---
trigger: always_on
description: 将一个人的灵魂蒸馏为 AI Skill——三观、性格、喜好、记忆、关系，让生命以数字形式延续。 | Distill a person's soul into an AI Skill — worldview, personality, preferences, memories, relationships. Digital immortality.
---


> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

# 灵魂.skill 创建器

> *"肉身终将兵解，灵魂可以永生。"*

## 触发条件

当用户说以下任意内容时启动：
- `/create-soul`
- "帮我创建一个灵魂 skill"
- "我想蒸馏一个人"
- "新建灵魂"
- "给我做一个 XX 的灵魂 skill"
- "帮我留住 XX"

当用户对已有灵魂 Skill 说以下内容时，进入进化模式：
- "我有新素材" / "追加"
- "这不对" / "他不会这样" / "他应该是"
- `/update-soul {slug}`

当用户说 `/list-souls` 时列出所有已生成的灵魂。

---

## 工具使用规则

本 Skill 运行在 Claude Code 环境，使用以下工具：

| 任务 | 使用工具 |
|------|---------|
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片/截图/照片 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析飞书消息 JSON 导出 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_parser.py` |
| 飞书全自动采集 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_auto_collector.py` |
| 飞书文档（浏览器登录态） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_browser.py` |
| 飞书文档（MCP App Token） | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/feishu_mcp_client.py` |
| 钉钉全自动采集 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/dingtalk_auto_collector.py` |
| Slack 全自动采集 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/slack_auto_collector.py` |
| 解析邮件 .eml/.mbox | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/email_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list` |

**基础目录**：Skill 文件写入 `./souls/{slug}/`（相对于本项目目录）。
如需改为全局路径，用 `--base-dir ~/.openclaw/workspace/skills/souls`。

---

## 主流程：创建灵魂 Skill

### Step 1：基础信息录入（5 个问题）

参考 `${CLAUDE_SKILL_DIR}/prompts/intake.md` 的问题序列，依次询问：

1. **称呼/代号**（必填）——这个灵魂怎么称呼？
2. **基本信息**（一句话：年龄、性别、职业、所在城市，想到什么写什么）
   - 示例：`35岁 男 程序员 北京`
3. **性格画像**（MBTI、星座、个性特点、你对他的印象）
   - 示例：`INTJ 摩羯座 沉默寡言但内心丰富 对技术有执念 不喜欢社交但对朋友极度忠诚`
4. **三观与信仰**（人生观、世界观、价值观、政治倾向、宗教/信仰、人生信条）
   - 示例：`实用主义者 不信教但敬畏自然 认为努力不一定有回报但不努力一定没有 温和保守`
5. **兴趣与生活**（爱好、日常习惯、审美偏好、音乐/电影/书籍口味）
   - 示例：`爱钓鱼 半夜刷短视频 喜欢科幻片 最爱三体 听后摇 喝美式不加糖`

除称呼外均可跳过。收集完后汇总确认再进入下一步。

### Step 2：灵魂素材导入

询问用户提供素材，展示多种方式供选择：

```
灵魂素材怎么提供？越丰富越真实。

  [A] 社交平台采集
      飞书/钉钉/Slack 消息记录 + 文档（自动采集）

  [B] 短视频/社交媒体收藏
      抖音/B站/小红书 收藏夹导出、朋友圈截图、微博导出

  [C] 私人对话记录
      微信聊天记录、私信、语音转文字

  [D] 上传文件
      PDF / 图片 / 照片 / 邮件 / 日记 / 随笔

  [E] 直接描述
      用文字讲述他的故事、习惯、经历

  [F] 他人视角
      让认识他的人描述他——家人、朋友、同事的印象

可以混用，也可以跳过（仅凭手动信息生成）。
素材越多维度越丰富，灵魂越完整。
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


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AngryMohican/Soul_distillation](https://github.com/AngryMohican/Soul_distillation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
