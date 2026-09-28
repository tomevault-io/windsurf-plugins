---
trigger: always_on
description: Project name: `discord-mcp-platform`
---

# Discord MCP Platform — Claude Project Instructions

## Project Identity

Project name: `discord-mcp-platform`

Working description:

> A multi-server Discord MCP control plane that allows users to connect their Discord identity, install/configure Discord bots, and manage Discord server automation through MCP-compatible AI clients.

This project is a **Discord MCP** because it exposes Discord operations as Model Context Protocol tools, resources, and prompts.

The product must use official Discord bot and OAuth2/application flows.

## Core Framing

Users do not connect their Discord account so the system can control the account.

Users connect their Discord account so the platform can:

- identify them;
- discover relevant guild context when permitted;
- verify whether they should be allowed to manage a guild;
- connect them to workspaces;
- authorize bot installation/configuration flows;
- audit actions performed through MCP clients.

Actual Discord automation must be executed through authorized Discord bot accounts or official webhooks.

## Strict Non-Goals

Do not implement self-bot behavior.

Do not use Discord user tokens.

Do not ask a user to paste their Discord token.

Do not automate a normal Discord user account.

Do not send messages as a personal Discord user.

Do not read user DMs by imitating a user.

Do not use browser automation to control Discord.

Do not bypass Discord permissions, rate limits, OAuth scopes, gateway restrictions, or API limits.

Do not build spam, raid, mass-DM, invite-spam, reaction-spam, or stealth-monitoring tools.

## Product Goal

Build a platform where a user can say, through an MCP-capable AI client:

- "Create a support channel in my server."
- "Summarize the last 100 messages in #general."
- "Create a moderation bot for #support."
- "Post this announcement in #updates."
- "Create a private incident thread and invite moderators."
- "Create an automation that answers FAQs and escalates unresolved questions."
- "List users with the Trial Member role."
- "Draft a moderation warning for this message."

The AI client should call MCP tools, and the platform should safely execute those operations through a Discord bot after permission checks.

## Architecture Style

Use a layered architecture.

```text
discord_mcp_platform/
  app/
    main.py
    settings.py
    logging.py
    lifecycle.py

  api/
    routes/
      health.py
      auth_discord.py
      workspaces.py
      guilds.py
      bots.py
      automations.py
      audit.py

  mcp/
    server.py
    auth.py
    context.py
    tools/
      guilds.py
      channels.py
      messages.py
      roles.py
      members.py
      threads.py
      automations.py
      moderation.py
      webhooks.py
      invites.py
      audit.py
    resources/
      guild.py
      channel.py
      message.py
      automation.py
    prompts/
      community_summary.py
      moderation_review.py
      automation_builder.py
      incident_report.py
      support_triage.py

  discord/
    bot_runtime.py
    rest_client.py
    gateway.py
    oauth.py
    permissions.py
    rate_limits.py
    slash_commands.py
    snowflake.py
    models.py

  control_plane/
    workspaces.py
    installations.py
    bot_configs.py
    mcp_clients.py
    guild_policies.py
    channel_policies.py

  automations/
    engine.py
    definitions.py
    triggers.py
    actions.py
    evaluators.py
    scheduler.py

  db/
    base.py
    session.py
    models.py
    repositories/
    migrations/

  security/
    policy.py
    redaction.py
    secrets.py
    allowlist.py
    validation.py

  services/
    guild_service.py
    channel_service.py
    message_service.py
    member_service.py
    role_service.py
    thread_service.py
    audit_service.py
    automation_service.py
    agent_service.py
    agent_tools.py
    moderation_service.py
    webhook_service.py
    invite_service.py
```

## Technical Stack

Use Python as the main implementation language.

Preferred stack:

- Python 3.12+
- uv
- FastAPI
- MCP Python SDK
- pydantic v2
- discord.py or nextcord
- httpx (Discord REST client)
- SQLAlchemy 2.x
- Alembic
- PostgreSQL
- Redis
- pytest
- ruff
- mypy
- Docker
- Docker Compose

For the dashboard, use a separate frontend later. Recommended frontend stack:

- Vue 3
- Vite
- TypeScript
- Tailwind CSS
- Pinia
- TanStack Query or equivalent

Do not build the dashboard in the first backend MVP unless explicitly requested.

## MCP Design

The MCP server must expose Discord capabilities as:

- tools: actions and queries the model can call;
- resources: addressable Discord/platform objects;
- prompts: reusable workflows for Discord operations.

The MCP server should support:

1. local STDIO mode for local development;
2. Streamable HTTP mode for hosted/remote usage.

HTTP transport should be the production target.

## Product Modes

### Mode A — Hosted Platform Bot

The user logs into the platform and installs the official platform bot into their server.

This is the MVP mode.

Benefits:

- easiest onboarding;
- single Discord app to manage;
- simpler security;
- simpler compliance;
- easier SaaS deployment.

### Mode B — Bring Your Own Bot

The user creates their own Discord application/bot, then connects bot credentials to the platform.

This is a later advanced mode.

Benefits:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rastrian/DiscordMCP](https://github.com/Rastrian/DiscordMCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
