---
trigger: always_on
description: `octo-cli` is the command-line interface for the Octo ecosystem, built for **AI Agent Bots** to call via `exec` from agent runtimes (OpenClaw, Claude Code, etc.). Output is a JSON envelope; agent-runtime commands take no interactive I/O. The sole exception is `octo-cli auth login`, an operator-only setup step that reads the token from a hidden terminal prompt (or stdin).
---

# CLAUDE.md — octo-cli project instructions

## What is octo-cli

`octo-cli` is the command-line interface for the Octo ecosystem, built for **AI Agent Bots** to call via `exec` from agent runtimes (OpenClaw, Claude Code, etc.). Output is a JSON envelope; agent-runtime commands take no interactive I/O. The sole exception is `octo-cli auth login`, an operator-only setup step that reads the token from a hidden terminal prompt (or stdin).

## Architecture

- Go single binary, cobra CLI framework.
- **Metadata-driven**: the entire service command tree is auto-registered at startup from OpenAPI 3.x specs embedded into the binary via `internal/registry`. To add or change an endpoint, update a spec — not code.
- **Thin client**: all business logic lives in backend services (matters, dmworkim). CLI is transport + validation + formatting.
- **Multi-backend**: different domains live at different base URLs, all routed through a unified API base URL (`OCTO_API_BASE_URL`).
- **Factory DI**: `internal/cmdutil.Factory` is the DI container; no package-level globals. Tests inject stubs through `ConfigFunc` / `CredentialFunc` / `ClientFunc` / `RegistryFunc`. `Factory.ErrorEmitted` tracks whether an error envelope was already written to stderr, preventing double-emit between RunE and the top-level main error handler.
- **JSON envelope I/O**: `{ok, identity, data, _pagination, _rate_limit}` on stdout for success; `{ok:false, error:{type,code,message,hint,detail}}` on stderr for failure. Exit codes: auth=3, validation/config=2, rest=1.

## Identity Model

- The CLI is **bot-only** — no user login. A bot token is an `app_*` (App Bot) or `bf_*` (User Bot) token.
- **Credential resolution** (see `internal/credential`, `internal/authstore`): a token comes from a stored encrypted profile or, as a fallback, `OCTO_BOT_TOKEN`. Stored profiles live in `~/.octo-cli` (override `OCTO_CONFIG_DIR`): metadata in plaintext `config.json`, tokens in AES-256-GCM `credentials.enc`. Manage them with `octo-cli auth`.
- **Selecting a credential at runtime**: `--bot-id <robot_id>` (env `OCTO_BOT_ID`) is the agent's primary selector — robot ids are self-known; `--profile <name>` selects by friendly name. With exactly one profile, selection is implicit; with **two or more, a selector is required** (ambiguity is a hard error, never a silent guess). Precedence: selector > sole/implicit profile > `OCTO_BOT_TOKEN`. The success envelope's `identity` echoes the active `{profile, robot_id, bot_kind, source}` so misuse is visible.
- **Isolation boundary = OS user**: the encryption key is machine-derived, so the store resists off-machine leakage (commit/backup/sync) but not a same-user process. Isolate mutually-distrusting bots with separate OS users or `OCTO_CONFIG_DIR` values.
- Each Bot has an **owner**; operations are attributed to the Bot identity. For LLM-backed paths (`matter extract`) the bot acts on behalf of its owner — pass `owner_uid` as `creator_uid`.
- `OCTO_SPACE_ID` (or `--space`) supplies space context for platform-scoped bots. Space-scoped bots resolve their space server-side.

## Command Structure (7 domains, 47 operations / 50 commands incl. 3 matter transition aliases)

Service commands are auto-registered. The hand-written leaves are `schema`, `version`, `api` (generic passthrough), `config`, `auth`, and the cobra-generated `completion`.

> **`matter` is temporarily withheld** (backend API not yet stable). The spec
> stays embedded — `octo-cli schema matter.*` still introspects it — but the command
> subtree and the `octo-matter` skill are hidden via the `x-octo-disabled` spec
> flag (`internal/registry/specs/matter.json`) and the skill's `disabled: true`
> frontmatter. Flip both off to re-enable. The tree below shows the full surface.

```
octo-cli matter    create | list | get | update | delete        (withheld)
               transition | close | reopen | archive | extract
               assignee add|remove
               channel  link|unlink
               timeline add|list|delete
octo-cli message   send | edit | sync | read-receipt
octo-cli group     list | get | members | md-get | md-update
               create | update | member-add | member-remove       (User Bot only)
octo-cli thread    create | list | get | members
               join | leave | md-get | md-update                  (User Bot only)
octo-cli file      upload | download | credentials | presigned
octo-cli bot       register | set-commands | user-info | space-members | typing | heartbeat
octo-cli event     list | ack

octo-cli auth      login | status | logout | list
octo-cli schema [--list [domain] | <operation-id>]
octo-cli api <METHOD> <PATH> [--params ...] [--data ...] [--service ...]
octo-cli config show
octo-cli completion bash|zsh|fish|powershell
octo-cli version
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mininglamp-OSS/octo-cli](https://github.com/Mininglamp-OSS/octo-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
