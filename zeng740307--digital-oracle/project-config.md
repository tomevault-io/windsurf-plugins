---
trigger: always_on
description: Answer prediction questions using market trading data, not opinions. Use when the user asks probability questions about geopolitics, economics, markets, industries, or any topic where real money is being traded on the outcome. Examples: 'What's the probability of WW3?', 'Will there be a recession?', 'Is AI in a bubble?', 'When will the Russia-Ukraine war end?', 'Is it a good time to buy gold?', 'Will SPY drop 5% this month?', 'Is NVDA options premium overpriced?'. The skill reads prices from pre
---


# digital-oracle

> Markets are efficient. Price contains all public information. Reading price = reading market consensus.

## Methodology

**Answer questions using only market trading data — no news, opinions, or statistical reports as causal evidence.** If something is true, some market has already priced it in.

Five iron rules:

1. **Trading data only** — prices, volume, open interest, spreads, premiums. Never cite analyst opinions.
2. **Explicit reasoning from price to judgment** — explain clearly "why this price answers this question."
3. **Multi-signal cross-validation** — never conclude from a single signal. At least 3 independent dimensions.
4. **Label the time horizon of each signal** — options price 3 months, equipment orders price 3 years — don't mix them in the same vote.
5. **Structured output** — the final report must follow the Step 5 template: layered signal tables → contradiction analysis → probability scenarios → signal consistency assessment. Do not substitute prose for structured reporting.

## Workflow

### Step 1: Understand the question

Decompose the user's question into:
- **Core variable**: What event or trend?
- **Time window**: Is the user asking about 3 months, 1 year, or 5 years?
- **Priceability**: Is there real money being traded on this outcome?

### Step 2: Select signals

Based on question type, select from the signal menu below. **Don't use just one category — cover at least 3.**

#### Geopolitical conflict / War risk
- Polymarket: Search for related event contracts (ceasefire, invasion, regime change, declaration of war)
- Kalshi: Search for related binary contracts
- Safe-haven assets: Gold (GC=F), silver (SI=F), Swiss franc (USDCHF=X)
- Conflict proxies: Crude oil (CL=F), natural gas (NG=F), wheat (ZW=F), defense ETF (ITA), defense stocks
- Risk ratios: Copper/Gold ratio (risk-off indicator), Gold/Silver ratio
- CFTC COT: Institutional positioning changes in crude/gold/wheat (which direction is smart money betting)
- BIS: Central bank policy rate trends in relevant countries
- FearGreedProvider: CNN Fear & Greed Index (composite of 7 price signals)
- Web search: VIX, MOVE index, sovereign CDS, war risk premiums, BDI freight rates, high-yield OAS
- Currencies: Currency pairs of relevant countries (e.g. USDRUB=X, USDCNY=X)
- Country ETFs: Asset flows in relevant countries (e.g. FXI, EWY)

#### Economic recession / Macro cycle
- Treasury: Yield curve shape (10Y-2Y spread, 10Y-3M spread), real rates, breakeven inflation
- YahooPriceProvider: SPY, copper (HG=F), crude oil (CL=F), price trends
- Risk ratios: Copper/Gold ratio
- CFTC COT: Speculative net positions in copper/crude (is managed money bullish or bearish)
- BIS: Credit-to-GDP gap (credit overheating = late cycle), policy rate directions
- World Bank: GDP growth rate historical trends, cross-country comparisons
- Deribit: BTC futures basis (risk appetite proxy)
- CoinGecko: Crypto total market cap + BTC dominance (risk appetite proxy)
- FearGreedProvider: CNN Fear & Greed Index (7 price signals composite → 0-100)
- CMEFedWatchProvider: Market-implied FOMC rate change probabilities from futures
- Polymarket: Recession-related contracts, central bank rate path
- Currencies: DXY/dollar strength, emerging market currencies
- Web search: High-yield bond spread (HY OAS), TED spread, MOVE index, TTF gas, BDI freight rates

#### Industry cycle / Bubble assessment
- YahooPriceProvider: Industry leader stock trends, sector ETFs
- Find the industry's "single-purpose commodity" (e.g. GPU rental price → AI, rebar → construction)
- Upstream equipment maker orders/stock price (e.g. ASML → semiconductors)
- Leader company valuation discount (e.g. TSMC vs peers → Taiwan Strait risk pricing)
- EDGAR: Industry leader insider trading cadence (Form 4) — concentrated selling = bearish signal
- CFTC COT: Institutional positioning changes in related commodities
- CoinGecko: For crypto industry, look at BTC/ETH/altcoin market cap distribution
- Web search: VC funding concentration, leveraged ETF concentration, margin debt levels
- Deribit: Implied volatility of related crypto assets

#### Asset pricing / Whether to buy
- YahooPriceProvider: Target asset price trend (daily/weekly/monthly)
- Relative price changes of correlated assets (divergence between two commodities = structural signal)
- Treasury: Risk-free rate as valuation anchor
- YFinance: Options chain (IV, put/call ratio, max pain, Greeks, implied move)
- EDGAR: Insider selling cadence (heavy Form 4 selling = insiders bearish)
- CFTC COT: Speculative vs commercial net position divergence for commodity assets
- CoinGecko: For crypto assets, check market cap, ATH/ATL distance, 24h volatility
- Deribit: Crypto options chain (implied volatility = market's expected range)
- Polymarket/Kalshi: Probability pricing of related events

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zeng740307/digital-oracle](https://github.com/zeng740307/digital-oracle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
