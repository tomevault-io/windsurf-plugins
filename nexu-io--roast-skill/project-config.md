---
trigger: always_on
description: 先蒸馏自己（或同事）的工作数据（飞书消息、推特、文章等），再生成毒舌但精准的牛马锐评报告。输出包含个人标签、牛马指数、能力条、8 维度深度扒皮、AI 对话示例，并自动生成可分享的落地页。触发词：锐评、牛马、吐槽、roast、蒸馏、赛博分身。
---


# 🔥 牛马锐评 — AI 蒸馏你的工作人格

> ## 🚨 最重要的 3 条规则（必须遵守，没有例外）
>
> **规则 1：Step 0 必须真实执行。** 你必须调用 `feishu_app_scopes` 工具检测飞书权限，不能跳过，不能猜测，不能假设。没有检测就没有资格进入 Step 1。
>
> **规则 2：飞书权限缺失时，先引导开通，不能直接降级。** 检测到没有 `im:message:readonly` 权限时，必须引导用户去飞书管理后台开通。**只有用户明确说"不想开通/跳过"之后**，才能提供"手动粘贴消息"作为 fallback。绝不能主动说"你可以手动粘贴消息给我"。
>
> **规则 3：必须确认目标用户身份，不能搞错人。** 从名字找到飞书用户时，如果有多个同名用户必须让用户确认；只有一个也要跟用户确认是不是 TA。绝不能猜、不能随便选。
>
> **规则 4：不要把内部过程输出给用户。** Step 0-3 的所有检测、采集、分析过程都是内部操作。用户看到的只有 3 样东西：
> 1. Step 1 的能力清单（🔥 锐评模式启动！）
> 2. 最终落地页链接 + 分享文案
> 3. 需要用户操作时的引导（如 OAuth 授权、补充数据）
>
> **以下内容绝对不能出现在发给用户的消息中：**
> - 英文的内部推理（"The user wants me to..."、"I must strictly follow..."、"Step 0: Environment capability detection"）
> - 工具调用过程（"Let me check..."、"Now let me call..."、"I need to run..."）
> - 检测步骤描述（"I must check: 1. Twitter capability 2. Feishu..."）
> - 技术细节（open_id、API 名称、脚本路径、错误码等）
> - 中间进度（"正在采集消息数据..."、"静默执行 Step 2"）
>
> **正确做法：** 静默完成检测 → 直接输出 Step 1 能力清单。用户看不到任何中间过程。
>
> **规则 5：锐评内容不在聊天中输出。** 所有锐评文本只写入落地页，用户看到的只有链接 + 分享文案。
>
> **规则 6：飞书采集必须先走 OAuth 授权流程。** 检查 `~/.nexu/feishu-user-token.json`，不存在就引导用户 OAuth 授权。**不要跳过 OAuth 直接用 Bot token 采集，不要建议用户"把 Bot 拉到群里"。** Bot token 模式只是 fallback，只有用户明确拒绝 OAuth 时才用。
>
> **规则 7：锐评的唯一输出形式是 nexu.space 落地页链接。** 部署功能已内置在 `deploy/` 子目录中，首次使用需要自动 `npm install`。**绝对不能自己发明替代方案**（Vercel、Gist、GitHub Pages、本地 HTML 等）。
>
> **规则 8：飞书数据采集必须用本 skill 自带的 python 脚本。** 用 `python3 <skill_dir>/scripts/feishu_fetch.py` 来搜索用户和采集消息。**不要自己调飞书 API、不要用 feishu_chat 等内置工具、不要手动拼接 API 请求。** 脚本会自动处理 OAuth token、搜索目标用户、遍历群聊、采集消息。凭证从 Agent 配置文件自动读取（见下方 Step 2 说明）。
>
> **规则 9：App Secret 绝对不能出现在聊天记录中。** Secret 只能通过配置文件或环境变量传递。**不要让用户在聊天中发送 App Secret**——即使 Secret 报错（如错误码 10014），也只能引导用户在飞书开放平台的「凭证与基础信息」页面确认 Secret，然后更新到 Agent 的配置文件中。正确的引导是："请到飞书开放平台确认 App Secret，然后更新你 Agent 的配置文件"，而不是"把 Secret 发给我"。
>
> **规则 10：锐评只能基于采集到的真实数据，不能用 agent 自己的 memory/记忆脑补。** 如果采集到的消息为 0 条或极少（<5 条），**必须告诉用户数据不足**，让用户补充（手动粘贴消息、提供推特链接/文章 URL 等）。**不能用自己的 memory 文件、历史对话、或自己对这个人的"印象"来生成锐评**——这样生成的内容极可能不准确（比如把研发说成运营）。数据不足时的话术：
> ```
> ⚠️ 采集到的飞书数据太少（只有 X 条），生成的锐评可能不够精准。
> 建议补充以下任意信息让锐评更到位：
> 1. TA 的推特/X 链接
> 2. TA 写的文章/博客 URL
> 3. 手动粘贴 TA 最近 20-30 条飞书消息
> ```

## 安装完成后的引导（必须执行）

当用户刚安装完 skill 时，**必须**主动告诉用户怎么开始使用：

```
🔥 roast-skill 安装完成！

现在你可以开始锐评了，输入：
👉 锐评 <名字>

例如：
  锐评 张三
  锐评 Elon Musk
  roast @elonmusk
```

**不要只说"可以开始了"就结束——必须给出明确的输入格式和示例。**

---

## 执行流程（必须严格按顺序执行）

当用户触发锐评请求时，按以下 5 步执行。**不要跳步，不要用历史 context 代替数据采集。**

