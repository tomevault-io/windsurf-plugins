---
trigger: always_on
description: **Name:** `china-market-gateway`
---

# China Market Gateway Skill

**Name:** `china-market-gateway`  
**Description:** Umbrella skill for retrieving Chinese finance data (A-shares, HK, funds, economic data) using Python. Pull stock prices, fund data, market news, and macro indicators from Eastmoney, Sina, CLS, Baidu, and other Chinese sources. Use when researching Chinese equities, mainland stock prices, HK market data, fund performance, or China-related financial news and economic indicators.  
**License:** MIT  
**Compatibility:** Requires Python 3.8+, network access to Chinese financial websites. May require proxy for regions with restricted access.  

**Metadata:**
- **Author:** Etherdrake
- **Version:** 1.0.2
- **Supported Markets:**  
  - A-shares  
  - HK  
  - Shanghai  
  - Shenzhen  
  - Fund  
  - Macro

# YuanData - Chinese Finance Data Retrieval (Python)

## When to Use This Skill

Use this skill when you need to:

- Get real-time or historical stock prices for Chinese A-shares (Shanghai/Shenzhen)
- Retrieve Hong Kong (HK) stock data
- Search for Chinese company financial news
- Research mainland China market information
- Access Baidu Gushitong for stock analysis
- Pull fund data and net asset values
- Fetch macroeconomic indicators (GDP, CPI, PPI, PMI)
- Track investment calendars and economic events

## Supported Sources

| Source           | URL Pattern                                                          | Coverage                     |
|------------------|----------------------------------------------------------------------|------------------------------|
| Eastmoney        | `https://quote.eastmoney.com/{stockCode}.html`                       | A-shares, HK, indices        |
| Sina Finance     | `https://finance.sina.com.cn/realstock/company/{stockCode}/nc.shtml` | A-shares, HK, US             |
| CLS (财联社)        | `https://www.cls.cn/searchPage`                                      | Market news, telegraph       |
| Baidu Gushitong  | `https://gushitong.baidu.com/stock/ab-{stockCode}`                   | Stock analysis, quotes       |
| Eastmoney Funds  | `https://fund.eastmoney.com/{fundCode}.html`                         | Fund data                    |
| Eastmoney Macro  | `https://datacenter-web.eastmoney.com/api/data/v1/get`               | GDP, CPI, PPI, PMI           |

## Stock Code Formats

Chinese stocks use specific prefixes:

- **A-shares (Shanghai):** `sh000001` (SSE), `sh600000` (SSE)
- **A-shares (Shenzhen):** `sz000001` (SZSE), `sz300000` (ChiNext)
- **HK stocks:** `hk00001` (HKEX), `hk00700` (Tencent)
- **US ADRs (for reference):** `usAAPL`

### Examples

| Stock Code     | Company                    | Type       |
|----------------|----------------------------|------------|
| `sh000001`     | Shanghai Composite Index   | Index      |
| `sh600519`     | Kweichow Moutai            | A-share    |
| `sz000001`     | Ping An Bank               | A-share    |
| `hk00700`      | Tencent                    | HK         |
| `hk09660`      | Horizon Robotics           | HK         |

---

# Part 1: Stock Data Retrieval (Python)

## 1. Setup and Dependencies

```python
import requests
from bs4 import BeautifulSoup
import json
import re
import time
from datetime import datetime, timedelta
from typing import Optional, Dict, List, Any
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class YuanData:
    """Chinese Finance Data Retrieval Class"""

    def __init__(self, proxy: Optional[str] = None):
        self.session = requests.Session()
        self.proxy = proxy
        self.headers = {
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36',
            'Accept': 'application/json, text/html,application/xhtml+xml',
            'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8',
        }
        if proxy:
            self.session.proxies = {'http': proxy, 'https': proxy}

    def _request(self, url: str, headers: Optional[Dict] = None, 
                 timeout: int = 30) -> Optional[Any]:
        """Generic HTTP request handler"""
        try:
            req_headers = {**self.headers, **(headers or {})}
            response = self.session.get(url, headers=req_headers, timeout=timeout)
            response.raise_for_status()
            return response
        except Exception as e:
            logger.error(f"Request failed: {url} - {e}")
            return None
```

## 2. Sina Finance API (Real-Time Stock Data)

```python
class SinaStockAPI:
    """Sina Finance Real-Time Stock Data"""

    BASE_URL = "http://hq.sinajs.cn"

    def __init__(self, yuan_data: YuanData):
        self.parent = yuan_data

    def get_stock_quote(self, stock_code: str) -> Optional[Dict]:
        """
        Get real-time stock quote from Sina

        Args:
            stock_code: Stock code (e.g., 'sh600519', 'hk00700')

        Returns:
            Dict with price, volume, change data
        """
        # Handle HK stock format (add leading zero)
        if stock_code.startswith('hk') and len(stock_code) == 6:
            stock_code = 'hk0' + stock_code[2:]

        timestamp = int(time.time() * 1000)
        url = f"{self.BASE_URL}/rn={timestamp}&list={stock_code}"

        headers = {
            'Host': 'hq.sinajs.cn',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Etherdrake/openclaw-china-market-gateway](https://github.com/Etherdrake/openclaw-china-market-gateway) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
