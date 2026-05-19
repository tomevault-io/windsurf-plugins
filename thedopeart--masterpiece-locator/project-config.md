---
trigger: always_on
description: Art discovery app with artwork pages featuring descriptions and FAQs. Content interlinks to internal pages (artists, museums, movements) and external LuxuryWallArt.com collections.
---

# Masterpiece Locator - Content Guidelines

## Project Overview
Art discovery app with artwork pages featuring descriptions and FAQs. Content interlinks to internal pages (artists, museums, movements) and external LuxuryWallArt.com collections.

---

## Workflow

### Step 1: Check the Checklist
Open `ARTWORK-CHECKLIST.md` to see what needs work. Artworks are prioritized by tier:
- **Tier 1**: High-traffic, famous paintings (do first)
- **Tier 2**: Medium priority
- **Tier 3**: Lower priority

Pick artworks marked `TODO`. When working with multiple agents, claim a batch by noting which slugs you're working on to avoid duplicates.

### Step 2: Research Each Artwork
Before writing, gather real information:
1. Check if Wikipedia URL exists in the checklist
2. Use WebFetch to read the Wikipedia page for facts
3. Look for: creation date, what's depicted, history, controversies, current location, interesting facts
4. Only write what you can verify - never invent details

### Step 3: Write Description & FAQs
Follow the guidelines below. Length depends on available content:
- Limited info → 130-150 words
- Rich history → up to 350 words

### Step 4: Update Database
Run Prisma update script for each batch.

### Step 5: Mark Complete
After updating, regenerate the checklist:
```bash
node get-artworks.js
```

---

## Artwork Descriptions

### Length
- **130-150 words** for paintings with limited context
- **Up to 350 words** only if there's genuinely interesting, true content
- Never pad text just because a painting is famous

### Structure
- Use `<p>` tags for paragraphs (typically 2-3 paragraphs)
- No `<br>` tags between paragraphs (CSS handles spacing)
- Vary sentence length dramatically

### Typical Paragraph Flow
1. **Opening paragraph**: Artist link, date, what the painting depicts visually
2. **Middle paragraph**: Historical context, technique, interesting facts, controversies
3. **Closing paragraph**: Artist bio context, current museum location with link

### What to Include (when true and interesting)
- What the painting actually shows (describe the scene)
- When and where it was created
- Historical context or story behind it
- Interesting facts (scandals, thefts, X-ray discoveries, controversies)
- Technical approach (technique, medium, series context)
- Artist's life context relevant to the work
- Current location and how it got there

### What NOT to Do
- Don't pad with generic art appreciation language
- Don't repeat information already in the metadata (title, year shown on page)
- Don't invent facts - only write what you actually know
- Don't write longer just because a painting is famous

### Bold Keywords
- Minimum **1 bold keyword per paragraph**
- Bold text is black unless inside a link
- Use `<strong>` tags

---

## Interlinking

### Internal Links (display in green #028161)
```html
<a href="/apps/masterpieces/artist/artist-slug"><strong>Artist Name</strong></a>
<a href="/apps/masterpieces/movement/movement-slug"><strong>Movement Name</strong></a>
```

### Museum Links (display in gold #C9A84C)
```html
<a href="/apps/masterpieces/museum/museum-slug"><strong>Museum Name</strong></a>
```

### LuxuryWallArt Links (display in gold #C9A84C)
Link to collections when content naturally matches. Great opportunities:

**Colors** (when describing palette):
- Gold `/collections/gold-art` | Black and Gold `/collections/black-and-gold`
- Blue `/collections/blue-wall-art` | Navy Blue `/collections/navy-blue` | Royal Blue `/collections/royal-blue`
- Red `/collections/red-wall-art` | Red and Black `/collections/red-and-black-art`
- Green `/collections/green-wall-art` | Emerald `/collections/emerald-green`
- Yellow `/collections/yellow-paintings` | Orange `/collections/orange-art`
- Pink `/collections/pink-wall-art` | Purple `/collections/purple-paintings`
- Gray/Grey `/collections/grey-art` | Beige `/collections/beige` | Brown `/collections/brown-art`
- Black and White `/collections/black-and-white-artwork` | Colorful `/collections/colorful-artwork`
- Earth Tones `/collections/earth-tones` | Neutral `/collections/neutral-art`

**Dark/Macabre themes** (skulls, death, horror):
- Skeleton & Skulls `/collections/skeleton-skull-art`
- Macabre `/collections/macabre-art`

**Religious/Spiritual** (angels, religious scenes):
- Angel `/collections/angel-art` | Spiritual `/collections/spiritual-art` | Zen `/collections/zen-art`

**Subjects** (when depicted):
- Women `/collections/women-art` | Portrait `/collections/portrait-art` | People `/collections/people-paintings`
- King and Queen `/collections/kings-and-queens` | Crown `/collections/crown-paintings`
- Floral `/collections/floral-art` | Nature `/collections/nature-art` | Tree `/collections/paintings-of-trees`
- Mountain `/collections/mountain-art` | Sunset `/collections/sunset-paintings` | Landscapes `/collections/landscapes`

**Animals** (when depicted):
- Horse `/collections/horse-art` | Dog `/collections/dog-paintings` | Cat `/collections/cats`
- Lion `/collections/lion-wall-art` | Tiger `/collections/tiger-paintings` | Big Cat `/collections/big-cat-art`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thedopeart/masterpiece-locator](https://github.com/thedopeart/masterpiece-locator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
