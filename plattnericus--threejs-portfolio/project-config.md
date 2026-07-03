---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **Status: greenfield / planning.** The repo currently contains only `README.md` and
> `.gitattributes`. Nothing below has been built yet — this document is the agreed product and
> technical plan. Keep it updated as the project is scaffolded and decisions firm up.

## The Idea

A living, low-poly 3D "GitHub Star Tree." A single tracked repository's GitHub stars drive a tree
that **grows as it gains stars**. **Every stargazer becomes a house** placed on/around the tree, and
each house has a **rarity tier** derived from that stargazer's standing. The more stars the repo has,
the bigger the tree and the denser the little village of houses around it.

The site is a personal portfolio piece: a fun, shareable, real-time visualization of community
support for one of the user's projects.

## Locked-In Decisions

These were chosen explicitly. Do not silently change them.

| Area            | Decision                                                                 |
| --------------- | ------------------------------------------------------------------------ |
| Framework       | **Next.js** (App Router)                                                  |
| 3D layer        | **react-three-fiber** + **@react-three/drei** (declarative, state-driven) |
| Data source     | **One repo, live via API** — track a single chosen repo's stargazers, fetched through a Next.js route with caching/ISR |
| Models          | **Pre-made GLTF/GLB low-poly assets** (tree + per-rarity houses), loaded via drei `useGLTF` |
| Rarity driver   | **Stargazer "clout" + contributor bonus** (see Rarity System below)      |
| Interaction     | Click house → stargazer profile panel; hover tooltips; free orbit/zoom/pan camera; rarity legend + stats HUD |
| Deployment      | **Vercel** (ISR + edge caching for the API route, env-based token)        |
| 2D UI / styling | **Tailwind CSS** for all HUD/panels/overlays                              |

## Rarity System (design)

Each house's rarity is a **score** computed from the stargazer, then bucketed into tiers. The score
blends the stargazer's own "clout" with a bonus if they also *contributed* to the tracked repo.

Proposed score formula (tune the weights during build):

```
clout   = w_followers * log10(followers + 1)
        + w_age       * accountAgeYears
        + w_repos     * log10(publicRepos + 1)

contributorBonus = isContributor ? (B_base + B_perCommit * log10(commits + 1)) : 0
                   + isForker     ? B_fork  : 0
                   + openedIssues  ? B_issue : 0

rarityScore = clout + contributorBonus
```

Bucket `rarityScore` into tiers (drop-rate / threshold table to be finalized):

| Tier      | Meaning                                  | House asset      |
| --------- | ---------------------------------------- | ---------------- |
| Common    | Low clout, no contribution               | `house_common`   |
| Uncommon  | Some clout                               | `house_uncommon` |
| Rare      | Notable clout **or** contributor         | `house_rare`     |
| Epic      | High clout **and** contributor           | `house_epic`     |
| Legendary | Top-tier clout + significant contributor | `house_legendary`|

**Open implementation questions to resolve when building:**
- Contributor data requires extra GitHub calls (`/contributors`, per-user commit counts). Decide
  whether to compute the bonus live or precompute it on a schedule and cache.
- Final weights/thresholds (`w_*`, `B_*`) should be calibrated against the real stargazer set so the
  tier distribution feels good (legendaries should stay rare).
- Tiers should be **deterministic** per user (same stargazer always gets the same house) — seed any
  randomness (e.g. house position/variant) from the user id.

## Location & Live Weather — Sterzing (Vipiteno), South Tyrol, Italy

> **IMPLEMENTED (first pass).** `/api/weather` fetches live Open-Meteo data for Sterzing;
> `lib/weather.ts` maps it (exaggerated) to `SceneParams` (sun pos/intensity/color, sky+fog color,
> ambient, wind, precip). `components/Weather.tsx` renders rain/snow particles; `SettingsMenu.tsx`
> toggles live↔manual with a time-of-day slider + conditions (clear/clouds/fog/rain/snow/storm).
> Wind drives leaf sway + whole-tree sway + house bob. Day/night from local hour. `components/
> SceneRig.tsx` eases background/fog/lights so weather & time changes fade smoothly. Seasonal foliage
> tint (`lib/weather.ts` LEAF_COLOR by calendar season) + snow accumulation on the island's up-facing
> surfaces (`uSnow` in the grass shader) + frosted leaves. Remaining polish: snow on house roofs,
> autumn falling-leaf particles, real organic growth (needs morph-target tree .glb).


The scene should mirror the **real-time, real-world weather** of **Sterzing / Vipiteno, South Tyrol,
Italy** (≈ 46.897° N, 11.430° E) — but **slightly exaggerated / "overstimulated"** so it reads
clearly on screen. If it's sunny+windy there, the site is bright with strong leaf sway; if it's
snowing, snow falls harder than reality; overcast → moody grey, etc.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Plattnericus/ThreeJS_Portfolio](https://github.com/Plattnericus/ThreeJS_Portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
