---
trigger: always_on
description: Arabic-language Moroccan football news site. Next.js 16 (App Router) + Payload CMS 3 on Neon Postgres (Frankfurt), deployed on Vercel.
---

# MFM Sport — working notes

Arabic-language Moroccan football news site. Next.js 16 (App Router) + Payload CMS 3 on Neon Postgres (Frankfurt), deployed on Vercel.

---

## Session state — SEO remediation

**Updated: 28 July 2026, phase boundary — 2024 import batch in flight.** Update this at every phase boundary. It is deliberately ground truth on disk rather than in a conversation summary.

### Resume here

**The 2024–2026 staged release is IMPORTED. Next action is yours, not the code's: review Search Console before importing any older year.**

| Batch | created | archive-full | archive-brief | video | no-date | empty | failed |
|---|---|---|---|---|---|---|---|
| 2024 | 6,570 | 5,706 | 864 | 0 | 0 | 0 | 0 |
| 2025 | 1,509 | 1,285 | 224 | 1 | 0 | 0 | 0 |
| 2026 | 463 | 461 | 2 | 0 | 0 | 0 | 0 |
| **total** | **8,542** | **7,452** | **1,090** | 1 | 0 | 0 | **0** |

Every batch matched its independently-audited prediction exactly — thin counts of 864 / 224, and 2026's `463 + 101 already-imported = 564`. Two code paths, same numbers.

Database after the release: **8,940 articles** (8,542 imported + 398 `editorial`), **8,742 redirects** (= 200 + 8,542), **0** null publish dates, **0** articles older than 2024. The staged boundary held; nothing leaked.

Spot-checked 10 evenly-spaced articles per batch against the XML — **30/30 at ratio 1.00**, dates and redirects correct (`pnpm spotcheck --year=<Y>`).

**BLOCKED on a deploy:** `/sitemap.xml` still serves the pre-import 906 URLs / 398 articles, byte-identical to before. It caches for 24h and the route that busts it is in this branch, undeployed. Once deployed, POST `{"collection":"sitemap"}` to `/api/revalidate` — or the importer now does it automatically at the end of a run. Expect the sitemap to go to roughly **7,850** article URLs (398 editorial + 7,452 released `archive-full`); `archive-brief` must stay out.

`/news-sitemap.xml` is verified still at **13 URLs** — the archive did not leak into the 48h feed.

```bash
# after this branch deploys, confirm the release is advertised:
curl -s https://www.mfmsport.ma/sitemap.xml | grep -o '<loc>' | wc -l

# then STOP. Do not import 2023 or earlier — see Hard gates.
```

**Still unfixed:** `<html>` carries no `lang`/`dir` (see open defects).

### Merged and deployed

| PR | What |
|---|---|
| #52 | Real 404s, ad-free error pages, `www` canonical, 307→308, Arabic description, match whitelist, archive importer |
| #53 | Reverted sitemap sharding (it served zero URLs and 404'd `/sitemap.xml`) |
| #54 | hreflang → Arabic only, canonicals on all page types, `docs/verification-principles.md`, this file |
| #55 | Upstream API failure no longer serves 404 for fixtures that exist |

All four are **live on production and verified against the served bytes** (deployment `dpl_9mR6QBDznjofSoHUSSn9xbafdG1i`):

- `/ar/transfers` and a missing article slug → **404** (real, not soft)
- apex → `/ar` → **308**
- `/sitemap.xml` → 200 with **906 `<loc>`**, of which 398 are articles
- `/news-sitemap.xml` → 200 with **13 `<loc>`** — the 48h window is holding, not leaking the archive
- article page → exactly **2** hreflang alternates, `ar-MA` + `x-default`, both `/ar`; canonical on `www`; no `robots` meta (indexable)
- 404 page → **0** real `adsbygoogle.js` script tags

### Redirect map — repaired and verified end-to-end

All 200 rows were stored in the dead format (100% had *both* a trailing slash and lowercase hex). `pnpm redirects:normalize` rewrote all 200; **0 duplicate collisions**.

Verified on the artefact, not the table:

| Check | Result |
|---|---|
| `/api/redirects?from=…` — the exact request middleware makes | **200/200** return the correct target |
| Stale `{to: null}` cached by the CDN before the repair | **0** (the deploy reset the cache) |
| Destination article URLs | **200/200** return HTTP 200 |
| One full legacy chain, end to end | `308` → `301` → `200` |

**Breakdown of the 200: all 200 → live published article.** Zero category-hub fallbacks, zero dead targets.

`pnpm redirects:verify` (`scripts/verify-redirects.ts`) re-runs the whole check any time. It probes the lookup endpoint rather than the legacy URLs, deliberately: fetching all 200 legacy URLs would consume the untouched sample someone may want for an independent spot-check.

The legacy URL consumed for the end-to-end chain test was **`/الجامعة-تبرم-اتفاقية-شراكة-مع-المكتب-ا`** (row 356). The other 199 are untouched.

### Database

Archive-fields DDL is **applied to production** (`broad-snow-50246164` / branch `br-royal-wildflower-a21skzaw`): `wp_post_id`, `legacy_slug`, `seo_tier`, three indexes, `payload_migrations` batch 8. All pre-existing articles default to `editorial`, so they stay indexable.

Verification branch **`br-gentle-hat-a2bzeay0`** is alive deliberately — keep it until the full import is done. It holds ~2,378 imported archive articles and 2,178 normalised redirects, and is useful to diff against.

**The 2024 import COMPLETED against production.**

| | |
|---|---|
| created | 6,570 (exactly the corpus-predicted count) |
| `archive-full` / `archive-brief` | 5,706 / 864 |
| skipped as already-imported | 101 (the backfill working) |
| redirects created | 6,570 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [newpublicity2024-cmyk/MFM-sport](https://github.com/newpublicity2024-cmyk/MFM-sport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
