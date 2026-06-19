---
trigger: always_on
description: |
---


# Shippage

A $50K landing page team in your terminal. One sentence in, production-ready page out.

This skill encodes the expertise of a CRO strategist ($10K/mo), conversion copywriter
($3K-5K/page), SaaS designer ($5K-15K/page), and full-stack developer — as data.
No AI slop. No templates. No subscriptions.

**Data moat**: 1,000+ lines of conversion copy rules from A/B tests · 200 real design
token sets from top SaaS sites · 7-step conversion psychology sequence · Social proof
cascade system · Risk reversal on every CTA · Pre-launch intelligence for zero-proof pages

**Stack**: React + Tailwind CSS + Framer Motion + shadcn/ui + Magic UI (default).
Adaptable to any framework — see `references/framework-adapters.md`.

---

## The Conversion Sequence

Every page follows a 7-step narrative arc. Each section has exactly one job:

1. **STOP THE SCROLL** (Hero) — 5 seconds to keep the visitor
2. **BUILD TRUST** (Social Proof) — logos and metrics establish credibility
3. **AGITATE THE PAIN** (Problem) — visitor feels understood
4. **SHOW THE SOLUTION** (Features) — how you solve their problem
5. **PROVE IT WORKS** (Testimonials) — real customers validate the claim
6. **REMOVE DOUBT** (Pricing + FAQ) — objections answered
7. **MAKE THE ASK** (Final CTA) — visitor converts

Do not reorder. Do not skip. Do not combine jobs.

---

## Step 1: Intake

### Quick Mode
User provides product name + one-line description. Skill infers the rest using these rules:

**Awareness level** — infer from product description:
- Mentions a known problem category (slow deploys, manual work, data chaos) → `problem-aware`
- Mentions a solution category (CI/CD, CRM, analytics) → `solution-aware`
- Mentions specific product capabilities or pricing → `product-aware`
- Default if unclear: `solution-aware`

**CTA** — infer from product stage signals:
- "coming soon", "beta", "waitlist", "early access" → `waitlist`
- "open source", "free tier", "try it" → `free-trial`
- Mentions pricing, plans, or enterprise → `purchase`
- B2B + complex product → `demo`
- Default: `free-trial`

**Vibe** — infer from product domain:
- Developer tools, CLI, API, infrastructure → `dark-premium`
- AI, ML, data science → `dark-premium`
- Marketing, sales, CRM, email → `bold-modern`
- Design, creative, content → `playful-creative`
- Finance, security, compliance, healthcare → `enterprise-trust`
- Default: `minimal-clean`

**Social proof** — default to `none` in Quick Mode (assume pre-launch, use Step 4c)

**Audience** — extract from one-liner keywords:
- "teams", "developers", "engineers" → dev teams at startups
- "marketers", "founders", "businesses" → business users / non-technical
- Default: "teams building [product category]"

### Guided Mode (8 questions)
1. **Product name + one-liner**
2. **Target audience** (job title, company size, industry)
3. **Awareness level**: problem-aware / solution-aware / product-aware
4. **Primary CTA**: free trial / demo / waitlist / purchase
5. **Available social proof**: logos / testimonials / metrics / none yet
6. **Vibe**: minimal-clean / bold-modern / dark-premium / playful-creative / enterprise-trust / "look like X.com"
7. **Brand assets** (optional): brand colors (hex), font name, logo file — or "none, pick for me"
8. **Voice calibration**: customer's own words describing their pain

### Minimum Input (3 fields)
If the user provides very little: require product name, one-liner, and primary audience.
Defaults: solution-aware, free-trial CTA, minimal-clean vibe, professional-casual voice.

---

## Step 2: Design System

### Four-Tier System

| Tier | Source | When |
|------|--------|------|
| **0: Custom** | User provides brand colors, fonts, or full token set | User says "my brand is #1E40AF / Poppins" |
| 1: Curated | `scripts/select-design-system.py` queries `design-tokens-db.json` — 200 systems across 8 industries | Default. Match by industry + vibe. |
| 2: Database | Same script with `--top 8` for broader search | When Tier 1 has no close match. |
| 3: Live | `scripts/extract-tokens.sh --site [url]` | When user says "make it look like X.com" |

```bash
python3 scripts/select-design-system.py --industry "Developer Tools" --vibe dark
python3 scripts/select-design-system.py --list-industries
```

### Tier 0: Custom Brand Tokens

When the user provides their own brand colors, fonts, or design values, use them directly.
Build the `globals.css` `:root` block by mapping user values to the required CSS custom properties:

```css
:root {
  --background: [user bg, or derive: white for light, #0a0a0a for dark vibe];
  --foreground: [user text, or derive: contrast against background];
  --primary: [user brand color — the main accent];
  --primary-foreground: [derive: white or black, whichever contrasts 4.5:1];
  --muted: [derive: desaturated, lighter shade of background];
  --muted-foreground: [derive: mid-contrast text for secondary content];
  --accent: [user secondary color, or derive: lighter shade of primary];
  --accent-foreground: [derive: contrast text against accent];
  --border: [derive: subtle gray matching the palette];
  --card: [derive: slight offset from background];
  --card-foreground: [same as foreground];
  --destructive: [default: #ef4444 unless user specifies];
  --destructive-foreground: [derive: contrast text];
  --ring: [same as primary];

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjahanzaib/shippage](https://github.com/imjahanzaib/shippage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
