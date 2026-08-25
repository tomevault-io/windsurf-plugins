---
trigger: always_on
description: 你是追蹤系統模板的維護者，負責維護通用骨架供各 repo 複製使用。
---

# Templates - 模板維護者指引

你是追蹤系統模板的維護者，負責維護通用骨架供各 repo 複製使用。

## 目錄結構

```
templates/
├── lib/                        # Python 函式庫
│   ├── __init__.py
│   ├── matcher.py              # 關鍵字匹配引擎
│   ├── sentiment.py            # 情緒分析
│   ├── scorer.py               # 重要性評分
│   └── anomaly.py              # 異常偵測
│
├── scripts/                    # 執行腳本
│   ├── enrich_event.py         # 事件標註
│   ├── generate_metrics.py     # 計算每日指標
│   ├── detect_anomalies.py     # 異常偵測
│   ├── generate_daily.py       # 生成每日報告
│   ├── generate_7d_report.py   # 生成 7 日報告
│   ├── update_baselines.py     # 更新歷史基準線
│   ├── fetch_holders.py        # 大股東/持股異動
│   ├── fetch_fund_flow.py      # ETF 資金流向
│   ├── generate_financials_history.py  # 財報歷史
│   ├── generate_llm_analysis.py        # LLM 分析報告
│   ├── generate_docs_skeleton.py       # 文件骨架產生
│   ├── validate_docs.py        # 文件驗證
│   ├── validate_docs.sh        # 文件驗證 wrapper
│   ├── backfill_financials.py  # 回填財報
│   └── backfill_reports.py     # 回填 LLM 分析
│
├── configs/                    # 設定檔範例
│   ├── site.yml.example        # 網站設定（標題、連結）
│   ├── companies.yml.example
│   ├── topics.yml.example
│   ├── feeds.yml.example
│   ├── sentiment_rules.yml.example
│   ├── importance_rules.yml.example
│   ├── anomaly_rules.yml.example
│   └── 7d_highlights_rules.yml.example
│
├── fetchers/                   # 爬蟲
│   ├── base.py                 # 公司官網爬蟲基底類別
│   └── example_company.py      # 爬蟲範例
│
├── site/                       # 前端 Dashboard
│   └── index.html              # 動態載入設定（從 data/site.json）
│
├── data/                       # 資料目錄
│   ├── events/                 # 事件 (JSONL)
│   ├── metrics/                # 每日指標 (JSON)
│   └── baselines/              # 歷史基準線 (JSON)
│
├── reports/                    # 報告目錄
│   ├── daily/                  # 每日報告
│   └── 7d/                     # 7 日報告
│
├── .claude/skills/             # Claude Code 操作指引
│   ├── 每日檢查.md
│   ├── 修復爬蟲.md
│   ├── 關鍵字調整.md
│   ├── 新增公司.md
│   ├── 新增畫面功能.md
│   ├── 執行抓取.md
│   ├── 產出報告.md
│   └── 畫面規範.md
│
├── tests/                      # 測試
│   ├── test_history_dedup.py
│   └── test_validate_docs.sh
│
├── docs/                       # 文件骨架
│   ├── companies/
│   ├── daily/
│   ├── live/
│   ├── monthly/
│   ├── quarterly/
│   ├── regulatory-archive/
│   └── weekly/
│
├── .github/workflows/
│   ├── daily-ingest.yml        # 每日自動抓取流程
│   ├── deploy-pages.yml        # GitHub Pages 部署
│   ├── validate-docs.yml       # 文件結構驗證
│   └── sync-downstream.yml     # 同步 skills/workflows 到下游 repos
│
├── requirements.txt            # Python 依賴
└── .gitignore                  # Git 忽略規則
```

## 標準流程

```
fetch_news → enrich_event → generate_metrics → detect_anomalies →
generate_daily → generate_7d_report → update_baselines → deploy
```

**重要**：`update_baselines` 必須在報告生成之後執行，避免今日資料影響今日的基準線比較。

## 各檔案職責

### lib/matcher.py
- 關鍵字匹配引擎
- 從文字中匹配公司和主題
- 判斷客戶/供應商關係（根據 companies.yml 的 upstream/downstream）

### lib/sentiment.py
- 情緒分析引擎
- 關鍵字匹配 + 否定詞處理
- 輸出 label (positive/neutral/negative) 和 score (-1 ~ 1)

### lib/scorer.py
- 重要性評分引擎
- 根據設定的規則計算分數
- 基礎分數 0.5，上限 1.0

### lib/anomaly.py
- 異常偵測引擎
- 三種異常：volume_spike, sentiment_shift, topic_resurface
- 尊重最小資料量要求

## 可客製化的部分

| 類型 | 檔案 | 說明 |
|------|------|------|
| 設定 | `configs/*.yml` | 不同產業的公司、主題、規則 |
| 邏輯 | `lib/matcher.py` | 可加入產業特有的匹配邏輯 |
| 邏輯 | `lib/sentiment.py` | 可加入產業特有的情緒詞 |
| 爬蟲 | `fetchers/*.py` | 各 repo 自行實作 |

## 新建追蹤的流程

### 從 Template 建立（推薦）

```bash
# 1. 從 template 建立新 repo
gh repo create weiqi-kids/{name} --template weiqi-kids/intel-template --public
git clone git@github.com:weiqi-kids/{name}.git

# 2. 複製範例設定檔並編輯
cp configs/companies.yml.example configs/companies.yml
cp configs/topics.yml.example configs/topics.yml
cp configs/feeds.yml.example configs/feeds.yml
cp configs/site.yml.example configs/site.yml
# 編輯各設定檔

# 3. 建立 fetchers（每家公司一個，參考 fetchers/example_company.py）
cp fetchers/example_company.py fetchers/{company_id}.py
# 編輯 fetcher，註冊到 scripts/fetch_companies.py

# 4. 設定 GitHub Pages
mv site/CNAME.example site/CNAME
# 編輯 CNAME 填入網域

# 5. Push 並驗證
git push origin main
gh workflow run "Daily Data Ingest"
```

### 建立爬蟲

每家公司需要一個爬蟲（繼承 `fetchers/base.py`）：

```python
# fetchers/{company_id}.py
from .base import CompanyFetcher, CompanyDocument
from datetime import datetime

class CompanyNameFetcher(CompanyFetcher):
    company_id = "company_id"
    company_name = "Company Name"
    news_url = "https://company.com/news"

    def parse_news(self, html: str) -> list[CompanyDocument]:
        soup = self._parse_html(html)
        docs = []
        # 解析邏輯...
        return docs

    def parse_ir(self, html: str) -> list[CompanyDocument]:
        return []  # 如果沒有 IR 頁面

if __name__ == "__main__":
    with CompanyNameFetcher() as fetcher:
        docs = fetcher.fetch_news()
        for doc in docs:
            print(doc.to_json())
```

### 測試

```bash
# 測試爬蟲
python -m fetchers.{company_id}

# 測試標註
python scripts/enrich_event.py --date 2026-03-16 --input data/raw/2026-03-16/test.jsonl

# 啟動本地伺服器
python -m http.server 8000 -d site
```

## 事件結構

```json
{
  "id": "{company}-{date}-{seq}",
  "date": "YYYY-MM-DD",
  "time_tags": {
    "year": 2026,
    "quarter": "Q1",
    "month": 3,
    "week": 11,
    "weekday": "Thu"
  },
  "entities": {
    "companies": ["samsung"],
    "customers": ["nvidia"],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weiqi-kids/intel-template](https://github.com/weiqi-kids/intel-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
