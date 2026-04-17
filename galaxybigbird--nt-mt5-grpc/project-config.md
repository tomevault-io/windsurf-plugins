---
trigger: always_on
description: Goal: Give AI agents the minimum, project-specific context to work productively in this repo.
---

# Copilot Instructions for NT-MT5 gRPC

Goal: Give AI agents the minimum, project-specific context to work productively in this repo.

## Architecture
- NT Addon ↔ BridgeApp (Go/Wails, pure gRPC) ↔ MT5 EA (MQL5 via DLL)
- Flow: NT trade → Bridge queue → MT5 hedge; MT5 hedge close → Bridge → NT close original
- Proto source of truth: `BridgeApp/proto/trading.proto` (synced to NT and MT5 folders)

## Key dirs/files
- Bridge server: `BridgeApp/internal/grpc/server.go` (+ `conversions.go`)
- Bridge app state/queue: `BridgeApp/app.go` (env `BRIDGE_GRPC_PORT`, default 50051)
- NT Addon gRPC client: `MultiStratManagerRepo/External/NTGrpcClient/` (public `TradingGrpcClient`)
- MT5 EA: `MT5/ACHedgeMaster_gRPC.mq5`; gRPC DLLs in `MT5/MT5GrpcClient/` and `MT5/cpp-grpc-client/`

## Contracts (examples)
- SubmitTrade: JSON → Trade proto with keys like `id, base_id, timestamp, action, quantity, price, total_quantity, contract_num, order_type, instrument, account_name, nt_*`
- Stream trades: Bridge polls queue every ~200ms and streams to MT5 (GetTrades)
- MT5 → Bridge: `MT5TradeResult`, `HedgeCloseNotification`
- Health: `HealthRequest{source}` → `HealthResponse{status, queue_size, net_position, hedge_size}`

## Conventions
- Bridge runs gRPC-only by default; no HTTP unless explicitly re-enabled elsewhere
- Edges use JSON; Bridge converts JSON↔proto (see `conversions.go` and JSON marshal in `app.go`)
- NT often keeps `useGrpc` feature flag with HTTP fallback still wired
- Queue is buffered (~100); stream buffers may drop on overflow (logged)

## Build/run (PowerShell)
- Bridge: from `BridgeApp/` → `make install-tools` (first time); `make proto`; `wails dev`; override port with `$env:BRIDGE_GRPC_PORT="50052"; wails dev`
  - **After making changes to Bridge code**: Use `./run-wails-dev-with-logs.ps1` from repo root for development with enhanced logging
  - **Build standalone Bridge server**: `cd 'c:\Documents\Dev\OfficialFuturesHedgebotv2\BridgeApp'; go build -o bridge-server.exe .`
- NT: build/use `External/NTGrpcClient.dll`; ensure .NET 4.8 and add `netstandard.dll` ref if CS0012 occurs (see gRPC progress docs)
- MT5: Build the native C++ gRPC client using your native dev prompt only. After any MT5-related changes, you MUST run `MT5/cpp-grpc-client/build_cpp_grpc.bat` to rebuild and deploy the DLLs to `MQL5\Libraries`. Do NOT use the PowerShell `build.ps1` for MT5 builds.

### NinjaTrader deploy script (recommended)
- Use `scripts/deploy-nt-to-ninjatrader.ps1` to build the NT gRPC client DLLs and deploy both DLLs and NinjaScript source `.cs` files for the MultiStratManager addon to their correct NinjaTrader folders.
- What the script does:
	- Builds `External/NTGrpcClient` for net48 by default.
	- Copies runtime DLLs (`NTGrpcClient.dll`, `Grpc.Core*.dll`, `Google.Protobuf.dll`, `System.Text.Json.dll`, etc.) into NinjaTrader roots under `bin/Custom` and `AddOns/MultiStratManager/External`.
	- Deploys ONLY top-level NinjaScript sources from `MultiStratManagerRepo/*.cs` into `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager`. Subfolders (e.g., `External/`) are treated as DLL project code and are not deployed as NinjaScript sources.
	- Purges any DLL-only `.cs` accidentally placed under `bin/Custom` to prevent NinjaScript compile errors.
- When to close NinjaTrader:
	- Editing DLL project code (the C# files that build the .NET DLLs): Close NinjaTrader first, then run the script to update DLLs (avoids file locks). Reopen NT and press F5 to rebuild NinjaScript.
	- Editing NinjaScript source `.cs` (files in `MultiStratManagerRepo` root): You do NOT need to close NinjaTrader. Run the script to sync sources, then press F5 in NinjaTrader to compile.
	- If the script reports locked DLLs in `bin/Custom`, close NinjaTrader and rerun.
 - Typical usage:
	 - For DLL + sources (NT closed): run the script from repo root; reopen NT → F5.
	 - For sources only (NT can stay open): run the script from repo root → F5.

## File sync to live environments
- Source of truth is THIS repo. Make edits here only; then copy the updated files to the live platform folders.
- MT5 (after editing files under `MT5/`): find the exact file in
	`C:\Users\marth\AppData\Roaming\MetaQuotes\Terminal\7BC3F33EDFDBDBDBADB45838B9A2D03F\MQL5` and copy over.
	- Example: for includes like `ACFunctions_gRPC.mqh`, target is
		`...\MQL5\Include\gRPC\ACFunctions_gRPC.mqh` (preserve folder structure).
	- Recompile the EA in MetaEditor after copying. If native DLLs are involved, rebuild them via `MT5/cpp-grpc-client/build_cpp_grpc.bat` first.
- NinjaTrader MultiStratManager: copy changes from `MultiStratManagerRepo/` to
	`C:\Users\marth\OneDrive\Desktop\OneDrive\Old video editing files\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager`, then rebuild inside NinjaTrader.

### NinjaTrader: exact copy targets for addon and gRPC client
- Source of truth (repo):
	- DLL build output: `MultiStratManagerRepo/External/NTGrpcClient/bin/Release/net48/`
	- Addon sources: `MultiStratManagerRepo/*.cs` (e.g., `MultiStratManager.cs`, `IndicatorCalculator.cs`, `SQLiteHelper.cs`)

- Copy these addon source files into the live NinjaTrader addon folder:
	- To: `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager\`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galaxybigbird) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
