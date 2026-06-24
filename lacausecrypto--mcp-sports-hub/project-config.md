---
trigger: always_on
description: Unified MCP server — 32 providers, 336 tools, one process.
---

# Sports Hub MCP Server

Unified MCP server — 32 providers, 336 tools, one process.
Covers: NFL, NBA, MLB, NHL, Soccer, F1, Tennis, Cricket, MMA, Golf, Esports, Rugby, Volleyball, Handball, College Sports, Chess, AFL, and more.

Works with any MCP client (Claude, ChatGPT, Gemini, Cursor, Windsurf, Continue, Cline, Zed).
Uses stdio transport — compatible with any LLM supporting the Model Context Protocol.

## Provider Reference

### No API key required (12 providers, ~109 tools)

| Prefix | Provider | Coverage | Tools |
|--------|----------|----------|-------|
| `espn_` | ESPN (unofficial) | 20+ sports — scores, standings, news | 10 |
| `nhl_` | NHL Web API | NHL schedules, rosters, player stats | 13 |
| `mlb_` | MLB Stats API | MLB/MiLB games, players, standings | 13 |
| `f1_` | Jolpica F1 | F1 results, standings, circuits (1950-now) | 13 |
| `openf1_` | OpenF1 | F1 live telemetry, lap times, race control | 12 |
| `openliga_` | OpenLigaDB | German football (Bundesliga focus) | 10 |
| `sportsdb_` | TheSportsDB | 40+ sports, teams, players, events | 13 |
| `ncaa_` | NCAA API | College sports (football, basketball, etc.) | 8 |
| `sportsrc_` | SportSRC | Football/Basketball/MMA + streams | 7 |
| `lichess_` | Lichess | Chess (users, top players, broadcasts, tournaments, daily puzzle) | 7 |
| `chesscom_` | Chess.com | Chess (profiles, stats, clubs, archives, leaderboards) | 7 |
| `squiggle_` | Squiggle | AFL (Australian Football League): teams, games, ladder, tips, sources | 6 |

`sportsdb_` defaults to test key "3" (free, watermarked images). Set `THESPORTSDB_API_KEY` for a personal key.
`sportsrc_` V1 endpoints are free with no key. V2 (xG, momentum, lineups) needs `SPORTSRC_API_KEY` and is currently NOT exposed.

### API key required (21 providers, ~227 tools)

| Prefix | Provider | Env var | Coverage | Tools | Free limit |
|--------|----------|---------|----------|-------|------------|
| `apisports_` | API-Sports | `API_SPORTS_KEY` | 9 sports multi-stat | 10 | 100/day/sport |
| `apifootball_` | API-Football | `API_FOOTBALL_KEY` | Soccer 960+ leagues | 15 | 100/day |
| `apitennis_` | API-Tennis | `API_TENNIS_KEY` | ATP, WTA, ITF | 12 | 100/day |
| `bdl_` | BallDontLie | `BALLDONTLIE_API_KEY` | NBA, NFL, MLB, NHL | 10 | Basic tier |
| `cricket_` | CricketData | `CRICKETDATA_API_KEY` | Cricket (Test, ODI, T20, IPL) | 10 | 100/day |
| `entitycricket_` | Entity Sport | `ENTITY_SPORT_KEY` | Cricket 250+ competitions | 12 | Free plan |
| `footballdata_` | football-data.org | `FOOTBALL_DATA_API_KEY` | Soccer 12 European leagues | 11 | 10/min |
| `sportmonks_` | Sportmonks | `SPORTMONKS_API_KEY` | Soccer (Danish + Scottish free) | 12 | 3000/hr |
| `sportsdata_` | SportsDataIO | `SPORTSDATA_IO_KEY` | 9 sports (data scrambled) | 12 | 1000/mo |
| `odds_` | The Odds API | `THE_ODDS_API_KEY` | Odds 70+ sports, 40+ bookmakers | 9 | 500/mo |
| `oddsio_` | Odds-API.io | `ODDS_API_IO_KEY` | Odds 34 sports, 265+ bookmakers | 10 | Free account |
| `sgo_` | Sports Game Odds | `SPORTS_GAME_ODDS_KEY` | Odds 55+ leagues, player props | 10 | Trial |
| `mma_` | Fighting Tomatoes | `FIGHTING_TOMATOES_API_KEY` | MMA fight history | 8 | 200/mo |
| `livegolf_` | Live Golf API | `LIVE_GOLF_API_KEY` | Golf PGA, DP World Tour | 8 | Free tier |
| `isports_` | iSportsAPI | `ISPORTSAPI_KEY` | Football + Basketball (Asia-Pacific) | 10 | Free tier |
| `sportdevs_` | SportDevs | `SPORTDEVS_API_KEY` | Rugby, Volleyball, Handball | 12 | Trial |
| `msf_` | MySportsFeeds | `MYSPORTSFEEDS_USER` + `_PASS` | NFL, NBA, MLB, NHL (detailed) | 12 | Free non-commercial |
| `pandascore_` | PandaScore | `PANDASCORE_TOKEN` | Esports 13 titles (LoL, CS2, Dota2...) | 14 | 1000/hr |
| `golfcourse_` | GolfCourseAPI | `GOLFCOURSE_API_KEY` | 30K+ golf courses worldwide | 6 | 300/day |
| `cfbd_` | College Football Data | `CFBD_API_KEY` | NCAA football (games, stats, recruiting) | 14 | 1000/mo |

## Choosing the right tool

**Live scores**: `espn_get_scoreboard` (multi-sport), `nhl_get_scores`, `openliga_get_current_matchday`, `sportsrc_get_live`
**Standings**: `espn_get_standings`, `nhl_get_standings`, `mlb_get_standings`, `apifootball_get_standings`, `footballdata_get_standings`
**Player stats**: `mlb_get_player_stats`, `nhl_get_player`, `bdl_get_stats`, `apisports_get_players`
**Game details**: `mlb_get_game_boxscore`, `nhl_get_game_boxscore`, `espn_get_event_summary`
**Soccer fixtures**: `apifootball_get_fixtures` (960+ leagues), `footballdata_get_matches` (12 top leagues), `sportsrc_get_matches`
**F1 data**: `f1_get_race_results` (historical 1950+), `openf1_get_laps` (live telemetry)
**Betting odds**: `odds_get_odds`, `oddsio_get_odds`, `sgo_get_odds`
**Esports**: `pandascore_get_matches`, `pandascore_get_lives` (LoL, CS2, Dota2, Valorant, etc.)
**Cricket**: `cricket_get_current_matches`, `entitycricket_get_matches`
**Golf**: `livegolf_get_leaderboard`, `golfcourse_search_courses`
**MMA**: `mma_search_fighters`, `mma_get_fighter_fights`
**College sports**: `ncaa_get_scoreboard`, `ncaa_get_rankings`, `cfbd_get_games`, `cfbd_get_plays`
**Chess**: `lichess_get_top_players`, `lichess_get_user`, `lichess_get_daily_puzzle`, `chesscom_get_player_stats`, `chesscom_get_leaderboards`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lacausecrypto/mcp-sports-hub](https://github.com/lacausecrypto/mcp-sports-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
