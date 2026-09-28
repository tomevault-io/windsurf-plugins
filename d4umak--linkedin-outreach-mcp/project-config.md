---
trigger: always_on
description: For repository changes, read CLAUDE.md and [docs/delivery.md](docs/delivery.md).
---

# HeyLead — Agent Integration Guide

For repository changes, read CLAUDE.md and [docs/delivery.md](docs/delivery.md).
Ship one complete outcome per PR, with its tests and self-review corrections.
Queue labels never replace the user's separate merge/release authorization.

## What HeyLead Does

HeyLead is an AI agent for LinkedIn outreach: it finds the right people, writes to them in the voice of your own LinkedIn posts, follows up, and handles replies. It runs from Claude Code, Cursor, any MCP client or a web dashboard. Every action is an MCP tool call.

**Use cases:** A campaign has one of six goals, and the ICP, the fit check and the messages follow it: Sell a product or service, Find a job, Hire people, Find partners or investors, Find a vendor and Research interviews.

- **Sell a product or service**: Reach the people who buy what you built.
- **Find a job**: Reach the people who hire for the role you want.
- **Hire people**: Reach candidates for a role you are filling. Works with a custom brief.
- **Find partners or investors**: Reach the people who can sign a partnership or an investment. Works with a custom brief.
- **Find a vendor**: You are the buyer. Reach the people who sell what you need.
- **Research interviews**: Reach people to interview, survey or test with. Works with a custom brief.

Hire people, Find partners or investors and Research interviews run on a custom brief until their message sets exist.

Also known as: LinkedIn lead generation, cold outreach automation, B2B prospecting, SDR automation, campaign management, ICP (Ideal Customer Profile) generation, multi-touch drip sequences, engagement warm-ups, and outreach analytics.

## Capabilities

| Category | What it does |
|----------|-------------|
| **ICP Generation** | RAG-powered buyer personas with pain points, fears, barriers, and LinkedIn search parameters |
| **Campaign Management** | Create, launch, pause, resume, archive, delete, and compare outreach campaigns |
| **Outreach Automation** | Personalized connection invitations, follow-up DMs, engagement warm-ups (comments, likes) |
| **Reply Handling** | Sentiment classification (positive/negative/question/neutral), auto-responses, meeting scheduling |
| **Intent Signals** | Company news, page engagement, website visitors, profile viewers — compounded into outreach angles |
| **Analytics** | Funnel reports, conversion rates, stale lead detection, engagement ROI |
| **Autonomous Scheduling** | Cloud is the default sender for hosted accounts (existing and new campaigns). Launching commissions the cloud: invitations, opening DMs, first-touch InMail, follow-ups, engagements, follows, endorsements, email fallbacks, campaign top-ups, auto-replies, inbound, warmup, signal collectors, post-intel, and housekeeping. This machine stays silent for that work unless the user runs `scheduler(action='send_from', host='local')`, which turns the cloud scheduler off. The local engine does not start on a hosted cloud account. Observe still means nobody sends, including the cloud. Direct / self-hosted installs send from this machine only. |

## Typical Workflow

```
1. setup_profile(backend_jwt="...")             → Connect LinkedIn account
2. generate_icp(target_description="CTOs fintech") → Create buyer personas
3. create_campaign(target_description="...", icp_id="...") → Find prospects (saved as a draft)
4. campaign(action="launch", campaign_id="...") → Start outreach — nothing sends before this
                                                   (hosted accounts: also starts 24/7 cloud sending)
5. scheduler(action="status")                   → Confirm sending is on the cloud (or send_from host=local)
6. inspect() / check_replies() / show_status()  → Monitor pipeline and agent holds
7. prospect(action="close", outcome="won")      → Track conversions
```

## Agent ops

When the user asks what the agents did, who is held, or why a reply was skipped, call `inspect()` first. It is read-only and never writes. If they ask what the agents decided on a hosted account, call `inspect(action='journal')`. If a campaign looks idle in the send window, call `inspect(action='review')`. If they ask what the agents left for the next tick, what the swarm thinks, or who went dark, call `inspect(action='commons')`. A campaign-wide coordinator hold: `campaign(action='clear_coordinator_hold', campaign_id='...')`.

A hold: `prospect(action="conversation", outreach_id="...")` then `send_message(action="reply", outreach_id="...")`. Operator replies skip the reply agent.

Never paste model-authored text as the LinkedIn message. The send tools generate it.

Never launch a draft unless the user asked.

HeyLead sends from your own LinkedIn account at a human pace: at most 20 invitations a day and 100 a week on a free LinkedIn account (more on Premium or Sales Navigator), Monday to Friday 08:00 to 22:00 in your time zone, minutes apart. It backs off when LinkedIn pushes back and resumes on its own. You can pause any campaign at any time.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [D4umak/linkedin-outreach-mcp](https://github.com/D4umak/linkedin-outreach-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
