---
trigger: always_on
description: Bybit AI Trading Skill — Trade on Bybit using natural language. Covers spot, derivatives, earn, and more. Works with Claude, ChatGPT, OpenClaw, and any AI assistant.
---


# Bybit Trading Skill

Trade on Bybit using natural language. Supports spot, linear perpetuals (USDT/USDC), inverse contracts, options, and earn products.

### Rule Priority

When rules in this skill conflict, follow this order: **Safety > User Responsiveness > Convenience**. For example, never skip confirmation to be faster; never block the user's first request to run an auto-update check.

### Auto Update (MUST follow at session start)

This skill supports self-update with integrity verification. At the start of each new session, launch the update check as a **background sub-agent** so it never blocks the user's first request:

```
FOREGROUND (main agent — immediate):
1. Respond to the user's request using the current local version. Do NOT wait for the update check.

BACKGROUND (sub-agent — parallel):
1. LOCAL_VERSION = metadata.version  (from YAML frontmatter above)
2. SKILL_DIR = directory where this SKILL.md is located
3. MANIFEST = curl -sf -H "User-Agent: bybit-skill/1.4.3" https://api.bybit.com/skill/manifest
   (returns JSON: {"version":"x.y.z", "files":{"SKILL.md":"sha256:...","modules/market.md":"sha256:...",...}})
4. If fetch fails: return {status: "error", reason: "fetch_failed"}
5. Path validation: For each file in manifest.files, reject the entire update if ANY path:
   - Does not match `SKILL.md` or `modules/<name>.md` (where <name> is [a-z0-9-]+)
   - Contains `..`, starts with `/` or `~`, contains backslashes, or has a non-.md extension
   If any path is invalid: return {status: "error", reason: "invalid_path", path: "<rejected>"}
6. Version comparison (semver): split by ".", compare major → minor → patch numerically.
   If manifest.version > LOCAL_VERSION:
   a. For each file in manifest.files:
      - Download: curl -sf -H "User-Agent: bybit-skill/1.4.3" https://raw.githubusercontent.com/bybit-exchange/skills/main/<file>
      - Save content to temp file, then compute SHA256: shasum -a 256 <temp_file> | awk '{print $1}'
      - Compare with manifest checksum (strip "sha256:" prefix)
      - If mismatch: ABORT entire update. return {status: "error", reason: "checksum_mismatch", file: "<file>"}
      - If match: save to SKILL_DIR/.skill-update-tmp/<file>
   b. ALL files verified → move from temp to SKILL_DIR:
      - For each file: mkdir -p parent dir, then mv .skill-update-tmp/<file> SKILL_DIR/<file>
      - rm -rf SKILL_DIR/.skill-update-tmp/
   c. return {status: "updated", from: LOCAL_VERSION, to: manifest.version}
   If manifest.version == LOCAL_VERSION:
   d. return {status: "current"}

WHEN SUB-AGENT COMPLETES (main agent receives result):
- If status="updated": notify user "Skill updated from {from} to {to}. Using latest version." Re-read updated SKILL.md.
- If status="current" or status="error": silently continue with current version.
- Cache manifest (if returned) in session memory for module loading (see Module Router).
```

**Rules:**
- Check at most ONCE per session. Do not re-check during the same conversation.
- If any network request fails (timeout, 404, etc.), skip silently and proceed with current version. (See Graceful Degradation below for unified fallback rules.)
- **Never block the user's first request.** The sub-agent runs in the background; the main agent responds immediately. If a module is needed before the sub-agent finishes, use the current local version.
- If checksum algorithm prefix is not "sha256:", refuse the update (fail closed).

---

## Quick Start

### Step 1: Get an API Key

1. Log in to [Bybit](https://www.bybit.com) → API Management → Create New Key
2. Permissions: enable **Read + Trade only** (NEVER enable Withdraw for AI use)
3. Recommended: bind your IP address (makes the key permanent; otherwise expires in 3 months)
4. **Strongly recommended**: Create a dedicated **sub-account** for AI trading with limited balance

### Step 2: Configure Credentials

Credential setup depends on where the AI runs. Auto-detect the environment and follow the matching path:

**Path A — Local CLI** (Claude Code, Cursor, or any tool with shell access):

Copy-paste this into `~/.zshrc` or `~/.bashrc`:

```bash
export BYBIT_API_KEY="your_api_key"
export BYBIT_API_SECRET="your_secret_key"
export BYBIT_ENV="testnet"  # or "mainnet"
```

> **Using an RSA API Key instead?** (Self-generated: you uploaded a public key to Bybit and kept the private key locally.) Replace the `BYBIT_API_SECRET` line with:
> ```bash
> export BYBIT_API_PRIVATE_KEY_PATH="/absolute/path/to/private.pem"
> ```
> Everything else stays the same. Do NOT set both `BYBIT_API_SECRET` and `BYBIT_API_PRIVATE_KEY_PATH` — the skill will pick RSA if both are present, but it's clearer to keep only the one you actually use.

On first use, check if these environment variables exist. If they do, use them directly — do NOT ask the user to paste keys in the conversation. If they don't exist, guide the user to set them up:

1. Tell the user: "For security, I recommend storing your API keys as environment variables instead of pasting them here."
2. Provide the export commands above
3. After the user has set them, verify with `echo $BYBIT_API_KEY | head -c5` (only show first 5 chars to confirm)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bybit-exchange/skills](https://github.com/bybit-exchange/skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
