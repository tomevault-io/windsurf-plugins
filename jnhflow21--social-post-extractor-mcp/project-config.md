---
trigger: always_on
description: Use this file when a user gives you this repository link and asks you to install `social-post-extractor-mcp`.
---

# Agent Install Protocol

Use this file when a user gives you this repository link and asks you to install `social-post-extractor-mcp`.

Your job is to configure the MCP end to end. Do not ask the user to copy a long prompt. Guide them through only the inputs that require human action, especially API Key creation and browser login.

## Outcome

The user should be able to call one MCP server, usually named `douyin`, that supports Douyin, Xiaohongshu, and Bilibili extraction.

Expected tools:

- `parse_social_post_info`
- `social_extract_transcript`
- `social_capture_url`
- `extract_social_post_script`
- `social_analyze_owner_posts`

## Rules

- Never commit or print the user's real API Key.
- Never put real API keys in README, tests, examples, Git commits, or public logs.
- Prefer `config/social-post-extractor.env` inside the cloned MCP repo for secrets.
- Preserve existing MCP client config entries when adding this server.
- Use absolute paths in MCP stdio config.
- Run tests before telling the user installation is complete.
- After config, automatically test Douyin, Xiaohongshu, and Bilibili before saying OK.

## Step 1: Check Prerequisites

Check the local machine:

```bash
git --version
python3 --version
uv --version
```

If `uv` is missing, install it using the user's normal package manager, or follow the official installer from Astral:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Then restart the shell or load the updated PATH.

Also identify the MCP client the user wants to use, such as mcporter, Claude Desktop, Claude Code, Codex, or OpenClaw.

## Step 2: Clone Or Update

If the repo is not present:

```bash
git clone https://github.com/JNHFlow21/social-post-extractor-mcp.git
cd social-post-extractor-mcp
uv sync
```

If the repo already exists:

```bash
cd /ABSOLUTE/PATH/social-post-extractor-mcp
git pull
uv sync
```

Run:

```bash
uv run python -m unittest discover -s tests
uv run python -m compileall social_post_extractor_mcp
```

## Step 3: Guide API Key Setup

Ask whether the user already has an Aliyun Bailian / DashScope API Key.

If they do not, guide them:

1. Open the direct API Key page: https://bailian.console.aliyun.com/cn-beijing?tab=model#/api-key
2. If the user is confused, open the official API Key documentation: https://help.aliyun.com/zh/model-studio/get-api-key
3. Log in to Aliyun.
4. If the page is not already in Beijing, select `华北2（北京）` in the top-right region selector.
5. Select the default business space.
6. Click `创建 API Key`.
7. Set permission to `全部`.
8. Click `确定`.
9. Ask the user to copy the `sk-...` key and send it to you in the private agent chat.
10. Write it into `config/social-post-extractor.env` for them.

Create the local config file inside the MCP repo. This is the preferred cross-platform path because both macOS and Windows MCP clients can read it as long as they start from the repo directory:

```bash
mkdir -p config
cp .env.example config/social-post-extractor.env
chmod 600 config/social-post-extractor.env
```

Windows PowerShell:

```powershell
New-Item -ItemType Directory -Force config
Copy-Item .env.example config/social-post-extractor.env
```

Edit `config/social-post-extractor.env` with the user's real key:

```bash
export ASR_PROVIDER=bailian
export ASR_MODEL=paraformer-v2
export VISION_PROVIDER=bailian
export VISION_MODEL=qwen3-vl-flash
export CLEAN_PROVIDER=bailian
export CLEAN_MODEL=qwen-flash
export BAILIAN_API_KEY=sk-your-real-api-key
export BAILIAN_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode/v1
export DASHSCOPE_ASR_URL=https://dashscope.aliyuncs.com/api/v1/services/audio/asr/transcription
```

This file is ignored by Git through the `config/` ignore rule. The server also accepts plain `KEY=value` lines, so Windows users do not need shell-style environment setup.

Supported config files, in precedence order:

1. `config/social-post-extractor.env`
2. `.env`
3. `~/.mcporter/secrets/social-post-extractor.env`
4. Windows `%APPDATA%\social-post-extractor-mcp\config.env`

## Step 4: Configure MCP Client

Use this macOS / Linux stdio server config. Replace the path with the actual repo path:

```json
{
  "mcpServers": {
    "douyin": {
      "command": "/bin/zsh",
      "args": [
        "-lc",
        "cd '/ABSOLUTE/PATH/social-post-extractor-mcp' && exec '.venv/bin/python' -m social_post_extractor_mcp"
      ]
    }
  }
}
```

Use this Windows PowerShell stdio server config:

```json
{
  "mcpServers": {
    "douyin": {
      "command": "powershell",
      "args": [
        "-NoProfile",
        "-ExecutionPolicy",
        "Bypass",
        "-Command",
        "Set-Location 'C:\\ABSOLUTE\\PATH\\social-post-extractor-mcp'; & '.\\.venv\\Scripts\\python.exe' -m social_post_extractor_mcp"
      ]
    }
  }
}
```

Notes:

- The server name `douyin` is kept for backward compatibility.
- The server supports Douyin, Xiaohongshu, and Bilibili even if the MCP server name is `douyin`.
- Do not put API keys in this JSON.

For mcporter:

```bash
mcporter config list
```

Find the active config file, then merge the server entry into `mcpServers` without deleting existing servers.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JNHFlow21/social-post-extractor-mcp](https://github.com/JNHFlow21/social-post-extractor-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
