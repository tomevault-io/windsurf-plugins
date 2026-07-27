---
trigger: always_on
description: Conventions for using Bitget.Net library when working with Bitget in C#/.NET. Apply when generating code that interacts with the Bitget API.
---


# Bitget.Net Conventions

This codebase uses **Bitget.Net** for Bitget exchange access. Do not write raw `HttpClient` calls to Bitget endpoints.

## Client setup pattern

```csharp
using Bitget.Net;
using Bitget.Net.Clients;

var restClient = new BitgetRestClient(options =>
{
    options.ApiCredentials = new BitgetCredentials("API_KEY", "API_SECRET", "PASSPHRASE");
});
```

For public market data only, no credentials are needed: `new BitgetRestClient()`.

## Result pattern

All REST methods return `WebCallResult<T>` or `WebCallResult`. WebSocket subscriptions return `CallResult<UpdateSubscription>`. Always check `.Success` before reading `.Data`:

```csharp
var ticker = await restClient.SpotApiV2.ExchangeData.GetTickersAsync("BTCUSDT");
if (!ticker.Success) { /* ticker.Error */ return; }
var price = ticker.Data.Single().LastPrice;
```

## API surface

- `restClient.SpotApiV2.ExchangeData` for spot public server time, assets, symbols, tickers, books, candles and trades
- `restClient.SpotApiV2.Account` for funding balances, spot balances, fees, transfers, deposits, withdrawals, ledgers and subaccounts
- `restClient.SpotApiV2.Trading` for spot orders, cancel/replace, fills and trigger orders
- `restClient.SpotApiV2.Margin` for cross and isolated spot margin
- `restClient.FuturesApiV2.ExchangeData` for futures contracts, tickers, books, candles, funding, open interest and tiers
- `restClient.FuturesApiV2.Account` for balances, leverage, margin mode, position mode, ADL and ledger
- `restClient.FuturesApiV2.Trading` for futures positions, orders, fills, close-position actions, triggers and TP/SL
- `restClient.CopyTradingFuturesV2.Trader` and `.Follower` for copy trading futures
- `restClient.UnifiedApi.ExchangeData`, `.Account` and `.Trading` for Unified/UTA market, account and trading endpoints
- `restClient.BrokerV2` for broker reporting on the concrete `BitgetRestClient`
- `socketClient.SpotApiV2` for spot public/private WebSocket streams and margin streams
- `socketClient.FuturesApiV2` for futures public/private WebSocket streams
- `socketClient.UnifiedApi` for Unified/UTA public/private WebSocket streams and socket order actions

## Symbols and product types

Bitget V2 symbol conventions matter:

- Spot and futures symbols: `BTCUSDT`, `ETHUSDT`, `BGBUSDT`
- Do not use `BTC-USDT`, `BTC/USDT`, `BTC_USDT`, or `tBTCUSD`

Futures product types:

```csharp
BitgetProductTypeV2.UsdtFutures
BitgetProductTypeV2.CoinFutures
BitgetProductTypeV2.UsdcFutures
BitgetProductTypeV2.SimUsdtFutures
```

Futures account and trading methods often require a margin asset such as `"USDT"`.

## Enum namespaces

Use `Bitget.Net.Enums` for product type and stream/futures kline enums:

```csharp
BitgetProductTypeV2.UsdtFutures
BitgetStreamKlineIntervalV2.OneMinute
BitgetFuturesKlineInterval.OneMinute
```

Use `Bitget.Net.Enums.V2` for V2 order/account enums:

```csharp
OrderSide.Buy
OrderType.Limit
TimeInForce.GoodTillCanceled
MarginMode.CrossMargin
TradeSide.Open
TransferAccountType.Spot
TransferType.OnChain
```

Use `Bitget.Net.Enums.Uta` for Unified/UTA category and account enums:

```csharp
ProductCategory.Spot
ProductCategory.UsdtFutures
KlineUaInterval.OneMinute
AccountLevel.Advanced
HoldingMode.OneWayMode
StpMode.CancelMaker
```

## Spot order placement

```csharp
var order = await restClient.SpotApiV2.Trading.PlaceOrderAsync(
    "BTCUSDT",
    OrderSide.Buy,
    OrderType.Limit,
    quantity: 0.001m,
    timeInForce: TimeInForce.GoodTillCanceled,
    price: 1m);
```

`GetTickersAsync("BTCUSDT")` returns an array. Use `.Single()` or `.FirstOrDefault()` after checking `.Success`.

## Futures order placement

```csharp
var productType = BitgetProductTypeV2.UsdtFutures;

var order = await restClient.FuturesApiV2.Trading.PlaceOrderAsync(
    productType,
    "BTCUSDT",
    "USDT",
    OrderSide.Buy,
    OrderType.Limit,
    MarginMode.CrossMargin,
    quantity: 0.001m,
    price: 1m,
    timeInForce: TimeInForce.GoodTillCanceled,
    tradeSide: TradeSide.Open);
```

## Spot margin

Spot margin is not a separate top-level API. Use `SpotApiV2.Margin`:

```csharp
var crossBalances = await restClient.SpotApiV2.Margin.GetCrossBalancesAsync("USDT");
var isolatedBalances = await restClient.SpotApiV2.Margin.GetIsolatedBalancesAsync();
```

## Unified/UTA

Unified/UTA endpoints are under `UnifiedApi`. Use `ProductCategory` from `Bitget.Net.Enums.Uta` for category parameters:

```csharp
var symbols = await restClient.UnifiedApi.ExchangeData.GetSpotSymbolsAsync("BTCUSDT");
var balances = await restClient.UnifiedApi.Account.GetBalancesAsync();

var order = await restClient.UnifiedApi.Trading.PlaceOrderAsync(
    ProductCategory.Spot,
    "BTCUSDT",
    OrderSide.Buy,
    OrderType.Limit,
    quantity: 0.001m,
    price: 1m);
```

## WebSocket pattern

```csharp
var socketClient = new BitgetSocketClient();
var sub = await socketClient.SpotApiV2.SubscribeToTickerUpdatesAsync(
    "BTCUSDT",
    update => { /* update.Data.First().LastPrice */ });
if (!sub.Success) { /* sub.Error */ return; }

// On shutdown:
await socketClient.UnsubscribeAsync(sub.Data);
```

For futures sockets:

```csharp

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JKorf/Bitget.Net](https://github.com/JKorf/Bitget.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
