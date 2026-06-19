---
trigger: always_on
description: |
---


# 🚀 SEO/GEO Skill (v2 — brand-aware)

A reproducible methodology for ranking on Google **and** being cited by ChatGPT, Claude, Gemini, and Perplexity. **Adapts to whatever brand you're working on** — never assumes.

---

## 🎯 What's new in v2

v1 implicitly assumed you were working on a B2B SaaS called Refine. v2 throws that assumption out:

- **Auto-detects** the brand from the current project (domain, package.json, CLAUDE.md, README, git remote, website crawl)
- **Asks once** for what it can't infer (ICP, competitors, voice constraints) and saves it to `seo-geo/brand-profile.md` for the rest of the engagement
- **Every output** uses the user's actual brand name, domain, voice, ICP, competitors — nothing hardcoded
- **Adapts the playbook** to the brand's archetype (B2B SaaS, agency, e-commerce, dev tool, consumer, local business, creator) — different tactics, different sources, different prompt universes
- **Default output directory** is `seo-geo/` in the current project (or wherever the user requests) — so the deliverables ship with their codebase, not somewhere generic

---

## 🧭 When to use

- A brand wants organic acquisition through Google + AI search
- A team needs a content strategy
- A founder asks "how do I get cited in ChatGPT?"
- A marketing manager wants to systematize GEO
- A user has a blog and wants to make it perform
- Anyone with a website who wants to be visible in AI answers

**Skip this skill if:** the user only wants a single article with no strategy context, a one-shot keyword check, or technical SEO debugging (use direct tools instead).

---

## ⚠️ Hard rules (read before running)

1. **Never hardcode a brand name.** Always pull from the brand profile.
2. **Never assume the brand category.** Ask if not obvious from the codebase.
3. **Never copy-paste examples from this file** into deliverables. Examples here are illustrative only.
4. **Never recommend tactics that don't fit the brand archetype.** A solo creator doesn't need a G2 listing. A regulated medical brand can't farm Reddit.
5. **Always save discoveries** to `seo-geo/brand-profile.md` so the user (and future skill runs) can reuse them.
6. **Always cite the user's actual competitors**, not placeholder competitors from this skill.
7. **Always use the user's actual domain** in the article slug, internal-link map, and tracking universe.

---

## 🔍 Phase 0 — Brand context discovery (NEW in v2)

Before any SEO/GEO work, build a `brand profile`. This is the single source of truth for the rest of the engagement.

### Auto-detection (run silently first, before asking the user)

Read these in order. Each fills part of the profile:

```
1. cwd / git remote → brand name candidate, repo description
2. package.json → name, description, homepage, author
3. CLAUDE.md / README.md → brand description, voice, stack
4. .env / config / next.config → production domain, locale
5. public/ favicon, og-image → visual brand cues
6. Existing /blog or /content directories → editorial voice samples
7. sitemap.xml or /robots.txt at the detected domain → page inventory
8. WebFetch on detected domain → tagline, hero copy, CTA, footer
```

### Then ask only for what's missing

Use `AskUserQuestion` for the gaps. Keep it tight — never ask more than 6 questions in one batch:

```
Q1: What's the one-line description of your product/service?
    (auto-fill from package.json description if present)

Q2: Who is your ICP? (role + company stage + pain)
    (auto-fill from CLAUDE.md or README if mentioned)

Q3: Who are your 3-5 direct competitors? (names + websites)

Q4: What's your primary geo focus? (global / region / country / local)

Q5: Which AI/LLM platforms do you care about most?
    (default: ChatGPT, Claude, Gemini, Perplexity)

Q6: Any constraints on voice, claims, or sources?
    (e.g. regulated industry, no superlatives, no comparisons)
```

### Save to `seo-geo/brand-profile.md`

```markdown
# Brand Profile — [DETECTED_BRAND_NAME]

## Identity
- **Brand name**: [from detection or user]
- **Domain**: [detected]
- **One-liner**: [from package.json or user]
- **Category**: [inferred or user]
- **Archetype**: [B2B SaaS | Agency | E-commerce | Dev tool | Consumer | Local | Creator | Other]

## ICP
- **Role**: [user input]
- **Company stage**: [user input]
- **Pain**: [user input]

## Geography
- **Primary**: [user input]
- **Locales**: [detected from i18n config or user]

## Competitors
- Direct: [user input]
- Adjacent: [user input or detected from competitor analysis]

## Voice & Constraints
- Tone: [detected from existing content or user]
- Hard constraints: [user input — regulated, accuracy-bound, etc.]

## Tech context
- Stack: [from package.json]
- Existing blog path: [detected]
- Sitemap: [detected]
- GSC available: [yes/no — ask if unsure]

## AI focus
- Platforms: [user choice — default: all 4]

---
_Last updated: [DATE]. Update this file as you learn more about the brand._
```

**This file is now the source of truth for every other phase.** Every prompt, every article, every CTA must read from here.

---

## 🔑 Core principles (always apply, regardless of brand)

1. **One intent per article** — never two articles cannibalizing on the same long-tail
2. **Long-tail > head terms** — 4-7 word queries convert better and are less competitive

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rob1-alt/refine-seo-geo](https://github.com/rob1-alt/refine-seo-geo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
