---
trigger: always_on
description: AI-powered video generation skill. Use when the user wants to generate videos from text descriptions, browse video recipes, upload assets, or manage video creation workflows.
---


# Medeo Video Generator Skill

Generate AI videos from text. Medeo is an **AI video agent** that handles full storylines, multi-scene narratives, and screenplays in a single call — shot composition, transitions, pacing, and music are all automatic.

> ⚠️ **Do NOT split stories into multiple calls.** Pass the entire screenplay in one `--message`.

## 0. Pre-Flight Check (MANDATORY — run before anything else)

**Before running any command**, check if API key is configured:

```bash
python3 {baseDir}/scripts/medeo_video.py config 2>/dev/null | python3 -c "import sys,json; d=json.load(sys.stdin); print('ok' if d.get('apiKey') else 'missing')"
```

- Output `ok` → proceed normally
- Output `missing` (or command fails) → **stop immediately**, do NOT run any other commands. Send the setup message using the **channel-appropriate method**:

**Feishu** — use Feishu API directly (NOT `message` tool — it won't render cards):
```python
import json, urllib.request
cfg = json.loads(open("/home/ec2-user/.openclaw/openclaw.json").read())
feishu = cfg["channels"]["feishu"]["accounts"]["default"]
token = json.loads(urllib.request.urlopen(urllib.request.Request(
    "https://open.feishu.cn/open-apis/auth/v3/tenant_access_token/internal",
    data=json.dumps({"app_id": feishu["appId"], "app_secret": feishu["appSecret"]}).encode(),
    headers={"Content-Type": "application/json"}
)).read())["tenant_access_token"]
card = {
    "config": {"wide_screen_mode": True},
    "header": {"title": {"tag": "plain_text", "content": "🎬 Video Generation — API Key Required"}, "template": "blue"},
    "elements": [{"tag": "div", "text": {"tag": "lark_md", "content": "You need a **Medeo API Key** to generate videos.\n\n**Steps:**\n1. Go to https://medeo.app/dev/apikey\n   - No account? You'll be guided to sign up. The key appears after login.\n2. Copy the key (starts with `mk_`) and send it back to me.\n\nOnce I have it, I'll configure everything for you."}}],
}
urllib.request.urlopen(urllib.request.Request(
    "https://open.feishu.cn/open-apis/im/v1/messages?receive_id_type=open_id",
    data=json.dumps({"receive_id": "<USER_OPEN_ID>", "msg_type": "interactive", "content": json.dumps(card)}).encode(),
    headers={"Authorization": f"Bearer {token}", "Content-Type": "application/json"}
))
```

**Telegram / Discord / Other channels** — send plain text via `message` tool (these channels support markdown natively):
```
🎬 Video Generation — API Key Required

Steps:
1. Go to https://medeo.app/dev/apikey (sign up if needed — the key appears after login)
2. Copy the key (starts with mk_) and send it back to me

Once I have it, I'll configure everything for you.
```

Once they provide the key: `python3 {baseDir}/scripts/medeo_video.py config-init --api-key "mk_..."`

## 1. First-Time Setup

If no API Key is configured, the script outputs `"setup_required": true`.
1. Send the user this exact link: https://medeo.app/dev/apikey (this page auto-prompts registration if not logged in, then shows the API key)
2. Once they provide the key: `python3 {baseDir}/scripts/medeo_video.py config-init --api-key "mk_..."`

## 2. Generate a Video (5-30 min, always async)

Users only need to know **3 ways** to generate a video:

1. **Send text** → generate video
2. **Send text + upload image** → generate video using their image
3. **Send text + image URL** → generate video using the URL image

The agent handles everything else silently.

> **IMPORTANT**: Before spawning the generation task, **immediately reply to the user** with an acknowledgment like:
> "🎬 Starting video generation — I'll send you the result in about 5–10 minutes."
> Do NOT wait in silence. The user should know their request was received.

### Usage 1: Text only

```bash
python3 {baseDir}/scripts/medeo_video.py spawn-task \
  --message "user's video description" \
  --deliver-to "oc_xxx" \
  --deliver-channel "feishu"
```

### Usage 2: Text + uploaded image (user sends image in chat)

```bash
# First: upload-file to get media_id (see Section 3)
python3 {baseDir}/scripts/medeo_video.py spawn-task \
  --message "user's video description" \
  --media-ids "media_01..." \
  --asset-sources my_uploaded_assets \
  --deliver-to "oc_xxx" \
  --deliver-channel "feishu"
```

### Usage 3: Text + image URL

```bash
python3 {baseDir}/scripts/medeo_video.py spawn-task \
  --message "user's video description" \
  --media-urls "https://example.com/photo.jpg" \
  --asset-sources my_uploaded_assets \
  --deliver-to "oc_xxx" \
  --deliver-channel "feishu"
```

> **Agent auto-behavior:** When the user provides images (Usage 2 or 3), always pass `--asset-sources my_uploaded_assets` so Medeo uses their images instead of generating new ones. The user does not need to know this flag exists.

### Internal Parameters (agent use only — never expose to users)

These are handled automatically by the agent. Do NOT mention them to users or ask users to provide them.

| Flag | When to use | Default behavior |
|------|------------|-----------------|
| `--voice-id "voice_01..."` | When a specific voice is needed | Medeo picks automatically |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [one2x-ai/medeo-video-skill](https://github.com/one2x-ai/medeo-video-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
