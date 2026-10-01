---
trigger: always_on
description: `jikan-edge` holds the initial vertical slice of **JikanV2** for WeebProfile: Cloudflare Workers, Hono, D1 and parsing of public MyAnimeList HTML. The goal is to validate viability on the Free plan without trying to reproduce Jikan's 100 routes.
---

# Project guide for agents

## State and goal

`jikan-edge` holds the initial vertical slice of **JikanV2** for WeebProfile: Cloudflare Workers, Hono, D1 and parsing of public MyAnimeList HTML. The goal is to validate viability on the Free plan without trying to reproduce Jikan's 100 routes.

The milestone's production code is authorized for user profile, statistics and lists, and — by a decision recorded in `docs/planning/jikan-v4-route-validation.md` ("Expansion decision (2026-07-26)") — for the P0 anime catalog: detail, genres, top and current season. Preserve the separation between source client, pure parser, domain, D1, services and HTTP.

**Full-parity decision (2026-07-26):** the user explicitly decided to widen the scope to every Jikan route (recorded in `docs/planning/jikan-v4-route-validation.md`, section "Full-parity decision"). This supersedes the earlier scope limit — new route groups (manga, characters, people, clubs, producers, seasons, watch, recommendations, reviews, magazines, schedules, search) are now authorized for incremental implementation, following the same per-route quality bar (real page probe, fixture, parser, tests, contract) already used for the anime catalog. Work in progress, in batches.

## Reading order

1. `README.md`
2. `docs/architecture.md`
3. `docs/planning/jikan-v4-route-validation.md`
4. `docs/results/initial-viability.md`
5. The applicable research in `docs/research/`

## Working rules

- Treat Jikan as a functional reference, not as a full-compatibility requirement.
- Use controlled refresh, D1 and stale fallback; never replace valid data with a suspicious document.
- Only use `https://myanimelist.net`; URLs are centralized in `src/source/mal-urls.ts`.
- Do not use a headless browser, internal endpoints, mass scraping or client-supplied URLs.
- Do not widen the scope to every Jikan route without a recorded decision. The full-parity decision (line above) already covers the implemented groups: anime, manga, characters, producers, clubs, people, seasons, watch, recommendations, reviews, magazines, schedules and anime/manga search.
- **When committing any change that affects API consumers** (response contract, a new error code, a visible bug fix, observable cache behavior), **add the corresponding entry to `CHANGELOG.md` in the same commit** — not later, not "when I remember". The entry's content (what, why, before/after) can be written at commit time; only the "Published versions" line depends on a confirmed deploy, because the id only exists after the build. Leave the entry with its date and content ready and an explicit note like "Published version: to be confirmed" rather than inventing an id or skipping the entry — later you just come back and fill in the real id (`npx wrangler deployments list` or the dashboard's Deployments tab).
  - **The Git integration works: every `git push` to `main` triggers a build and deploy, in ~30-60 s** (verified 2026-08-18 in the Builds tab, which lists each build's commit). The earlier note here said the opposite, based on 2026-08-10 — but on that date `pnpm-workspace.yaml` made *every* build fail at dependency installation; `2a6bea6` removed the file on 2026-08-16. "Does not trigger" was a broken build, not asynchronous integration.
  - **Consequence: do not run `wrangler deploy` before `git push`.** It is redundant (the same change ships twice, generating two version ids) and deliberately reintroduces the risk the deploy rule below describes — a manual deploy publishes the *file tree* of that moment, while the build publishes the *commit*. On 2026-08-18 this produced 9 versions for 3 changes. **The id that matters is the one from the build the push triggered**, because it arrives last and is the one left serving.
  - **An entry can never name the version that publishes it.** Any given day's last id can only be filled in afterwards, in a following commit — the absence is structural, not a lost deploy. That was the case for `3853a33d` (2026-08-16), tracked down on 2026-08-18.
- Anime/manga search (`GET /v1/anime?q=`, `GET /v1/manga?q=`) implemented 2026-07-26 — see `docs/routes.md` for the contract and the note about MAL's fallback behavior for queries with no match.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LucasHenriqueDiniz/jikan-edge](https://github.com/LucasHenriqueDiniz/jikan-edge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
