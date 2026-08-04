---
trigger: always_on
description: You are looking at the **GTM System repo** maintained by [La Growth Machine](https://lagrowthmachine.com). It bundles a library of public Claude skills (sourcing, list building, campaigns, copywriting, analytics, reply handling) and an MCP server that lets Claude act directly inside the user's LGM workspace.
---

# LGM GTM Skills — Claude setup

You are looking at the **GTM System repo** maintained by [La Growth Machine](https://lagrowthmachine.com). It bundles a library of public Claude skills (sourcing, list building, campaigns, copywriting, analytics, reply handling) and an MCP server that lets Claude act directly inside the user's LGM workspace.

## First-time setup

Check if the LGM MCP is already configured by running `claude mcp list` (look for a line starting with `LaGrowthMachine:`). The installer registers the MCP via `claude mcp add --scope user --transport http`, which stores the config in `~/.claude.json` at user scope (available in every project). If the `claude` CLI isn't available, ask the user whether they've already run `sh install.sh`.

**If it is NOT configured**, proactively tell the user:

> "I can see this is the LGM GTM Skills repo. Would you like me to install the LGM MCP and skills? It takes ~1 minute — I'll run `sh install.sh` for you. On first use, a browser tab opens to sign in to La Growth Machine (OAuth) — new accounts can be created from that tab. No API key to copy, nothing to paste."

If they say yes, run `sh install.sh` and confirm when done.

**If it IS configured**, present the onboarding to the user:

1. Confirm MCP is connected and list the available MCP tools grouped by category (Audiences, Campaigns, Inbox, Send, Workspace, LinkedIn, BigQuery).
2. List the available GTM Skills grouped by category.
3. Show the suggested first-steps table.
4. Ask: *"What would you like to do?"*

---

## What LGM gives you

Once the MCP is connected and skills are installed, two layers work together.

---

### Layer 1 — LGM MCP (Claude acts inside La Growth Machine)

The MCP gives Claude direct access to the user's LGM workspace. No copy-paste, no tab switching. Tools are exposed under the namespace `mcp__LaGrowthMachine__*`.

**Audiences**
- `create_audience_from_linkedin_url` — create an audience directly from a Sales Navigator search URL
- `get_audience` — fetch an existing audience by ID
- `get_audience_leads` — list all leads in an audience (paginated)

**Campaigns — read**
- `list_campaigns` — list all campaigns in the workspace
- `get_campaign_stats` — open rates, reply rates, meetings booked, performance data
- `get_campaign_messages` — fetch the messages of a campaign's sequence
- `get_campaign_steps` — fetch the sequence structure (channels, waits, branches)

**Campaigns — build / edit**
- `duplicate_campaign` — clone an existing campaign as the starting point for a new one
- `add_campaign_step_message` — add a message to a specific step of a draft campaign
- `edit_campaign_message` — edit an existing message in a draft campaign

**Inbox / conversations — read**
- `get_lead_conversations` — all conversations for a specific lead
- `get_conversation_messages` — the full message thread of a conversation
- `get_lead_logs` — the activity log for a lead (visits, clicks, replies)
- `get_unread_conversations` — inbox: unread conversations
- `get_conversations_to_reply` — inbox: conversations waiting for a reply
- `get_favourite_conversations` — inbox: starred conversations
- `search_conversations` — search the inbox by keyword, lead, campaign, status

**Inbox / conversations — actions**
- `send_email_message` — send an email reply in a conversation
- `send_linkedin_message` — send a LinkedIn reply in a conversation
- `snooze_conversation` / `unsnooze_conversation` — snooze a thread for later
- `archive_conversation` / `unarchive_conversation` — archive / restore a thread

**Workspace**
- `list_workspaces` — list the workspaces the user has access to
- `list_members` — list the members of the current workspace
- `list_identities` — list all identities (LinkedIn accounts, email accounts)
- `save_identity_preference` — set a preferred identity for outreach

**LinkedIn**
- `get_linkedin_post` — fetch a LinkedIn post and its engagers (likers, commenters)

**BigQuery**
- `execute_bigquery_query` — run a BigQuery query against LGM data
- `get_bigquery_logs_guide` — get the guide to LGM's log schema (what tables and fields exist)

---

### Layer 2 — GTM Skills (published in this repo)

Skills guide Claude through complex GTM workflows. Just describe what you want. Skills are categorized into 4 outcomes mirroring the GTM motion:

**Fuel my pipeline** — sourcing, list building, ICP

- `sales-nav-search-builder` — turn a natural-language ICP into a precise LinkedIn Sales Navigator search URL, ready to import as an LGM audience
- `post-to-campaign` — turn a LinkedIn post into a ready-to-launch campaign: scrape the post's likers and commenters into an audience and fill a draft sequence
- `audience-icp-filter` — filter an existing audience against an ICP; sorts every lead into match / needs review / no match, strips your team and competitors, never silently drops anyone
- `won-deal-icp-finder` — audit your biggest closed-won deals to find your proven ICP and a look-alike target list

**Get qualified meetings** — campaigns, copywriting, sequences


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LaGrowthMachine/gtm-system](https://github.com/LaGrowthMachine/gtm-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
