---
trigger: always_on
description: Guide for Claude Code in this repo. Goal, complete SDK capability overview, and
---

# CLAUDE.md

Guide for Claude Code in this repo. Goal, complete SDK capability overview, and
constraints for the project "Control KNX ETS via LLM/MCP".

## Goal

A path through which an LLM (via MCP or scripts) controls KNX ETS: create devices,
create group addresses, link communication objects to group addresses, set parameters,
and program devices.

## Core decisions

- **Architecture: Option A** = ETS AddIn (in-process, C#/net48, System.AddIn) with
  a local IPC endpoint, fronted by a platform-independent MCP server as adapter.
- **Multi-version:** the AddIn builds against the **ETS6 6.4 SDK (Build 8658)** by
  default and runs on both ETS 6.3 and 6.4. An `AppDomain.AssemblyResolve` handler
  binds the `Knx.Ets.*` references to whatever SDK version the host ETS has actually
  loaded, bridging the strong-name version difference. The code uses only the common
  API surface. A separate **ETS5 target** (`dotnet build -c Release -p:Ets5=true`,
  SDK 5.7) exists, with `#if ETS5` deltas.
- ETS access is encapsulated behind `IEtsProjectGateway` (implementation:
  `Ets6ProjectGateway`) so that the ETS5 adapter shares the same interface.
- No ETS-free reimplementation (koolenex model): that would require reimplementing
  programming itself, which is incomplete. Option A uses ETS' own download engine.

## SDK version rule

- The SDK assembly is **strong-named with build-exact version**:
  `Knx.Ets.Sdk, Version=6.4.8658.0, PublicKeyToken=61439873ec5e1159`.
- ETS ships its own SDK DLLs at runtime. The `AssemblyResolve` handler in the AddIn
  resolves `Knx.Ets.*` references to whatever version the host process has loaded,
  so a single build can run on multiple ETS versions as long as only the common API
  surface is used.
- SDK DLLs come from the official KNX SDK downloads (see BUILD.md), referenced with
  `Private=false` (do NOT ship them). Point `EtsDllPath` at the folder holding them;
  hm-knx alternatively references the DLLs directly from the ETS install directory.
- **No runtime version gating.** The AddIn detects and logs the loaded SDK version at
  startup but never rejects a version -- it runs on ETS5 5.x and ETS6 6.3/6.4 alike.
  Compile-time safety comes from building against a specific SDK; the runtime is not
  gated.

### SDK known quirks

- **DPT reading:** `ComObject.DatapointType` returns empty on some ETS versions. Use
  `ComObjectInstanceRef.DatapointTypes` or the application model instead.
- **Linking inactive objects:** `Link(...)` throws on an inactive ComObject on older
  ETS builds. Check `IsActive` beforehand or handle the exception.
- **KNX Secure:** `ComObjectInstanceRef.DetermineLinkSecurityImpacts(...)` exists only
  in SDK 6.4.0+. On ETS 6.3 a device can become secure in a non-password project;
  `GroupMessageReceived` does not fire for Secure messages.
- **6.4-only API:** the AddIn avoids `DetermineLinkSecurityImpacts(...)` to stay
  compatible with 6.3.

## What the SDK can do (complete overview)

All information from the documented public API (`Knx.Ets.Sdk.xml`, 3602 members),
namespace `Knx.Ets.Sdk.Project` unless stated otherwise. Mutations require the ETS UI
thread and an undo marker (see Hard constraints).

### Topology (physical)

- Areas: `AreaCollection.Add(name[, addr[, AddressAllocations, addr]])`, `Delete`,
  `Area.Move(Line, ...)`.
- Lines/Segments: types `Line`, `Segment` present (Add analogous via the respective
  collection).
- Create devices from catalog:
  `DeviceCollection.Add(Line, CatalogItem, addr, AddressAllocations, count)` plus
  overloads for `Segment`/`BuildingPart`/`Trade`, for `UnifiedCatalogItem`, and for
  `(Product, Hardware2Program)`. `Delete`, additional addresses via
  `AdditionalDeviceAddressCollection.Add`.
- Physical address: via the `addr` parameters and `AddressAllocations` strategy.

### Building / Trades (logical)

- Building structure: `BuildingPartCollection.Add(...)` (with `BuildingPartType`,
  optionally `SpaceUsage`) for floors/rooms/distribution boards, `Delete`, `Move`.
- Assign devices to rooms: `BuildingPart.Link(Device)` / `Unlink`.
- Building functions: `BuildingFunctionCollection.Add(...)`,
  `BuildingFunction.Link(GroupAddress)` / `Unlink`.
- Trades: type `Trade` (devices also via `DeviceCollection.Add(Trade, ...)`).

### Group addresses and links

- GA structure: `GroupRangeCollection.Add(name, uint[, AddressAllocations, uint[, uint]])`
  (main/middle groups), `GroupAddressCollection.Add(name, uint[, AddressAllocations,
  uint])`, `Delete`, `GroupRange.Move(GroupAddress, ...)`.
- **Link ComObject<->GA:** `ComObjectInstanceRef.Link(GroupAddress)` /
  `Link(IEnumerable<GroupAddress>)`, `Unlink(...)`.
- Coupler/filter view: `BusInterface.Link(GroupAddress|GroupRange)` / `Unlink`
  (BusInterfaceConnector).
- Additional/sending GAs: `AdditionalGroupAddressCollection.Add/Delete`.

### Parameters, communication objects, modules

- Set parameters: `ParameterInstanceRef.Value` (settable), `IsActive`, `IsDefault`.
- ComObject properties/flags settable: `ComObjectInstanceRef.CommunicationFlag`,
  `ReadFlag`, `WriteFlag`, `TransmitFlag`, `UpdateFlag`, `ReadOnInitFlag`, `Priority`,
  `DatapointTypes`, `Description`.
- Channels/modules: `ChannelInstance`, `ModuleInstance`, `ModuleArgument`.

### KNX Secure


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [knx-ai/knx-ets-mcp](https://github.com/knx-ai/knx-ets-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
