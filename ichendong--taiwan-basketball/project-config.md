---
trigger: always_on
description: Taiwan professional basketball stats, scores, schedules, player data, live scores, box scores, notifications, and transactions for PLG and TPBL.
---


# Taiwan Basketball Skill - 台灣職籃資訊查詢 🏀

Query PLG (P. LEAGUE+) and TPBL (台灣職業籃球大聯盟) game results, schedules, standings, player stats, league leaders, live scores, box scores, notifications, and transactions.

## Data Sources

| Source | Description |
|--------|-------------|
| PLG official website | HTML scraping (server-side rendered) |
| TPBL official REST API | `api.tpbl.basketball` |
| 台灣籃球維基館 | Scrapling StealthyFetcher (bypasses Anubis protection) |
| Local SQLite DB | `~/.local/share/taiwan-basketball/basketball.db` |

## Features

| Feature | Script | Source |
|---------|--------|--------|
| Schedule (with countdown) | `basketball_schedule.py` | PLG website / TPBL API |
| Standings | `basketball_standings.py` | PLG website / TPBL API |
| Game results | `basketball_games.py` | PLG website / TPBL API |
| Player stats | `basketball_player.py` | PLG website / TPBL API |
| League leaders | `basketball_leaders.py` | PLG website / TPBL API |
| Player comparison | `basketball_compare.py` | PLG website / TPBL API |
| **Live scores** ✨ | `basketball_live.py` | TPBL API / PLG time-based |
| **Box Score** ✨ | `basketball_boxscore.py` | TPBL API / PLG website |
| **Notifications** ✨ | `basketball_notify.py` | PLG website / TPBL API |
| **Transactions** ✨ | `basketball_transactions.py` | PLG news / TPBL API |
| **Wiki (awards/history)** ✨ | `_wiki_api.py` | 台灣籃球維基館 (Scrapling StealthyFetcher) |

## Architecture

```
scripts/
  _cache.py            # 磁碟 TTL 快取模組
  _http.py             # HTTP 工具（重試 / 快取）
  _utils.py            # 共用工具（格式化、球隊別名、球員別名、並行擷取）
  _tpbl_api.py         # TPBL REST API 封裝
  _plg_api.py          # PLG HTML 爬蟲封裝
  _wiki_api.py         # 台灣籃球維基館（Scrapling StealthyFetcher 繞過 Anubis）
  _basketball_api.py   # 兼容性匯入層（維持所有腳本相容）
  _db.py               # SQLite 資料持久化模組
  basketball_*.py      # CLI 腳本
```

**並行擷取**：所有 `--league all` 查詢均使用 `ThreadPoolExecutor` 並行發送 PLG/TPBL 請求，大幅縮短等待時間。

## Quick Start

All scripts use `uv run` for dependency management.

### Schedule

```bash
uv run scripts/basketball_schedule.py --league plg
uv run scripts/basketball_schedule.py --league tpbl
uv run scripts/basketball_schedule.py --league all       # PLG + TPBL 合併查詢
uv run scripts/basketball_schedule.py -l plg --team 勇士
uv run scripts/basketball_schedule.py -l all --next      # 只顯示下一場比賽及倒數
uv run scripts/basketball_schedule.py -l all --format table
uv run scripts/basketball_schedule.py -l all --stage playoffs    # 只顯示季後賽賽程
uv run scripts/basketball_schedule.py -l tpbl --stage play-in   # 只顯示季後挑戰賽賽程
```

### Standings

```bash
uv run scripts/basketball_standings.py --league plg
uv run scripts/basketball_standings.py --league tpbl
uv run scripts/basketball_standings.py --league plg --format table
```

### Game Results

```bash
uv run scripts/basketball_games.py --league plg
uv run scripts/basketball_games.py --league tpbl
uv run scripts/basketball_games.py --league all            # PLG + TPBL 合併查詢
uv run scripts/basketball_games.py --league all --last 5   # 最近 5 場結果
uv run scripts/basketball_games.py -l tpbl --team 戰神
uv run scripts/basketball_games.py -l all --last 10 --format table
uv run scripts/basketball_games.py -l tpbl --stage play-in  # 只顯示季後挑戰賽結果
uv run scripts/basketball_games.py -l all --stage playoffs  # 只顯示季後賽結果
```

### Player Stats

```bash
uv run scripts/basketball_player.py --league plg --player 林書豪
uv run scripts/basketball_player.py --league tpbl --player 林書豪
uv run scripts/basketball_player.py --league all --player 林書豪
uv run scripts/basketball_player.py -l plg -p 林書豪 --season 2023-24
uv run scripts/basketball_player.py -l tpbl -p 夢想家           # 球隊搜尋
```

Player names support aliases (e.g. `高柏鎧` finds `吉爾貝克` in PLG, `吉爾貝克` finds `高柏鎧` in TPBL).

### League Leaders（排行榜）

```bash
uv run scripts/basketball_leaders.py --league plg --stat pts          # PLG 得分王
uv run scripts/basketball_leaders.py --league tpbl --stat reb --top 5 # TPBL 籃板前5名
uv run scripts/basketball_leaders.py -l tpbl -s ast --format table    # 表格輸出
uv run scripts/basketball_leaders.py -l all -s pts --top 10           # 雙聯盟得分榜
```

Supported `--stat` values: `pts`（得分）、`reb`（籃板）、`ast`（助攻）、`stl`（抄截）、`blk`（阻攻）、`tov`（失誤）、`pf`（犯規）、`eff`（效率值，TPBL 限定）

### Player Comparison（球員比較）

```bash
uv run scripts/basketball_compare.py --league plg --player1 林書豪 --player2 戴維斯
uv run scripts/basketball_compare.py -l tpbl -p1 林志傑 -p2 陳盈駿
uv run scripts/basketball_compare.py -l plg -p1 林書豪 -p2 戴維斯 --season 2023-24
uv run scripts/basketball_compare.py -l plg -p1 林書豪 -p2 戴維斯 --format table
```

Supports fuzzy search by player name or team name. Returns per-season stats (GP, avg minutes/pts/reb/ast/stl/blk, FG/3P/FT splits, efficiency, PIR) plus career totals.

- **PLG**: Scrapes `/stat-player` + `/all-players` for player index, then `/player/{ID}` for detailed per-season stats.
- **TPBL**: Queries `/games/stats/players?division_id={id}` for all divisions across all seasons. Recalculates FG%/3P%/FT% from accumulated makes/attempts for cross-division accuracy.

### Live Scores（即時比分）✨

```bash
uv run scripts/basketball_live.py --league all
uv run scripts/basketball_live.py --league tpbl --format table
uv run scripts/basketball_live.py --league plg
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ichendong/taiwan-basketball](https://github.com/ichendong/taiwan-basketball) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
