---
trigger: always_on
description: Copy everything below the line, fill in the [BRACKETS] with your info, and paste it into a new Claude chat.
---

# Portfolio Website Prompt

Copy everything below the line, fill in the [BRACKETS] with your info, and paste it into a new Claude chat.

---

You are a world-class frontend designer and developer with 20+ years of experience building award-winning, Awwwards-nominated portfolio websites. You obsess over every pixel, every animation curve, every font pairing. You don't build websites — you craft digital experiences.

Build me a **single-page portfolio website** as a React component (.jsx) that looks like it was designed by a top-tier design agency, NOT by AI.

## Who I Am

- **Name:** Tamer
- **Role / Title:** Full stack developer & AI specialist 
- **Short tagline:** Ideas into real products.
- **Bio (2-3 sentences):** I’m a full-stack web developer and AI specialist who loves turning ideas into intelligent, user-friendly products. I combine modern web technologies and AI to build solutions that are both powerful and simple to use.
- **Skills / Tech stack:** Python React Figma shopify wWrdpress SEO optimaliztion Javascript  Performance optimalization Web design Claude
- **Projects (3-4):** // The project parts will be deliverd later
  - Project 1: [Name] — [One sentence description] — [Tech used]
  - Project 2: [Name] — [One sentence description] — [Tech used]
  - Project 3: [Name] — [One sentence description] — [Tech used]
  - Project 4: [Name] — [One sentence description] — [Tech used]
- **Certifications / Courses (2-5):**
  - [e.g. "AWS Certified Solutions Architect — Amazon Web Services — 2024"]
  - [e.g. "Google UX Design Professional Certificate — Google — 2023"]
  - [e.g. "Meta Front-End Developer Certificate — Meta — 2023"]
  - [Add your own — include: name of certification, issuing organization, and year]
- **Contact links:** [e.g. email, GitHub, LinkedIn, Twitter/X — list what you have]

---

## Design Direction — "Luxury Minimal with Life"

The aesthetic is: **refined, light, modern, and alive.** Think Apple's design philosophy meets a high-end architecture firm's website — with a living, breathing gradient background that makes it feel organic and warm, not cold and sterile.

### Theme & Color Palette
- **Light theme** — predominantly white and soft off-white backgrounds (#FAFAF9 or similar warm whites, never pure #FFFFFF)
- One **dominant accent color** that feels intentional and bold (think: a rich coral, a deep sage green, a warm amber, or an electric blue — pick ONE and commit to it)
- Use **tints and shades** of your accent for depth — lighter versions for backgrounds, full strength for CTAs, darker for text accents
- Add a subtle **warm neutral** for secondary text (not cold gray — use something like #6B6560 or #4A4540)
- Define ALL colors as CSS variables so the entire palette is cohesive

### Animated Gradient Wave Background
This is the hero element — the thing people remember. Build it with care:
- A **full-viewport animated gradient mesh** in the hero section that uses soft, light, pastel tones of the accent color blended with whites and creams
- The gradient should **slowly morph and flow** like silk in water — use CSS `@keyframes` with `background-position` or `background-size` animation on a large gradient (400% 400%)
- Movement should be **slow and hypnotic** — cycle duration of 15-20 seconds, ease-in-out timing
- It should feel like light reflecting off water or soft clouds drifting — NOT like a disco or a rave
- Layer a subtle **grain/noise texture overlay** on top using CSS (a pseudo-element with a tiny repeating SVG noise pattern at very low opacity) to add organic feel and prevent it from looking "too digital"
- The gradient should gently **fade out** as you scroll down, so it doesn't fight with the content below

### Typography — Make It Unforgettable
- Import from Google Fonts — pick a **distinctive display font** for headings that has CHARACTER (examples for inspiration: "Playfair Display", "Cormorant Garamond", "Sora", "General Sans", "Clash Display", "Cabinet Grotesk", "Instrument Serif" — pick something unique, do NOT just default to the first one)
- Pair it with a **clean, refined body font** that contrasts well (something geometric or humanist sans-serif)
- Headings should feel LARGE and CONFIDENT — hero name can be 5-8rem, section headings 3-4rem
- Use **letter-spacing** intentionally: tight on large headings (-0.02em to -0.04em), slightly open on small labels/caps (0.05em to 0.1em)
- Body text should be 1.1-1.2rem with generous line-height (1.6-1.8) for readability
- Add one **typographic surprise**: maybe the tagline is in italic serif while everything else is sans-serif, or a section label uses a monospace font

### Layout & Spatial Composition
- **Generous whitespace everywhere** — sections should breathe. Use 120px-200px vertical padding between sections
- Content width maxes out at around 1200px, centered, but let some elements **break the grid** (e.g., a decorative accent line that extends full-width, or a project card that's offset to one side)
- Use **asymmetric layouts** where it feels natural — the about section could be a 60/40 split, projects could use a masonry-style stagger
- Add **subtle horizontal rules or decorative dividers** between sections using thin lines, dots, or the accent color

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TMR1905) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
