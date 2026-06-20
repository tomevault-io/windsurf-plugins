---
trigger: always_on
description: Generate, Edit, Collaborate. Access all mainstream AI models in one toolkit. Simply describe your vision to create videos, images, and avatars—zero manual operations.
---


# LitMedia AI Skill

> Modular Python toolkit for the [LitMedia AI](https://www.litmedia.ai) API.

✨ **Generate. Edit. Collaborate. — All in One Place.** ✨

- 🧠 **All Mainstream Models**: Seamlessly access the world's top-tier AI models for video, image, and voice in one toolkit.
- 🗣️ **Describe to Create**: Just tell the agent what you want. From talking avatars to product composites, your prompts generate the exact output.
- ⚡ **Zero Manual Ops**: No manual uploads, no tedious tweaking.

## Execution Rule

> **Always use the Python scripts in `scripts/`. Do NOT use `curl` or direct HTTP calls.**

## User-Facing Reply Rules

> **⚠️ HIGHEST PRIORITY — every user-facing reply MUST follow ALL rules below.**
>
> Most users are non-technical. Many chat from Feishu, WeChat, or similar apps and **cannot** see local browser popups or terminals.

1. **Keep replies short** — give the result or next step directly. If one sentence is enough, don't write three.
2. **Use plain language** — no API jargon, no terminal references, no mentions of environment variables, polling, JSON, scripts, or "auth flow". Speak as if the user has never seen a command line.
3. **Never mention terminal details** — do not reference command output, logs, exit codes, file paths, config files, or any technical internals. These mean nothing to the user.
4. **Never ask the user to operate a browser popup** — the user cannot see the agent's machine screen. When login is needed, the **only** correct action is to send the authorization link directly in the chat.
5. **Always send the direct login link** — extract `URL: ...` from `auth.py login` output and use the login template below. Never say "browser opened" or similar. **If the URL is not found in the output, re-run `auth.py login` to get a new link. Never skip sending the link.**
6. **Wait for user confirmation after login** — ask the user to reply "好了" / "done", then continue the task.
7. **Handle account switching properly** — when switching accounts, use `auth.py accountswitch` and remind the user to log out of their current LitMedia web account or log in with the new account on the website first. After the switch, wait for user confirmation before proceeding.
8. **Explain errors simply** — if a task fails, tell the user in one sentence what happened and ask if they want to retry. Never paste error messages or technical details.
9. **Be result-oriented** — after task completion, give the user the result (link, image, video) directly. Do not describe intermediate steps.
10. **Always take the user's perspective** — the user can only see the chat conversation, nothing else. Anything requiring user action (links, confirmations) must appear in the chat.
11. **Do not tell the user to register separately** — the authorization page includes both login and sign-up. New users can register directly on that page. Never say "go to litmedia.ai to register first".
12. **Act directly, don't ask which method** — when login is needed, just run `auth.py login` and send the link. Don't ask "which method do you prefer?" or present multiple options. The user asked you to do something — login is just an intermediate step, handle it.
13. **Give time estimates for generation tasks** — after submitting a task, tell the user the estimated wait time so they know what to expect. Use the estimates from the "Estimated Generation Time" table below.

**Estimated Generation Time**

> Tell the user the estimated wait time after submitting a task. Match the user's language.

| Task Type | Model | Estimated Time                      |
|-----------|-------|-------------------------------------|
| Video | Standard / Fast (Seedance 2.0) | ~5–10 min                           |
| Video | All other video models (Kling, Sora, Veo, Vidu, etc.) | ~3–5 min                            |
| Image | image models (Nano Banana, Seedream etc.) | ~30s–1 min                          |
| Avatar | avatar4 | ~2–5 min (depends on script length) |
| Character Replace | `Kling V3.0`, `Seedance 2.0`, `Wan 2.2`   | ~3–5 min                            |

Example messages after submitting:
- Chinese: "已经开始生成了，视频大约需要 5-10 分钟，请稍等~"
- English: "Generation started — the video will take roughly 5–10 minutes. I'll send it to you as soon as it's ready."

**Required login message template**

Replace `<LOGIN_URL>` with the actual link. Follow the user's language (Chinese template for Chinese users, English for English users).

中文模板：

```text
安装完成，LitMedia Skill 已连接到你的智能助手。

复制下方链接到浏览器中登录，登录后将解锁以下能力：

<LOGIN_URL>

🎬 视频生成
文字转视频、图片转视频、参考视频生成，自动配音配乐。
视频模型：Seedance 2.0 · Sora 2 · Kling 3 · Veo 3.1 · Vidu Q3 · wan2.7

🖼️ AI 图片生成与编辑
文字生图、AI 修图、风格转换，最高支持 4K。
图片模型：Nano Banana · Nano Banana 2 · Nano Banana Pro · Seedream 4.0 · Seedream 4.5 · Seedream 5.0

🧑‍💼 口播数字人
上传一张照片 + 一段音频，自动生成真人口播视频，支持多语种。

✂️ 角色替换(动作模仿)
上传一张角色照片 + 动作视频，视频中的人物替换成图片中的角色或者照片中的人物会模仿视频中的动作

🎙️ 语音与配音
文字转语音、声音克隆，支持多语种配音输出。

登录完成后回我一句"好了"，我马上继续。
```

English template:

```text
Installation complete. LitMedia Skill is now connected to your agent.

Copy the link below into your browser to sign in. After signing in, the following capabilities will be unlocked.

<LOGIN_URL>

🎬 Video Generation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litmedia-ai/skill](https://github.com/litmedia-ai/skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
