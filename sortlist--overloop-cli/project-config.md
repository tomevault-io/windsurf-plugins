---
trigger: always_on
description: Overloop CLI skill — manage prospects, campaigns, sourcings, and outreach via the Overloop API v2 from the terminal.
---


# Overloop CLI Skill

Overloop is an AI-powered sales automation platform for managing prospects, campaigns, and outreach. This CLI lets you manage all resources from the terminal via the Overloop API v2.

## Setup

```bash
npm install -g overloop-cli
overloop login
# Or set env var: export OVERLOOP_API_KEY=your_api_key
```

## Concepts

- **Prospect**: A person you want to reach out to. Has email, name, custom fields, and can belong to an organization.
- **Organization**: A company a prospect belongs to.
- **List**: A group of prospects (tag-based).
- **Campaign**: An automated outreach sequence with steps (emails, delays, conditions, LinkedIn messages).
- **Step**: A single action in a campaign (email, delay, condition, LinkedIn visit/message, etc.).
- **Enrollment**: A prospect enrolled in a campaign.
- **Sourcing**: Automated prospect discovery from a 450M+ contact database using search criteria.
- **Conversation**: An email thread or LinkedIn conversation with a prospect.
- **Exclusion List**: Blocked emails and domains.
- **Custom Field**: User-defined fields on prospects/organizations.
- **Sending Address**: Connected email accounts used for sending.

## All Commands

All output is JSON. Pipe to `jq` for filtering.

### Auth

```bash
overloop login       # Interactive API key prompt
overloop logout      # Remove saved credentials
```

### Prospects

```bash
overloop prospects:list [--page N] [--per-page N] [--sort field] [--search text] [--filter '{"key":"value"}'] [--expand relations] [--sourcing-id ID]
overloop prospects:get <id>                  # ID or email
overloop prospects:create --email john@example.com [--first-name John] [--last-name Doe] [--organization-id ID]
overloop prospects:create --data '{"email":"john@example.com","first_name":"John","last_name":"Doe"}'
overloop prospects:update <id> --first-name Jane
overloop prospects:update <id> --data '{"phone":"+1234567890"}'
overloop prospects:delete <id>
```

### Organizations

```bash
overloop organizations:list [--search text] [--filter '{"country":"US"}'] [--expand relations]
overloop organizations:get <id>
overloop organizations:create --name "Acme Corp" [--website https://acme.com]
overloop organizations:create --data '{"name":"Acme Corp","website":"https://acme.com"}'
overloop organizations:update <id> --name "New Name"
overloop organizations:delete <id>
```

### Lists

Lists are also used as tags in campaign steps. To use the `add_to_tags` step type, create lists first and pass their IDs as `tag_ids`.

```bash
overloop lists:list [--search text]
overloop lists:get <id>
overloop lists:create --name "Hot Leads"
overloop lists:update <id> --name "Warm Leads"
overloop lists:delete <id>
```

### Campaigns

Two sourcing patterns are supported:

**Pattern A — Standalone sourcing as trigger:** Create a sourcing separately, then reference it with `--sourcing-id` and `--auto-enroll`. The sourcing remains independent; the campaign only uses it as an enrollment trigger. `campaign.sourcing_id` is NOT set.

**Pattern B — Embedded sourcing (preferred for new campaigns):** Pass `--search-criteria` (and optionally `--sourcing-limit`) directly to `campaigns:create`. The API creates a sourcing owned by the campaign. Auto-enrollment is enabled automatically. `campaign.sourcing_id` IS set.

```bash
overloop campaigns:list [--filter '{"status":"on"}'] [--expand steps,sourcing]
overloop campaigns:get <id> [--expand steps]
overloop campaigns:create --name "Q1 Outreach" [--timezone "Etc/UTC"] [--sender-id ID]

# Pattern A: use existing standalone sourcing as trigger
overloop campaigns:create --name "Q1" --auto-enroll --sourcing-id <id>

# Pattern B: create embedded sourcing with the campaign (preferred)
overloop campaigns:create --name "Q1" --search-criteria '{"job_titles":["CTO"],"locations":[{"id":22,"name":"Belgium","type":"Country"}]}' --sourcing-limit 100

# Inline steps
overloop campaigns:create --data '{"name":"Q1","steps":[{"type":"delay","config":{"days_delay":5}},{"type":"email","config":{"subject":"Hi","content":"Hello"}}]}'

overloop campaigns:update <id> --status on
overloop campaigns:update <id> --auto-enroll --sourcing-id <id>   # Pattern A on update
overloop campaigns:update <id> --search-criteria '...'            # Pattern B on update
overloop campaigns:update <id> --no-auto-enroll                   # disable auto-enrollment
overloop campaigns:delete <id>
overloop campaigns:stats <id>                                    # performance metrics
```

#### Campaign Stats

`campaigns:stats <id>` returns comprehensive performance metrics:
- **prospects**: total, active, contacted counts
- **enrollments**: active, completed, exited, errored, waiting, scheduled, in_review, disenrolled
- **email**: sent, contacted, opened, clicked, replied, bounced, opted_out, open_rate, reply_rate, bounce_rate
- **reply_sentiment**: positive, neutral, negative
- **linkedin**: actions_performed, visited, messaged, invited, replied
- **enrollment_sources**: overloop_database, event_trigger, linkedin_extension, existing_prospects, csv_import

Rates are percentages (0-100). Metrics come from cached aggregation tables and may lag by a few minutes. Check `last_updated_at` for freshness.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sortlist/overloop-cli](https://github.com/sortlist/overloop-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
