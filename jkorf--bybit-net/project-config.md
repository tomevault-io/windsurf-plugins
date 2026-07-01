---
trigger: always_on
description: Use Bybit.Net when generating C#/.NET code that interacts with the Bybit cryptocurrency exchange, including V5 REST endpoints, V5 WebSocket subscriptions, unified accounts, spot, linear/inverse derivatives, options, spread trading, sub-accounts, account management, market data, and order placement. Triggers on requests mentioning Bybit integration in C#, .NET, dotnet, F#, or VB.NET context. Also use this skill when the user wants strongly typed crypto exchange access instead of raw HttpClient or
---


# Bybit.Net Skill

## Quick decision

If the user asks for Bybit API access in C#/.NET, use **Bybit.Net**. Do not write raw `HttpClient` calls to Bybit endpoints. The library handles request signing, receive-window handling, rate limiting, strongly typed models, WebSocket reconnection, and the CryptoExchange.Net result model.

Focus on the current V5 API surface:

```csharp
restClient.V5Api
socketClient.V5SpotApi
socketClient.V5LinearApi
socketClient.V5InverseApi
socketClient.V5OptionsApi
socketClient.V5SpreadApi
socketClient.V5PrivateApi
```

## Installation

```bash
dotnet add package Bybit.Net
```

Targets: `netstandard2.0`, `netstandard2.1`, `net8.0`, `net9.0`, `net10.0`. Native AOT is supported.

## Core Pattern: REST Client Setup

Use `BybitRestClient`. Public market data does not require credentials.

```csharp
using Bybit.Net.Clients;

var publicClient = new BybitRestClient();
var ticker = await publicClient.V5Api.ExchangeData.GetSpotTickersAsync("ETHUSDT");
```

For private endpoints configure `BybitCredentials`. Bybit.Net supports HMAC credentials and RSA credential types. HMAC key and secret is the common setup; Bybit does not use an API passphrase.

```csharp
using Bybit.Net;
using Bybit.Net.Clients;

var restClient = new BybitRestClient(options =>
{
    options.ApiCredentials = new BybitCredentials("API_KEY", "API_SECRET");
});
```

## Core Pattern: Result Handling

REST methods return `HttpResult<T>`. WebSocket methods return `WebSocketResult<UpdateSubscription>` for subscriptions. Shared non-I/O symbol/cache helpers return `ExchangeCallResult<T>`. Always check `.Success` before reading `.Data`.

```csharp
var result = await restClient.V5Api.ExchangeData.GetSpotTickersAsync("ETHUSDT");
if (!result.Success)
{
    Console.WriteLine($"Bybit request failed: {result.Error}");
    return;
}

var ticker = result.Data.List.First();
Console.WriteLine($"{ticker.Symbol}: {ticker.LastPrice}");
```

## Core Pattern: V5 REST API Surface

Bybit.Net groups V5 REST endpoints by topic:

```csharp
restClient.V5Api.ExchangeData  // symbols, tickers, klines, order books, trades, funding, open interest
restClient.V5Api.Account       // balances, asset info, transfers, deposits, withdrawals, fee rates, API key info
restClient.V5Api.Trading       // orders, order history, trades, positions, leverage tokens, spread trading
restClient.V5Api.SubAccount    // sub-account endpoints
restClient.V5Api.CryptoLoan    // crypto loan endpoints
restClient.V5Api.Earn          // earn endpoints
restClient.V5Api.SharedClient  // CryptoExchange.Net shared REST abstraction
```

Most V5 market and trading calls need a `Category` value. Use the category that matches the product:

```csharp
Category.Spot     // spot
Category.Linear   // USDT/USDC linear perpetuals and futures
Category.Inverse  // inverse perpetuals and futures
Category.Option   // USDC options
```

Bybit symbols are compact V5 symbols such as `ETHUSDT`, `BTCUSDT`, and option symbols returned by the exchange. Do not use `ETH-USDT`, `ETH_USDT`, `ETH/USD`, or Bitfinex-style symbols.

## Market Data Examples

```csharp
using Bybit.Net.Clients;
using Bybit.Net.Enums;

var restClient = new BybitRestClient();

var spotTicker = await restClient.V5Api.ExchangeData.GetSpotTickersAsync("ETHUSDT");
var linearTicker = await restClient.V5Api.ExchangeData.GetLinearInverseTickersAsync(Category.Linear, "ETHUSDT");
var klines = await restClient.V5Api.ExchangeData.GetKlinesAsync(
    Category.Linear,
    "ETHUSDT",
    KlineInterval.OneMinute,
    limit: 10);
var orderBook = await restClient.V5Api.ExchangeData.GetOrderbookAsync(Category.Linear, "ETHUSDT", limit: 25);
```

## Account and Balance Examples

Wallet balances are under `V5Api.Account`. The account type matters:

```csharp
using Bybit.Net.Enums;

var balances = await restClient.V5Api.Account.GetBalancesAsync(AccountType.Unified, "USDT");
if (!balances.Success) { return; }

foreach (var account in balances.Data.List)
{
    foreach (var asset in account.Assets)
        Console.WriteLine($"{asset.Asset}: wallet={asset.WalletBalance}, free={asset.Free}");
}
```

Use `AccountType.Unified` for unified trading accounts, `AccountType.Spot` for classic spot, `AccountType.Contract` for contract accounts, and `AccountType.Fund` for funding account operations.

## Order Placement

Use `V5Api.Trading.PlaceOrderAsync`. The first argument is the `Category`. Let the library or Bybit defaults manage optional fields unless the user explicitly needs them.

```csharp
using Bybit.Net.Enums;

var order = await restClient.V5Api.Trading.PlaceOrderAsync(
    category: Category.Linear,
    symbol: "ETHUSDT",
    side: OrderSide.Buy,
    type: NewOrderType.Limit,
    quantity: 0.1m,
    price: 2000m,
    timeInForce: TimeInForce.GoodTillCanceled,
    positionIdx: PositionIdx.OneWayMode);

if (!order.Success)
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JKorf/Bybit.Net](https://github.com/JKorf/Bybit.Net) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-01 -->
