---
trigger: always_on
description: Use when the user wants to create, generate, build, update, refresh, or improve their personal writing style guide from their own sent writing (Generate mode), OR when they ask the agent to write, draft, reply, rewrite, edit, polish, or humanize something in their voice (Use mode). Teach your agent how to write like you.
---


# Ghostwriter

**Teach your agent how to write like you.**

Ghostwriter turns your sent writing into a practical style guide your agent can use for drafts that sound like you, not like AI. It has two modes:

- **Generate mode** builds your personal style guide (usually `ghostwriter-guide.md`) by analyzing email, texts, Slack, or other writing you've actually sent.
- **Use mode** loads that guide and drafts or edits in your voice, with your personal style as the primary authority.

Once your personal guide is in place, agent-written drafts should feel spooky good.

This skill is agent-agnostic. It works with Claude Code, Codex, OpenCode/OpenClaw-style agents, Hermes, and similar local agents. It does not require any single vendor, tool, or data source — it adapts to whatever access the current environment already has.

---

## Step 0: Route to a mode

Read the user's request and pick a mode. Do this before anything else.

**Generate mode** — the user wants to *create or update the guide itself*. Triggers include:
> "create / generate / build / make my Ghostwriter guide", "update / refresh / improve my style guide", "analyze how I write", "teach you how I write", "learn my voice".

**Use mode** — the user wants the agent to *produce or edit writing* in their voice. Triggers include:
> "write / draft / reply / respond as me", "rewrite / edit / polish / humanize this in my voice", "make this sound like me", "draft an email/text/Slack message for me".

If the request is ambiguous (e.g. "help me with my writing"), ask one short clarifying question: are they building/updating the guide, or do they want something written now?

---

## Generate mode

Goal: produce a standalone markdown style guide the user can paste into *any* agent without the rest of this skill. Walk through five stages. Keep the user in control at each decision point.

Reference files (read the one that matches the chosen source — don't preload all of them):

- `references/source-detection.md` — how to inventory what access already exists
- `references/gmail-gogcli.md` — Gmail via `openclaw/gogcli`
- `references/gmail-googleworkspace-cli.md` — Gmail via `googleworkspace/cli`
- `references/local-email-exports.md` — `.mbox` / `.eml` / Takeout exports
- `references/imessage-macos.md` — local iMessage database on macOS
- `references/slack.md` — Slack via MCP or API
- `references/generic-corpora.md` — `.txt` / `.json` / `.csv` / mailbox / pasted text
- `references/analysis-framework.md` — what dimensions to extract and how
- `references/humanizer-cleanup.md` — the secondary AI-artifact cleanup pass

> **The user can point you at any source.** If they name a specific method they want used — a connector, MCP server, CLI, plugin, app integration, built-in tool, an export they've already made, or even "use the email access you already have" — **use that method** and skip ahead to privacy + collection. You don't need a reference file for it: apply the same guidelines to any source (sent messages only, filter noise, analyze the dimensions, write the guide). The reference files are examples, not an allow-list. When a tool behaves differently than a reference describes, adapt — the reference shows the *shape* of the workflow, not the only way to do it.

### Stage 1 — Detect existing access (do this first)

**Before suggesting any new setup, check what the current agent environment already has.** Least setup wins. Look for:

- Email access through a connector, MCP server, plugin, app integration, or built-in tool (e.g. a Gmail/Outlook connector, an email MCP).
- Slack access through an MCP server or app integration.
- A local messages database (iMessage on macOS).
- Any already-connected writing source.

Inventory what's available, then **recommend the existing path first** because it requires no new setup. See `references/source-detection.md` for how to probe each agent environment.

If nothing is connected, move to setup-based options in Stage 2.

### Stage 2 — Select source(s)

Present the available source methods and let the user choose one or more. Explain tradeoffs briefly (one line each), don't lecture:

- **Existing connector / MCP / plugin / built-in tool** — least setup; use if detected in Stage 1.
- **Gmail via `openclaw/gogcli`** — `gog` CLI, simple `gmail search`/`get`; good for quick local pulls. See `references/gmail-gogcli.md`.
- **Gmail via `googleworkspace/cli`** — `gws` CLI, dynamic Discovery-based access; good if you want the full Workspace surface. See `references/gmail-googleworkspace-cli.md`.
- **Local sent email export** — `.mbox` from Google Takeout or `.eml` files; no API access needed. See `references/local-email-exports.md`.
- **Local iMessage (macOS)** — adds short-form / texting voice. See `references/imessage-macos.md`.
- **Slack (MCP or API)** — workplace voice. See `references/slack.md`.
- **Other local corpora** — exported text, JSON, CSV, mailbox files, pasted samples. See `references/generic-corpora.md`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joshferrara/ghostwriter](https://github.com/joshferrara/ghostwriter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
