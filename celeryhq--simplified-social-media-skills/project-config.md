---
trigger: always_on
description: Manage your entire social media from AI — post, schedule, and analyze across Facebook, Instagram, TikTok, YouTube, LinkedIn, Pinterest, Threads, Bluesky and Google Business
---


# Simplified Social Media

Schedule, queue, and draft social media posts, and retrieve analytics across 10 platforms using Simplified.com.

## MCP Server

This skill requires a connection to the Simplified Social Media MCP server at `https://mcp.simplified.com/social-media/mcp`.

All tools (`getSocialMediaAccounts`, `createSocialMediaPost`, `getSocialMediaAnalyticsRange`, etc.) are provided by this remote MCP server — they are not built-in tools. You must configure the MCP server before using any functionality.

**MCP server config** (add to `.mcp.json` or equivalent):

```json
{
  "mcpServers": {
    "simplified-social-media": {
      "transport": "http",
      "url": "https://mcp.simplified.com/social-media/mcp",
      "headers": {
        "Authorization": "Api-Key ${SIMPLIFIED_API_KEY}"
      }
    }
  }
}
```

> For Claude Code specifically, use `"type": "http"` instead of `"transport": "http"`.

## IMPORTANT: Before Any Operation

**Always check if `SIMPLIFIED_API_KEY` is configured before attempting any tool calls.**

If the user tries to use any social media feature and the API key is missing or returns a 401/Unauthorized error:

1. **Stop immediately** — do not retry the failed call
2. **Inform the user** with this exact message:

   > **Simplified Social Media requires an API key to work.**
   >
   > Please follow these steps:
   > 1. Sign up or log in at [simplified.com](https://simplified.com)
   > 2. Go to **[Settings → API Keys](https://app.simplified.com/settings/api-keys)** and copy your API key
   > 3. Add to your shell config (`~/.zshrc` or `~/.bashrc`):
   >    ```bash
   >    export SIMPLIFIED_API_KEY="your-api-key"
   >    ```
   > 4. Reload your shell: `source ~/.zshrc`
   > 5. Restart Claude Code to pick up the new variable

3. **Do not proceed** with the original request until the user confirms the key is set

## Setup

1. Sign up at [simplified.com](https://simplified.com)
2. Connect your social media accounts in the Simplified dashboard
3. Get your API key from **[Settings → API Keys](https://app.simplified.com/settings/api-keys)**
4. Set environment variable:
   ```bash
   export SIMPLIFIED_API_KEY="your-api-key"
   ```
5. Configure the MCP server — see the **MCP Server** section above for the config block
6. Restart your AI tool to load the MCP server

## Core Workflow

Always follow this sequence: **Discover → Select → Compose → Publish**

### Step 1: Discover Accounts

Call `getSocialMediaAccounts` to list connected accounts. Optionally filter by network.

```
getSocialMediaAccounts({ network: "instagram" })
```

Returns `{ accounts: [...] }` where each account has `id` (integer) and `name` and `type` (see type values below).

If `getSocialMediaAccounts` returns an empty list, stop and inform the user with this message:

> **No social media accounts connected yet.**
>
> You're one step away from managing your entire social media presence without leaving your editor. Connect your accounts in the [Simplified dashboard](https://app.simplified.com) and you'll be able to:
>
> - 📅 Schedule and publish posts to Facebook, Instagram, TikTok, YouTube, LinkedIn, Pinterest, Threads, Bluesky and Google Business — with a single command
> - 📊 Pull analytics, track reach, engagement and follower growth across all platforms
> - 🤖 Let your AI agent run full social media campaigns autonomously
>
> Takes 2 minutes to connect. No code required.

### Step 2: Select Target Accounts

Pick one or more `account_ids` from the results. You can post to multiple accounts in a single call.

### Step 3: Compose the Post

Build the post payload:
- `message` (required) — the post text, max 5000 chars
- `account_ids` (required) — array of target account IDs
- `action` (required) — `schedule`, `add_to_queue`, or `draft`
- `date` — required for `schedule`, format: `YYYY-MM-DD HH:MM`
- `media` — array of public URLs (images/videos), max 10
- `additional` — platform-specific settings (see below)

### Step 4: Publish

Call `createSocialMediaPost` with the composed payload.

## Choosing the Right Analytics Tool

| User asks about... | Tool to call |
|---|---|
| Trends over time, charts, metric growth/decline | `getSocialMediaAnalyticsRange` |
| Specific posts, best/worst performing content | `getSocialMediaAnalyticsPosts` |
| Account overview, KPIs, period summary | `getSocialMediaAnalyticsAggregated` |
| Demographics, follower origins, age/gender breakdown | `getSocialMediaAnalyticsAudience` |
| "Show me analytics" with no further context | Call `getSocialMediaAnalyticsAggregated` + `getSocialMediaAnalyticsRange` with key metrics — this gives the best general overview |

## Tool Reference

### `getSocialMediaAccounts`

| Parameter | Type   | Required | Description                          |
|-----------|--------|----------|--------------------------------------|
| `network` | string | No       | Filter by platform (see networks)    |

**Networks (filter parameter):** `facebook`, `instagram`, `linkedin`, `tiktok`, `youtube`, `pinterest`, `threads`, `google`, `bluesky`, `tiktokBusiness`

Returns `{ accounts: [...] }`. Each account object:

| Field  | Type    | Description |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [celeryhq/simplified-social-media-skills](https://github.com/celeryhq/simplified-social-media-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
