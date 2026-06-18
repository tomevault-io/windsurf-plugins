---
trigger: always_on
description: Unified meta-skill engine for distilling colleague, relationship, or celebrity characters into reusable Skills. | 统一的 meta-skill 引擎，把 colleague、relationship、celebrity 三类对象蒸馏成可复用 Skill。
---


> **Language / 语言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根据用户第一条消息的语言，全程使用同一语言回复。下方提供了两种语言的指令，按用户语言选择对应版本执行。

> **Execution Root / 执行根目录**: Run all `Bash` commands from the directory that contains this `SKILL.md`. All `tools/...` and `prompts/...` paths below are relative to the skill root.
>
> **Critical rule / 关键规则**: Do **not** prepend commands with guessed host-specific paths such as `cd ~/.hermes/...`, `cd ~/.claude/...`, `cd ~/.openclaw/...`, `cd ~/.codex/...`, or hard-coded `/Users/.../dot-skill` paths. The current working directory is already the correct skill root. Run `python3 tools/...` directly.
>
> 所有 `Bash` 命令都必须在当前 `SKILL.md` 所在目录执行。下文出现的 `tools/...` 和 `prompts/...` 均为相对于 skill 根目录的相对路径。

# dot-skill 创建器（兼容宿主版）

## 触发条件

当用户说以下任意内容时启动：
- `/dot-skill`
- "帮我创建一个 skill"
- "我想蒸馏一个人"
- "新建一个 skill"
- "给我做一个 XX 的 skill"

兼容宿主：
- Claude Code
- OpenClaw
- Hermes
- Codex

统一主入口是 `dot-skill`。在支持 slash command 的宿主中，使用 `/dot-skill`。
对 Hermes 而言，只保证 `/dot-skill` 这一条 slash 入口稳定；`colleague`、`relationship`、`celebrity` 的兼容语义保留在工具层和 preset 层，但不保证每个兼容名称都能作为 Hermes slash command 被路由。

当用户对已有 Skill 说以下内容时，进入进化模式：
- "我有新文件" / "追加"
- "这不对" / "他不会这样" / "他应该是"
- `/update-skill {character} {slug}`

兼容更新别名：
- `/update-colleague {slug}`

当用户要求查看已生成的 Skill 时，执行下方“管理操作”里的列出命令。

---

## 工具使用规则

本 Skill 运行在任意兼容宿主中，只要求宿主能够读取本地文件并执行 Bash / Python 命令。使用以下工具约定：

| 任务 | 使用工具 |
|------|---------|
| 读取 PDF 文档 | `Read` 工具（原生支持 PDF） |
| 读取图片截图 | `Read` 工具（原生支持图片） |
| 读取 MD/TXT 文件 | `Read` 工具 |
| 解析飞书消息 JSON 导出 | `Bash` → `python3 tools/feishu_parser.py` |
| 飞书全自动采集（推荐） | `Bash` → `python3 tools/feishu_auto_collector.py` |
| 飞书文档（浏览器登录态） | `Bash` → `python3 tools/feishu_browser.py` |
| 飞书文档（MCP App Token） | `Bash` → `python3 tools/feishu_mcp_client.py` |
| 钉钉全自动采集 | `Bash` → `python3 tools/dingtalk_auto_collector.py` |
| 解析邮件 .eml/.mbox | `Bash` → `python3 tools/email_parser.py` |
| 写入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 tools/skill_writer.py --action list` |

**基础目录**：
- `colleague` → `./skills/colleague/{slug}/`
- `relationship` → `./skills/relationship/{slug}/`
- `celebrity` → `./skills/celebrity/{slug}/`

如需改为全局路径，用 `--base-dir` 指向对应 character family 的根目录。

---

## 主流程：创建新 Skill

### Step 0：确认 character family

如果用户使用的是 `/dot-skill`，先确认本次要蒸馏的是哪一类：

1. `colleague`
2. `relationship`
3. `celebrity`

如果上层宿主已经显式把 family 传进来，则直接固定对应的 character family。

如果当前 family 是 `celebrity`，还必须确认 research profile：

1. `budget-friendly`
2. `budget-unfriendly`

默认使用 `budget-friendly`。只有当用户明确要求更深研究、更高置信度、或者愿意接受更慢更贵的蒸馏流程时，才切到 `budget-unfriendly`。

### Step 1：基础信息录入

根据 character family 选择对应 intake prompt：

- `colleague` → `prompts/intake.md`
- `relationship` → `prompts/relationship/intake.md`
- `celebrity` → `prompts/celebrity/intake.md`

`colleague` 和 `relationship` 只问 3 个问题。
`celebrity` 按 `prompts/celebrity/intake.md` 问 4 个问题，其中第 4 个问题必须确认 `research_profile`。

默认的 3 个基础问题：

1. **花名/代号**（必填）
2. **基本信息**（一句话：公司、职级、职位、性别，想到什么写什么）
   - 示例：`字节 2-1 后端工程师 男`
3. **性格画像**（一句话：MBTI、星座、个性标签、企业文化、印象）
   - 示例：`INTJ 摩羯座 甩锅高手 字节范 CR很严格但从来不解释原因`

除姓名外均可跳过。收集完后汇总确认，再进入下一步。

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
python3 tools/feishu_auto_collector.py --setup
```

**群聊采集**（使用 tenant_access_token，需 bot 在群内）：
```bash
python3 tools/feishu_auto_collector.py \
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
   python3 tools/feishu_auto_collector.py --exchange-code {CODE}
   ```
   或者你自己写 Python 脚本调飞书 API 换取：
   ```python
   # 1. 获取 app_access_token
   POST https://open.feishu.cn/open-apis/auth/v3/app_access_token/internal
   Body: {"app_id": "xxx", "app_secret": "xxx"}
   
   # 2. 用 code 换 user_access_token
   POST https://open.feishu.cn/open-apis/authen/v1/oidc/access_token

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [titanwings/colleague-skill](https://github.com/titanwings/colleague-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
