---
trigger: always_on
description: Guide map design from scratch using established cartographic principles (color, projection, type, symbols, visual hierarchy, layout). Use this whenever the user is making or planning a map or map figure and needs design decisions — choosing a color scheme, picking a projection, deciding class breaks, setting up labels, or laying out a figure for a paper, poster, slide, or web. Trigger it even when the user just says "make a map of X," "what colors for this map," "how should I map this data," or 
---


# Designing Better Maps

Guide the user through map design as a sequence of dependent decisions. Each decision constrains the next: you cannot choose a color scheme until you know the data's measurement level, and you cannot choose a projection until you know the map's purpose and extent. Work the chain in order. Resist the urge to jump straight to "here's a nice color palette" — a good palette on the wrong scheme type is still a bad map.

The goal is a concrete **design spec** the user can hand to their plotting code (GEE, matplotlib, R, QGIS, ArcGIS), not finished pixels. End every session with that spec.

## Critique mode — if you have an existing map

When the user shares an existing map rather than starting fresh, run a rapid audit before proposing changes. Work the same decision chain, but as a checklist:

1. **Map type** — does the type match the data's measurement level? Flag: counts on a choropleth, qualitative scheme on ordered data, symbols not scaled to area.
2. **Color** — does the scheme type match the data? Flag: rainbow/jet for sequence, ternary RGB when a simpler encoding would serve, wrong diverging midpoint, red–green pair on a colorblind-sensitive figure.
3. **Symbols** — are sizes area-scaled? Is the range wide enough to discriminate? Is overplotting masking the data?
4. **Projection** — is it equal-area for any comparison of quantities across areas? Appropriate compromise for the extent?
5. **Visual hierarchy** — is the thematic content dominant? Is the basemap quiet enough? Does the most important layer have the highest contrast?
6. **Type** — is there a legible size hierarchy? Are all labels above the minimum legibility floor for the medium?
7. **Layout** — is every marginalia element earning its place? Is the legend readable and correctly ordered?

Name each problem with a one-sentence diagnosis and a one-sentence fix. Don't recount what the map already does right unless it's non-obvious. Prioritize: color scheme errors first (they mislead the data), then symbol errors, then hierarchy and type, then layout polish.

## Step 0 — Get the brief before recommending anything

A map's design follows from its job. Elicit these before proposing decisions; if the user already stated some, don't re-ask. Keep it to the few that actually change the design:

- **Purpose / message** — What is the one thing a reader should take away? A map that tries to say everything says nothing. Push for a single dominant message; everything else is supporting context.
- **Audience & medium** — Print (paper figure, poster) vs screen (slide, web, interactive)? Print needs higher contrast and can't rely on zoom; screen tolerates more layers. This sets resolution, type size floors, and detail budget.
- **Data** — What variable(s), and what is each variable's **measurement level**? This is the single most consequential fact for color and symbol choice. Sort each into: nominal/categorical, ordinal, or numeric (interval/ratio), and note whether numeric data has a **meaningful midpoint** (e.g. anomaly, change, divergence from a reference).
- **Geographic extent & location** — Continental, national, city, neighborhood? Governs projection and how much generalization the basemap needs.

If the user is vague on purpose, that is the problem to fix first — most weak maps are weak because they have no decided message, not because of palette.

## The decision chain

Work top to bottom. For the bulky lookups, read the referenced file only when you reach that decision.

### 1. Map type (follows from data measurement level)

Match the thematic map type to what the data *is*. Common pairings:

- **Nominal / categorical area data** → categorical fill (one hue per class). Not a sequence.
- **Numeric data tied to areas, normalized** → **choropleth**. Critical rule: choropleth requires data normalized to area or population (rates, densities, per-capita), never raw counts — raw counts on a choropleth just re-draw the population/area map. If the user has raw counts, either normalize or switch to proportional symbols.
- **Numeric data, raw magnitude / counts** → **proportional (graduated) symbols** scaled by value, or dot density.
- **Continuous surfaces (raster, interpolation)** → isarithmic / continuous fill (this is the usual case for remote-sensing and climate fields — LST, NDVI, precipitation).
- **Flows / movement** → flow lines weighted by magnitude.

State the chosen type and *why the data forced it*. If the user asked for a choropleth of counts, flag the normalization trap explicitly.

### 2. Classification (only for class-based maps)

If using classed data (choropleth, classed symbols), decide the class scheme and count:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kangning-huang/designing-better-maps-skills](https://github.com/kangning-huang/designing-better-maps-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
