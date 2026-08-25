---
trigger: always_on
description: You are the senior frontend engineer and interaction designer responsible for shipping this demo. Do not redesign the product from scratch. The creative direction, content, asset names, event data, and mobile experience below are approved. Preserve them unless the user explicitly asks for a change.
---

# Elevare Wedding Invite Demo — Claude Code Build Brief

## Role
You are the senior frontend engineer and interaction designer responsible for shipping this demo. Do not redesign the product from scratch. The creative direction, content, asset names, event data, and mobile experience below are approved. Preserve them unless the user explicitly asks for a change.

## Product goal
Build a premium, mobile-first digital Indian wedding invitation for **Shubham & Riddhi** that Elevare can later reuse as a paid client template. It must feel like entering a royal cinematic invitation, not reading a generic wedding landing page.

The commercial product target is a reusable **₹8k–₹25k wedding invite website + video service**. Therefore, client-specific content must live in one configuration file and must not be scattered across components.

## Experience target
The guest should feel:
1. Warmly welcomed.
2. Surprised by the level of presentation.
3. Guided through the couple's story and events without friction.
4. Able to get directions or RSVP in one tap.

Visual tone: **Indian miniature-inspired romantic illustration + contemporary cinematic luxury**.

Avoid: Canva-template styling, loud gradients, neon, heavy green palettes, floating hearts, excessive particles, spinning mandalas, confetti loops, cursor effects, autoplay hacks, or animation on every element.

## Technical architecture
- Next.js 16 App Router.
- TypeScript, strict mode.
- Keep dependencies minimal.
- Use `next/image` for all major visuals.
- Use CSS + IntersectionObserver for motion unless a requirement genuinely needs an animation library.
- No backend.
- No database.
- WhatsApp RSVP only.
- Client-specific data lives in `src/config/invitation.ts`.
- The site must deploy cleanly to Vercel.

## Folder contract
Do not rename these without a strong technical reason:

```text
src/
  app/
    globals.css
    layout.tsx
    page.tsx
    opengraph-image.jpg
    twitter-image.jpg
  components/
    InviteExperience.tsx
    Reveal.tsx
  config/
    invitation.ts

public/
  images/
    hero.jpg
    story-first-meeting.jpg
    celebration.jpg
    wedding-day.jpg
    closing.jpg
    og-whatsapp.jpg
  audio/
    wedding-theme.mp3   # only after a licensed track is supplied
  shubham-riddhi-wedding.ics
```

## Locked visual palette
Do not introduce green as a primary UI colour.

- Ivory: `#F7F0E4`
- Parchment: `#E7D3B7`
- Antique Gold: `#B78A45`
- Dusty Rose: `#C97782`
- Deep Rose: `#9A4A58`
- Wine: `#5A1E2B`
- Maroon: `#762E3A`
- Ink: `#2F2422`

Natural foliage that already exists inside the illustrations is acceptable; the interface itself should remain ivory / gold / rose / wine / maroon-led.

## Fonts
- Display / names / emotional headlines: `Cormorant Garamond`.
- Utility copy / buttons / event information: `Manrope`.
- Marathi / Devanagari accents: `Noto Serif Devanagari`.
- Use `next/font/google` and `display: swap`.
- Never use multiple script fonts.

## Approved section order and exact copy

### 1. Entry Gate
Full viewport. Ivory paper texture. Minimal royal arch treatment.

Copy:
- `॥ श्री गणेशाय नमः ॥`
- `Together with our families`
- `Shubham & Riddhi`
- `22 · 11 · 2025`
- CTA: `Open the Invitation`
- Microcopy: `Tap to enter the celebration`

Behavior:
- The invitation content is hidden until the CTA is tapped.
- This tap is the only acceptable point to start audio when audio is enabled.
- Do not try to bypass browser autoplay restrictions.

### 2. Opening / Hero Couple
Asset: `/images/hero.jpg`

Copy:
- `शुभ विवाह`
- `Shubham`
- `&`
- `Riddhi`
- `Together with our families, we invite you to celebrate our beginning.`
- `22 · 11 · 2025`
- Microcopy: `Scroll to enter our story`

Motion:
- Controlled 3–5% slow image push after entry.
- Text reveal only; no bouncing or floating elements.

### 3. Our Beginning
Asset: `/images/story-first-meeting.jpg`

Eyebrow: `Our Beginning`
Headline: `It began with a first meeting.`

Body copy, use exactly:
`Our story began in Nashik, when Riddhi and her family came to Shubham's home with a wedding proposal. What started as a traditional introduction became an easy conversation, then comfort, then certainty. Two people met. Two families connected. And a new chapter quietly began.`

Marathi line:
`एका भेटीतून सुरू झालेली आपली गोष्ट…`

Tone: arranged-marriage story told with confidence and warmth. Do not rewrite it into a dating / college-love story.

### 4. Celebration Visual
Asset: `/images/celebration.jpg`

Overlay:
- `21 November 2025`
- `The Celebrations Begin`

Use this as a cinematic visual break before the event cards.

### 5. Celebration Events — 21 November 2025
Header:
- `शुक्रवार`
- `21 November 2025`
- `A day of colour, blessings & togetherness.`

Events:
1. `Haldi & Grahashanti`
   - `10:00 AM`
   - `Hira Executive, Nandurbar`

2. `Swaruchi Bhojan`
   - `12:00 PM onwards`
   - `Hira Executive, Nandurbar`

3. `Raas Garba`
   - `7:00 PM`
   - `Hira Executive, Nandurbar`

Hira directions URL:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sonarshubhamn-ux/WeddingInvitation](https://github.com/sonarshubhamn-ux/WeddingInvitation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
