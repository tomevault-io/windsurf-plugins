---
trigger: always_on
description: **Local working tree:** `/Users/apple/Cursor-Claude/`
---

# Velonyx Systems — Claude Code Handoff

**Local working tree:** `/Users/apple/Cursor-Claude/`
**GitHub:** [`carlitolamar1989/velonyx-portfolio`](https://github.com/carlitolamar1989/velonyx-portfolio) (default branch: `main`)
**Live site:** `https://velonyxsystems.com` / `https://www.velonyxsystems.com`
**Subdomain (RETIRED 2026-06-16):** `gdk.velonyxsystems.com` is being taken down — the 6 in-site demos (`/demos/*`) replace it. The hero "Explore the Live Demo" button now routes to `/demos/tax/` (Benjamin Lewis). Do not re-link the gdk subdomain.
**Founder/Operator:** Carlos Glover ([admin@velonyxsystems.com](mailto:admin@velonyxsystems.com), (877) 317-8643, San Diego CA)

> **Last updated: 2026-06-13.** This file was significantly rewritten after the June audit
> because the previous version was stale (it still described the $3,000 Founding Member model,
> a Stripe checkout funnel, a hero video, and a `for-barbers.html` page — none of which match
> the current site). If something here disagrees with the code, trust the code and fix this file.

---

## ⚠️ Repo hygiene — local clone drifts behind `origin/main`

Recent site work has been landing through GitHub PRs (#22–#39). The local clone is easy to leave
several commits behind, which means editing stale files. **Always `git pull` before starting work**
and verify `git rev-list --left-right --count main...origin/main` shows `0 0`.

---

## What this site is

A marketing site for **Velonyx Systems**. As of **June 16, 2026** the site is **repositioned** around
an aspirational, worldwide message: the hero is *"The Future of Business Runs on AI."* — eyebrow
*"The businesses that embrace AI win,"* subhead *"A custom AI that works while you do."* The buyer is
**any business owner, anywhere in the world** (no industry/geo targeting); the pitch is empowerment, not
fear. Core promise: a custom AI that does the work of a help desk / assistant — answers chat + phone
24/7, captures, qualifies, and books leads, and runs the busywork — so owners scale **without hiring**.

> **⚠️ Retired framing — do NOT reintroduce:** "Never Miss Another Lead" and all fear/loss language
> ("leads slip," "78% first to respond," "losing money"); home-services / trades targeting (HVAC,
> plumbing, electrical, garage doors, "under a sink/truck" + those keywords + JSON-LD `serviceType`);
> San Diego / local / `geo.*` meta (areaServed is now "Worldwide"); and the rent-vs-own angle
> ("yours to own," "own it," "no rent"). The 6 demo cards (Garage Door Kings, etc.) stay — they're
> intentional cross-industry *examples* that prove the AI works for any business.

Underneath the message, the actual product is the same **custom platform**: branded website + AI chatbot
+ AI voice agent + lead automation + booking + payments + customer financing + SMS + an owner dashboard.

The site exists to:
1. Capture leads via the floating lead-form widget and route serious prospects to **book a call**.
2. Showcase the 6 in-site live demos under `/demos/*` (the old `gdk.velonyxsystems.com` subdomain is retired).
3. Provide an "in-bio" landing at `/connect/` for QR-code / DM / SMS sharing.

### ✅ Positioning alignment (resolved 2026-06-16)
The old title/meta/JSON-LD vs. body mismatch is **fixed** — the `<title>`, meta description, keywords,
OG/Twitter, and all JSON-LD now carry the unified "The Future of Business Runs on AI" / worldwide /
AI-system message. Next marketing step on Carlos's list: **SEO + organic content** built around this
new positioning (rank for "AI for business / custom AI system," not local/trades terms).

---

## Funnel & pricing (current)

**Funnel = book a call.** There is **no live self-serve payment** on the site right now.
- Primary CTAs open the floating lead-form widget (`data-vx-form-open`) → POSTs a lead.
- `book.html` is the real booking step — an inline **Calendly** embed (`calendly.com/admin-velonyxsystems/30min`).
- `checkout.html` and `financing.html` are informational; their CTAs route to **`/book.html`** ("book a
  call and we'll send the secure link"). They must **not** present "Pay now / Go to checkout" dead-ends.

**Pricing:** **$700 one-time build + $70/month.** Optional growth add-ons ($250 / $500 / $1,500/mo)
appear in the pricing/JSON-LD. The old **$3,000 Founding Member + $100/mo** model is **retired** — if you
still see it anywhere, it's stale.

**Stripe (pending):** Carlos is creating **new Stripe Payment Links for the $700 / $70 prices**. Until
those exist and are wired in, keep the funnel honest as book-a-call. When the links are ready, wire them
into `checkout.html` (and anywhere a "pay" CTA belongs) — do not reintroduce "pay" language before then.

---

## Lead capture plumbing

- **Floating lead-form widget:** `assets/velonyx-lead-form.js`, opened by any `data-vx-form-open` element.
  POSTs to the conversational endpoint `…/form-turn` with a fallback to the leads endpoint:
  `https://jyo775chsk.execute-api.us-east-1.amazonaws.com/leads`.
- **Lead payload shape** (reused by `sms-opt-in.html`): `{ firstName, lastName, phone, email, service, description }`;
  success when the JSON response has `success: true`.
- **Floating AI demo widget:** `assets/velonyx-ai-demo-widget.js` — a self-contained scripted/canned
  conversation that demos the product. No live backend.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlitolamar1989/velonyx-portfolio](https://github.com/carlitolamar1989/velonyx-portfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
