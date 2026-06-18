---
trigger: always_on
description: |
---


# Feast

A meal planning skill that transforms weekly cooking into a cultural experience.

## Quick Start

1. **New user?** Run onboarding: "Let's set up Feast" or "Onboard me for meal planning"
2. **Returning user?** Check status: "What's the meal plan status?"
3. **Planning day?** Start planning: "Let's plan next week's meals"
4. **Cooking day?** Get reveal: "What's for dinner?"

## Core Files

User data lives in their workspace:

```
workspace/meals/
├── profile.yaml          # User preferences (created during onboarding)
├── history.yaml          # What they've eaten
├── favourites.yaml       # Loved recipes
├── failures.yaml         # Never again
└── weeks/
    └── YYYY-MM-DD.md     # Each week's plan (self-contained)
```

**Note:** Weekly plans are fully self-contained — each day's recipe, theme research, music playlist, and cultural context is embedded directly in the week file. There are no separate recipe or theme files.

## Weekly Cadence

Default schedule (user-configurable):

| Day | Activity | Trigger |
|-----|----------|---------|
| Thursday | Research & draft | "Let's plan next week" |
| Friday | Confirm plan | "Confirm the meal plan" |
| Saturday | Shopping list | "Generate shopping list" |
| Sunday | Shopping | User shops |
| Week | Daily reveals | "What's for dinner?" |
| End of week | Review | "Review this week's meals" |

## Notifications

Feast sends reminders at key moments: planning day, confirmation, shopping list, daily reveals, and week review. These are delivered via cron jobs that spawn isolated agents to send notifications.

### Notification Channels

Users configure their preferred channel in `profile.yaml` under `schedule.notifications.channel`:

| Channel | Delivery Method |
|---------|-----------------|
| `auto` | Delivers to the current session or first available channel |
| `telegram` | Sends via Telegram (requires Telegram channel configured in OpenClaw) |
| `discord` | Sends via Discord (requires Discord channel configured in OpenClaw) |
| `signal` | Sends via Signal (requires Signal channel configured in OpenClaw) |
| `webchat` | Outputs to the chat session |

### Push Notifications (Optional)

For notifications to mobile devices independent of chat channels, users can enable push notifications:

```yaml
schedule:
  notifications:
    push:
      enabled: true
      method: "pushbullet"    # or "ntfy"
```

**Supported methods:**

- **Pushbullet** — Requires the `pushbullet-notify` skill installed separately with API key configured
- **ntfy** — Uses ntfy.sh (or self-hosted); configure topic in profile

Push notifications are sent *in addition to* the primary channel, not instead of it. If push delivery fails, the notification still goes to the primary channel.

### Timing

Notifications are delivered via OpenClaw's cron system with `wakeMode: "next-heartbeat"`. This means notifications arrive within the heartbeat interval (typically up to 1 hour) after the scheduled time. For most meal planning purposes, this slight delay is acceptable.

### Managing Notifications

Users can adjust their notification preferences anytime:

- "Change my Feast notifications to Telegram"
- "Turn off morning hints"
- "Enable Pushbullet notifications"

When updating, remove old cron jobs using stored IDs and create new ones with updated settings.

## Workflows

### Onboarding

Read [references/onboarding.md](references/onboarding.md) for the full flow.

Essential questions:
1. Location (for seasonality, units, stores)
2. Household size & portion needs
3. Week structure (start day, cooking days, cheat day)
4. Dietary requirements & phase
5. Equipment & cooking confidence
6. Preferences (cuisines, spice, budget)

Save to `workspace/meals/profile.yaml`.

### Planning (Thursday)

1. Check user profile
2. Review history (avoid recent repeats)
3. Check upcoming cultural events (see [references/events.md](references/events.md))
4. Check seasonality for location
5. Select 6-7 meals with:
   - Cuisine variety
   - Ingredient overlap
   - Balanced nutrition
   - Mix of quick/involved
6. **For each meal, research and embed:**
   - **The Place:** Identify specific region of origin (drill down to province, city, or area). Research regional context, history, current events. Write an evocative description.
   - **The Dish:** Research authentic recipe from native sources (search in original language). Include origin story, cultural significance, full ingredients and method.
   - **The Soundtrack:** Curate a 1-2 hour playlist with contemporary hits + classic/traditional from the region (see [references/theme-research.md](references/theme-research.md)). Include full tracklist with links.
   - **Setting the Scene:** How to serve, what to drink, atmosphere tips.
7. Draft plan to `workspace/meals/weeks/YYYY-MM-DD.md` (all content embedded in this single file)
8. Present summary (themes only, not full reveals)

### Confirmation (Friday)

1. Present draft plan with themes only, not full reveals
2. Allow amendments
3. Mark as confirmed
4. Set up daily reveal reminders

### Shopping List (Saturday)

1. Generate from confirmed plan
2. Optimise:
   - Group by category
   - Combine overlapping ingredients
   - Check pack sizes vs needs
   - Flag seasonal items
3. **Price check key ingredients** (see [references/price-checking.md](references/price-checking.md)):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smadgerano/feast](https://github.com/smadgerano/feast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
