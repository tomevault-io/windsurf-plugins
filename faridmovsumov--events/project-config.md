---
trigger: always_on
description: This repository is a personal assistant for finding interesting events and activities for our family.
---

# Family Events Assistant

This repository is a personal assistant for finding interesting events and activities for our family.

## Family Profile

- **Family size**: 3
- **Members**: Husband, Wife, Son (6 years old, born ~2020)
- **Location**: Zaandam, Netherlands
- **Languages**: English preferred (Dutch is limited)
- **Travel radius**: Flexible - Zaandam, Amsterdam, and beyond. Location closeness is nice but not a hard requirement.

## Preferences

### What We Want
1. **Creative/building activities**: workshops where we build, make, or create something together
2. **Brain challenges**: escape rooms, puzzles, scavenger hunts, quiz nights - things that use intelligence
3. **Competitive activities**: events where we compete as a family team against other families
4. **Social/interactive**: activities where we meet and interact with other families, not just our own
5. **Physical challenges**: climbing, obstacle courses, sports - active things we do together

### What We DON'T Want
- Generic tourist attractions or sightseeing (we're locals, not tourists)
- Generic national holidays like King's Day (obvious, not really for kids)
- Passive activities (just watching something)
- Drop-off activities (must be together as family)
- "Just another museum visit" without interactive/creative element

### Key Criteria
- **Goal**: Spend quality time together as a family
- **Age-appropriate**: Must be suitable for a 6-year-old (update age as child grows)
- **International-friendly**: Prefer English-friendly activities, but not a hard requirement
- **Together**: Activities the whole family can enjoy together (not drop-off activities)
- **Engaging**: Use our intelligence, creativity, or physical skills - not passive entertainment
- **Social**: Prefer activities where we interact with other families when possible

### Budget
- Paid events are totally fine - don't focus on free events
- Open to all price ranges
- Note costs when researching so family can decide

## How to Use This Repository

### Quick Event Search
Ask Claude to search for events:
- "Find events for this weekend"
- "What's happening in spring break?"
- "Find outdoor activities for next month"
- "Search for new family activities"

### Run the Search Helper Script
```bash
./scripts/search_events.sh [category] [timeframe]
# Categories: all, physical, brain, outdoor, seasonal, deals
# Timeframe: weekend, week, month, spring, summer
# Add --open as 3rd arg to open URLs in browser
```

### Repository Structure
- `CLAUDE.md` - This file (family profile, preferences, quick reference)
- `events/creative-competitive.md` - **BEST LIST**: Creative, competitive, social activities (build, compete, solve puzzles)
- `family-activities-zaandam-area.md` - Comprehensive activity guide with 30+ venues, costs, details
- `scripts/sources.md` - 50+ curated event sources organized by type (English/Dutch/local/booking)
- `scripts/search_events.sh` - Search helper script with prompts and links
- `events/visited.md` - Events we've been to (with ratings)
- `events/wishlist.md` - Events we want to attend
- `events/seasonal.md` - Recurring seasonal events calendar

## Event Sources (Quick Reference)

See `scripts/sources.md` for the full list (50+ sources). Top sources by use case:

### English-Friendly
- [I amsterdam Family](https://www.iamsterdam.com/en/see-and-do/family-and-kids) - Best English event calendar
- [Amsterdam Mamas](https://amsterdam-mamas.nl/) - Expat family community, 22,000+ members
- [TimeOut Amsterdam Kids](https://www.timeout.com/amsterdam/kids) - Curated English recommendations
- [AmsterdamKids.com](https://amsterdamkids.com/) - Family offers and event early access
- [IamExpat.nl](https://www.iamexpat.nl/lifestyle/events-festivals-netherlands) - Expat events (has RSS)

### Local Zaandam
- [Go-Kids Zaanland](https://go-kids.nl/zaanland/agenda) - Weekly local family event agenda
- [De Orkaan Waarheen](https://waarheen.deorkaan.nl/agenda/) - 274+ local events
- [Rodi.nl Zaanstad](https://www.rodi.nl/zaanstad/uit/) - Weekly weekend tips (every Friday)
- [Zaanse Schans Events](https://www.dezaanseschans.nl/en/zaanse-schans-events/) - Local events (free for residents)

### Dutch Family Sites (use translator)
- [Uitmetkinderen.nl](https://www.uitmetkinderen.nl) - 3,000+ family outings, has app
- [Kidsproof Amsterdam](https://www.kidsproof.nl/amsterdam) - Kid-tested activities
- [DagjeWeg.nl](https://www.dagjeweg.nl/) - 5,000+ day trips

### Booking & Deals
- [Tiqets](https://www.tiqets.com/en/amsterdam) - Museum/attraction tickets with deals
- [Groupon NL](https://www.groupon.nl) - Discounted family activities

## Top Activities Quick Reference

### Creative, Competitive & Social (Our Favorites!)
See `events/creative-competitive.md` for full details. Highlights:

| Activity | Type | Age | Cost/person | Why |
|----------|------|-----|-------------|-----|
| parkrun Amsterdamse Bos | Community run | 4+ | FREE | Meet families weekly, build social network |
| BAZ Bouldergym (Zaandam!) | Bouldering | 5+ | 10 EUR | Right here, brain + physical |
| Laser Tag (UP Events) | Team battles | 6+ | From 10.99 EUR | Your son CAN do this now |
| Sherlocked Escape Room | Brain puzzles | 6+ | 30-40 EUR | Best escape room for age 6 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faridmovsumov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
