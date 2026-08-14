---
trigger: always_on
description: An automated agent team that analyzes every 2026 FIFA World Cup match (June 11 – July 19, 2026, 104 matches), produces per-match analysis tickets, composes daily betting-ticket recommendations with **virtual stakes only**, and tracks its own performance.
---

# World Cup 2026 AI Analysis Team

An automated agent team that analyzes every 2026 FIFA World Cup match (June 11 – July 19, 2026, 104 matches), produces per-match analysis tickets, composes daily betting-ticket recommendations with **virtual stakes only**, and tracks its own performance.

## Hard guardrails

- **Never place, automate, or facilitate real-money bets.** All stakes are virtual units against a simulated bankroll. Recommendations are analysis, not financial advice.
- **Never invent data.** Every odds value must come from a fetched source recorded in `sources`. If odds can't be found for a market, omit that market.
- **The ledger is append-only truth.** Never hand-edit `data/ledger.json`; it is rebuilt by `scripts/settle.py`.
- Timestamps are always UTC ISO-8601 (`2026-06-11T19:00:00Z`). A **matchday** `YYYY-MM-DD` covers kickoffs from 06:00 UTC that day to 05:59 UTC the next day — North-America evening games that cross UTC midnight belong to the previous (local) matchday.

## Pipeline (what /daily-run does)

1. Settle yesterday: `python3 scripts/fetch_results.py <yesterday>`, stats enrichment (see /settle skill), then `python3 scripts/settle.py`
2. Fetch today: `python3 scripts/fetch_fixtures.py <today>`
3. Polymarket prices: `python3 scripts/fetch_polymarket.py <today>` and `python3 scripts/fetch_polymarket.py --futures`
4. `odds-researcher` agent → `data/odds/<today>.json`
5. One `match-analyst` agent per fixture (parallel) → `data/tickets/<today>/<match_id>.json`
6. `ticket-builder` agent → `data/betslips/<today>.json`
7. Futures review (agent step in /daily-run) → may add positions to `data/futures.json`
8. `python3 scripts/build_index.py` (regenerates `data/index.json` for the dashboard)
9. Commit (`daily: YYYY-MM-DD`) and push — the dashboard on GitHub Pages updates on push.

## Data sources (free tier)

- **Fixtures/results:** football-data.org v4, competition code `WC`, header `X-Auth-Token` from `FOOTBALL_DATA_TOKEN` (in gitignored `.env`). 10 req/min. If the token is missing, the scripts exit with code 2 and agents fall back to web research (fifa.com, bbc.com/sport) writing the identical schema.
- **Odds:** no free bookmaker odds API covers the World Cup — the `odds-researcher` agent web-researches public odds pages (oddschecker.com, covers.com, oddsportal.com, bookmaker public sites) and records the sources used.
- **Polymarket:** free public Gamma API (`gamma-api.polymarket.com`, no key) via `scripts/fetch_polymarket.py` — per-match prices and futures (tournament/group winners). Prices are probabilities (0–1); decimal-odds equivalent = 1/price. Polymarket uses FIFA team naming ("Korea Republic"); aliases live in `fetch_polymarket.py`. Tomas places his real bets on Polymarket, so PM recommendations must be actionable: outcome, limit price ("buy ≤ 43¢"), and the market URL.

## JSON schemas (data/)

All files written by scripts or agents MUST match these shapes exactly.

### fixtures/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "source": "football-data.org", "fetched_at": "<iso>",
 "matches": [{"match_id": "fd-12345", "kickoff_utc": "<iso>", "stage": "GROUP_STAGE",
              "group": "GROUP_A", "home": "Mexico", "away": "South Africa", "venue": "Estadio Azteca"}]}
```
`match_id` is `fd-<football-data id>`; web-research fallback uses `wc-<seq>` slugs but must stay stable across files.

### results/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "source": "football-data.org", "fetched_at": "<iso>",
 "matches": [{"match_id": "fd-12345", "home": "Mexico", "away": "South Africa",
              "home_goals": 2, "away_goals": 1, "status": "FINISHED",
              "home_cards": 2, "away_cards": 3, "home_reds": 0, "away_reds": 1,
              "home_corners": 7, "away_corners": 3,
              "stats_sources": ["https://..."]}]}
```
`status`: `FINISHED` | `POSTPONED` | `CANCELLED`. Goals are full-time (after extra time in knockouts; note shootout in `notes` if any — h2h market in knockout stage settles on the result after ET, draw possible only in group stage).

The six stats fields (cards = yellows+reds per team, reds, corners) are **enriched by the /settle agent from web-researched match reports** (FIFA/BBC/ESPN) — football-data.org free tier doesn't provide them. Set a field to `null` whenever it can't be verified from a real source; `settle.py` then **voids** any leg depending on it. Never guess stats.

### odds/YYYY-MM-DD.json
```json
{"date": "2026-06-11", "collected_at": "<iso>",
 "matches": [{"match_id": "fd-12345",
   "h2h": {"home": 2.10, "draw": 3.30, "away": 3.60},
   "double_chance": {"1x": 1.28, "12": 1.32, "x2": 1.72},
   "dnb": {"home": 1.55, "away": 2.45},
   "totals": {"line": 2.5, "over": 1.95, "under": 1.85},
   "totals_alt": [{"line": 1.5, "over": 1.45, "under": 2.70},
                  {"line": 3.5, "over": 3.10, "under": 1.36}],
   "team_totals": {"home": {"line": 1.5, "over": 2.00, "under": 1.80},
                   "away": {"line": 0.5, "over": 1.65, "under": 2.20}},
   "btts": {"yes": 2.05, "no": 1.75},
   "cards": {"line": 4.5, "over": 1.90, "under": 1.90},

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomikng/worldcup-2026-analysis](https://github.com/tomikng/worldcup-2026-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
