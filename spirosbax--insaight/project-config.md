---
trigger: always_on
description: Scrapes LinkedIn company pages and personal profiles via Apify, stores in SQLite,
---

# Insaight — LinkedIn Intelligence Tool

Scrapes LinkedIn company pages and personal profiles via Apify, stores in SQLite,
and exposes them to Claude via MCP for prospect research and cold outreach.

Skills live in `skills/<name>/SKILL.md` — each is independently invocable and they chain
conversationally (research → outreach → save).

---

## Setup

User config lives in `~/.insaight/config.md` (created by `get_config()` on
first call) — **not** in this file. The block below is the template; skills
call `insaight:get_config` to read it.

```
NOTION_WORKSPACE:     YourWorkspaceName   # your Notion workspace / team name
NOTION_RESEARCH_PAGE: Prospect Research   # parent page for research briefs
NOTION_OUTREACH_LOG:  Sent Log            # your sent-messages archive (READ-ONLY)
COMPANY_NAME:         YourCompany         # your company (used by draft-post)
COMPANY_LINKEDIN:     your-company-slug   # your LinkedIn company page slug
```

---

## Insaight MCP — Tool Reference

All tools must be loaded first via `tool_search(query="insaight")`.

| Tool | Purpose | Key params |
|------|---------|------------|
| `insaight:list_accounts` | Show all tracked accounts + slugs | — |
| `insaight:get_stats` | DB-wide overview (counts, date range, categories) | — |
| `insaight:list_posts` | Slim index: metadata + 150-char snippet, NO full text | `account`, `days_ago`, `limit`, `min_engagement` |
| `insaight:search_posts` | Full-text keyword search → returns full content | `query`, `account`, `days_ago`, `limit` |
| `insaight:get_posts` | Fetch full content for specific URNs (max 20) | `urns: string[]` |
| `insaight:scrape_profile` | Scrape fresh posts from any LinkedIn URL via Apify | `url`, `max_posts` |
| `insaight:list_people` | List stored employees/leadership from DB (instant) | `account`, `role`, `limit` |
| `insaight:scrape_people` | Scrape + store company leadership via Apify | `url`, `job_titles`, `max_items`, `full_mode` |
| `insaight:scrape_person_profile` | Enrich ONE person with full profile (experience, education, skills, etc.) | `url`, `with_email`, `company_url` |
| `insaight:scrape_post_comments` | Scrape comments on a LinkedIn post (with replies + author info) | `post_url`, `max_items`, `include_replies`, `profile_mode` |
| `insaight:list_comments` | List stored comments for a post, ranked by likes | `post_urn` or `post_url`, `limit`, `min_likes`, `include_replies` |
| `insaight:log_outreach` | Record a SENT message in the ledger (flags prior contact) | `target_url`, `message`, `target_name`, `channel`, `variant`, `hook_type` |
| `insaight:record_outcome` | Record replied/positive/meeting/ghosted; flags when reflection is due | `outreach_id` or `target_url`, `outcome`, `reply_snippet` |
| `insaight:list_outreach` | Query the outreach ledger (prior-contact checks, pending review) | `outcome`, `target`, `limit`, `full` |
| `insaight:get_outreach_stats` | Reply-rate breakdown by hook/variant/channel + reflection state | — |
| `insaight:get_memory` | Read distilled style + playbook memory (draft skills read THIS) | — |
| `insaight:update_memory` | Rewrite a memory file — only after user approval | `kind`, `content`, `mark_reflection_done` |
| `insaight:get_config` | User config: Notion pages + company name/slug (`~/.insaight/config.md`) | — |

### Efficient reading pattern

```
1. list_accounts()                    → find the correct slug
   └─ not found? → scrape_profile(url) to fetch and store fresh posts

2. list_posts(account=slug, limit=50) → survey what exists (slim, token-cheap)

3. get_posts(urns=[...])              → read full content for chosen posts only
   OR search_posts(query=...)         → targeted lookup when you know what to look for

4. list_people(account=slug)          → check stored leadership
   └─ empty? → scrape_people(url, job_titles=["CEO","Founder","CTO","Head of"])

5. scrape_person_profile(url=...)     → enrich ONE person with full profile
   (experience, education, skills, volunteer, languages, projects, recommendations)
   Use for commonality mining — overlapping companies, schools, volunteer work.

6. list_comments(post_url=...)        → read commenters on a post (free if already scraped)
   └─ empty? → scrape_post_comments(post_url, include_replies=True)
   Use for mining a thread for warm leads, decision-makers, competitor mentions.
```

**Never call `get_posts` on all URNs at once.** Select the most signal-rich posts
to stay within the 20-URN limit per call.

### Cost warnings

`scrape_profile` and `scrape_people` call Apify (costs tokens/money). Only use
them when the account isn't tracked yet or data is stale. `list_people` is
free/instant — always try it first.

### Account slug format

- Company page: `acme-charging` (matches the LinkedIn URL slug)
- Personal profile: `jane-doe-12345678` (matches the LinkedIn profile ID)

### Engagement benchmarks (example: EV charging niche, NL/EU B2B — adjust for yours)

- < 10 likes = low
- 10–40 likes = normal
- 40+ likes = high signal post, read in full

### Outreach memory loop

The ledger + memory system replaces re-reading a raw sent-log every session:

1. **Log every send** (`log_outreach`) with the exact sent text + `variant` + `hook_type`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spirosbax/insaight](https://github.com/spirosbax/insaight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
