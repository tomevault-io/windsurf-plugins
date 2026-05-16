---
trigger: always_on
description: 輸入一場 League of Legends ARAM (queueId=450) 的雙方英雄組合 (5v5)，輸出藍方獲勝機率。
---

<!-- lines: 146 -->
# aram-winrate-nn — ARAM 英雄組合勝率預測 NN，Python / PyTorch

## Why
輸入一場 League of Legends ARAM (queueId=450) 的雙方英雄組合 (5v5)，輸出藍方獲勝機率。
目標是驗證「提供英雄組合資訊後，模型準確率能否超過藍方 base rate (~51%)」。

## Architecture
- **Python 3.13**, PyTorch 2.11, polars, scikit-learn, httpx, psutil, click
- `src/aram_nn/ingest/` — Riot API 爬蟲：`riot_client.py` / `snowball.py` / `extract.py`
- `src/aram_nn/lcu/` — 本機 LCU collector / graph snowball：`process.py` / `client.py` / `poller.py` / `snowball.py`
- `src/aram_nn/models/` — `logreg.py` / `deepsets.py`
- `src/aram_nn/train.py` / `eval.py` / `data.py` — 訓練 pipeline（完成）
- `data/raw/` — parquet 原始資料；`data/lcu/games.db` — LCU SQLite 資料庫（`games` + `crawl_seen` set + `crawl_queue` priority frontier）
- `scripts/` — `probe_user.py`, `probe_queues.py`, `lcu_collector.py`, `build_tier_list.py`
- `docs/index.html` — 公開 tier-list 網站（GitHub Pages, `main` branch `/docs` folder）→ https://lanternko.github.io/ARAM-Mayhem-Database/
- `data/cache/` — `kiwi.bin.json` + `lol_stringtable_zh_tw.json` (CommunityDragon mirror, ~30 MB) 用來解析 Mayhem augment 中文敘述
- 深度技術決策見 `PLAN.md`（v3，已經 Codex review）；部署流程見「Site deploy」節

## Commands
```bash
# 安裝（editable，每次加新 entry point 後要重跑）
python -m pip install -e .

# 資料抓取：從指定 Riot ID snowball，不過濾 patch = 全收
python -m aram_nn.ingest.snowball \
    --region tw \
    --seed-riot-id "Name#TAG" \
    --target-matches 500 \
    --out data/raw/tw_aram_all_patch.parquet

# 資料抓取：過濾特定 patch
python -m aram_nn.ingest.snowball \
    --region tw \
    --seed-riot-id "Name#TAG" \
    --target-matches 2000 \
    --patch 16.9 \
    --out data/raw/tw_aram_16_9.parquet

# 診斷：查某 Riot ID 最近打了哪些 queue
python scripts/probe_user.py --region tw --riot-id "Name#TAG" --count 100

# Tier list 網站重 build（見「Site deploy」節）
python scripts/build_tier_list.py --site-url "https://lanternko.github.io/ARAM-Mayhem-Database/"
```

## LCU Collector (Mayhem data, local client only)

Riot blocks queueId=2400 in the public API.  The League client's own local APIs work.

```powershell
# Run BEFORE launching League — leave open in a separate terminal
python scripts/lcu_collector.py collect          # captures both ARAM (450) + Mayhem (2400)
python scripts/lcu_collector.py collect --queue 2400  # Mayhem only

python scripts/lcu_collector.py status           # see how many games captured
python scripts/lcu_collector.py metrics          # record growth / speed / seed-efficiency snapshots
python scripts/lcu_collector.py snowball --target-games 500 --max-players 200
python scripts/lcu_collector.py snowball --target-games 5000 --max-players 5000 --games-per-player 6 --seed-ladder --seed-apex
python scripts/lcu_collector.py snowball --target-games 5000 --max-players 5000 --games-per-player 6 --seed-riot-tier --riot-tier GOLD --riot-page-limit 3
python scripts/lcu_collector.py snowball-workers --workers 3 --target-games 5000 --max-players 5000 --games-per-player 4 --seed-ladder --seed-apex
python scripts/lcu_collector.py snowball-workers --workers 8 --target-games 5000 --max-players 5000 --games-per-player 4 --seed-riot-tier --riot-tier GOLD --riot-page-limit 3
python scripts/lcu_collector.py seed-opgg --tier diamond --region tw --pages 2 --topn 200 --out data/seeds/opgg_tw.txt
python scripts/lcu_collector.py seed-opgg-plan --region tw --tier diamond --tier emerald --tier platinum --tier gold --pages-per-tier 80 --topn-total 0 --out data/seeds/opgg_tw.txt
python scripts/lcu_collector.py snowball --seed-riot-id-file data/seeds/opgg_tw.txt --target-games 5000 --max-players 5000
python scripts/lcu_collector.py snowball --db data/lcu/games_account_a.db --target-games 5000 --max-players 5000
python scripts/lcu_collector.py snowball --db data/lcu/games_account_b.db --target-games 5000 --max-players 5000
python scripts/lcu_collector.py merge-db --out-db data/lcu/games_merged.db data/lcu/games_account_a.db data/lcu/games_account_b.db
python scripts/lcu_collector.py dataset --queue 2400 --patch-prefix 16.9 --topn 20 --min-games 30
python scripts/lcu_collector.py stats --queue 2400 --patch-prefix 16.9 --out-dir data/stats/mayhem_16_9
python scripts/lcu_collector.py export --out data/raw/lcu_games.parquet
python scripts/lcu_collector.py export --queue 2400 --out data/raw/mayhem_games.parquet
```

`--seed-riot-tier` 會先用 Riot `account-v1 by-puuid` 轉成 `GameName#TagLine`，再用 LCU `/lol-summoner/v2/summoners/names` 橋接成 36-char LCU puuid；這條路可行，但 seed 速度會比 friend / apex 慢。

LCU retains only the **last ~20 games**.  Run the collector every session or you'll miss games.
`snowball` 會從 self / friends / discovered participants 擴張；**exact match dedupe 一律用 `game_id`**，不要用 10 人英雄組合作唯一鍵。
`crawl_seen` + `crawl_queue` 讓 snowball 可中斷續跑；queue 依發現該玩家的最新對戰時間排序，越新的 match 衍生 ID priority 越高。`crawl_seen` 就是 persistent puuid set；worker 另外有 local puuid cache 減少重複 DB enqueue。
`snowball-workers` 會開多個背景 worker 共用同一個 frontier；預設只有第一個 worker 負責 seed，其他 worker 直接消化 queue，避免重複 startup 成本。
`--seed-riot-id-file` 可吃一行一筆的 `Name#TAG`，也接受 OPGG summoner/profile URL；crawler 會先解析成 Riot ID，再經 LCU bridge 成本地 puuid 後入 queue。
多 client 時，每個 client 應各自寫自己的 `--db`；`merge-db` 只合併 `games` 表並以 `game_id` 去重，`crawl_seen` / `crawl_queue` frontier 不要跨 client 合併。

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Lanternko/ARAM-Mayhem-Database](https://github.com/Lanternko/ARAM-Mayhem-Database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
