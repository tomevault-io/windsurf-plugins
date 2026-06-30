---
trigger: always_on
description: This repository is **Bitget.Net**, a strongly typed C#/.NET client library for the Bitget REST and WebSocket APIs. It is part of the CryptoExchange.Net ecosystem.
---

# Copilot Instructions for Bitget.Net

This repository is **Bitget.Net**, a strongly typed C#/.NET client library for the Bitget REST and WebSocket APIs. It is part of the CryptoExchange.Net ecosystem.

When generating code that consumes Bitget.Net, follow these conventions.

## Use Bitget.Net, not raw HTTP

Never generate `HttpClient` calls to Bitget API URLs. Always use `BitgetRestClient` or `BitgetSocketClient`. This ensures correct request signing, passphrase handling, rate limiting, serialization, and error handling.

## Client setup

```csharp
using Bitget.Net;
using Bitget.Net.Clients;

var restClient = new BitgetRestClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

For public market data, credentials are not required:

```csharp
var restClient = new BitgetRestClient();
```

Socket client:

```csharp
var socketClient = new BitgetSocketClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

## Result handling

REST methods return `WebCallResult<T>` or `WebCallResult`. Socket subscription methods return `CallResult<UpdateSubscription>`. Always check `.Success` before reading `.Data`. The error is on `.Error`.

```csharp
var result = await restClient.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");
if (!result.Success)
{
    Console.WriteLine(result.Error);
    return;
}

Console.WriteLine(result.Data.Single().LastPrice);
```

## API structure

- `restClient.SpotApiV2.ExchangeData` - spot public server time, announcements, assets, symbols, tickers, order books, klines and trades
- `restClient.SpotApiV2.Account` - funding balances, spot balances, fees, transfers, deposits, withdrawals, ledgers, BGB deduction and subaccounts
- `restClient.SpotApiV2.Trading` - spot orders, cancel/replace, open/closed orders, fills and spot trigger orders
- `restClient.SpotApiV2.Margin` - spot cross and isolated margin balances, borrowing, repayment, risk, orders and histories
- `restClient.FuturesApiV2.ExchangeData` - futures contracts, tickers, order books, klines, funding, open interest, prices and tiers
- `restClient.FuturesApiV2.Account` - futures balances, leverage, margin mode, position mode, ledger, ADL, liquidation price and openable quantity
- `restClient.FuturesApiV2.Trading` - futures positions, orders, fills, close-position actions, trigger orders and TP/SL
- `restClient.CopyTradingFuturesV2.Trader` and `.Follower` - copy trading futures endpoints
- `restClient.UnifiedApi.ExchangeData`, `.Account` and `.Trading` - Unified/UTA market, account and trading endpoints
- `restClient.BrokerV2` - broker reporting endpoints on the concrete `BitgetRestClient`
- `socketClient.SpotApiV2` - spot public/private WebSocket streams and margin streams
- `socketClient.FuturesApiV2` - futures public/private WebSocket streams
- `socketClient.UnifiedApi` - Unified/UTA public/private WebSocket streams and socket order actions

## Symbols and product types

Bitget symbols are compact:

- Correct: `BTCUSDT`, `ETHUSDT`, `BGBUSDT`
- Wrong: `BTC-USDT`, `BTC/USDT`, `BTC_USDT`, `tBTCUSD`

Futures calls usually require a product type:

```csharp
BitgetProductTypeV2.UsdtFutures
BitgetProductTypeV2.CoinFutures
BitgetProductTypeV2.UsdcFutures
BitgetProductTypeV2.SimUsdtFutures
```

Futures account and trading calls often require a margin asset such as `"USDT"`.

## Enum namespaces

Use `Bitget.Net.Enums` for:

- `BitgetProductTypeV2`
- `BitgetStreamKlineIntervalV2`
- `BitgetFuturesKlineInterval`
- `BitgetBusinessType`

Use `Bitget.Net.Enums.V2` for:

- `OrderSide`, `OrderType`, `TimeInForce`
- `MarginMode`, `TradeSide`, `PositionSide`, `PositionMode`
- `TransferAccountType`, `TransferType`, `AccountType`
- spot REST `KlineInterval`
- trigger-order enums

Use `Bitget.Net.Enums.Uta` for Unified/UTA enums:

- `ProductCategory`, `KlineUaInterval`
- `AccountLevel`, `AccountMode`, `HoldingMode`
- `StpMode`, `StrategyType`, `TpslMode`

## Spot examples

Market data:

```csharp
var ticker = await restClient.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");
if (!ticker.Success) { Console.WriteLine(ticker.Error); return; }

Console.WriteLine(ticker.Data.Single().LastPrice);
```

Spot order:

```csharp
using Bitget.Net.Enums.V2;

var order = await restClient.SpotApiV2.Trading.PlaceOrderAsync(
    "BTCUSDT",
    OrderSide.Buy,
    OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 1m);
```

Spot margin:

```csharp
var crossBalances = await restClient.SpotApiV2.Margin.GetCrossBalancesAsync("USDT");
var isolatedBalances = await restClient.SpotApiV2.Margin.GetIsolatedBalancesAsync();
```

## Futures examples

```csharp
using Bitget.Net.Enums;
using Bitget.Net.Enums.V2;

var productType = BitgetProductTypeV2.UsdtFutures;
const string symbol = "BTCUSDT";
const string marginAsset = "USDT";

var ticker = await restClient.FuturesApiV2.ExchangeData.GetTickerAsync(productType, symbol);
var positions = await restClient.FuturesApiV2.Trading.GetPositionsAsync(productType, marginAsset);

var order = await restClient.FuturesApiV2.Trading.PlaceOrderAsync(
    productType,
    symbol,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JKorf/Bitget.Net](https://github.com/JKorf/Bitget.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
