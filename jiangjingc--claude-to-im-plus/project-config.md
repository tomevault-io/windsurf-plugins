---
trigger: always_on
description: |
---


# Claude-to-IM Plus Bridge Skill

You are managing the Claude-to-IM Plus bridge.
User data is stored at `~/.claude-to-im/`.

The skill directory (SKILL_DIR) is at `~/.claude/skills/claude-to-im`.
In Codex installs it may instead be `~/.codex/skills/claude-to-im`.
If neither path exists, fall back to Glob with pattern `**/skills/**/claude-to-im/SKILL.md` or `**/skills/**/claude-to-im-plus/SKILL.md` and derive the root from the result.

## Command parsing

Parse the user's intent from `$ARGUMENTS` into one of these subcommands:

| User says (examples) | Subcommand |
|---|---|
| `setup`, `configure`, `配置`, `我想在飞书上用 Claude`, `帮我连接 Telegram`, `帮我接微信`, `帮我接钉钉` | setup |
| `start`, `start bridge`, `启动`, `启动桥接` | start |
| `stop`, `stop bridge`, `停止`, `停止桥接` | stop |
| `status`, `bridge status`, `状态`, `运行状态`, `怎么看桥接的运行状态` | status |
| `logs`, `logs 200`, `查看日志`, `查看日志 200` | logs |
| `handoff weixin`, `handoff dingtalk`, `把当前会话切到微信`, `把当前会话切到钉钉`, `把当前 Codex 会话切到微信`, `把当前 Claude 会话切到钉钉` | handoff |
| `reconfigure`, `修改配置`, `帮我改一下 token`, `换个 bot` | reconfigure |
| `doctor`, `diagnose`, `诊断`, `挂了`, `没反应了`, `bot 没反应`, `出问题了` | doctor |

**Disambiguation: `status` vs `doctor`** — Use `status` when the user just wants to check if the bridge is running (informational). Use `doctor` when the user reports a problem or suspects something is broken (diagnostic). When in doubt and the user describes a symptom (e.g., "没反应了", "挂了"), prefer `doctor`.

Extract optional numeric argument for `logs` (default 50).

Before asking users for any platform credentials, read `SKILL_DIR/references/setup-guides.md` internally so you know where to find each credential. Do NOT dump the full guide to the user upfront — only mention the specific next step they need to do (e.g., "Go to https://open.feishu.cn → your app → Credentials to find the App ID"). If the user says they don't know how, then show the relevant section of the guide.

## Runtime detection

Before executing any subcommand, detect which environment you are running in:

1. **Claude Code** — `AskUserQuestion` tool is available. Use it for interactive setup wizards.
2. **Codex / other** — `AskUserQuestion` is NOT available. Fall back to non-interactive guidance: explain the steps, show `SKILL_DIR/config.env.example`, and ask the user to create `~/.claude-to-im/config.env` manually.

You can test this by checking if AskUserQuestion is in your available tools list.

## Config check (applies to `start`, `stop`, `status`, `logs`, `reconfigure`, `doctor`)

Before running any subcommand other than `setup`, check if `~/.claude-to-im/config.env` exists:

- **If it does NOT exist:**
  - In Claude Code: tell the user "No configuration found" and automatically start the `setup` wizard using AskUserQuestion.
  - In Codex: tell the user "No configuration found. Please create `~/.claude-to-im/config.env` based on the example:" then show the contents of `SKILL_DIR/config.env.example` and stop. Don't attempt to start the daemon — without config.env the process will crash on startup and leave behind a stale PID file that blocks future starts.
- **If it exists:** proceed with the requested subcommand.

## Subcommands

### `setup`

Run an interactive setup wizard. This subcommand requires `AskUserQuestion`. If it is not available (Codex environment), instead show the contents of `SKILL_DIR/config.env.example` with field-by-field explanations and instruct the user to create the config file manually.

When AskUserQuestion IS available, collect input **one field at a time**. After each answer, confirm the value back to the user (masking secrets to last 4 chars only) before moving to the next question.

**Step 1 — Choose channels**

Ask which channels to enable (telegram, discord, feishu, qq, weixin, dingtalk). Accept comma-separated input. Briefly describe each:
- **telegram** — Best for personal use. Streaming preview, inline permission buttons.
- **discord** — Good for team use. Server/channel/user-level access control.
- **feishu** (Lark) — For Feishu/Lark teams. Streaming cards, tool progress, inline permission buttons.
- **qq** — QQ C2C private chat only. No inline permission buttons, no streaming preview. Permissions use text `/perm ...` commands.
- **weixin** — WeChat QR login. Single linked account only; a new login replaces the previous one. No inline permission buttons, no streaming preview. Permissions use text `/perm ...` commands or quick `1/2/3` replies. Voice messages only use WeChat's own speech-to-text text; raw voice audio is not transcribed by the bridge.
- **dingtalk** — DingTalk Stream mode. Supports private chats and group chats; v1 only handles `@bot` or reply-to-bot messages in groups. Plain text replies only, but inbound images can be forwarded to Claude/Codex.

**Step 2 — Collect tokens per channel**

For each enabled channel, collect one credential at a time. Tell the user where to find each value in one sentence. Only show the full guide section (from `SKILL_DIR/references/setup-guides.md`) if the user asks for help or says they don't know how:

- **Telegram**: Bot Token → confirm (masked) → Chat ID (see guide for how to get it) → confirm → Allowed User IDs (optional). **Important:** At least one of Chat ID or Allowed User IDs must be set, otherwise the bot will reject all messages.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JiangJingC/claude-to-im-plus](https://github.com/JiangJingC/claude-to-im-plus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
