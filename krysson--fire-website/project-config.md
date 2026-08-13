---
trigger: always_on
description: This is the FIRE convention website - a static Next.js site for a rope bondage educational organization hosting three annual events in Orlando, FL.
---

# CLAUDE.md - Instructions for Claude Code

## Project Context

This is the FIRE convention website - a static Next.js site for a rope bondage educational organization hosting three annual events in Orlando, FL.

**Key constraint:** Keep it simple. This is a static marketing/information site with no auth, no database, no payments.

---

## Tech Stack

```
Framework:    Next.js 14+ (App Router)
Styling:      Tailwind CSS
Components:   shadcn/ui
Language:     TypeScript
Content:      Markdown + JSON files
Hosting:      Vercel
```

---

## Project Structure

```
fire-website/
├── app/
│   ├── layout.tsx              # Root layout with nav/footer
│   ├── page.tsx                # Homepage
│   ├── about/page.tsx          # About FIRE org
│   ├── faq/page.tsx            # FAQ & Policies
│   ├── contact/page.tsx        # Contact info
│   ├── sponsors/page.tsx       # Sponsors & vendors
│   ├── past-events/page.tsx    # Archive (placeholder)
│   └── events/
│       ├── blaze-2026/
│       │   ├── page.tsx        # BLAZE landing
│       │   ├── schedule/page.tsx
│       │   ├── presenters/page.tsx
│       │   ├── presenters/[slug]/page.tsx
│       │   ├── classes/page.tsx
│       │   └── venue/page.tsx
│       ├── flare-2026/
│       │   └── ...
│       └── fire-2027/
│           └── ...
├── components/
│   ├── ui/                     # shadcn components
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Navigation.tsx
│   ├── home/
│   │   ├── Hero.tsx
│   │   └── EventCards.tsx
│   └── events/
│       ├── ScheduleGrid.tsx
│       ├── PresenterCard.tsx
│       ├── ClassCard.tsx
│       └── VenueMap.tsx
├── content/
│   ├── blaze-2026/
│   │   ├── event.json
│   │   ├── schedule.json
│   │   ├── sponsors.json
│   │   ├── presenters/
│   │   └── classes/
│   ├── flare-2026/
│   ├── fire-2027/
│   └── organization/
│       ├── config.json         # SINGLE SOURCE OF TRUTH: contact emails, social links,
│       │                       # homepage event cards, class level colors
│       ├── about.md
│       ├── faq.md
│       └── policies.md
├── lib/
│   ├── content.ts              # Content loading utilities
│   └── utils.ts                # Helper functions
├── public/
│   ├── logos/
│   │   ├── blaze-2025.png      # Update year to 2026 when received
│   │   ├── fire-logo.png
│   │   └── flower-flat.png
│   └── images/
│       └── presenters/
└── styles/
    └── globals.css
```

---

## Coding Standards

### General
- Use TypeScript strict mode
- Prefer server components; use 'use client' only when needed
- Keep components small and focused
- Use descriptive variable names

### Styling
- Tailwind CSS for all styling
- Use CSS variables for brand colors (defined in globals.css)
- Mobile-first responsive design
- Consistent spacing using Tailwind scale

### Content
- Load markdown with gray-matter for frontmatter
- Parse markdown with remark/rehype
- Type all JSON content with interfaces

---

## Brand Colors (Tailwind Config)

```javascript
// tailwind.config.ts
colors: {
  fire: {
    black: '#0a0a0a',
    charcoal: '#1a1a1a',
    dark: '#2a2a2a',
    red: '#e63946',
    orange: '#f4a261',
    yellow: '#f9c74f',
  }
}
```

---

## Content File Formats

### config.json (Single Source of Truth)

`content/organization/config.json` controls site-wide settings. Edit this file — not component code — for:

- **Contact emails** (`contact.general`, `contact.presenters`, `contact.volunteers`, `contact.vendors`)
- **Social media links** (`social.fetlife`, `social.instagram`, `social.facebook`, `social.tiktok`)
- **Homepage event cards** (`homepage.events[]`) — dates, focus level, badge color, ticket slug, logo
- **Featured event** (`homepage.featuredEventId`) — which event shows the "Next Event" banner
- **Class level badge colors** (`classLevels[]`)

Never hard-code these values in component files. Always read from `getSiteConfig()`.

---

### event.json
```json
{
  "name": "BLAZE",
  "year": 2026,
  "tagline": "Ignite Your Rope Journey",
  "description": "A short paragraph about the event.\n\nUse \\n\\n to separate paragraphs. HTML tags like <strong> and <em> are supported.",
  "dates": {
    "display": "April 17-19, 2026",
    "start": "2026-04-17",
    "end": "2026-04-19"
  },
  "focus": "Beginner to Intermediate",
  "venue": {
    "name": "The Woodshed",
    "address": "6431 Milner Blvd Suite #4, Orlando, FL 32809"
  },
  "tickets": {
    "url": "https://forbiddentickets.com/events/...",
    "onSaleDate": "February 14, 2026"
  }
}
```

> **`description` formatting:** Use `\n\n` (double backslash-n) between paragraphs to create line breaks. HTML tags such as `<strong>`, `<em>`, and `<br>` are rendered. The `dates.start` ISO value drives the homepage countdown timer.

### schedule.json
```json
{
  "days": [
    {
      "date": "2026-04-17",
      "label": "Friday",
      "slots": [
        {
          "time": "7:00 PM",
          "title": "Registration Opens",
          "type": "general"
        },
        {
          "time": "8:00 PM",
          "title": "Intro to Floor Work",
          "presenter": "desmond-ellise",
          "room": "Main Room",
          "type": "class"
        }
      ]
    }
  ]
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Krysson/fire-website](https://github.com/Krysson/fire-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
