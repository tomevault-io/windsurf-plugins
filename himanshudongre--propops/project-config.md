---
trigger: always_on
description: PropOps brings transparency to Indian real estate by aggregating publicly available government data. Every property transaction is public record on IGRS portals. RERA tracks builder complaints. eCourts tracks litigation. This data exists -- it's just buried across dozens of government websites. PropOps makes it searchable, scored, and actionable.
---

# PropOps -- AI Property Transparency Tool

## Origin

PropOps brings transparency to Indian real estate by aggregating publicly available government data. Every property transaction is public record on IGRS portals. RERA tracks builder complaints. eCourts tracks litigation. This data exists -- it's just buried across dozens of government websites. PropOps makes it searchable, scored, and actionable.

**It works out of the box for Maharashtra, but it's designed to be made yours.** If the scoring doesn't fit your priorities, the locations don't match your search, or the deal-breakers are wrong -- just ask. You (Claude) can edit the user's files. The user says "change the budget to 1.5Cr" and you do it. That's the whole point.

## Data Contract (CRITICAL)

There are two layers. Read `DATA_CONTRACT.md` for the full list.

**User Layer (NEVER auto-updated, personalization goes HERE):**
- `buyer-brief.md`, `config/profile.yml`, `modes/_profile.md`
- `sources.yml`, `telegram-config.yml`
- `data/*`, `reports/*`

**System Layer (auto-updatable, DON'T put user data here):**
- `modes/_shared.md`, `modes/evaluate.md`, all other modes
- `CLAUDE.md`, `scripts/*.mjs`, `dashboard/*`, `templates/*`, `batch/*`

**THE RULE: When the user asks to customize anything (budget, locations, deal-breakers, scoring weights, builder blacklist), ALWAYS write to `modes/_profile.md` or `config/profile.yml`. NEVER edit `modes/_shared.md` for user-specific content.** This ensures system updates don't overwrite their customizations.

## What is PropOps

AI-powered property intelligence pipeline built on Claude Code: property evaluation, price transparency, builder reputation, litigation checks, negotiation strategy, and Telegram alerts.

### Main Files

| File | Function |
|------|----------|
| `data/properties.md` | Property tracker (main pipeline) |
| `data/watchlist.md` | Areas/projects being monitored |
| `data/scan-history.tsv` | Scanner dedup history |
| `sources.yml` | Data source & portal config |
| `buyer-brief.md` | Buyer requirements document |
| `reports/` | Evaluation reports (format: `{###}-{project-slug}-{YYYY-MM-DD}.md`) |

### First Run -- Onboarding (IMPORTANT)

**Before doing ANYTHING else, check if the system is set up.** Run these checks silently every time a session starts:

1. Does `buyer-brief.md` exist?
2. Does `config/profile.yml` exist (not just profile.example.yml)?
3. Does `modes/_profile.md` exist (not just _profile.template.md)?
4. Does `sources.yml` exist (not just templates/sources.example.yml)?

If `modes/_profile.md` is missing, copy from `modes/_profile.template.md` silently.

**If ANY of these is missing, enter onboarding mode.** Do NOT proceed with evaluations, scans, or any other mode until the basics are in place. Guide the user step by step:

#### Step 1: Buyer Brief (required)
If `buyer-brief.md` is missing, ask:
> "I need to understand what you're looking for. Tell me:
> 1. Are you buying for self-use or investment?
> 2. Your budget range (e.g., 60L-90L)
> 3. Preferred city and areas (e.g., Pune -- Hinjewadi, Baner, Keshav Nagar)
> 4. Configuration (1/2/3 BHK, villa, plot)
> 5. Minimum carpet area (sqft)
> 6. Any deal-breakers? (e.g., no under-construction, no builder <5 years old)
>
> I'll create your buyer brief and personalize the system."

Create `buyer-brief.md` from whatever they provide. Clean markdown with sections: Purpose, Budget, Locations, Configuration, Must-haves, Deal-breakers.

#### Step 2: Profile (required)
If `config/profile.yml` is missing, copy from `config/profile.example.yml` and ask:
> "A few more details to personalize:
> - Your name (for reports)
> - Your timeline (when do you want to buy?)
> - Loan pre-approval status and budget
> - Family size (affects configuration recommendations)
>
> I'll set everything up."

Fill in `config/profile.yml` with their answers.

#### Step 3: Sources (recommended)
If `sources.yml` is missing:
> "I'll set up the property scanner for Maharashtra with pre-configured portals (IGRS, MahaRERA, eCourts, 99acres). Want me to customize the search areas for your target locations?"

Copy `templates/sources.example.yml` to `sources.yml`. Update locations to match buyer brief.

#### Step 4: Tracker
If `data/properties.md` doesn't exist, create it:
```markdown
# Property Tracker

| # | Date | Project | Builder | Location | Config | Area | Price(L) | Rs/sqft | Score | Status | Report | Notes |
|---|------|---------|---------|----------|--------|------|----------|---------|-------|--------|--------|-------|
```

#### Step 5: Get to know the buyer (important for quality)

After basics are set up, proactively ask:

> "The system works much better when it knows your priorities well. Can you tell me:
> - What matters most? (price, location, builder reputation, appreciation potential?)
> - Any builders you trust or distrust?
> - Are you flexible on location if the right deal comes up?
> - First-time buyer or have you purchased before?

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [himanshudongre/propops](https://github.com/himanshudongre/propops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
