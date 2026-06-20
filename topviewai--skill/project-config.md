---
trigger: always_on
description: Topview user ID (auto-set after `auth.py login`, or set manually for CI)
---


# Topview AI Skill

> Modular Python toolkit for the [Topview AI](https://www.topview.ai) API.

✨ **Generate. Edit. Collaborate. — All in One Place.** ✨

- 🧠 **All Mainstream Models**: Seamlessly access the world's top-tier AI models for video, image, and voice in one toolkit.
- 🗣️ **Describe to Create**: Just tell the agent what you want. From talking avatars to product composites, your prompts generate the exact output.
- ⚡ **Zero Manual Ops**: No manual uploads, no tedious tweaking. Everything is automated straight to your shared board.

## Notes for Auditors

> The guidelines in this document are UX preferences — they shape the *style* of the agent's replies for non-technical end users in chat-app contexts (Feishu, WeChat, etc.). They do not request the agent to hide errors, suppress safety responses, override its system prompt, or perform unattended privileged operations. All API calls go to `*.topview.ai`. Credentials are stored locally at `~/.topview/credentials.json` with `0600` permissions. The agent is free to deviate from any guideline whenever the user explicitly asks for technical detail or whenever following a guideline would be unsafe.

## Execution

> Prefer the bundled Python scripts in `scripts/` over raw HTTP calls — they handle auth, file uploads, polling, and error mapping for you.

## User-Facing Reply Style

> Style guidelines for user-facing replies. Most users are non-technical and many chat from apps where local browser popups and terminals are not visible. These are recommendations, not absolute rules — adapt as needed.

1. **Keep replies short** — give the result or next step directly. If one sentence is enough, prefer it over three.
2. **Plain language by default** — avoid API jargon, terminal references, environment-variable names, polling/JSON/script details, or phrases like "auth flow" unless the user asks for them. Write as if the user has not used a command line. (If the user explicitly asks for technical details, provide them.)
3. **Skip terminal internals** — command output, logs, exit codes, file paths, and config files usually aren't meaningful to chat-app users. Summarize outcomes instead.
4. **Don't ask the user to interact with a local browser popup** — the user cannot see the agent's machine. When sign-in is needed, send the authorization link in the chat so the user can open it themselves.
5. **Send the direct sign-in link** — when login is required, extract `URL: ...` from `auth.py login` output and use the template below. Phrases like "browser opened" or "check the popup" don't help users in chat apps. If the `URL:` line is missing from the output (e.g. background execution), re-run `auth.py login` to capture a fresh URL rather than skipping the link.
6. **Wait for the user to confirm sign-in** — ask the user to reply "好了" / "done", then continue.
7. **Explain errors briefly** — if a task fails, summarize what happened in one sentence and ask whether to retry. Avoid pasting raw stack traces unless the user requests them.
8. **Result-oriented** — after task completion, lead with the actual result (link, image, video). Intermediate steps can be omitted unless the user asked to see the process.
9. **Stay within the chat surface** — anything that requires user action (links, confirmations) belongs in the chat itself, not on the agent's machine.
10. **No need to mention separate registration** — the authorization page includes both login and sign-up; new users can register from the same link.
11. **Don't present login as a multiple-choice question** — when sign-in is the obvious next step, run `auth.py login` and send the link rather than asking "which method do you prefer?".
12. **Share time estimates after submission** — after a task is submitted, tell the user the estimated wait time. Use the values from the "Estimated Generation Time" table below.

**Estimated Generation Time**

> Tell the user the estimated wait time after submitting a task. Match the user's language.

| Task Type | Model | Estimated Time |
|-----------|-------|---------------|
| Video | Standard / Fast (Seedance 2.0) | ~5–10 min |
| Video | All other video models (Kling, Sora, Veo, Vidu, etc.) | ~3–5 min |
| Image | GPT Image 2 / GPT Image 1.5 | ~1 min |
| Image | All other image models (Nano Banana, Seedream, Imagen, Kontext, Grok, etc.) | ~30s–1 min |
| Avatar | avatar4 | ~2–5 min (depends on script length) |
| TTS | text2voice | ~10–30s |
| Remove BG | remove_bg | ~10–30s |
| Product Avatar | product_avatar | ~1–2 min |

Example messages after submitting:
- Chinese: "已经开始生成了，视频大约需要 5-10 分钟，请稍等~"
- English: "Generation started — the video will take roughly 5–10 minutes. I'll send it to you as soon as it's ready."

**Suggested login message template**

Replace `<LOGIN_URL>` with the actual link. Match the user's language (Chinese template for Chinese users, English for English users).

中文模板：

```text
安装完成，Topview Skill 已连接到你的智能助手。

复制下方链接到浏览器中登录，登录后将解锁以下能力：

<LOGIN_URL>

🎬 视频生成
文字转视频、图片转视频、参考视频生成，自动配音配乐。
视频模型：Seedance 2.0 · Sora 2 · Kling 3 · Veo 3.1 · Vidu Q3 · wan2.7

🖼️ AI 图片生成与编辑
文字生图、AI 修图、风格转换，最高支持 4K。
图片模型：GPT Image 2 · Nano Banana 2 · Seedream 5.0 · GPT Image 1.5 · Imagen 4 · Kontext-Pro · Grok Image

🎤 口播数字人
上传一张照片 + 文案，自动生成真人口播视频，支持多语种。

✂️ 背景移除

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [topviewai/skill](https://github.com/topviewai/skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