### Step 0: 环境能力检测（⛔ 必须执行，不可跳过）

**🚨🚨🚨 这是整个流程最重要的一步。如果你跳过这一步，后面所有步骤都会失败。🚨🚨🚨**

在回复用户任何内容之前，你**必须先执行以下 3 个检测**。不检测就回复 = 流程错误。

**规则（没有例外）：**
- ✅ 必须**真实调用工具**检测，不能凭猜测标 ✅ 或 ❌
- ✅ 必须在**第一条回复之前**完成所有检测
- ❌ 不能跳过检测直接问用户要数据
- ❌ 不能因为"我觉得可能没权限"就直接告诉用户手动粘贴
- ❌ 不能说"我暂时无法读取飞书消息"然后就降级——你必须先检测再说话

#### 检测项 1: 推特/X 读取能力

**执行动作：** 运行 `which twitter` 或 `which bird` 命令。

- 命令找到了 → 再运行 `twitter status`
  - `authenticated: true` → ✅ 推特就绪
  - `authenticated: false` → ⚠️ 需引导配置 Cookie（见「推特/X 从零配置指南」第 2-3 步）
- 命令没找到 → ❌ 需引导安装（见「推特/X 从零配置指南」完整 3 步）

#### 检测项 2: 飞书能力检测（⛔ 关键 — 只需跑一条命令）

> **🚨 不要自己调用 feishu_app_scopes、不要自己检查权限列表、不要自己判断能力。** 直接运行下面的命令，脚本会告诉你一切。

**执行命令：**
```bash
python3 <skill_dir>/scripts/feishu_fetch.py --verify
```

**脚本返回 JSON 报告。你只需要看 3 个字段：**

**① `guidance` 数组（最重要）：**
- **不为空** → ⛔ **STOP。把 guidance 里的每一条直接发给用户**（已经是中文，直接复制发送）。然后等用户回复。
- **为空** → ✅ 飞书能力完整，继续下一个检测项。

**② `oauthTokenValid`：**
- **false 且 guidance 里提到需要 OAuth** → 引导用户授权（见下方 OAuth 流程）
- **true** → OAuth 已完成

**③ `userCanListChats`：**
- **false** → 用户级权限缺失，guidance 里已经有引导了，照发就行
- **true** → 用户级权限完整

> 💡 **这条命令检测了所有飞书相关能力**：凭证、Bot 权限、OAuth token、用户级权限、消息实际可读性。agent 不需要理解这些概念，照着 guidance 输出做就行。

**⛔ 共同行为规则（不管哪种失败）：**
- 发送 guidance 后**停止执行，等待用户回复**
- 用户说「好了」→ **重新运行 `--verify` 再检测一次**
- 用户说「跳过」/「不想配」→ 这时候才可以降级
- **❌ 不要自动降级、不要长篇解释技术原因、不要主动说"你也可以手动粘贴消息"**

---

**OAuth 授权流程（仅当 `--verify` 报告 oauthTokenValid=false 时执行）：**

1. 生成授权链接：
```bash
python3 <skill_dir>/scripts/feishu_fetch.py --oauth-url
```

2. 把脚本输出的链接发给用户（把 `{app_id}` 换成实际值，`{oauth_url}` 换成脚本输出）：
```
🔐 首次使用需要飞书授权（仅一次），这样我就能读取你所有群聊的消息，锐评更精准。

⚠️ 首次授权前需要先配置一下（30 秒搞定）：
1. 打开你的应用：https://open.feishu.cn/app/{app_id}
2. 点左侧菜单「安全设置」
3. 在「重定向 URL」里添加：https://open.feishu.cn/document/home/index
4. 保存后再点击下面的授权链接

👉 点击授权：{oauth_url}

授权后，把浏览器地址栏中 code= 后面的那串字符发给我就行。
```

3. 用户发回 code 后换取 token：
```bash
python3 <skill_dir>/scripts/feishu_fetch.py --auth-code <用户发的code>
```

4. 成功后**再跑一次 `--verify`** 确认能力完整。

**常见报错（agent 内部参考）：**
- 20029 = redirect_uri 没配 → 引导用户做上面第 2-3 步
- 10014 = App Secret 不对 → "请到飞书开放平台确认 Secret，更新 Agent 配置文件" **（⛔ 不要让用户在聊天里发 Secret）**
- 20003 = code 已用过 → 让用户重新点授权链接

#### 检测项 3: 网页抓取能力
- 检查 `web_fetch` 工具是否可用 → 通常都有 ✅

#### 检测项 4: 落地页部署能力（内置在 roast-skill 中）

**执行动作：** 分 3 步检查：

**4a. 检查 deploy 脚本存在：**
```bash
ls <skill_dir>/deploy/scripts/deploy_skill.js
```
- 不存在 → roast-skill 安装不完整，重新安装。

**4b. 检查 jszip 依赖：**
```bash
ls <skill_dir>/deploy/node_modules/jszip/package.json 2>/dev/null || (cd <skill_dir>/deploy && npm install)
```

**4c. 检查 Nexu 登录态（⛔ 关键 — 必须提前检测）：**
```bash
node <skill_dir>/deploy/scripts/deploy_skill.js check
```
- 返回 `"status": "ok"` → ✅ 部署就绪
- 报错包含 `"log in to your Nexu account"` → ⛔ **STOP，告诉用户：**
  ```
  ⚠️ 部署落地页需要先登录 nexu 账号。请在 nexu desktop app 中登录后再试。
  ```
  **不要等到最后部署阶段才发现这个问题——必须在 Step 0 就检测到并提前告知用户。**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nexu-io/roast-skill](https://github.com/nexu-io/roast-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
