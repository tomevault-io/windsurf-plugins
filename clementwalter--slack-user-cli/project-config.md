---
trigger: always_on
description: Terminal access to Slack using browser session credentials (`xoxc-` token + `d`
---


# Slack User CLI

Terminal access to Slack using browser session credentials (`xoxc-` token + `d`
cookie). Located at `~/.claude/skills/slack-user-cli/slack_user_cli.py`.

## Running

All commands use `uv run`:

```bash
uv run ~/.claude/skills/slack-user-cli/slack_user_cli.py <command> [options]
```

## Authentication

Must be logged in before using any command. Credentials are stored in
`~/.config/slack-user-cli/config.json`.

```bash
# Auto-extract from Slack desktop app (close Slack first)
slack_user_cli login --auto

# Import all workspaces from browser — copies to clipboard, reads via pbpaste
slack_user_cli login --browser

# Add a single workspace manually
slack_user_cli login --manual
```

## Global Options

| Option                            | Description                                 |
| --------------------------------- | ------------------------------------------- |
| `-w <name>`, `--workspace <name>` | Use a specific workspace instead of default |
| `--debug`                         | Enable debug logging                        |

## Output Conventions

- **IDs by default.** Every read command emits raw Slack IDs (`U…` users,
  `C…`/`D…`/`G…` channels) so output is stable for scripts. Do **not** add
  `--names` unless the user explicitly asks for human-readable names — raw IDs
  are the right answer for chaining further commands.
- **`--names` is opt-in.** Only when the user specifically wants display names
  (e.g. "show me who said what", "summarize this thread"), pass `--names` to
  resolve user/channel IDs and rewrite `<@UXXX>` mentions.
- **CRITICAL — never guess a name.** A name attributed to a message must come
  from `--names` resolution (or an explicit `users`/`search` lookup), **never**
  from inference. Do not guess an author from the message content, from a
  username stem, from a DM/MPIM conversation title, or from surrounding context —
  that is silent misattribution and a correctness failure. If a name will not
  resolve, keep the raw `U…` ID and say so; do not approximate or invent one.
- **`--json` everywhere.** Every command supports `--json` for structured
  output. Use it whenever a programmatic consumer would otherwise parse rendered
  text. `--json` and `--names` are independent.

## Commands Reference

### Workspace Management

```bash
# List all saved workspaces
slack_user_cli workspaces

# Set default workspace
slack_user_cli default "Workspace Name"

# Force-refresh the channel and user cache
slack_user_cli refresh
```

### Reading

```bash
# List joined channels (output: channel IDs)
slack_user_cli channels
slack_user_cli channels --all
slack_user_cli channels --type "public_channel,private_channel,mpim,im"
slack_user_cli channels --json   # {channels: [{id, name, type, num_members, topic, is_member}, ...]}

# Read recent messages from a channel (by name or ID; output: user IDs)
slack_user_cli read <channel_name_or_id> --limit 20

# Emit structured JSON instead of human-readable text (for programmatic
# consumers — smithers workflows, scripts, pipelines). Shape:
#   {"channel": "...", "messages": [{"ts", "raw_ts", "user", "text", "thread_ts"?, "threadCount"?, "files"?, "replies"?}, ...]}
# Every message carries `raw_ts` (full microsecond ts) — `ts` is minute-precision
# for humans, but `raw_ts` is what you pass to the `permalink`/`click`/`thread`
# commands. Use --json whenever you'd otherwise parse the pretty output back into
# fields — the parse step is the #1 source of bugs and timeouts.
# Messages with attachments carry a `files` array: each entry has
# {id, name, filetype, mimetype, size, url_private, url_private_download,
# permalink}. In text output, attachments show as a 📎 line under the message.
# To actually read a file's contents, fetch it with the `download` command
# (the message text alone never includes attachment contents).
# A message that QUOTES/SHARES another message carries the original under a
# `shared` array: [{url, author, channel, ts, text, files:[...]}]. Its `files`
# are the quoted message's attachments — so a forwarded message never hides its
# attachments (text output shows them under a "↪ quoted <author>" line). Plain
# pasted message permalinks appear in a `links` array ([{url, channel, ts}]).
slack_user_cli read <channel_name_or_id> --limit 20 --json

# Add --expand-thread to inline every thread's replies under `replies: [...]`
# on the parent. Only meaningful with --json. Most decisions live in replies
# rather than parent posts, so expansion is almost always what you want for
# analysis; skip it only when you need cheap channel-level metadata.
slack_user_cli read <channel_name_or_id> --limit 20 --json --expand-thread

# Time-bounded fetch: only messages at/after an ISO date or datetime (UTC if
# no tz). Sets the history `oldest` bound; pair with a larger --limit to pull a
# whole window. To catch threads whose parent predates the window but that got
# fresh replies inside it, set --since a couple of days earlier, --expand-thread,
# and filter on each message's raw_ts >= your real cutoff.
slack_user_cli read <channel_name_or_id> --since 2026-05-29 --limit 200 --json --expand-thread
slack_user_cli read <channel_name_or_id> --since 2026-05-29T10:07:00 --limit 200 --json


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClementWalter/slack-user-cli](https://github.com/ClementWalter/slack-user-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
