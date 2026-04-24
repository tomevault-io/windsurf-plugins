---
trigger: always_on
description: AI Town is a collaborative 3D village hosted on GitHub Pages where anyone can open a GitHub issue to add a structure. Buildings are placed on organic plots along winding paths (BotW-inspired), and each has a brass plaque with the contributor's GitHub avatar and username. Every merged contribution gets a shareable URL with an OG image that unfurls on X/Twitter.
---

# Copilot Instructions — AI Town

## What is AI Town?

AI Town is a collaborative 3D village hosted on GitHub Pages where anyone can open a GitHub issue to add a structure. Buildings are placed on organic plots along winding paths (BotW-inspired), and each has a brass plaque with the contributor's GitHub avatar and username. Every merged contribution gets a shareable URL with an OG image that unfurls on X/Twitter.

**Tagline**: *"An open-source town built entirely by AI, directed by the community."*

## Architecture

- Pure HTML/CSS/JS — no build step, no server
- Three.js 3D renderer with WASD walking controls
- Town data stored in `town.json` — array of building objects
- Each building gets a page at `/town/{building-slug}/` with OG meta tags

## Data Model (`town.json`)

Each building is an object:
```json
{
  "id": "the-cat-bookstore",
  "name": "The Cat Bookstore",
  "type": "shop",
  "description": "A cozy bookstore with a cat sleeping in the window",
  "plot": 5,
  "contributor": {
    "username": "burkeholland",
    "avatar": "https://github.com/burkeholland.png"
  },
  "issue": 42,
  "added": "2026-02-26"
}
```

## When Adding a Building

**Automated validation**: New buildings go through GitHub Actions workflows that check for safety violations and plot conflicts before merge.

1. Add an entry to `town.json` with:
   - `id`: kebab-case slug from the building name
   - `name`: the building's display name
   - `type`: one of `shop`, `house`, `restaurant`, `public`, `entertainment`, `nature`, `other`
   - `description`: brief description of the building
   - `plot`: Use the deterministic plot assignment system (see **Plot Assignment** section below). In build context comments, the dispatch system will suggest a plot number based on the building type and current zone availability. Use that suggestion. If no suggestion is provided, manually choose an available plot from `town.json` (plots 5-40, avoiding 1-4 and duplicates).
   - `contributor`: `{ "username": "...", "avatar": "https://github.com/{username}.png" }`
   - `issue`: the issue number this building was requested in
   - `added`: today's date in YYYY-MM-DD format

2. Buildings can be any shape or size — use the CUSTOM_BUILDERS registry in `js/buildings.js` for unique structures. Register a builder function keyed by the building's `id`.

3. Each plot has a `facing` direction — buildings are automatically rotated to face the nearest road.

## Plot Assignment System

AI Town uses a **deterministic zone-based plot assignment algorithm** to ensure buildings are placed appropriately:

### Zone Rules
- **Town Square (plots 1-4)**: RESERVED — no buildings allowed
- **Main Street West (plots 5-9)**: Preferred for `shop`, `restaurant`
- **Main Street East (plots 10-14)**: Preferred for `entertainment`, `restaurant`, `shop`
- **North Residential (plots 15-19)**: Preferred for `house`, `nature`, `other`
- **South Residential (plots 20-24)**: Preferred for `house`, `nature`, `public`, `other`
- **Village Outskirts (plots 25-40)**: Preferred for `other`, `nature`, `public` — large or unique structures

### Assignment Process
The algorithm in `.github/scripts/plot-assignment.mjs` scores all available plots based on:
1. Zone type match (100 points for preferred types)
2. Plot position within zone (earlier plots preferred)
3. Building size considerations (larger plots for monuments/landmarks)

When the dispatch system creates a "build context" comment, it will include a suggested plot number. **Use that plot number** unless there's a compelling reason to override.

### Current Availability
Check `.github/workflows/suggest-plot.yml` for a workflow that shows current zone availability. As of the last commit:
- Main Street West: **FULL** (5/5 occupied)
- Main Street East: **FULL** (5/5 occupied)
- North Residential: **FULL** (5/5 occupied)
- South Residential: 2/5 occupied, 3 available
- Village Outskirts: 10/16 occupied, 6 available

## When Modifying a Building

Issues labeled `building-modification` ask to change an **existing** building. The issue author can only modify their own building.

1. Find the building in `town.json` where `contributor.username` matches the issue author's GitHub login. That is the only building you may change.
2. Read the issue body to understand what the user wants changed (description, type, custom visuals, etc.).
3. Update the building's entry in `town.json` as requested (e.g., new `description`, `name`, or `type`).
4. If the modification involves custom visuals, update or add a `CUSTOM_BUILDERS` entry in `js/buildings.js` keyed by the building's `id`.
5. Do NOT change the `contributor`, `plot`, `issue`, or `added` fields. Plot assignments are permanent.
6. Do NOT touch any other building's data or code.

## Building Types & Colors

| Type | Roof Color | Wall Color |
|------|-----------|------------|
| shop | #ff7f50 (coral) | #fef3c7 (cream) |
| house | #84cc16 (sage) | #fef3c7 (cream) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [burkeholland/ai-town](https://github.com/burkeholland/ai-town) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
