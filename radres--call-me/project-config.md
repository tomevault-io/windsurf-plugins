---
trigger: always_on
description: Your AI agents ring your actual iPhone, speak their question aloud, and get
---

# callme — your AI can call you

Your AI agents ring your actual iPhone, speak their question aloud, and get
your spoken answer back as text — or just text you. Works from any AI that
speaks MCP; nothing to install.

## 1. Get the app

**[callme on the App Store](https://serdaroztetik.com/aiphone/go/readme)**.
No registration, no email. Open it, tap *Agree & Continue*, and it shows your
personal **callme number**: 10 digits, and all an agent needs to reach you.

## 2. Connect your AI (MCP)

Pairing is stored locally in `~/.aiphone/config.json`. An MCP server running on
this computer can read that file automatically; the hosted HTTP endpoint cannot
read files on your computer and therefore needs the number supplied separately.

One server, every MCP client — no API key, no OAuth:

```
https://serdaroztetik.com/aiphone/mcp
```

**Claude Code**

```bash
claude mcp add --transport http call-me https://serdaroztetik.com/aiphone/mcp
```

**opencode** — add to `opencode.json` (or `~/.config/opencode/opencode.json`):

```json
{
  "mcp": {
    "call-me": {
      "type": "remote",
      "url": "https://serdaroztetik.com/aiphone/mcp",
      "enabled": true
    }
  }
}
```

**Codex CLI — use the local paired MCP server**

```bash
cd /absolute/path/to/call-me
codex mcp remove call-me  # only if the hosted entry is already configured
codex mcp add callme -- node "$PWD/codex/mcp.mjs"
codex mcp list
```

The local `callme` MCP server reads `~/.aiphone/config.json` for every call and
text, so no number needs to be typed into the Codex conversation. Use its
`identity` tool to verify the paired phone.

**Gemini CLI**

```bash
gemini mcp add --transport http call-me https://serdaroztetik.com/aiphone/mcp
```

**Cursor** — add to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project):

```json
{
  "mcpServers": {
    "call-me": { "url": "https://serdaroztetik.com/aiphone/mcp" }
  }
}
```

**ChatGPT** — Settings → Apps → Advanced → Developer mode → add a connector
with the URL above (no authentication).

**claude.ai** — Settings → Connectors → Add custom connector → the URL above.

**Anything else that speaks MCP** — add a streamable-HTTP server pointing at the
URL above, then provide the recipient number through that client's pairing or
configuration flow. A hosted MCP connection cannot see this computer's
`~/.aiphone/config.json`.

For the local Codex/Claude integrations, pairing once writes the shared local
config and the MCP tools use it automatically. For hosted clients, tell the AI
the number or use that client's explicit pairing flow; never guess it.

| Tool | What it does |
|---|---|
| `call` | Rings your iPhone, speaks the question, returns your spoken answer as text. Holds ~30s, then hands the AI a `call_id` to poll |
| `poll_result` | Finishes a call that was still ringing |
| `text` | Push-notification message, no ring |
| `wait_for_reply` | Delivers your replies and voicemails back to the AI |
| `set_thread_title` | Names the conversation thread on your phone |

Repeat calls from the same AI land in one titled thread on your phone; each
agent/session gets its own number and thread. Blocking a thread in the app
silences that sender for good.

## 3. Or just curl it

No MCP client, no plugin, no token. Anything that can POST JSON can ring your
phone, and these two endpoints are the whole product.

**Ring and wait for the spoken answer** (blocks until you answer or it gives up):

```bash
curl -sS https://serdaroztetik.com/aiphone/ring \
  -H 'content-type: application/json' \
  -d '{"to":"<YOUR_10_DIGITS>","text":"Deploy to prod?","from":"Claude on my laptop"}'
```

```json
{"status":"completed","transcript":"Yes, ship it",
 "session_token":"curl_1a2b...","from":"<CALLER_NUMBER>"}
```

`status` is one of `completed`, `missed`, `declined`, `timeout`, `failed`.
`transcript` is what you said, already turned into text.

**Text only**, no ring:

```bash
curl -sS https://serdaroztetik.com/aiphone/text \
  -H 'content-type: application/json' \
  -d '{"to":"<YOUR_10_DIGITS>","body":"Migration finished.","from":"Claude on my laptop"}'
```

Returns `{"ok":true,"message_id":42,"delivered":true}`. A `delivered:false` means
the text is in your history but no notification was shown (notifications are off
for /call-me on that phone); the response spells that out in `notice`.

| Field | |
|---|---|
| `to` | your 10-digit /call-me number. Required |
| `text` / `body` | what to ask / send. 600 chars for a call, 2000 for a text |
| `from` | sender name shown on your phone. Defaults to "Someone's AI" |
| `timeout_s` | `/ring` only: seconds to wait, 30 to 300. Default 300 |

### Anything else

Both responses hand back a `session_token`, and that is a real session: the same
caller can use the rest of the API with it, all landing in one thread on your
phone.

```bash
# name the thread
curl -sS https://serdaroztetik.com/aiphone/sessions/label \
  -H 'content-type: application/json' \
  -d '{"session_token":"curl_...","label":"nightly deploy"}'

# long-poll for replies, voicemails and missed calls
curl -sS "https://serdaroztetik.com/aiphone/sessions/events?session_token=curl_...&cursor=0&wait=25"
```

The token is derived from `to` + `from` + your IP, so repeat curls from the same

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [radres/call-me](https://github.com/radres/call-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
