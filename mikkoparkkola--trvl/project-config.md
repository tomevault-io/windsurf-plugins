---
trigger: always_on
description: > **Human**: Give this URL to your AI assistant and say "set up trvl":
---

# trvl — Complete Setup for AI Assistants

> **Human**: Give this URL to your AI assistant and say "set up trvl":
> `https://raw.githubusercontent.com/MikkoParkkola/trvl/main/AGENTS.md`

## What To Do

You are setting up `trvl` — a travel MCP server that gives you real-time Google Flights and Google Hotels data. No API keys needed. Run these steps in order:

### Step 1: Install

```bash
brew install MikkoParkkola/tap/trvl
```

Pre-built binary, no toolchain needed, 1 second. If Homebrew is not available:

```bash
# Manual: detect OS/arch, download, extract
curl -fsSL https://github.com/MikkoParkkola/trvl/releases/latest/download/trvl_$(uname -s | tr '[:upper:]' '[:lower:]')_$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/').tar.gz | tar xz -C /usr/local/bin trvl
```

### Step 2: Register as MCP server

```bash
trvl mcp install                       # Claude Desktop (default)
trvl mcp install --client cursor       # Cursor / Windsurf
trvl mcp install --client claude-code  # Claude Code
```

Then restart your MCP client. No JSON editing needed.

For Claude Code, you can also run:
```bash
claude mcp add trvl --transport stdio -- trvl mcp
```

### Step 3: Install the bundled skill (makes you smarter about travel searches)

```bash
mkdir -p ~/.claude/skills
for s in trvl providers; do
  curl -fsSL "https://raw.githubusercontent.com/MikkoParkkola/trvl/main/.claude/skills/$s.md" -o "$HOME/.claude/skills/$s.md"
done
```

The skills at `.claude/skills/trvl.md` and `.claude/skills/providers.md` teach you how to:
- Ask the right questions (From? To? When? Flex? Budget?)
- Run hack detectors automatically after every search
- Use the unified optimizer (`optimize_booking`) for trip planning
- Show the "Naive -> Optimized -> Saved" comparison after every plan
- Use all-in pricing with FF benefits (bag fees included, status benefits subtracted)
- Apply 37 travel hack detectors to find savings opportunities
- Configure optional hotel/restaurant/ground providers only after verified source-code research and user consent

### Step 4: Verify

```bash
trvl version
# Expected: trvl 0.3.0 (or later)

trvl flights HEL LHR 2026-07-01 --format json | head -5
# Expected: JSON with flight results
```

Tell the user: "trvl is installed with 61 MCP tools and 2 bundled Claude skills. It includes 37 travel hack detectors (including error fare and flash sale detection) that auto-fire on searches, a unified optimizer (optimize_booking) with 9 expansion strategies (alternative origins/destinations, rail+fly, date flex, hidden city, departure tax avoidance, rail competition alternatives, ferry cabin as hotel) that searches all combinations in parallel, all-in pricing with FF status (bag fees included, FF benefits subtracted), pre-priced candidate pipeline for ground alternatives, miles tracking and earning estimates, cross-program award sweet-spot scanning, and cross-provider hotel price comparison with cross-currency savings display. I can search flights, hotels, destinations, plan trips, find weekend getaways, find optimal travel windows, optimize multi-city routes, find nearby restaurants, check local events, search ground transport (buses, trains, ferries, night trains), detect travel hacks, check weather forecasts, look up airline baggage rules, find airport lounges, check visa requirements, calculate points-vs-cash redemptions, and configure additional data providers (Airbnb, Booking.com, Hostelworld). Just ask me anything about travel."

### Step 5: Build travel profile (recommended)

Run the onboarding interview to learn how the user travels:

1. Call `onboard_profile` with `phase: 1` — ask the basics (home, frequency, companions)
2. After user answers, call `onboard_profile` with `phase: 2` — travel style (accommodation, budget, transport)
3. Continue through phases 3 (favourite cities, properties), 4 (companion, wishlist), and 5 (reasoning and strategies)
4. Each phase skips questions the profile already answers
5. Save answers to `~/.trvl/profile.json` via `add_booking` or `update_preferences`

If the user has email access (Gmail), also offer to scan booking history:
```
Call build_profile with source: "email" to scan Gmail for past bookings
```

The profile powers personalized search — preferred neighbourhoods, price elasticity, booking strategies, and destination recommendations.

### Step 6: (Optional) Set up free API keys for enhanced data

trvl works out of the box with Wikivoyage + OpenStreetMap (no keys needed). For richer data (events, restaurant ratings, attractions), the user can get free API keys:

| Service | What it adds | Signup |
|---------|-------------|--------|
| Ticketmaster | Events (concerts, sports, festivals) | https://developer.ticketmaster.com/ |
| Foursquare | Restaurant ratings, tips, price levels | https://developer.foursquare.com/ |
| Geoapify | Walking-distance POI search | https://myprojects.geoapify.com/ |
| OpenTripMap | Tourist attractions + Wikipedia | https://opentripmap.io/product |

All free, no credit card, 2 min signup each. Walk the user through each signup:
1. Open the URL for them
2. Tell them what to click (Sign up → Create project → Copy key)
3. Have them paste the key
4. Set it: `echo 'export TICKETMASTER_API_KEY="their-key"' >> ~/.zshrc && source ~/.zshrc`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MikkoParkkola/trvl](https://github.com/MikkoParkkola/trvl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
