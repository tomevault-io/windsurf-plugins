---
trigger: always_on
description: CLI-based multi-tier web scraper with intelligent tier escalation, AI extraction, CAPTCHA solving, and anti-bot bypass.
---

# ultimate-scraper

CLI-based multi-tier web scraper with intelligent tier escalation, AI extraction, CAPTCHA solving, and anti-bot bypass.

## Running

```bash
# Basic scrape
python scripts/scrape.py "https://example.com"

# With AI extraction
python scripts/scrape.py "https://example.com" -e "Extract product names and prices" -o json

# Protected sites (stealth browser + US proxy)
python scripts/scrape.py "https://example.com" -m stealth -g us

# Batch processing
python scripts/scrape.py --batch urls.jsonl -p 10 --output-dir ./results/
```

## Core Loop

1. Detect site profile (protection level, content type)
2. Start at lowest viable tier (or known-good tier from history)
3. Attempt extraction at current tier
4. On failure → rotate proxy → retry → escalate to next tier
5. On CAPTCHA → attempt solve (CapSolver/2Captcha) → escalate if unsolvable
6. Return content as markdown, JSON, or raw HTML

## Tier System

| Tier | Mode | Technology | Use Case |
|------|------|------------|----------|
| 0 | static | chompjs/extruct | `__NEXT_DATA__`, JSON-LD (fastest) |
| 1 | http | curl_cffi | TLS fingerprint spoofing |
| 2 | browser | CloakBrowser/Patchright | Stealth browser + CAPTCHA solving |
| 2.5 | agent | agent-browser CLI | CLI automation + tracker blocking |
| 3 | stealth | Camoufox | Full anti-detect (Cloudflare bypass) |
| 4 | ai | Crawl4AI + LLM | AI-powered extraction |
| 5 | visual | Screenshot + Vision LLM | Visual extraction (bypasses DOM detection) |

Auto-escalation: Tier N fails → rotate proxy → retry → escalate to N+1.

## Decision Tree

```
Simple page, no anti-bot → python scripts/scrape.py "URL"
Need specific data       → python scripts/scrape.py "URL" -e "Extract X" -o json
Protected site           → python scripts/scrape.py "URL" -m stealth -g us
Multiple URLs            → python scripts/scrape.py URL1 URL2 -p 10
URLs from file           → python scripts/scrape.py --batch urls.jsonl -p 10
Save to separate files   → python scripts/scrape.py --batch urls.txt --output-dir ./
Long-running job         → python scripts/scrape.py --batch FILE --output-dir ./ --checkpoint job.json
Detect protection        → python scripts/scrape.py "URL" --probe-only
```

## Key Files

| File | Purpose |
|------|---------|
| `scripts/scrape.py` | CLI entry point |
| `scripts/core/scraper.py` | Main orchestrator, tier escalation, rate limiting |
| `scripts/core/config.py` | Centralized configuration (env vars + YAML) |
| `scripts/core/result.py` | Result types and error classification |
| `scripts/tiers/tier0_static.py` | Static data extraction (`__NEXT_DATA__`, JSON-LD) |
| `scripts/tiers/tier1_http.py` | TLS fingerprint spoofing via curl_cffi |
| `scripts/tiers/tier2_scrapling.py` | CloakBrowser/Patchright stealth browser |
| `scripts/tiers/tier2_5_agentbrowser.py` | agent-browser CLI automation |
| `scripts/tiers/tier3_camoufox.py` | Camoufox anti-detect Firefox |
| `scripts/tiers/tier4_ai.py` | Crawl4AI + LLM extraction |
| `scripts/tiers/tier5_visual.py` | Screenshot + Vision LLM |
| `scripts/extraction/ai_router.py` | 3-tier LLM routing |
| `scripts/proxy/manager.py` | Proxy management with geo-targeting |
| `scripts/detection/` | Site profiling, paywall detection, loop detection |
| `scripts/captcha/solver.py` | CAPTCHA solving (CapSolver + 2Captcha) |
| `scripts/rate_limiting/limiter.py` | Per-domain sliding window rate limiter |
| `scripts/fingerprint/manager.py` | Browser fingerprint persistence |

## Configuration

All settings via environment variables (see `.env.example`) or `config/default.yaml`. Env vars take precedence.

### Proxy

Bring your own proxy. Any HTTP/SOCKS5 proxy provider works:

```bash
PROXY_HOST=your-proxy-host        # Hostname
PROXY_PORT=5000                   # Port
PROXY_USERNAME=your_username      # Auth username
PROXY_PASSWORD=your_password      # Auth password
```

- Proxy is optional — all tiers work without one
- Residential proxies recommended for anti-bot sites
- Geo-targeting via `-g us`, `-g uk`, `-g de`, etc.
- GeoIP timezone/locale auto-correlation when proxy is configured

### AI Extraction

3-tier LLM routing for `--extract` mode. The default chain (local → z.ai → Haiku) reflects the developer's setup. **Models and providers are user preference** — modify `scripts/extraction/ai_router.py` to wire in your own. Any OpenAI-compatible API works for Tier 1.

Recommended local model: [GLM-4.7-Flash-UD Q4](https://huggingface.co/THUDM/glm-4-9b-hf) via vLLM/llama.cpp, or any instruction-following model with JSON output (Qwen 2.5, Llama 3.1, Mistral, etc.).

```bash
# Tier 1: Any OpenAI-compatible local LLM (free)
LOCAL_LLM_URL=http://localhost:8080/v1/chat/completions
LOCAL_LLM_ENABLED=true

# Tier 2: z.ai GLM-4.5-Air (rate-limited — or swap for your preferred cloud LLM)
ZAI_API_KEY=your_key

# Tier 3: Anthropic Claude Haiku (paid fallback — or swap for any provider)
ANTHROPIC_API_KEY=your_key
```

### CAPTCHA Solving

```bash
CAPSOLVER_API_KEY=your_key        # AI solver (fast, ~$1-3/1000)
TWOCAPTCHA_API_KEY=your_key       # Human fallback (slower, broadest coverage)
```

Without keys, CAPTCHAs trigger tier escalation.

## Error Handling

| Error | Auto-Handled | Action |
|-------|--------------|--------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yoloshii/ultimate-scraper-skill](https://github.com/yoloshii/ultimate-scraper-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
