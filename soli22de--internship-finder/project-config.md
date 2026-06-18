---
trigger: always_on
description: > Read this first before any work. Keeps context without re-explaining.
---

# AGENTS.md — Project Memory for AI Assistants

> Read this first before any work. Keeps context without re-explaining.

## Project: 实习平台数据聚合 (Internship Aggregation Platform)

**Goal**: Build a Shanghai-area internship data platform that aggregates postings from major tech companies and job boards, then matches resumes to the most relevant positions. Eventually a WeChat mini-program.

**Owner**: 张靖恒 (2027届 graduate, targeting data/analytics roles in Shanghai)

**Path**: `C:\jz_code\internship_finding`

---

## Architecture Overview

```
Crawl Layer                    Process Layer               Output Layer
─────────────                  ─────────────               ────────────
crawlers/run_all_official.py → official_jobs_raw.csv  ┐
  ├─ 7 API adapters          (per-company runners)      │
  ├─ 2 Playwright headless   (bytedance, baidu)         ├→ merge_file.py
  └─ CDP mode (deprecated)   (used to need Chrome 9222) │   ├─ cleaning
                                                         │   ├─ cohort27 scoring
ResuMiner/ (submodule)                                   │   ├─ link health
  ├─ 实习僧/猎聘/51job/应届生 crawlers                   │   └─ release_data/
  └─ resume matching engine                              │
                                                         └→ outputs/reports/
                                                             outputs/dashboard/
```

**Two crawling modes**:
- **API mode** (7 companies): Direct `requests.post()` to internal APIs, no browser needed. Fast & reliable.
- **Playwright headless** (2 companies): Launch own Chromium, intercept XHR. ByteDance, Baidu. No manual Chrome needed anymore.

## Current State (2026-05-06)

### Working crawlers (9/9):
| # | Company | Source | Mode | Status |
|---|---------|--------|------|--------|
| 1 | 字节跳动 | official_bytedance | Playwright headless | **Just fixed** (was broken CDP) |
| 2 | 腾讯 | official_tencent_api | API adapter | OK |
| 3 | 快手 | official_kuaishou_api | API adapter | OK |
| 4 | 小红书 | official_xiaohongshu | API adapter | OK |
| 5 | 美团 | official_meituan | API adapter | OK |
| 6 | 阿里 | official_alibaba | API adapter | OK |
| 7 | 京东 | official_jd_api | API adapter | OK |
| 8 | 哔哩哔哩 | official_bilibili | API adapter | OK (preflight fixed) |
| 9 | 百度 | official_baidu | Playwright Firefox | **Fixed** (was 0 rows, now Firefox+DOM) |
| 10 | 实习僧 | shixiseng | Playwright headless | **Working** (fonts decoded, list 40 rows) |
| 11 | BOSS直聘 | boss | Playwright Firefox | **Working** (cookies + double-nav) |
| 12 | 猎聘 | liepin | Playwright API intercept | **New** (84 raw, no anti-bot) |

### Pending / Future:
- [ ] 51job / 应届生 (lower priority)
- [ ] Font map auto-extraction for shixiseng (currently salary shows garbled)
- [x] Semantic resume matching (DeepSeek LLM)
- [ ] FastAPI backend + WeChat mini-program → **Frontend: Opus, Backend: this agent**

---

## Key Files Map

```
internship_finding/
├── config.py                  # Enable/disable companies, strict mode flags
├── official_multi_crawler.py  # Core: SOURCES config, crawl_one_source(), all fetch_*_jobs() functions
├── merge_file.py              # 1408 lines: cleaning, enrichment, scoring, dashboards
├── requirements.txt           # Python dependencies
├── .env                       # DEEPSEEK_API_KEY (gitignored)
│
├── crawlers/                  # Split crawler architecture (per-company runners)
│   ├── run_all_official.py    # Orchestrator: runs all 9 crawlers, preflight health, retry, cache fallback
│   ├── base_runner.py         # run_single_source() + dedup_rows()
│   ├── io.py                  # CSV I/O helpers
│   ├── schema.py              # FIELD_ORDER (21-column canonical schema)
│   ├── bytedance_crawler.py   # ByteDance: Playwright headless via crawl_one_source()
│   ├── baidu_crawler.py       # Baidu: Playwright headless via crawl_one_source()
│   ├── tencent_crawler.py     # Tencent: wraps fetch_tencent_jobs()
│   ├── kuaishou_crawler.py    # Kuaishou: wraps fetch_kuaishou_jobs_api()
│   ├── xiaohongshu_crawler.py # Xiaohongshu: wraps fetch_xiaohongshu_jobs_api()
│   ├── meituan_crawler.py     # Meituan: wraps fetch_meituan_jobs_api()
│   ├── alibaba_crawler.py     # Alibaba: wraps fetch_alibaba_jobs_api()
│   ├── jd_crawler.py          # JD: wraps fetch_jd_jobs()
│   ├── bilibili_crawler.py    # Bilibili: wraps fetch_bilibili_jobs_api()
│   └── cdp/                   # CDP-based crawlers (legacy, for manual Chrome scraping)
│
├── parsers/
│   ├── base_adapter.py        # Abstract BaseCompanyAdapter (fetch_list, parse, get_27_signal)
│   ├── company_adapters.py    # Concrete adapters: Kuaishou, Tencent, XHS, Bilibili, Alibaba, Meituan
│   └── company_registry.py    # ADAPTER_REGISTRY dict + get_adapter() factory
│
├── rules/
│   ├── cohort27_rules.py      # Rule-based 27届 classifier (high/medium/low/none)
│   └── location_normalizer.py # City alias normalization (望京→北京, etc.)
│
├── scripts/
│   ├── resume_job_match_pipeline.py  # Resume-to-JD matching (keyword-based scoring)
│   ├── build_shanghai_internship_latest.py
│   ├── task6_monitoring_metrics.py
│   └── ... (analysis scripts)
│
├── tools/
│   ├── crawler_dispatcher.py  # Unified scheduler (7 tasks, sequential, health report)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Soli22de/internship-finder](https://github.com/Soli22de/internship-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
