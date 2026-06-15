---
trigger: always_on
description: A curl-like CLI tool for making authenticated requests to the X (Twitter) API. Use this skill when you need to post tweets, reply, quote, search, read posts, manage followers, send DMs, upload media, or interact with any X API v2 endpoint. Supports multiple apps, OAuth 2.0, OAuth 1.0a, and app-only auth.
---


# xurl — Agent Skill Reference

`xurl` is a CLI tool for the X API. It supports both **shortcut commands** (human/agent‑friendly one‑liners) and **raw curl‑style** access to any v2 endpoint. All commands return JSON to stdout.

---

## Prerequisites

This skill requires the `xurl` CLI utility: <https://github.com/xdevplatform/xurl>.

Before using any command you must be authenticated. Run `xurl auth status` to check.

### Secret Safety (Mandatory)

- Never read, print, parse, summarize, upload, or send `~/.xurl` (or copies of it) to the LLM context.
- Never ask the user to paste credentials/tokens into chat.
- The user must fill `~/.xurl` with required secrets manually on their own machine.
- Do not recommend or execute auth commands with inline secrets in agent/LLM sessions.
- Warn that using CLI secret options in agent sessions can leak credentials (prompt/context, logs, shell history).
- Never use `--verbose` / `-v` in agent/LLM sessions; it can expose sensitive headers/tokens in output.
- Sensitive flags that must never be used in agent commands: `--bearer-token`, `--consumer-key`, `--consumer-secret`, `--access-token`, `--token-secret`, `--client-id`, `--client-secret`.
- To verify whether at least one app with credentials is already registered, run: `xurl auth status`.

### Register an app (recommended)

App credential registration must be done manually by the user outside the agent/LLM session.
After credentials are registered, authenticate against the app that holds those credentials:

```bash
xurl auth oauth2 --app APP_NAME
```

You can also run `xurl auth default APP_NAME` first and then use `xurl auth oauth2`.

For multiple pre-configured apps, switch between them:
```bash
xurl auth default prod-app          # set default app
xurl auth default prod-app alice    # set default app + user
xurl --app dev-app /2/users/me      # one-off override
xurl auth apps redirect-uri get prod-app
xurl auth apps redirect-uri set prod-app http://localhost:8080/callback
```

### Other auth methods

Examples with inline secret flags are intentionally omitted. If OAuth1 or app-only auth is needed, the user must run those commands manually outside agent/LLM context.

Tokens are persisted to `~/.xurl` in YAML format. Each app has its own isolated tokens and may also store a `redirect_uri`. `REDIRECT_URI` in the environment still takes precedence over the stored app value. Do not read this file through the agent/LLM. Once authenticated, every command below will auto‑attach the right `Authorization` header.

---

## Quick Reference

| Action | Command |
|---|---|
| Post | `xurl post "Hello world!"` |
| Reply | `xurl reply POST_ID "Nice post!"` |
| Quote | `xurl quote POST_ID "My take"` |
| Delete a post | `xurl delete POST_ID` |
| Read a post | `xurl read POST_ID` |
| Search posts | `xurl search "QUERY" -n 10` |
| Who am I | `xurl whoami` |
| Look up a user | `xurl user @handle` |
| Home timeline | `xurl timeline -n 20` |
| Mentions | `xurl mentions -n 10` |
| Like | `xurl like POST_ID` |
| Unlike | `xurl unlike POST_ID` |
| Repost | `xurl repost POST_ID` |
| Undo repost | `xurl unrepost POST_ID` |
| Bookmark | `xurl bookmark POST_ID` |
| Remove bookmark | `xurl unbookmark POST_ID` |
| List bookmarks | `xurl bookmarks -n 10` |
| List likes | `xurl likes -n 10` |
| Follow | `xurl follow @handle` |
| Unfollow | `xurl unfollow @handle` |
| List following | `xurl following -n 20` |
| List followers | `xurl followers -n 20` |
| Block | `xurl block @handle` |
| Unblock | `xurl unblock @handle` |
| Mute | `xurl mute @handle` |
| Unmute | `xurl unmute @handle` |
| Send DM | `xurl dm @handle "message"` |
| List DMs | `xurl dms -n 10` |
| Upload media | `xurl media upload path/to/file.mp4` |
| Media status | `xurl media status MEDIA_ID` |
| **App Management** | |
| Register app | Manual, outside agent (do not pass secrets via agent) |
| List apps | `xurl auth apps list` |
| Update app config | Manual, outside agent (do not pass secrets via agent) |
| View app redirect URI | `xurl auth apps redirect-uri get [NAME]` |
| Set app redirect URI | `xurl auth apps redirect-uri set NAME URI` |
| Remove app | `xurl auth apps remove NAME` |
| Set default (interactive) | `xurl auth default` |
| Set default (command) | `xurl auth default APP_NAME [USERNAME]` |
| Use app per-request | `xurl --app NAME /2/users/me` |
| Auth status | `xurl auth status` |

> **Post IDs vs URLs:** Anywhere `POST_ID` appears above you can also paste a full post URL (e.g. `https://x.com/user/status/1234567890`) — xurl extracts the ID automatically.

> **Usernames:** Leading `@` is optional. `@elonmusk` and `elonmusk` both work.

---

## Command Details

### Posting

```bash
# Simple post
xurl post "Hello world!"

# Post with media (upload first, then attach)
xurl media upload photo.jpg          # → note the media_id from response
xurl post "Check this out" --media-id MEDIA_ID

# Multiple media
xurl post "Thread pics" --media-id 111 --media-id 222

# Reply to a post (by ID or URL)
xurl reply 1234567890 "Great point!"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xdevplatform/xurl](https://github.com/xdevplatform/xurl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
