---
trigger: always_on
description: **Before ANY cross-site work (marketing a new product on multiple sites, adding a CTA, running an audit, generating content), open `~/social-autoposter/config.json` first.** It is the authoritative list of every website we run. Do not `ls ~/` looking for site repos, do not guess domains, do not hardcode a "list of our sites" anywhere.
---

# Social Autoposter

## Source of truth for active projects: config.json

**Before ANY cross-site work (marketing a new product on multiple sites, adding a CTA, running an audit, generating content), open `~/social-autoposter/config.json` first.** It is the authoritative list of every website we run. Do not `ls ~/` looking for site repos, do not guess domains, do not hardcode a "list of our sites" anywhere.

Each entry under `projects[]` exposes (at minimum):
- `name` (e.g. `fazm`, `mediar`, `assrt`)
- `website` production domain
- `local_repo` path to the product repo (e.g. `~/fazm`)
- `landing_pages.repo` path to the website repo (e.g. `~/fazm-website`) <- use this for marketing pages, blog posts, CTAs
- `landing_pages.github_repo`, `landing_pages.base_url`, `landing_pages.gsc_property`
- `posthog.project_id`, `booking_link`, `get_started_link`, `qualification`

Rules:
- New website? Add it to `config.json` first; SEO pipeline, analytics checker, dashboard, and cross-site marketing scripts pick it up automatically.
- Never hardcode project names, repo paths, or domains outside `config.json`.
- Any script that iterates over "all our websites" MUST read `projects[]`.

## Shared UI library: @m13v/seo-components (~/seo-components)

**`~/seo-components` is where cross-site UI lives.** Published as `@m13v/seo-components`, consumed by every website in `config.json`. Before building a new component on one site (CTA block, newsletter signup, comparison table, FAQ, proof band), check if it already exists in the library, and if not, **add it to the library instead of building it site-local**. One site-local copy today becomes four divergent copies next quarter.

Already shipped (partial): `InlineCta`, `StickyBottomCta`, `BookCallCTA`, `GetStartedCTA`, `NewsletterSignup`, `FullSiteAnalytics`, `ComparisonTable`, `FaqSection`, `RelatedPostsGrid`, `ProofBand`, `GlowCard`, `ShimmerButton`, `BeforeAfter`, `AnimatedDemo`, `BentoGrid`, `Breadcrumbs`, `ArticleMeta`, `MetricsRow`, `TypingAnimation`.

Consumer sites import via the `@seo/components` alias. When adding a new component: build in `~/seo-components/src/components/`, bump version, then update each consumer (the `bump:consumers` script automates this).

## Dashboard colors: black and white only (per user instruction 2026-05-27)

**The dashboard at `bin/server.js` (and any HTML/CSS surface it renders) MUST use only black, white, and shades of gray.** No chromatic colors. No green for "good", no red for "bad", no purple/blue/amber/yellow accents on pills, badges, charts, or text.

Use `var(--text)` for foreground, `var(--muted)` for secondary, `var(--bg)` and `var(--card)` for backgrounds, and `var(--border)` for separators. The existing `pill(label, n)` helper at `bin/server.js` already enforces this shape (`color:var(--muted)` for the label, `color:var(--text)` for the number) and accepts a `_color` arg that is **deliberately ignored**; do not "fix" it to apply the color.

Forbidden in any new code on this dashboard:
- Hardcoded hex colors: `#22c55e` (green), `#ef4444` (red), `#a78bfa` (purple), `#eab308` (amber), `#3b82f6` (blue), etc.
- Tailwind palette classes: `text-green-*`, `bg-red-*`, `border-purple-*`, etc.
- Color-coded "good/bad" pills, status badges, or chart series.

If you need to convey severity, use weight (`font-weight:600`), brackets, parens, or a tooltip line, never color. Example: `age 77.1h (cap 1h, leak)` not `age <span style="color:red">77.1h</span>`.

Existing hardcoded chromatic colors elsewhere in `bin/server.js` are tech debt; do NOT proactively refactor them, but do NOT add new ones, and when you touch a line that has one, drop the color while you are there.

To audit when asked to "remove all colors": grep `bin/server.js` for `#[0-9a-fA-F]{3,8}` and `color:\s*(?!var\(|inherit|transparent|currentColor)` to find remaining chromatic usages.

## No retention pruning, ever (per user instruction 2026-05-08)

**Never delete `*_candidates` rows by age.** The user explicitly requires that every candidate row (`twitter_candidates`, `linkedin_candidates`, `reddit_candidates`, etc.) be kept forever, regardless of `status`. The full history (chosen, skipped, expired, posted) powers analytics on skip reasons, project routing, engagement curves, and growth dynamics; pruning destroys that signal.

Forbidden patterns anywhere in this repo (Python, shell, SQL, schedulers):

```sql
-- DO NOT REINTRODUCE
DELETE FROM <table>_candidates
 WHERE status IN ('posted', 'expired', 'skipped')
   AND discovered_at < NOW() - INTERVAL 'N days';
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m13v/s4l](https://github.com/m13v/s4l) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
